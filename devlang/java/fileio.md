# File IO

## `java.nio` (Modern IO - 권장)

| class | package | Blocking 여부 |
| :--- | :--- | :--- |
| `Files` | `java.nio.file` | Blocking |
| `FileChannel` | `java.nio.channels` | Blocking |
| `AsynchronousFileChannel` | `java.nio.channels` | Non-Blocking |

- **NIO (java.nio)**는 **데이터 전송(Buffer, Channel)**과 성능에 초점을 맞춘 저수준 API
- **NIO (java.nio.channels )**는 [`ByteBuffer`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/ByteBuffer.html) 타입을 read/write 하며, 대용량/고성능 IO 용도임
- ByteBuffer(JVM 힙(Heap))가 아닌, Direct ByteBuffer(OS의 Native Memory) 사용시 `JVM 버퍼 > OS Buffer` 과정 생략 가능

### `Direct ByteBuffer`

- `Direct ByteBuffer`는 **JVM 힙(Heap) 메모리가 아닌, 운영체제(OS)의 네이티브 메모리(Native Memory)에 할당된 버퍼**
- `Direct ByteBuffer`는 **"생성은 느리지만, I/O 속도는 매우 빠른"** 네이티브 메모리 버퍼

#### 1. 특징

- **생성 방법**: `ByteBuffer.allocateDirect(size)` 메서드로 생성
- **위치**: JVM 힙 바깥(Off-heap)에 존재, 따라서 가비지 컬렉션(GC)의 관리 대상에서 다소 벗어남(참조 객체만 힙에 있고 실제 데이터는 밖)
- **비용**: 생성 및 해제 비용이 일반 힙 버퍼(`allocate()`)보다 높음 (OS에 메모리를 요청해야 하므로)

#### 2. 왜 사용하는가? (성능 이점)

- **Zero-Copy (복사 최소화)**:
  - 일반 힙 버퍼(`byte[]`)를 사용하여 I/O를 수행하면, JVM은 내부적으로 이 데이터를 **네이티브 메모리로 한 번 복사**한 후 OS에 전달해야 함 (GC가 힙 메모리를 이동시킬 수 있기 때문에, I/O 도중 주소가 바뀌면 안 되기 때문)
  - `Direct ByteBuffer`는 이미 네이티브 메모리에 고정되어 있으므로, **중간 복사 과정 없이** 바로 OS의 I/O 작업에 전달될 수 있음
- **대용량 I/O 최적화**: 파일 입출력이나 네트워크 전송 등 대량의 데이터를 다룰 때 속도가 훨씬 빠름

#### 3. 언제 사용해야 하는가?

- **추천**:
  - 대용량 파일 I/O가 빈번할 때
  - 네트워크 소켓 통신 등 고성능 I/O가 필요할 때
  - 버퍼를 한 번 만들어서 **오래 재사용**할 때 (생성 비용 상쇄)
- **비추천**:
  - 단순히 데이터를 잠깐 담았다가 버리는 경우 (생성 비용이 더 큼)
  - 작은 크기의 I/O 작업

## `java.io` (Classic IO - 비권장)

전통적인 블로킹(Blocking) I/O 방식입니다.

### **바이트 기반 (이미지, 동영상, 바이너리 데이터)**

- **`FileOutputStream`**: 가장 기본적인 바이트 출력 스트림입니다.
- **`BufferedOutputStream`**: `FileOutputStream`에 버퍼링 기능을 추가하여 성능을 높입니다. (필수적으로 같이 사용 권장)
- **`DataOutputStream`**: 자바의 기본 타입(int, float, boolean 등)을 그대로 파일에 쓸 때 사용합니다.
- **`ObjectOutputStream`**: 자바 객체 자체를 직렬화(Serialization)하여 파일에 쓸 때 사용합니다.

### **문자 기반 (텍스트 파일, JSON, XML)**

- **`FileWriter`**: 텍스트 파일 쓰기용 기본 클래스입니다.
- **`BufferedWriter`**: `FileWriter`에 버퍼링을 추가합니다. `newLine()` 같은 줄바꿈 메서드를 제공합니다.
- **`PrintWriter`**: `System.out.println()`처럼 포맷팅된 출력을 파일에 쓸 때 편리합니다.
- **`RandomAccessFile`**: 파일의 특정 위치로 이동(seek)하여 읽거나 쓸 때 사용합니다. (현재 코드에서 사용 중), `java.nio.channels`의 `FileChannel`, `AsynchronousFileChannel`이 대체제로 사용 가능

---

## 요약 및 추천

| 상황 | 추천 클래스/메서드 | 예시 코드 |
| :--- | :--- | :--- |
| **간단한 텍스트 저장** | `Files.writeString()` | `Files.writeString(path, "Hello");` |
| **간단한 바이너리 저장** | `Files.write()` | `Files.write(path, bytes);` |
| **대용량 텍스트 처리** | `BufferedWriter` | `Files.newBufferedWriter(path)` |
| **대용량/고성능 바이너리** | `FileChannel` | `FileChannel.open(...)` |
| **파일 임의 위치 접근** | `FileChannel` | `.read(dst, pos)` |
| **객체 저장** | `ObjectOutputStream` | `new ObjectOutputStream(stream)` |

- 모든 `FileChannel` 사용 처에서 비동기 지원이 필요한 경우 `AsynchronousFileChannel`을 사용