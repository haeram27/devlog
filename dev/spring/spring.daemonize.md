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

- 방법 1: main()에서 무한 루프 Thread 사용
  - 사용처: 테스트용 어플리케이션
  - 장점: 실행중 여부를 판단(hear beat)하기 위한 로그 출력 가능
  - 단점: main 메소드가 복잡해짐

- 방법 2: main()에서 Condition await() 사용
  - 사용처: 상용 서비스
  - 장점: main() 메소드가 깔끔해짐
  - 단점: 별도로 로그가 거의 없는 어플리케이션의 경우 실행중 여부를 판단하기 어려움

### main()에서 무한 루프 Thread 사용하여 daemonize

spring-web이 아닌 경우 별도로 main thread의 종료를 막아 daemonize 함

#### ServiceApplication.java

```java
package com.example.service;

import java.util.concurrent.atomic.AtomicInteger;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextClosedEvent;
import lombok.extern.slf4j.Slf4j;

@SpringBootApplication
@Slf4j
public class ServiceApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(ServiceApplication.class)
                .listeners(new ApplicationListener<ApplicationEvent>() {
                    private static AtomicInteger ai = new AtomicInteger(0);

                    @Override
                    public void onApplicationEvent(ApplicationEvent event) {
                        // log.info(":: springboot :: event :: " + event.toString());
                        if (event instanceof ApplicationReadyEvent) {
                            // implementations under HERE after springboot initialization

                            Thread.startVirtualThread({
                                try {
                                    while (true) {
                                        log.info("log message ID:"
                                                + String.valueOf(ai.incrementAndGet()));
                                        Thread.sleep(200);
                                    }
                                } catch (Exception e) {
                                    log.error("Exception: ", e);
                                }
                            });
                        } else if (event instanceof ContextClosedEvent) {
                            // implementations under HERE before applcation closed
                        }
                    }
                }).run(args);

        new Thread() {
            @Override
            public void run() {
                try {
                    while (true) {
                        log.info("heart beat");
                        Thread.sleep(1000);
                    }
                } catch (Exception e) {
                    log.error("Exception: ", e);
                }
            }
        }.start();
    }
}
```

### main()에서 main thread에 Condition await() 사용하여 daemonize

#### ServiceApplication.java

```java
package com.example.service;

import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.event.ContextClosedEvent;
import org.springframework.context.event.EventListener;
import lombok.extern.slf4j.Slf4j;

@SpringBootApplication
@Slf4j
public class ServiceApplication {

    private static ReentrantLock LOCK_TERM = new ReentrantLock();
    private static Condition COND_TERM = LOCK_TERM.newCondition();
    private static AtomicInteger ATOM_INT = new AtomicInteger(0);

    @EventListener
    public void onApplicationEvent(ApplicationReadyEvent e) {
        // applicaiton started
        log.info("@@@ ApplicationReadyEvent");

        Thread.startVirtualThread({
            try {
                while (true) {
                    log.info("log message ID:" + String.valueOf(ai.incrementAndGet()));
                    Thread.sleep(200);
                }
            } catch (Exception e) {
                log.error("Exception: ", e);
            }
        });
    }

    @EventListener
    public void onApplicationEvent(ContextClosedEvent e) {
        // applicaiton terminated
        log.info("@@@ ContextClosedEvent");

        // wakeup main thread
        LOCK_TERM.lock(); // acquire lock object
        try {
            COND_TERM.signalAll(); // send signal to all waiting thread
        } finally {
            LOCK_TERM.unlock(); // release lock object
        }
    }

    public static void main(String[] args) throws Exception {
        new SpringApplicationBuilder(ServiceApplication.class).run(args);

        // wait on main thread
        LOCK_TERM.lock(); // acquire lock object
        try {
            COND_TERM.await(); // make main thread wait status
        } finally {
            LOCK_TERM.unlock(); // release lock object
        }
    }
}
```
