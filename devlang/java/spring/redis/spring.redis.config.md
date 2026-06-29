# Spring Redis Configuration

## example

Spring Redis 3.x 기준

```java
import org.springframework.boot.cache.autoconfigure.RedisCacheManagerBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import com.example.com.CacheDataSourceManager;

@Configuration
public class RedisConfig {
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return CacheDataSourceManager.getInstance().getLettuceConnectionFactoryFromDatabase();
    }

    // java serialization: save java object as JavaSerialize type value
    @Primary
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTetemplatemplate = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory());
        template.setEnableTransactionSupport(true);

        return template;
    }

    // json serialization: save java object as json type value
    @Bean(name = "jsonRedisTemplate")
    public RedisTemplate<String, Object> jsonRedisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(this.redisConnectionFactory());
        template.setEnableTransactionSupport(true);

        StringRedisSerializer stringSerializer = new StringRedisSerializer();
        GenericJackson2JsonRedisSerializer jsonSerializer = new GenericJackson2JsonRedisSerializer();

        template.setKeySerializer(stringSerializer);
        template.setValueSerializer(jsonSerializer);
        template.setHashKeySerializer(stringSerializer);
        template.setHashValueSerializer(jsonSerializer);

        return template;
    }

    // json serialization: save java object as json type value
    @Bean
    public StringRedisTemplate stringRedisTemplate() {
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate(redisConnectionFactory());
        stringRedisTemplate.setEnableTransactionSupport(true);

        return stringRedisTemplate;
    }

    @Bean
    RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
        return builder -> builder.cacheDefaults(
            RedisCacheConfiguration.defaultCacheConfig()
                .computePrefixWith(cacheName -> cacheName + ":")
        );
    }
}
```

### 주의 사항

- type이 같은 Bean을 등록한 경우 (예: `RedisTemplate<String, Object>`의 redistTemplate, jsonRedisTemplate)
  - `@Primary` 사용하여 해당 Type의 기본 Bean을 설정한다. Spring은 중복된 Type의 Bean이 존재하는 경우 주입에 선택될 Bean을 자체적으로 판단하지 않으므로, 충돌 Error를 출력하고 실행을 종료한다.
  - `lombok`은 생성자를 생성하는 annotation(예: `@RequiredArgsConstructor` 등)에서 기본적으로 annotation을 복사하지 않는다. 필요한 경우 properties에 lombok 설정을 별도로 추가하여 복사할 annotion을 지정할 수 있으나 관리가 번거롭기 때문에, 의존성 주입이 필요한 필드에 `@Qualifier`등의 annotation 사용이 필요한 경우 `lombok` 의 생성자 annotation을 사용하지 않고 직접 생성자를 구현하여 필드에 `@Qualifier`를 명시하도록 한다.
