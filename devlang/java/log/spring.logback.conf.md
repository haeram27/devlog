# spring logback configuration

## LogBackEncoder

```java
package com.example.log.LogBackEncoder;

import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.encoder.LayoutWrappingEncoder;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.security.SecureRandom;
import java.util.Base64;

public class LogBackEncoder extends LayoutWrappingEncoder<ILoggingEvent> {

    // 앞서 생성한 32바이트 키를 Base64에서 복원하거나 직접 바이트 배열로 정의
    private static final byte[] SECRET_KEY = Base64.getDecoder().decode("여기에_생성한_Base64_키_입력");

    @Override
    public byte[] encode(ILoggingEvent event) {
        // 1. 내부 Layout(PatternLayout)을 통해 로그 한 줄 생성
        String rawLog = layout.doLayout(event);
        if (rawLog == null) return new byte[0];

        try {
            // 2. 랜덤 IV 생성 (16바이트)
            byte[] iv = new byte[16];
            new SecureRandom().nextBytes(iv);
            IvParameterSpec ivSpec = new IvParameterSpec(iv);

            // 3. AES-256 암호화
            Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
            SecretKeySpec keySpec = new SecretKeySpec(SECRET_KEY, "AES");
            cipher.init(Cipher.ENCRYPT_MODE, keySpec, ivSpec);
            
            byte[] encrypted = cipher.doFinal(rawLog.getBytes(StandardCharsets.UTF_8));

            // 4. [IV(16) + 암호문] 결합
            byte[] combined = new byte[iv.length + encrypted.length];
            System.arraycopy(iv, 0, combined, 0, iv.length);
            System.arraycopy(encrypted, 0, combined, iv.length, encrypted.length);

            // 5. Base64 인코딩 및 개행 추가
            String base64Result = Base64.getEncoder().encodeToString(combined) + "\n";
            return base64Result.getBytes(StandardCharsets.UTF_8);

        } catch (Exception e) {
            // 암호화 실패 시 안전을 위해 원본 출력 또는 에러 처리
            return (rawLog).getBytes(StandardCharsets.UTF_8);
        }
    }
}
```

## logback 설정 - LogBackEncoder

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>
    <appender name="ROLLING_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>${FILE_LOG_THRESHOLD}</level>
        </filter>

        <!-- use private encoder -->

        <encoder class="com.example.log.LogBackEncoder">
            <layout class="ch.qos.logback.classic.PatternLayout">
                <pattern>${FILE_LOG_PATTERN}</pattern>
                <charset>${FILE_LOG_CHARSET}</charset>
            </layout>
        </encoder>

        <!-- ========================== -->


        <!-- use plain text encoder output -->

        <!-- <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>${FILE_LOG_CHARSET}</charset>
        </encoder> -->

        <!-- ========================== -->

        <file>${LOG_FILE}</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN:-${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz}</fileNamePattern>
            <cleanHistoryOnStart>${LOGBACK_ROLLINGPOLICY_CLEAN_HISTORY_ON_START:-false}</cleanHistoryOnStart>
            <maxFileSize>${LOGBACK_ROLLINGPOLICY_MAX_FILE_SIZE:-10MB}</maxFileSize>
            <totalSizeCap>${LOGBACK_ROLLINGPOLICY_TOTAL_SIZE_CAP:-0}</totalSizeCap>
            <maxHistory>${LOGBACK_ROLLINGPOLICY_MAX_HISTORY:-7}</maxHistory>
        </rollingPolicy>
    </appender>
    
    <springProfile name="local">
        <logger name="com.example" level="DEBUG" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
    
        <root level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
    
    <springProfile name="product">
        <logger name="com.example" level="DEBUG" additivity="false">
            <appender-ref ref="ROLLING_FILE"/>
        </logger>
    
        <root level="INFO" additivity="false">
            <appender-ref ref="ROLLING_FILE"/>
        </root>
    </springProfile>
</configuration>
```