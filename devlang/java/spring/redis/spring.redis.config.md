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

    // java serialization: save java object as JavaSerialize
    @Primary
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTetemplatemplate = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory());
        template.setEnableTransactionSupport(true);

        return template;
    }

    // json serialization: save java object as json
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