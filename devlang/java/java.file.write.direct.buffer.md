# DirectBuffer 사용하여 File 쓰기

목적: OS PageCache 메모리 누적하지 않고, 대용량 파일 Write하기

O_DIRECT(Direct I/O)를 사용하면 시스템 전체의 캐시를 건드리지 않으면서, 본인의 Java 프로세스가 사용하는 메모리만 깔끔하게 관리하겠다는 가장 전문적인 접근입니다.
하지만 앞서 말씀드린 '정렬(Alignment) 제한' 때문에 일반적인 FileChannel 사용법과는 조금 다릅니다. Linux 환경에서 Java로 O_DIRECT를 사용할 때 반드시 지켜야 할 3가지 핵심 규칙을 정리해 드립니다.

## 1. 블록 크기 맞추기 (Alignment)

Direct I/O를 쓰려면 파일 시스템의 블록 크기(보통 4096바이트, 즉 4KB) 단위로만 쓰고 읽어야 합니다.

* Buffer 크기: 반드시 4KB의 배수여야 합니다. (예: 4KB, 8KB, 1MB 등)
* Write 위치(Offset): 파일의 어느 지점에 쓸 때도 반드시 4KB의 배수 지점부터 써야 합니다.
* 마지막 부분 처리: 파일 크기가 4KB로 딱 나누어떨어지지 않는다면, 마지막에는 4KB를 채워서 쓰고 나중에 파일을 truncate하여 크기를 맞추는 테크닉이 필요할 수 있습니다.

## 2. Direct Buffer 사용

ByteBuffer.allocate()가 아닌 ByteBuffer.allocateDirect()를 사용해야 합니다. 그래야 Java 힙 메모리를 거치지 않고 OS가 디스크로 바로 데이터를 쏠 수 있습니다.

## 3. 실전 코드 예시 (Java 10+ 기준)

리눅스 환경에서 O_DIRECT를 활성화하여 대용량 파일을 작성하는 기본 구조입니다.

```java
import com.sun.nio.file.ExtendedOpenOption;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.*;

public class LargeFileBatchWriter {
    public static void main(String[] args) {
        Path path = Paths.get("huge_batch_data.bin");
        
        try {
            FileStore store = Files.getFileStore(path.getParent() != null ? path.getParent() : Paths.get("."));
            int blockSize = (int) store.getBlockSize();
            
            // 1. 버퍼 크기 설정: 보통 2MB ~ 8MB 정도가 I/O 성능이 가장 좋습니다.
            // 반드시 blockSize(4KB)의 배수여야 합니다.
            int bufferSize = blockSize * 1024; // 4MB
            ByteBuffer buffer = ByteBuffer.allocateDirect(bufferSize);

            long totalBytesWritten = 0;
            long targetSize = 10L * 1024 * 1024 * 1024; // 예: 10GB 파일 생성 테스트

            try (FileChannel fc = FileChannel.open(path, 
                    StandardOpenOption.WRITE, 
                    StandardOpenOption.CREATE, 
                    StandardOpenOption.TRUNCATE_EXISTING,
                    ExtendedOpenOption.DIRECT)) {

                // 2. 메인 루프: 버퍼 단위로 대량 쓰기
                while (totalBytesWritten < targetSize) {
                    buffer.clear();
                    
                    // 남은 데이터가 버퍼보다 작을 경우 처리
                    if (targetSize - totalBytesWritten < bufferSize) {
                        // 자투리 데이터를 위해 버퍼 전체를 0으로 초기화(선택) 후 일부만 채움
                        // O_DIRECT는 이 상황에서도 bufferSize 전체를 write해야 함
                        buffer.limit(bufferSize); 
                    }

                    // 실제로는 여기서 데이터를 buffer에 채움 (예시로 생략)
                    // fillBuffer(buffer); 

                    buffer.position(0);
                    fc.write(buffer);
                    totalBytesWritten += bufferSize;
                }

                // 3. 마지막에 실제 원하는 크기로 자르기
                // 이 명령으로 인해 블록 단위로 기록된 파일 끝부분의 패딩이 제거됨
                fc.truncate(targetSize);
                
                System.out.println("대용량 파일 생성 완료: " + targetSize + " bytes");

            }
        } catch (IOException | UnsupportedOperationException e) {
            e.printStackTrace();
        }
    }
}

```

### 주의사항

* 파일 시스템 지원: ext4, xfs 등 대부분의 리눅스 파일 시스템은 지원하지만, 일부 특수한 환경이나 네트워크 파일 시스템(NFS)에서는 O_DIRECT 사용 시 오류가 날 수 있습니다.
* 소량 데이터 작성 시 불리: 100바이트씩 자주 쓰는 작업이라면 O_DIRECT는 매우 느려집니다. 반드시 애플리케이션 레벨에서 큰 단위(MB 단위 권장)로 버퍼링해서 한 번에 내려써야 성능이 나옵니다.

이제 이 방식을 도입하시면 free -h를 확인했을 때 buff/cache 영역이 더 이상 치솟지 않는 것을 보실 수 있을 겁니다.

## 파일 시스템의 블록 크기 확인 방법

Java 7 이상에서 제공하는 FileStore API를 사용하면 OS의 파일 시스템 블록 크기(Block Size)를 직접 확인할 수 있습니다. [1, 2, 3] 
O_DIRECT를 적용하기 위해 특정 경로의 블록 크기를 확인하는 Java 코드는 다음과 같습니다.

```java
import java.io.IOException;
import java.nio.file.FileStore;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class FileSystemBlockSize {
    public static void main(String[] args) {
        try {
            // 파일을 저장할 디렉토리 경로 지정
            Path path = Paths.get("."); 
            FileStore store = Files.getFileStore(path);

            // getBlockSize()는 해당 파일 저장소의 블록 크기를 바이트 단위로 반환합니다.
            long blockSize = store.getBlockSize();

            System.out.println("File System Type: " + store.type());
            System.out.println("Block Size: " + blockSize + " bytes");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 참고 사항

* 지원 범위: getBlockSize() 메서드는 Java 9부터 공식적으로 추가되었습니다. Java 8 이하 버전에서는 시스템 명령어를 호출하거나 기본값인 4096(4KB)으로 가정하고 구현하는 경우가 많습니다.
* 정렬 필수: O_DIRECT로 파일을 쓸 때, ByteBuffer.allocateDirect()로 생성한 버퍼의 크기는 반드시 이 blockSize의 배수여야 오류 없이 작동합니다.
* 성능 팁: 일반적으로 리눅스의 현대적 파일 시스템(ext4, xfs)은 4KB(4096 bytes)를 사용하므로, 하드코딩 대신 위 API를 사용하여 환경에 맞는 버퍼 크기를 동적으로 설정하는 것이 안전합니다. [3, 4, 5, 6, 7, 8, 9] 
