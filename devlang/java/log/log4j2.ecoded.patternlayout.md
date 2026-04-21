# log4j2 EncodedPatternLayout

## log4j2 log encryptor plugin

```java
@Plugin(name = "EncodedPatternLayout", category = "Core", elementType = "layout", printObject = true)
public class EncodedPatternLayout extends AbstractStringLayout {

    private final PatternLayout patternLayout;
    private final byte[] secretKey = {
        0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x30,
        0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x30,
        0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x30,
        0x31, 0x32
    };

    protected EncodedPatternLayout(Charset charset, PatternLayout patternLayout) {
        super(charset);
        this.patternLayout = patternLayout;
    }

    @Override
    public byte[] toByteArray(LogEvent event) {
        try {
            byte[] rawData = patternLayout.toByteArray(event);
            
            // 1. 랜덤 IV 생성 (16바이트)
            byte[] iv = new byte[16];
            new SecureRandom().nextBytes(iv);
            IvParameterSpec ivSpec = new IvParameterSpec(iv);

            // 2. AES 암호화
            Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
            SecretKeySpec keySpec = new SecretKeySpec(secretKey, "AES");
            cipher.init(Cipher.ENCRYPT_MODE, keySpec, ivSpec);
            byte[] encrypted = cipher.doFinal(rawData);

            // 3. [IV(16) + Encrypted Data] 결합
            byte[] combined = new byte[iv.length + encrypted.length];
            System.arraycopy(iv, 0, combined, 0, iv.length);
            System.arraycopy(encrypted, 0, combined, iv.length, encrypted.length);

            // 4. Base64 인코딩 및 개행 추가
            return (Base64.getEncoder().encodeToString(combined) + "\n").getBytes(getCharset());
            
        } catch (Exception e) {
            StatusLogger.getLogger().error("Encryption failed", e);
            return patternLayout.toByteArray(event);
        }
    }

    @Override
    public String toSerializable(LogEvent event) {
        return new String(toByteArray(event), getCharset());
    }

    @PluginFactory
    public static EncodedPatternLayout createLayout(
            @PluginAttribute(value = "pattern", defaultString = PatternLayout.DEFAULT_CONVERSION_PATTERN) String pattern,
            @PluginConfiguration Configuration config,
            @PluginAttribute(value = "charset", defaultString = "UTF-8") Charset charset) {
        
        PatternLayout patternLayout = PatternLayout.newBuilder()
                .withPattern(pattern)
                .withConfiguration(config)
                .build();

        return new EncodedPatternLayout(charset, patternLayout);
    }
}
```

## log4j2 설정 - EcodedPatternLayout

```xml
<Configuration status="WARN" packages="com.your.package.plugin">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <!-- 기존 PatternLayout 대신 커스텀 레이아웃 사용 -->
            <EncodedPatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m" />
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

## secret key generator

```java
import java.security.SecureRandom;
import java.util.Base64;

public class KeyGenerator {
    public static void main(String[] args) {
        String newKey = generateAES256Key();
        System.out.println("Generated AES-256 Key (Base64): " + newKey);
    }

    /**
     * AES-256용 32바이트 랜덤 키를 생성하고 Base64로 인코딩하여 반환
     */
    public static String generateAES256Key() {
        // 1. 보안상 안전한 랜덤 바이트 32개 생성 (256비트)
        byte[] key = new byte[32];
        new SecureRandom().nextBytes(key);

        // 2. Base64 인코딩하여 문자열로 반환
        return Base64.getEncoder().encodeToString(key);
    }
}
```

## decryptor

```java
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.io.File;
import java.io.RandomAccessFile;
import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class LogTailDecryptor {

    private static final String SECRET_KEY = "12345678901234567890123456789012";
    private static final String IV = "1234567890123456";

    public static void main(String[] args) throws Exception {
        if (args.length < 1) {
            System.out.println("Usage: java LogTailDecryptor /path/to/encrypted.log");
            return;
        }

        File logFile = new File(args[0]);
        tailAndDecrypt(logFile);
    }

    public static void tailAndDecrypt(File file) throws Exception {
        // 읽기 전용으로 파일 열기
        try (RandomAccessFile reader = new RandomAccessFile(file, "r")) {
            long lastPointer = file.length(); // 시작 시점: 파일의 맨 끝

            System.out.println("복호화 모니터링 시작: " + file.getName());

            while (true) {
                long fileLength = file.length();

                if (fileLength < lastPointer) {
                    // 로그 파일이 로테이트(비워짐) 되었을 경우 처리
                    System.out.println("--- 파일이 갱신되었습니다 ---");
                    lastPointer = 0;
                    reader.seek(0);
                }

                if (fileLength > lastPointer) {
                    // 새로운 데이터가 추가됨
                    reader.seek(lastPointer);
                    String line;
                    while ((line = reader.readLine()) != null) {
                        // RandomAccessFile은 ISO-8859-1로 읽으므로 UTF-8 변환 필요
                        String utf8Line = new String(line.getBytes(StandardCharsets.ISO_8859_1), StandardCharsets.UTF_8);
                        
                        try {
                            if (!utf8Line.trim().isEmpty()) {
                                String decrypted = decrypt(utf8Line);
                                System.out.println(decrypted);
                            }
                        } catch (Exception e) {
                            // 암호화되지 않은 줄이나 오류 발생 시 원본 출력
                            System.out.println("[ERROR/RAW] " + utf8Line);
                        }
                    }
                    lastPointer = reader.getFilePointer();
                }

                Thread.sleep(500); // 0.5초 간격으로 체크
            }
        }
    }

    private static String decrypt(String encryptedLog) throws Exception {
        // 1. Base64 디코딩
        byte[] combined = Base64.getDecoder().decode(encryptedLog.trim());

        // 2. 앞 16바이트에서 IV 추출
        if (combined.length < 16) throw new IllegalArgumentException("Invalid log data");
        byte[] iv = new byte[16];
        System.arraycopy(combined, 0, iv, 0, 16);
        IvParameterSpec ivSpec = new IvParameterSpec(iv);

        // 3. 나머지 바이트에서 암호문(Ciphertext) 추출
        int cipherTextLen = combined.length - 16;
        byte[] cipherText = new byte[cipherTextLen];
        System.arraycopy(combined, 16, cipherText, 0, cipherTextLen);

        // 4. 추출한 IV와 고정 키로 복호화
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        SecretKeySpec keySpec = new SecretKeySpec(SECRET_KEY.getBytes(StandardCharsets.UTF_8), "AES");
        cipher.init(Cipher.DECRYPT_MODE, keySpec, ivSpec);
        
        return new String(cipher.doFinal(cipherText), StandardCharsets.UTF_8);
    }
}
```
