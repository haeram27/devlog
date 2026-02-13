# How to prevent container occurpies all memory with OS Page Cache

컨테이너가 Page Cache를 과도 사용해 호스트의 메모리를 전부 소비하는 것을 막는 방법은 무엇인가?
프로그래밍 방식이나 컨테이너 설정이나 커널 변수는 방식에는 제한이 없다.

GitHub Copilot: 컨테이너 환경에서 대용량 파일 쓰기 시 Page Cache(Clean Page)가 누적되어 호스트 메모리를 고갈시키는 문제는 매우 흔한 현상입니다. 특히 **"메모리 제한 설정이 없다"**는 점이 문제의 핵심 원인입니다.

리눅스 커널은 유휴 메모리를 최대한 Page Cache로 활용하도록 설계되어 있습니다. 컨테이너에 메모리 제한(Limit)이 없으면, 컨테이너는 호스트의 모든 메모리를 자신의 캐시 영역으로 간주하고 채워버립니다. `posix_fadvise`는 커널에 대한 '힌트'일 뿐이며, 메모리 압박이 없다고 판단되면 커널은 이를 무시할 수 있습니다.

이 문제를 해결하기 위한 3가지 계층(인프라, 프로그래밍, 커널)의 해결책을 제시합니다.

---

### 1. 컨테이너 설정 (가장 확실한 해결책)

가장 먼저 적용해야 할 필수 조치입니다. 컨테이너가 사용할 수 있는 메모리의 상한선을 두어야 커널이 Page Cache를 능동적으로 회수(Eviction)하기 시작합니다.

-   **Docker 사용 시:**
    `--memory` 옵션을 사용하여 컨테이너 메모리를 제한합니다.
    ```bash
    docker run -d --memory="4g" --memory-swap="4g" ...
    ```

-   **Kubernetes 사용 시:**

    Pod Spec에 `resources.limits.memory`를 설정합니다.
    ```yaml
    resources:
      limits:
        memory: "4Gi"
      requests:
        memory: "2Gi"
    ```

**효과:** 제한된 메모리(예: 4GB) 내에서만 Page Cache가 생성됩니다. 쓰기 작업이 계속되면 커널은 OOM을 발생시키는 대신, 오래된 Page Cache를 비우고 새 데이터를 씁니다.

---

### 2. 프로그래밍 방식: Direct I/O (`O_DIRECT`) 사용

`posix_fadvise`가 작동하지 않는다면, 아예 Page Cache를 거치지 않고 디스크에 직접 쓰는 **Direct I/O** 방식을 사용해야 합니다. Java의 표준 API(`RandomAccessFile`, `FileChannel`)는 이를 직접 지원하지 않으므로 JNA를 통해 `open` 시스템 콜을 호출할 때 `O_DIRECT` 플래그를 사용해야 합니다.

**주의:** `O_DIRECT` 사용 시 버퍼의 메모리 주소와 크기가 디스크 블록 크기(보통 512 또는 4096 바이트)에 정렬(Aligned)되어야 합니다.

#### 수정된 코드 제안 (JNA 활용)

기존 `RandomAccessFile` 대신 JNA의 `open`, `pwrite`, `close`를 직접 매핑하여 사용합니다.

