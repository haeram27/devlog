# spring logback configuration

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
        <!--
        <encoder class="com.example.log.Encoder">
            <layout class="ch.qos.logback.classic.PatternLayout">
                <pattern>${FILE_LOG_PATTERN}</pattern>
                <charset>${FILE_LOG_CHARSET}</charset>
            </layout>
        </encoder>
        -->
        <!-- ========================== -->


        <!-- use plain text encoder output -->

        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>${FILE_LOG_CHARSET}</charset>
        </encoder>

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