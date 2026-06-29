# Channel

## Channel vs Stream

핵심적으로 Stream은 한 방향으로만 흐르는 '빨대'와 같고, Channel은 양방향 통신이 가능한 '도로'와 같습니다. 두 방식의 주요 차이점은 다음과 같습니다.

### 1. 데이터의 방향 (Direction)

- Stream: 단방향입니다. 데이터를 읽으려면 InputStream, 쓰려면 OutputStream을 각각 생성해야 합니다.
- Channel: 양방향입니다. 하나의 채널 객체로 읽기(Read)와 쓰기(Write)를 동시에 수행할 수 있습니다. (예: ByteChannel)

### 2. 버퍼 사용 여부 (Buffering)

- Stream: 스트림 기반입니다. 버퍼를 사용하지 않고 데이터를 1바이트씩 즉시 처리하거나, 별도의 BufferedStream을 감싸서 사용해야 합니다.
- Channel: 버퍼 기반입니다. 반드시 ByteBuffer에 데이터를 먼저 담은 뒤 채널을 통해 주고받습니다. 이 덕분에 데이터 처리가 더 효율적입니다.

기본적인(Primitive) Stream 클래스들은 JVM 힙 메모리에 별도의 중간 버퍼를 생성하지 않고 데이터를 즉시 전달합니다.
구체적으로 비교하면 다음과 같습니다.

#### 기본 Stream (예: FileInputStream)

- read() 메소드를 호출하면 OS의 시스템 콜을 통해 데이터를 가져온 뒤, 사용자가 인자로 넘긴 byte[] 배열(JVM 힙에 위치)에 직접 담아줍니다.
- 이 과정에서 스트림 객체 자체가 내부적으로 들고 있는 별도의 힙 버퍼는 없습니다. 그래서 1바이트씩 읽을 경우 매번 시스템 콜이 발생해 속도가 매우 느려지는 것입니다.

#### BufferedStream (예: BufferedInputStream)

- 속도 문제를 해결하기 위해 내부적으로 new byte[8192] 같은 JVM 힙 버퍼를 하나 가집니다.
- OS로부터 한꺼번에 8KB를 힙 버퍼로 긁어온 뒤, 사용자가 원할 때마다 힙 내에서 데이터를 쪼개서 줍니다.

#### Channel의 특수성 (Direct Buffer)

반면, Channel은 JVM 힙이 아닌 OS의 커널 메모리 영역에 직접 버퍼를 생성(Direct Buffer)할 수 있습니다.

- Stream: OS 커널 버퍼 → JVM 힙 버퍼(복사) → 어플리케이션 (복사 발생)
- Channel: OS 커널 버퍼(Direct Buffer) → 어플리케이션 (복사 생략)

결론적으로:
일반적인 Stream은 데이터를 담기 위한 '사용자 배열'은 힙에 두지만, 내부적인 '중간 대기용 버퍼'는 사용하지 않을 수 있습니다. 반면 Channel은 힙을 완전히 거치지 않는 통로(DirectBuffer)를 만들 수 있다는 점이 가장 큰 차이입니다. DirectBuffer를 사용하면 JVM에서 복사 과정이 생략되는 Zero-copy 방식을 사용할 수 있습니다.

### 3. 차단 vs 비차단 (Blocking vs Non-blocking)

- Stream: 기본적으로 블로킹(Blocking) 방식입니다. 데이터를 읽거나 쓸 때 작업이 완료될 때까지 쓰레드가 대기 상태에 빠집니다.
- Channel: 넌블로킹(Non-blocking) 모드를 지원합니다. 데이터가 준비되지 않았을 때 쓰레드가 대기하지 않고 다른 일을 할 수 있어, 다수의 연결을 효율적으로 관리할 수 있습니다.

### 요약 비교표

| 구분 | Stream (IO) | Channel (NIO) |
|---|---|---|
| 작동 방식 | 스트림 기반 (Byte/Char) | 버퍼 기반 (Buffer) |
| 방향성 | 단방향 (In/Out 분리) | 양방향 (하나의 객체로 가능) |
| 차단 방식 | 항상 블로킹 (Blocking) | 블로킹 + 넌블로킹 지원 |
| 주 사용처 | 단순 파일 읽기/쓰기 | 대규모 동시 접속, 고성능 네트워크 |

## Zero Copy의 원리와 DirectBuffer(OS 커널 버퍼)

Zero-copy의 핵심은 데이터를 전송할 때 CPU가 메모리 사이에서 데이터를 복사하는 횟수를 최소화(0에 가깝게) 하는 기술입니다.

일반적인 방식과 Zero-copy 방식의 차이를 보면 이해가 빠릅니다.

### 1. 일반적인 방식 (4번의 복사)

파일을 읽어서 네트워크로 전송할 때, 보통 다음과 같은 단계를 거칩니다.

   1. 하드디스크 → 커널 버퍼: OS가 파일을 읽어 커널 메모리에 저장 (DMA 복사)
   2. 커널 버퍼 → 유저 버퍼: [CPU 복사] JVM 힙 메모리(애플리케이션)로 데이터 복사
   3. 유저 버퍼 → 소켓 버퍼: [CPU 복사] 전송을 위해 다시 커널의 소켓 버퍼로 복사
   4. 소켓 버퍼 → NIC(랜카드): 네트워크 장치로 전송 (DMA 복사)

문제점: 중간에 CPU가 개입하여 커널 ↔ 유저 영역 간에 데이터를 복사하는 과정(2, 3번)에서 자원이 낭비됩니다.

### 2. Zero-copy 방식 (2번의 복사)

