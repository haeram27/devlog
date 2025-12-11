# Linux 메모리 사용량 확인 및 Cache 비우기

## 호스트 메모리 사용량 확인 (간략)

```bash
free
```

## 호스트 메모리 사용량 확인

```bash
cat /proc/meminfo
```

## 컨테이너(cgroup) 메모리 사용량 확인

```bash
docker exec my-app cat /sys/fs/cgroup/memory/memory.stat
```

## Cache(디스크 IO 캐시) 삭제

### cli

```bash
# 1 = page cache 비우기 (현재 목적)
echo 1 > /proc/sys/vm/drop_caches

# 2 = dentries와 inodes 비우기  
echo 2 > /proc/sys/vm/drop_caches

# 3 = page cache + dentries + inodes 모두 비우기
echo 3 > /proc/sys/vm/drop_caches
```

- `/proc/sys/vm/drop_caches`는 호스트 시스템 전역의 모든 캐시를 삭제함 (컨테이너 범위 아님)
- 프로덕션 환경에서 사용 금지(테스트용)
- 정상적인 운영에서는 필요하지 않음
- 시스템 성능을 급격히 저하시킴 (캐시를 다시 쌓아야 함)
- 주로 벤치마킹이나 테스트 목적으로만 사용

### posix 함수

```bash
posix_fadvise(POSIX_FADV_DONTNEED)
```

- 선택적으로 특정 파일의 캐시를 비움
- 파일내 특정 부분(offset)만 삭제도 가능
- 어플리케이션이 커널에게 처리를 권고하는 것일뿐 강제 실행력은 없음

#### posix_fadvise man page

프로그램에서 posix_fadvise()를 사용해 향후 특정 패턴으로 파일 데이터에 접근하겠다는 의도를 선언할 수 있다. 그러면 커널이 적절한 최적화를 수행할 수 있게 된다.

fd가 가리키는 파일 내에서 offset에서 시작해 len 바이트만큼 (len이 0이면 파일 끝까지) 이어지는 (꼭 존재해야 하는 것은 아닌) 영역에 advice를 적용한다. ***advice에는 강제력이 없으며 그저 어플 입장에서의 가이드일 뿐이다.(커널이 반드시 실행한다는 보장이 없음)***

advice에 다음 값이 가능하다.

- POSIX_FADV_NORMAL
  - 지정한 데이터에 대한 접근 패턴에 관해 응용에서 해 줄 조언이 없음을 나타낸다. 열린 파일에 아무 조언도 주지 않으면 기본으로 상정하는 값이다.

- POSIX_FADV_SEQUENTIAL
  - 응용에서 지정한 데이터에 순차적으로 (오프셋이 작은 데이터를 먼저 읽는 식으로) 접근할 예정이다.

- POSIX_FADV_RANDOM
  - 지정한 데이터에 임의 순서로 접근하려 한다.

- POSIX_FADV_NOREUSE
  - 지정한 데이터에 한 번만 접근하려 한다.

  - 2.6.18 전의 커널에서 POSIX_FADV_NOREUSE는 POSIX_FADV_WILLNEED와 동작이 같았다. 버그였던 것 같다. 커널 2.6.18부터 이 플래그는 no-op이다.

- POSIX_FADV_WILLNEED
  - 지정한 데이터에 조만간 접근하려 한다.

  - POSIX_FADV_WILLNEED는 지정한 영역을 페이지 캐시로 읽어 들이는 논블록 동작을 개시한다. 가상 메모리 부하에 따라 읽는 데이터 양을 커널이 줄일 수도 있다. (보통 몇 메가바이트 정도는 완전히 충족되고 그 이상으로는 잘 쓰지 않는다.)

