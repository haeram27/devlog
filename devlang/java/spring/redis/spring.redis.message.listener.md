# spring cache

spring은 외부 캐시 솔루션(redis 등)과 편리하게 연동하기 위해서 Cache 관련 어노테이션을 제공함

## 주요 캐시 어노테이션 설명

```java
import java.util.List;

import org.springframework.data.redis.connection.MessageListener;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import com.example.MyData;
import com.example.service.redis.RedisService;

@Slf4j
@RequiredArgsConstructor
public class RedisMessageListener implements MessageListener {

    private final MyRedisService redisService;
    private final MyData data;

    public RedisMessageListener(MyData myData, MyRedisService redisService) {
        refresh();  // refresh data from cache when MessageListener is created
    }

    @Override
    public void onMessage(Message message, byte[] pattern) {
        log.info("## Updated SYSLOG configuration.");

        refresh();  // refresh data from cache when receive message from redis 
    }
    private void refresh() {
        redisService.evictCacheMyData(); // remove data from cache
        List<MyData> dataList = redisService.getMyDataList(); // read data from cache
    }
}
```

### Redis MessageListener 등록

- `MessageListener`가 `Redis`로 부터 이벤트를 전달 받을 수 있도록 `RedisConnectionFactory`에 등록
- 이벤트 주체인 `Cache Key`를 반드시 지정해 주어야 함

#### RedisMessageListenerConfig.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.listener.ChannelTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;

import lombok.RequiredArgsConstructor;

import com.example.MyData;
import com.example.service.redis.RedisService;

@Configuration
@RequiredArgsConstructor
public class RedisMessageListenerConfig {
    private static final String CACHE_KEY_MY_DATA = "mydata-cache-key";

    private final MyData myData;
    private final RedisService redisService;

    @Bean
    public RedisMessageListenerContainer redisListener(
            RedisConnectionFactory redisConnectionFactory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(redisConnectionFactory);
        container.addMessageListener(messageListenerAdapter(), new ChannelTopic(CACHE_KEY_MY_DATA));

        return container;
    }

    @Bean
    public MessageListenerAdapter messageListenerAdapter() {
        MessageListenerAdapter adapter = new MessageListenerAdapter();
        adapter.setDelegate(new RedisMessageListener(myData, redisService));

        return adapter;
    }
}
```