Java NIO의 FileChannel.transferTo() 같은 메소드를 사용하면, 유저 영역(JVM)을 거치지 않고 커널 영역 내에서 데이터를 직접 토스합니다.

   1. 하드디스크 → 커널 버퍼: OS가 파일을 읽어 커널 메모리에 저장 (DMA 복사)
   2. 커널 버퍼 → NIC(랜카드): 커널이 유저 영역을 거치지 않고 바로 소켓/네트워크로 전송 (DMA 복사)

장점: CPU가 데이터를 복사하는 작업이 사라집니다. 데이터는 유저 영역으로 올라오지 않고 커널 안에서만 움직이므로 Context Switching 비용과 메모리 복사 비용이 획기적으로 줄어듭니다.

### 왜 Channel에서 이게 가능한가요?

Java의 Direct Buffer나 FileChannel은 운영체제의 저수준 기능(Linux의 sendfile 시스템 콜 등)을 직접 호출할 수 있도록 설계되었기 때문입니다. 반면 일반 Stream은 데이터를 JVM 내부 배열로 가져와야만 처리가 가능하므로 이 최적화를 쓰기 어렵습니다.

정리하자면:

- Zero-copy는 "데이터를 내 집(JVM 힙)으로 들고 오지 않고, 택배 기사(OS)들끼리 밖에서 바로 전달하게 만드는 것"과 같습니다.
- 이 기능이 대규모 트래픽 처리에 필수적인데, 혹시 Kafka나 Nginx 같은 기술에서 이 원리가 어떻게 쓰이는지도 궁금하신가요?

## Channel에서 버퍼 사용(ByteBuffer)

ByteBuffer는 Direct Buffer일 수도 있고, 아닐 수도 있습니다. 사용자가 어떤 메소드로 생성하느냐에 따라 결정됩니다.
ByteBuffer를 만드는 방식은 크게 두 가지입니다.

### 1. HeapByteBuffer (Non-direct)

- 생성법: ByteBuffer.allocate(1024);
- 특징: JVM 힙 메모리에 바이트 배열을 생성합니다.
- 채널과의 작동: 채널을 통해 데이터를 읽고 쓸 때, 결국 OS 수준의 I/O를 수행해야 하므로 내부적으로 임시 Direct Buffer를 만들어 데이터를 복사하는 과정을 한 번 더 거칩니다.

### 2. DirectByteBuffer (Direct)

- 생성법: ByteBuffer.allocateDirect(1024);
- 특징: JVM 힙이 아닌 OS 커널 영역(네이티브 메모리)에 직접 버퍼를 생성합니다.
- 채널과의 작동: 앞서 설명한 Zero-copy 효과를 보려면 반드시 이 Direct Buffer를 사용해야 합니다. 복사 과정 없이 OS가 이 메모리에 직접 데이터를 쓰고 읽기 때문입니다.

### 요약하자면

- Channel은 양쪽 모두와 일할 수 있습니다.
- 하지만 성능 최적화가 목적이라면 allocateDirect()로 만든 Direct Buffer를 채널에 물려주는 것이 유리합니다.
- 반대로 데이터 가공이 많고 버퍼 크기가 작다면 관리가 편한 Heap Buffer가 나을 수도 있습니다. (Direct Buffer는 할당/해제 비용이 더 비쌉니다.)

## Channels methods

Java의 java.nio.channels.Channels 클래스는 java.io 패키지의 스트림(Stream)과 NIO 채널(Channel) 간의 상호 운용성을 지원하는 유틸리티 클래스입니다. 모든 메소드는 정적(static)이며, 주요 목록은 다음과 같습니다.

- 참고: [Oracle 공식 API 문서](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/channels/Channels.html)

### 1. 스트림에서 채널 생성 (Stream → Channel)

스트림 객체를 NIO 채널로 변환하여 사용할 때 사용합니다.

- newChannel(InputStream in): 입력 스트림으로부터 바이트를 읽는 ReadableByteChannel을 생성합니다.
- newChannel(OutputStream out): 출력 스트림에 바이트를 쓰는 WritableByteChannel을 생성합니다. [3, 4] 

### 2. 채널에서 스트림 생성 (Channel → Stream) [4] 

채널 객체를 기존 스트림 기반의 API와 연결할 때 사용합니다.

- newInputStream(ReadableByteChannel ch): 채널에서 데이터를 읽는 InputStream을 생성합니다.
- newInputStream(AsynchronousByteChannel ch): 비동기 채널에서 데이터를 읽는 InputStream을 생성합니다.
- newOutputStream(WritableByteChannel ch): 채널에 데이터를 쓰는 OutputStream을 생성합니다.
- newOutputStream(AsynchronousByteChannel ch): 비동기 채널에 데이터를 쓰는 OutputStream을 생성합니다.

### 3. 채널에서 Reader/Writer 생성 (Channel → Reader/Writer)

채널과 특정 문자셋(Charset)을 연결하여 텍스트 데이터를 처리할 때 사용합니다.

- newReader(ReadableByteChannel ch, String charsetName): 지정된 이름의 문자셋을 사용하여 바이트를 디코딩하는 Reader를 생성합니다.
- newReader(ReadableByteChannel ch, Charset charset): 지정된 Charset 객체를 사용하는 Reader를 생성합니다.
- newReader(ReadableByteChannel ch, CharsetDecoder dec, int minBufferCap): 지정된 디코더를 사용하여 Reader를 생성합니다.
- newWriter(WritableByteChannel ch, String charsetName): 문자를 인코딩하여 채널에 쓰는 Writer를 생성합니다.
- newWriter(WritableByteChannel ch, Charset charset): 지정된 Charset 객체를 사용하는 Writer를 생성합니다.
- newWriter(WritableByteChannel ch, CharsetEncoder enc, int minBufferCap): 지정된 인코더를 사용하는 Writer를 생성합니다.