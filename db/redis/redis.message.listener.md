# Redis keyspace event notification

## ref

- [redis official docs: keyspace-notifications](https://redis.io/docs/latest/develop/pubsub/keyspace-notifications/)
- [blog: keysapce notification](https://px201226.github.io/redis-event-notifications/)

### redis config

```java
@Configuration
public class RedisConfig {

    private static final String EXPIRED_EVENT_PATTERN = "__keyevent@*__:expired";
    private static final String EXPIRED_SPACE_PATTERN = "__keyspace@*__:expired";
    private static final String SET_EVENT_PATTERN = "__keyevent@*__:set";
    private static final String PUBLISH_KEY = "settings:testpubkey";

    ... // RedisTemplate 관련 코드 생략

    @Bean(name = "redisMessageTaskExecutor")
    public Executor redisMessageTaskExecutor() {
        ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
        threadPoolTaskExecutor.setCorePoolSize(2); 
        threadPoolTaskExecutor.setMaxPoolSize(4);
        return threadPoolTaskExecutor;
    }

    @Bean
    public RedisMessageListenerContainer redisMessageListenerContainer(
               RedisConnectionFactory redisConnectionFactory, RedisMessageListener listener) {
        var container = new RedisMessageListenerContainer();
        container.setConnectionFactory(redisConnectionFactory);
        container.addMessageListener(listener, new PatternTopic(SET_EVENT_PATTERN));
        container.addMessageListener(listener, new PatternTopic(EXPIRED_EVENT_PATTERN));
        container.addMessageListener(listener, new PatternTopic(EXPIRED_SPACE_PATTERN));
        container.addMessageListener(messageListenerAdapter(), new ChannelTopic(PUBLISH_KEY));
        container.setTaskExecutor(asyncThreadTaskExecutor());
        return container;
    }
}
```

```java
import java.util.List;
import org.springframework.data.redis.connection.Message;
import org.springframework.data.redis.connection.MessageListener;

import lombok.extern.slf4j.Slf4j;

@Component
@RequiredArgsConstructor
@Slf4j
public class RedisMessageListener implements MessageListener {

    @Override
    public void onMessage(final Message message, final byte[] pattern) {
        log.info("## redis message pattern: " + new String(pattern));
        log.info("## redis message : " + message.toString());
    }
}
```
