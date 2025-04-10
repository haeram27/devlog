# javax.sql.DataSource

database url, id, password와 기타 접속 관련 메타 정보를 포함하는 클래스
datasource는 RDB와 연결하기 위한 정보를 가진 설정 클래스이며 기본 JDBC이외에도 connetion-pool(hikari, DBCP, DBCP2 등) 라이브러리들이 DataSource 클래스 인스턴스를 생성한다.

# hikariCP datasource 생성
<https://escapefromcoding.tistory.com/712>

## 방법1. application.properties, application.yml에서 설정하기

"spring.datasource.hikari" 접두사를 이용해서 속성 설정

```yaml
---
spring.datasource.hikari.connectionTimeout=30000
spring.datasource.hikari.idleTimeout=600000
spring.datasource.hikari.maxLifetime=1800000
```

## 방법2. hikari 전용 별도의 properties 파일(resource/hikari.properties)을 이용하여 생성

### hikari.properties

```
driverClassName=com.mysql.jdbc.Driver
jdbcUrl=jdbc:mysql://localhost:3306/testdb?useSSL=false
maximumPoolSize=20
username=testuser
password=test623
dataSource.cachePrepStmts=true
dataSource.prepStmtCacheSize=250
dataSource.prepStmtCacheSqlLimit=2048
```

properties를 가져와서 설정할 수도 있습니다.

```
@Configuration
public class HikariCPConfig {
    @Bean(destroyMethod = "close")
    public DataSource dataSource() throws SQLException {
        HikariConfig config = new HikariConfig("/hikari.properties");
        HikariDataSource dataSource = new HikariDataSource(config);

        return dataSource;
    }
}
```

## 방법3. Configuration Bean을 이용한 설정

```
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

@Bean(destroyMethod = "close")
public DataSource dataSource(){    
    HikariConfig hikariConfig = new HikariConfig();    
    hikariConfig.setDriverClassName("oracle.jdbc.driver.OracleDriver");    
    hikariConfig.setJdbcUrl("jdbc:oracle:thin:@ip:port:SID");     
    hikariConfig.setUsername("id");    
    hikariConfig.setPassword("password");     
    //hikariConfig.setMaximumPoolSize(5);    
    //hikariConfig.setConnectionTestQuery("SELECT 1");    
    //hikariConfig.setPoolName("springHikariCP");     
    HikariDataSource dataSource = new HikariDataSource(hikariConfig);     
    return dataSource;
}
```

# 구조

SqlSessionFactoryBean > DataSource(Hikari

spring 2.x 부터 HikariCP는 spring-boot의 기본 Connection Pool로 설정되어 있음
별다른 hikaricp 설정 없어도 hikaricp는 RDB 연결시 사용되고 이때 초기 설정은 default 값으로 자동 생성됨

단, RDB와 연결하려면 최소한의 jdbc를 위한 기본 datasource 설정은 반드시 필요함

```
spring.datasource.driver-class-name: org.postgresql.Driver
spring.datasource.url: jdbc:postgresql://localhost:5432/dvdrental
spring.datasource.username: postgres
spring.datasource.password: postgres
```

# HikariConfig:: default values

HikariCP의 로그를 활성화하면 최초 DB 연결 동작시 초기화 로그를 볼 수 있다.

```
HikariPool-1 - configuration:
allowPoolSuspension.............false
autoCommit......................true
catalog.........................none
connectionInitSql...............none
connectionTestQuery.............none
connectionTimeout...............30000
dataSource......................none
dataSourceClassName.............none
dataSourceJNDI..................none
dataSourceProperties............{password=<masked>}
driverClassName................."org.postgresql.Driver"
exceptionOverrideClassName......none
healthCheckProperties...........{}
healthCheckRegistry.............none
idleTimeout.....................600000
initializationFailTimeout.......1
isolateInternalQueries..........false
jdbcUrl.........................jdbc:postgresql://localhost:5432/dvdrental
keepaliveTime...................0
leakDetectionThreshold..........0
maxLifetime.....................1800000
maximumPoolSize.................10
metricRegistry..................none
metricsTrackerFactory...........none
minimumIdle.....................10
password........................<masked>
poolName........................"HikariPool-1"
readOnly........................false
registerMbeans..................false
scheduledExecutor...............none
schema..........................none
threadFactory...................internal
transactionIsolation............default
username........................"postgres"
validationTimeout...............5000
```

# HikariConfig 초기화 로그 보기

application.yml 이나 환경변수에 logging.level.{package-path} 설정을 하면 해당 패키지 또는 클래스의 로그 레벨 설정이 가능한다.

HikariConfig 클래스의 로그 레벨을 DEBUG 이상으로 설정하면 최초 DB 연결시 HikariConfig의 설정값을 확인할 수 있다.

## com.zaxxer.hikari package 이하에 포함되는 모든 클래스 로깅 레벨 설정

위치: resource/application.yaml
설정: logging.level.com.zaxxer.hikari: TRACE

## com.zaxxer.hikari.HikariConfig 클래스 로깅 레벨 설정

위치: resource/application.yaml
설정: logging.level.com.zaxxer.hikari.HikariConfig: TRACE

## 실행시 환경변수로 설정(대소문자 구분X)

$ logging_level_com_zaxxer_hikari_hikariconfig=debug gradle bootRun

# HikariCP 추천 설정
<https://netmarble.engineering/hikaricp-options-optimization-for-game-server/>
autoCommit......................true
maximumPoolSize == minimumIdle
maximumPoolSize=<CPU 코어수>로 테스트 시작하여 하기 기준에 맞도록 설정
> 최종 기준
>> DB의 리소스 사용량 임계치: CPU 60%
> 풀 크기 산정 방법
>> HikariCP의 IdleConnections 수치가 크다면 풀 크기를 축소
>> HikariCP의 PendingConnections 수치가 크다면 풀 크기를 증가
>> DB의 리소스 사용률이 임계치보다 크다면 풀 크기를 축소 또는 DB에 연결되는 게임 서버의 수를 축소

connectionTestQuery=none
validationTimeout=500ms
connectionTimeout=2000ms
maxLifetime=0ms           (Infinite Lifetime)
idleTimeout=0ms           (Infinite Lifetime)
keepaliveTime=30000ms
StatementTimeout=1000ms
ConnectionTimeout=3000ms
ConnectionTimeout=13000ms (DNS 사용시)