```java
// ...existing code...
import com.sun.jna.Pointer;
import com.sun.jna.Memory;
import com.sun.jna.Native;
// ...existing code...

public abstract class ResumableFileUploadService {
    // ...existing code...

    // O_DIRECT 플래그 상수 (Linux 기준, 시스템마다 다를 수 있음. 보통 040000)
    private static final int O_WRONLY = 0x0001;
    private static final int O_CREAT = 0x0040;
    private static final int O_DIRECT = 0x4000; // 시스템 헤더 확인 필요 (fcntl.h)

    // JNA CLibrary 인터페이스 확장
    public interface CLibrary extends com.sun.jna.Library {
        CLibrary INSTANCE = com.sun.jna.Native.load("c", CLibrary.class);
        
        int open(String path, int flags, int mode);
        int close(int fd);
        // pwrite는 file offset을 변경하지 않고 특정 위치에 씀 (스레드 안전)
        long pwrite(int fd, Pointer buf, long count, long offset);
        int posix_fadvise(int fd, long offset, long len, int advice);
        String strerror(int errno);
    }

    private void writeFile(HttpServletRequest request, int flowChunkNumber, FlowInfoVo info, String filePath) throws IOException {
        File dir = new File(filePath);
        if (!dir.exists() && dir.mkdirs()) {
            log.trace("create directory success!");
        }

        String targetFile = filePath + '/' + info.getFlowFilename() + ".temp";
        long startOffset = (flowChunkNumber - 1) * info.getFlowChunkSize();
        
        // 1. O_DIRECT로 파일 열기
        // 주의: O_DIRECT는 Linux 전용이며, 파일 시스템이 지원해야 함.
        int fd = CLibrary.INSTANCE.open(targetFile, O_WRONLY | O_CREAT | O_DIRECT, 0644);
        
        if (fd < 0) {
            // O_DIRECT 실패 시(지원 안하는 FS 등) 일반 모드로 재시도 하거나 예외 처리
            log.warn("O_DIRECT open failed, fallback to normal open. errno: " + Native.getLastError());
            fd = CLibrary.INSTANCE.open(targetFile, O_WRONLY | O_CREAT, 0644);
        }

        try (InputStream is = request.getInputStream()) {
            long contentLength = request.getContentLength();
            
            // 2. Aligned Buffer 준비 (Direct Memory)
            // O_DIRECT는 블록 사이즈(보통 4096) 정렬이 필수입니다.
            int bufferSize = 1024 * 1024; // 1MB
            long bytesWritten = 0;
            
            // JNA Memory는 기본적으로 page-aligned 됨
            Memory buffer = new Memory(bufferSize); 

            byte[] javaBuffer = new byte[bufferSize];
            
            while (bytesWritten < contentLength) {
                int readLen = is.read(javaBuffer);
                if (readLen == -1) break;

                // Java Heap -> Native Memory 복사
                buffer.write(0, javaBuffer, 0, readLen);

                // 3. pwrite 호출 (Direct I/O)
                // 주의: O_DIRECT 사용 시 쓰기 크기도 블록 사이즈의 배수여야 할 수 있음.
                // 마지막 청크가 블록 사이즈와 맞지 않으면 O_DIRECT 쓰기가 실패할 수 있으므로
                // 마지막 조각은 일반 I/O로 처리하거나 패딩 처리가 필요할 수 있음.
                long written = CLibrary.INSTANCE.pwrite(fd, buffer, readLen, startOffset + bytesWritten);
                
                if (written < 0) {
                     throw new IOException("pwrite failed: " + CLibrary.INSTANCE.strerror(Native.getLastError()));
                }
                bytesWritten += written;
            }
            
            // O_DIRECT는 OS 캐시를 안 거치므로 sync/dropCache 불필요
            
        } catch (Exception e) {
            log.error("File write error", e);
            throw new IOException(e);
        } finally {
            if (fd > 0) CLibrary.INSTANCE.close(fd);
        }
    }
    // ...existing code...
}
```
*참고: 위 코드는 개념 증명용이며, 실제 프로덕션 적용 시 `O_DIRECT`의 정렬 제약조건(메모리 주소 정렬, 쓰기 크기 정렬)을 엄격히 처리해야 합니다. 마지막 청크 크기가 4096 배수가 아닐 경우 `O_DIRECT` 쓰기가 실패할 수 있어, 해당 부분만 `RandomAccessFile`로 처리하는 하이브리드 방식이 필요할 수 있습니다.*

---

### 3. 커널 파라미터 튜닝 (보조 수단)

컨테이너 호스트(또는 특권 컨테이너)에서 커널의 Dirty Page 처리 방식을 변경하여, 캐시가 너무 많이 쌓이기 전에 디스크로 플러시하도록 유도합니다.

-   `vm.dirty_background_ratio`: 백그라운드에서 디스크 쓰기를 시작할 메모리 비율 (기본값 10% -> **5% 이하로 축소**)
-   `vm.dirty_ratio`: 프로세스가 쓰기를 멈추고 디스크 쓰기를 기다리게 하는 절대 한계 (기본값 20% -> **10% 이하로 축소**)

```bash
sysctl -w vm.dirty_background_ratio=5
sysctl -w vm.dirty_ratio=10
```
이 설정은 Page Cache가 "Dirty" 상태로 오래 머무는 것을 막아주지만, 이미 디스크에 기록된 "Clean" Page Cache가 메모리를 점유하는 것을 막지는 못합니다.

### 요약 및 권장 사항

1.  **최우선:** **컨테이너 메모리 제한(`--memory`)**을 설정하십시오. 이것이 근본적인 해결책입니다. 제한을 걸면 커널이 알아서 Page Cache를 조절합니다.
2.  **코드 수정:** `posix_fadvise`가 효과가 없다면, 현재 구현된 `RandomAccessFile` 방식은 유지하되, **`FileChannel`**을 사용하여 성능을 높이고, 컨테이너 메모리 제한 하에서 동작하도록 두는 것이 가장 안정적입니다. `O_DIRECT`는 구현 복잡도가 높고 파일 시스템 호환성을 타므로 차선책으로 고려하십시오.