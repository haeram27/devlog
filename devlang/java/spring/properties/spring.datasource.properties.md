# datasource

## DataSource application.yml

```yml
application:
  datasource:
    first-db:
      entity-packages:
        - com.example.datasource.repository.domain
        - com.example.datasource.repository.domain.entity
      url: jdbc:postgresql://1.1.1.1:1111/first
      username: user
      password: passwd
    second-db:
      entity-packages: com.example.datasource.repository.domain
      url: jdbc:postgresql://1.1.1.1:1111/second
      username: user
      password: passwd
```

## DataSoruce Configuration

```java

import java.util.List;
import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.jpa.autoconfigure.JpaProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import jakarta.persistence.EntityManagerFactory;

@Configuration
@EnableTransactionManagement
@EnableConfigurationProperties({
    RdbDataSourceConfig.FirstDbProperties.class,
    RdbDataSourceConfig.SecondDbProperties.class
})
public class RdbDataSourceConfig {

    @Primary
    @Bean(name = "firstDataSource")
    public DataSource firstDataSource(FirstDbProperties properties) {
        return createDataSource(properties);
    }

    @Bean(name = "dataSource")
    public DataSource dataSource(@Qualifier("firstDataSource") DataSource firstDataSource) {
        return firstDataSource;
    }

    @Bean(name = "secondDataSource")
    public DataSource secondDataSource(SecondDbProperties properties) {
        return createDataSource(properties);
    }

    @Primary
    @Bean(name = "firstEntityManagerFactory")
    public LocalContainerEntityManagerFactoryBean firstEntityManagerFactory(
        @Qualifier("firstDataSource") DataSource firstDataSource,
        FirstDbProperties properties,
        JpaProperties jpaProperties
    ) {
        return createEntityManagerFactory(
            firstDataSource,
            properties.entityPackages(),
            "firstDb",
            jpaProperties
        );
    }

    @Bean(name = "secondEntityManagerFactory")
    public LocalContainerEntityManagerFactoryBean secondEntityManagerFactory(
        @Qualifier("secondDataSource") DataSource secondDataSource,
        SecondDbProperties properties,
        JpaProperties jpaProperties
    ) {
        return createEntityManagerFactory(
            secondDataSource,
            properties.entityPackages(),
            "secondDb",
            jpaProperties
        );
    }

    @Bean(name = "entityManagerFactory")
    public EntityManagerFactory entityManagerFactory(
        @Qualifier("firstEntityManagerFactory") EntityManagerFactory firstEntityManagerFactory
    ) {
        return firstEntityManagerFactory;
    }

    @Primary
    @Bean(name = "firstTransactionManager")
    public PlatformTransactionManager firstTransactionManager(
        @Qualifier("firstEntityManagerFactory") EntityManagerFactory entityManagerFactory
    ) {
        return new JpaTransactionManager(entityManagerFactory);
    }

    @Bean(name = "secondTransactionManager")
    public PlatformTransactionManager secondTransactionManager(
        @Qualifier("secondEntityManagerFactory") EntityManagerFactory entityManagerFactory
    ) {
        return new JpaTransactionManager(entityManagerFactory);
    }

    @Bean(name = "transactionManager")
    public PlatformTransactionManager transactionManager(
        @Qualifier("firstTransactionManager") PlatformTransactionManager firstTransactionManager
    ) {
        return firstTransactionManager;
    }

    private DataSource createDataSource(BaseDbProperties properties) {
        var builder = DataSourceBuilder.create()
            .url(properties.url())
            .username(properties.username())
            .password(properties.password());

        if (properties.driverClassName() != null && !properties.driverClassName().isBlank()) {
            builder.driverClassName(properties.driverClassName());
        }

        return builder.build();
    }

    private LocalContainerEntityManagerFactoryBean createEntityManagerFactory(
        DataSource dataSource,
        List<String> entityPackages,
        String persistenceUnitName,
        JpaProperties jpaProperties
    ) {
        String[] packagesToScan = entityPackages == null ? new String[0] : entityPackages.toArray(String[]::new);

        var entityManagerFactory = new LocalContainerEntityManagerFactoryBean();
        entityManagerFactory.setDataSource(dataSource);
        entityManagerFactory.setPackagesToScan(packagesToScan);
        entityManagerFactory.setPersistenceUnitName(persistenceUnitName);
        entityManagerFactory.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        entityManagerFactory.setJpaPropertyMap(jpaProperties.getProperties());
        return entityManagerFactory;
    }

    private interface BaseDbProperties {
        String url();
        String username();
        String password();
        String driverClassName();
        List<String> entityPackages();
    }

    @ConfigurationProperties(prefix = "application.datasource.first-db")
    public record FirstDbProperties(
        List<String> entityPackages,
        String url,
        String username,
        String password,
        String driverClassName
    ) implements BaseDbProperties {}

    @ConfigurationProperties(prefix = "application.datasource.second-db")
    public record SecondDbProperties(
        List<String> entityPackages,
        String url,
        String username,
        String password,
        String driverClassName
    ) implements BaseDbProperties {}
}

```

## test

```java

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assumptions.assumeTrue;

import java.util.List;
import java.util.Map;

import javax.sql.DataSource;

import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.condition.EnabledIfEnvironmentVariable;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;

import com.example.server.SpringApplication;

/**
 * Required env:
 * - RUN_RDB_GEOIP_IT=true
 */
@EnabledIfEnvironmentVariable(named = "RUN_RDB_GEOIP_IT", matches = "(?i)true")
@SpringBootTest(
    classes = SpringApplication.class,
    webEnvironment = SpringBootTest.WebEnvironment.NONE
)
class RdbGeoIpCityIntegrationTest {

    private static final Logger log = LoggerFactory.getLogger(RdbGeoIpCityIntegrationTest.class);
    private static final String TEST_ENABLE_KEY = "RUN_RDB_GEOIP_IT";
    private static final String QUERY_GEOIP_CITY_TOP10 = "SELECT * FROM tb_geoip_city LIMIT 10";

    private final DataSource firstDataSource;

    RdbGeoIpCityIntegrationTest(
        @Qualifier("firstDataSource") DataSource firstDataSource
    ) {
        this.firstDataSource = firstDataSource;
    }

    @BeforeAll
    static void guardExecution() {
        boolean enabled = "true".equalsIgnoreCase(System.getenv().getOrDefault(TEST_ENABLE_KEY, "false"));
        assumeTrue(enabled, () -> "Set " + TEST_ENABLE_KEY + "=true to run this integration test");
    }

    @Test
    void shouldPrintTop10TbGeoipCityRowsFromAdminDb() {
        JdbcTemplate firstJdbc = new JdbcTemplate(firstDataSource);
        List<Map<String, Object>> adminRows = firstJdbc.queryForList(QUERY_GEOIP_CITY_TOP10);

        assertTrue(adminRows.size() <= 10, "admin-db query result size must be <= 10");

        logRows("admin-db", adminRows);
    }

    private void logRows(String dbName, List<Map<String, Object>> rows) {
        String header = "tb_geoip_city top rows from %s (count=%d):".formatted(dbName, rows.size());
        log.info(header);
        for (int i = 0; i < rows.size(); i++) {
            String line = "[%s][%d] %s".formatted(dbName, i + 1, rows.get(i));
            log.info(line);
        }
    }
}
```