- POSIX_FADV_DONTNEED
  - 지정한 데이터에 당분간은 접근하지 않을 것이다.
  - POSIX_FADV_DONTNEED는 지정한 영역과 연계된 캐싱 된 페이지들을 해제 시도한다. 예를 들어 큰 파일을 스트리밍 할 때 유용하다. 프로그램에서 이미 사용한 데이터를 캐시에서 해제하라고 주기적으로 커널에게 요청할 수 있고, 그러면 캐시 내의 더 유용한 페이지들이 폐기되지 않을 수 있다.

  - 페이지 일부만 폐기하는 요청은 무시한다. 불필요한 데이터를 폐기하는 것보다 필요한 데이터를 보존하는 쪽이 중요하다. 데이터 폐기를 고려하게 하려면 offset과 len이 페이지에 정렬되어 있어야 한다.

  - 지정 영역 내의 변경된 페이지들을 구현체가 기반 장치로 기록하려 시도할 수도 있다. 하지만 이를 보장하지는 않는다. 기록 안 된 변경 페이지가 있으면 해제되지 않을 것이다. 변경된 페이지가 해제되게 하고 싶으면 응용에서 먼저 fsync(2)나 fdatasync(2)를 호출해야 할 것이다.

#### JAVA - posix_fadvise 사용하기(JNA)

pom.xml:

```xml
<dependency>
    <groupId>net.java.dev.jna</groupId>
    <artifactId>jna</artifactId>
    <version>5.18.1</version>
</dependency>
```

CLibrary.java

```java
import com.sun.jna.Library;
import com.sun.jna.Native;
import com.sun.jna.NativeLong;

public interface CLibrary extends Library {
    // load "libc"
    CLibrary INSTANCE = (CLibrary) Native.load("c", CLibrary.class);

    // Constants for posix_fadvise
    int POSIX_FADV_NORMAL = 0;
    int POSIX_FADV_RANDOM = 1;
    int POSIX_FADV_SEQUENTIAL = 2;
    int POSIX_FADV_WILLNEED = 3;
    int POSIX_FADV_DONTNEED = 4;
    int POSIX_FADV_NOREUSE = 5;

    /**
     * int posix_fadvise(int fd, off_t offset, off_t len, int advice);
     */
    int posix_fadvise(int fd, NativeLong offset, NativeLong len, int advice);
}
```

```java
import com.sun.jna.NativeLong;
import com.sun.jna.Platform;
import java.io.FileDescriptor;
import java.lang.reflect.Field;

public class CleanCache {
    /**
     * Advise the OS to free cache pages for the entire file (len=0)
     * Ensure offset and length are page-aligned for best results
     * 
     * @param fd
     * @param offset start offset to drop cache of file. Offset 0 means from the beginning of the file.
     * @param length length to drop cache of file from the beginning of the offset.  Length 0 means until the end of the file.
     */
    private void dropPageCache(FileDescriptor fd, long offset, long length) {
        if (!Platform.isLinux()) return;
        try {
            Field fdField = FileDescriptor.class.getDeclaredField("fd");
            fdField.setAccessible(true);
            int nativeFd = fdField.getInt(fd);

            CLibrary.INSTANCE.posix_fadvise(
                nativeFd, new NativeLong(offset), new NativeLong(length), CLibrary.POSIX_FADV_DONTNEED
            );
        } catch (Exception e) {
            log.warn("Failed to drop cache: {}", e.getMessage());
        }
    }
}
```

```text
Failed to drop cache: Unable to make field private int java.io.FileDescriptor.fd accessible: module java.base does not "opens java.io" to unnamed module @2de23121
```

- 원인 :
  - `FileDescriptor.class.getDeclaredField("fd");`는 리플렉션 사용 코드
  - Java 9 이상에서는 java.base 모듈(여기서는 `java.io.FileDescriptor`)의 내부 필드에 리플렉션으로 접근하는 것이 기본적으로 차단됨

- 해결 방법:
  - JVM 시작 시 --add-opens 옵션을 추가하여 java.io 패키지를 열어주는 것

```bash
java --add-opens java.base/java.io=ALL-UNNAMED -jar my-app.jar
```