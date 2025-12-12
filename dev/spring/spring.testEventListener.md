# Create Thread for Test in spring

## Test: Log per 1 second

```java
import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.Channels;
import java.nio.channels.FileChannel;
import java.nio.channels.ReadableByteChannel;
import java.time.Instant;
import java.time.OffsetDateTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.util.Random;
import java.util.UUID;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.ApplicationContext;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
@RequiredArgsConstructor
public class TestEventListener {
    private final ApplicationContext applicationContext;

@EventListener
    public void handleApplicationReady(ApplicationReadyEvent event) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                Thread.currentThread().setName("TestEventListener-Thread");
                log.info("TestEventListener Thread Started: " + OffsetDateTime.ofInstant(Instant.now(),
                            ZoneId.systemDefault()).format(DateTimeFormatter.ISO_OFFSET_DATE_TIME));

                try {
                    // 시스템 부팅 후 안정화 대기
                    Thread.sleep(30000); 
                } catch (InterruptedException e1) {
                    log.error("## Error", e1);
                }

                while (true) {
                    try {
                        var now = OffsetDateTime.ofInstant(Instant.now(),
                                    ZoneId.systemDefault()).format(DateTimeFormatter.ISO_OFFSET_DATE_TIME);

                        log.info("now: {}", now);

                        Thread.sleep(1000);
                    } catch (Exception e) {
                        log.error("## File Write Error", e);
                        try { Thread.sleep(1000); } catch (InterruptedException ie) {}
                    }
                }
            }
        }).start();
    }
}
```

## Test: Infinite File Write

```java
import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.Channels;
import java.nio.channels.FileChannel;
import java.nio.channels.ReadableByteChannel;
import java.time.Instant;
import java.time.OffsetDateTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.util.Random;
import java.util.UUID;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.ApplicationContext;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
@RequiredArgsConstructor
public class TestEventListener {
    private static final String TEST_FILE_PATH = "/tmp/test-upload";
    private static final int CHUNK_SIZE = 1024 * 1024 * 10; // 10MB Chunk

    private final ApplicationContext applicationContext;

@EventListener
    public void handleApplicationReady(ApplicationReadyEvent event) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                Thread.currentThread().setName("TestEventListener-Thread");
                log.info("TestEventListener Thread Started: " + OffsetDateTime.ofInstant(Instant.now(),
                            ZoneId.systemDefault()).format(DateTimeFormatter.ISO_OFFSET_DATE_TIME));

                try {
                    // 시스템 부팅 후 안정화 대기
                    Thread.sleep(30000); 
                } catch (InterruptedException e1) {
                    log.error("## Error", e1);
                }

                // 테스트용 더미 데이터 생성 (매번 생성하면 CPU 낭비가 심하므로 재사용)
                byte[] dummyData = new byte[CHUNK_SIZE];
                new Random().nextBytes(dummyData);

                // 디렉토리 생성
                new File(TEST_FILE_PATH).mkdirs();

                long totalBytesWritten = 0;
                int count = 0;

                while (true) {
                    try {
                        count++;
                        String fileName = UUID.randomUUID().toString() + ".temp";
                        long startTime = System.currentTimeMillis();

                        // 파일 쓰기 수행
                        writeTestFile(fileName, dummyData);

                        long duration = System.currentTimeMillis() - startTime;
                        totalBytesWritten += CHUNK_SIZE;

                        log.info("Chunk #{} Written. File: {}, Size: 10MB, Time: {}ms, Total: {}MB", 
                            count, fileName, duration, totalBytesWritten / (1024 * 1024));

                        // 너무 빠른 루프 방지 (필요 시 주석 해제)
                        // Thread.sleep(100);

                    } catch (Exception e) {
                        log.error("## File Write Error", e);
                        try { Thread.sleep(1000); } catch (InterruptedException ie) {}
                    }
                }
            }
        }).start();
    }

    private void writeTestFile(String fileName, byte[] sourceData) throws IOException {
        File targetFile = new File(TEST_FILE_PATH, fileName);
        
        // 1. Input Stream 준비 (메모리 데이터를 스트림으로 변환)
        InputStream is = new ByteArrayInputStream(sourceData);

        try (
            // 2. RandomAccessFile & FileChannel 열기
            RandomAccessFile raf = new RandomAccessFile(targetFile, "rw");
            FileChannel fileChannel = raf.getChannel();
            
            // 3. InputStream을 ReadableByteChannel로 변환
            ReadableByteChannel inputChannel = Channels.newChannel(is)
        ) {
            // Direct Buffer 할당 (Native I/O 성능 최적화)
            ByteBuffer buffer = ByteBuffer.allocateDirect(1024 * 64); // 64KB Buffer

            // 4. Channel 간 데이터 전송
            while (inputChannel.read(buffer) != -1) {
                buffer.flip(); // 읽기 모드 -> 쓰기 모드
                while (buffer.hasRemaining()) {
                    fileChannel.write(buffer);
                }
                buffer.clear(); // 버퍼 비우기
            }

            // 5. 디스크 강제 동기화 (Page Cache Flush 유도)
            fileChannel.force(true);
        }
    }
}
```
