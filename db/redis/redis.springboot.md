# java redis client library

lettuce: async client  
jedis: sync client

[redist data types](https://redis.io/docs/latest/develop/data-types)

## configuration

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {
  private final CacheDataSourceManager cacheDataSourceManager = CacheDataSourceManager.getInstance();

  /**
   - @return
   */
  @Primary
  @Bean
  public RedisConnectionFactory redisConnectionFactory() {
    return cacheDataSourceManager.getLettuceConnectionFactory();
  }

  @Bean(name = "redisConnectionFactoryFromDatabase")
  public RedisConnectionFactory redisConnectionFactoryFromDatabase() {
    return cacheDataSourceManager.getLettuceConnectionFactoryFromDatabase();
  }

  @Bean
  public RedisTemplate<String, Object> redisTemplate() {
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(this.redisConnectionFactory());
    redisTemplate.setEnableTransactionSupport(true);
    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(String.class));

    return redisTemplate;
  }

  @Primary
  @Bean
  public StringRedisTemplate stringRedisTemplate() {
    StringRedisTemplate stringRedisTemplate = new StringRedisTemplate(this.redisConnectionFactory());
    stringRedisTemplate.setEnableTransactionSupport(Boolean.TRUE);

    return stringRedisTemplate;
  }

  @Bean(name = "stringRedisTemplateDatabase")
  public StringRedisTemplate stringRedisTemplateDatabase() {
    StringRedisTemplate stringRedisTemplateDatabase = new StringRedisTemplate(redisConnectionFactoryFromDatabase());
    stringRedisTemplateDatabase.setEnableTransactionSupport(Boolean.TRUE);

    return stringRedisTemplateDatabase;
  }

  @Bean
  public RedisMessageListenerContainer redisListener() {
    RedisMessageListenerContainer container = new RedisMessageListenerContainer();
    container.setConnectionFactory(this.redisConnectionFactory());

    return container;
  }
}
```
