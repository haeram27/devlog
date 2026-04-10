# spring daemonize

spring의 경우 spring-web 사용시 자동적으로 http 서버를 구동하며 daemonize가 된다.
spring-web이 아닌 경우 별도로 main thread가 종료되지 않도록 하여 daemonize 해야 한다.

## spring-web disable daemonize

- spring-web 사용시 자동적으로 http 서버를 구동하며 daemonize가 됨
- 단 rest client와 같은 어플리케이션 구현시 spring-web을 사용하여 http library를 사용해야하지만 어플리케이션이 daemonize 되지 않도록 설정 해야함

### File: SpringApplication.java

```java
package com.example.httpclient;

import org.springframework.boot.Banner;
import org.springframework.boot.WebApplicationType;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;

import lombok.extern.slf4j.Slf4j;


@SpringBootApplication
@Slf4j
public class SpringApplication {

    public static void main(String[] args) throws Exception {
        log.trace("*** main() ***");
        new SpringApplicationBuilder(SpringApplication.class)
        .web(WebApplicationType.NONE)   // Do NOT run as server
        .bannerMode(Banner.Mode.OFF)    // Do NOT display Spring Banner on LOG when app is started
        .run(args);
    }
}
```

## non-web spring daemonize

CountDownLatch를 이용하는 것이 가장 깔끔

- CountDownLatch: daemonize(프로세스 생존 유지) 담당
- scheduler: 주기적인 작업에 권장

### Java: ServiceApplication.java

```java
package com.example.service;

import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.CountDownLatch;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.Banner;
import org.springframework.boot.WebApplicationType;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextClosedEvent;

@SpringBootApplication
@Slf4j
public class ServiceApplication {

    private static final CountDownLatch shutdownLatch = new CountDownLatch(1);
    private static final AtomicInteger counter = new AtomicInteger(0);
    private static final ScheduledExecutorService scheduler =
            Executors.newSingleThreadScheduledExecutor();

    public static void main(String[] args) throws InterruptedException {
        new SpringApplicationBuilder(ServiceApplication.class)
                .web(WebApplicationType.NONE)
                .bannerMode(Banner.Mode.OFF)
                .listeners((ApplicationListener<ApplicationEvent>) event -> {
                    log.info("event={}", event.getClass().getSimpleName());

                    if (event instanceof ApplicationReadyEvent) {
                        scheduler.scheduleAtFixedRate(() -> {
                            int id = counter.incrementAndGet();
                            log.info("tick id={}", id);
                        }, 0, 200, TimeUnit.MILLISECONDS);
                    }

                    if (event instanceof ContextClosedEvent) {
                        scheduler.shutdown();
                        try {
                            if (!scheduler.awaitTermination(5, TimeUnit.SECONDS)) {
                                scheduler.shutdownNow();
                            }
                        } catch (InterruptedException ie) {
                            Thread.currentThread().interrupt();
                            scheduler.shutdownNow();
                        } finally {
                            shutdownLatch.countDown();
                        }
                    }
                })
                .run(args);

        shutdownLatch.await();
    }
}
```

### Kotlin : ServiceApplication.kt

```kotlin
package com.example.service

import io.github.oshai.kotlinlogging.KotlinLogging
import java.util.concurrent.CountDownLatch
import java.util.concurrent.Executors
import java.util.concurrent.TimeUnit
import java.util.concurrent.atomic.AtomicInteger
import org.springframework.boot.Banner
import org.springframework.boot.WebApplicationType
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.builder.SpringApplicationBuilder
import org.springframework.boot.context.event.ApplicationReadyEvent
import org.springframework.context.ApplicationEvent
import org.springframework.context.ApplicationListener
import org.springframework.context.event.ContextClosedEvent

@SpringBootApplication
class ServiceApplication

private val log = KotlinLogging.logger {}
private val shutdownLatch = CountDownLatch(1)
private val counter = AtomicInteger(0)
private val scheduler = Executors.newSingleThreadScheduledExecutor()

fun main(args: Array<String>) {
    SpringApplicationBuilder(ServiceApplication::class.java)
        .web(WebApplicationType.NONE)
        .bannerMode(Banner.Mode.OFF)
        .listeners(
            ApplicationListener<ApplicationEvent> { event ->
                log.info { "event=${event.javaClass.simpleName}" }

                if (event is ApplicationReadyEvent) {
                    scheduler.scheduleAtFixedRate(
                        {
                            val id = counter.incrementAndGet()
                            log.info { "tick id=$id" }
                        },
                        0,
                        200,
                        TimeUnit.MILLISECONDS,
                    )
                }

                if (event is ContextClosedEvent) {
                    scheduler.shutdown()
                    try {
                        if (!scheduler.awaitTermination(5, TimeUnit.SECONDS)) {
                            scheduler.shutdownNow()
                        }
                    } catch (e: InterruptedException) {
                        Thread.currentThread().interrupt()
                        scheduler.shutdownNow()
                    } finally {
                        shutdownLatch.countDown()
                    }
                }
            }
        )
        .run(*args)

    shutdownLatch.await()
}
```

## Non-Spring Daemonize

### Kotlin: CountDownLatch 기반 (가장 단순한 daemonize)

```kotlin
import java.util.concurrent.CountDownLatch
import java.util.concurrent.atomic.AtomicBoolean

private val running = AtomicBoolean(true)
private val shutdownLatch = CountDownLatch(1)

fun main() {
    // 종료 훅: Ctrl+C(SIGINT)나 정상 종료 시 호출
    Runtime.getRuntime().addShutdownHook(
        Thread {
            println("shutdown hook called")
            running.set(false)
            shutdownLatch.countDown()
        }
    )

    // 실제 작업 스레드(예: 폴링/소비 루프)
    Thread.startVirtualThread {
        while (running.get()) {
            println("worker alive...")
            Thread.sleep(1000)
        }
        println("worker stopped")
    }

    // 메인 스레드 대기 (프로세스 유지)
    shutdownLatch.await()
    println("main exit")
}
```

### Kotlin: CountDownLatch + Scheduler 기반 (주기 작업(heartbeat, polling, batch) 에 권장)

```kotlin
import java.util.concurrent.CountDownLatch
import java.util.concurrent.Executors
import java.util.concurrent.ScheduledExecutorService
import java.util.concurrent.TimeUnit
import java.util.concurrent.atomic.AtomicInteger

private val shutdownLatch = CountDownLatch(1)
private val counter = AtomicInteger(0)
private val scheduler: ScheduledExecutorService = Executors.newSingleThreadScheduledExecutor()

fun main() {
    // 종료 훅: Ctrl+C(SIGINT)나 정상 종료 시 호출
    Runtime.getRuntime().addShutdownHook(
        Thread {
            println("shutdown hook called")
            scheduler.shutdown()
            try {
                if (!scheduler.awaitTermination(5, TimeUnit.SECONDS)) {
                    scheduler.shutdownNow()
                }
            } finally {
                shutdownLatch.countDown()
            }
        }
    )

    scheduler.scheduleAtFixedRate(
        {
            val id = counter.incrementAndGet()
            println("heartbeat #$id")
        },
        0,
        1,
        TimeUnit.SECONDS,
    )

    // 메인 스레드 대기 (프로세스 유지)
    shutdownLatch.await()
    println("main exit")
}
```