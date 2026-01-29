# RestTemplate: spring에서 http client 사용하기

RestTemplate 은 블로킹 방식의 http request를 사용하기 위하여 설계되었다.
블로킹 방식은 request 당 1 thread, 1 socket을 사용한다.

Async 방식의 http client pool을 사용하려면 WebClient를 사용하라.
Async 방식의 request 처리는 많은 requets에서도 더 적은 thread와 socket을 사용할 수 있지만, 응답 처리를 위한 listener 구조를 별도로 구현해야 한다.

## trustAllRestTemplate Bean Exmaple (apache httpclient 5.3.1)

for org.apache.httpcomponents.client5:httpclient5:jar:5.3.1

```java
import java.security.GeneralSecurityException;
import java.time.Duration;
import org.apache.hc.client5.http.impl.classic.HttpClients;
import org.apache.hc.client5.http.impl.io.PoolingHttpClientConnectionManagerBuilder;
import org.apache.hc.client5.http.ssl.NoopHostnameVerifier;
import org.apache.hc.client5.http.ssl.SSLConnectionSocketFactoryBuilder;
import org.apache.hc.client5.http.ssl.TrustAllStrategy;
import org.apache.hc.core5.http.io.SocketConfig;
import org.apache.hc.core5.ssl.SSLContexts;
import org.apache.hc.core5.util.Timeout;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

@Configuration
public class HttpClientConfig {

    @Bean
    public RestTemplate trustAllRestTemplate() {
        try {
            var sslContext = SSLContexts.custom()
                .loadTrustMaterial(null, TrustAllStrategy.INSTANCE)
                .build();

            var sslSocketFactory = SSLConnectionSocketFactoryBuilder.create()
            .setSslContext(sslContext)
            .setHostnameVerifier(NoopHostnameVerifier.INSTANCE)
            .build();

            // var cm = new BasicHttpClientConnectionManager(socketFactoryRegistry);
            var cm = PoolingHttpClientConnectionManagerBuilder.create()
                .setSSLSocketFactory(sslSocketFactory)
                // read timeout
                .setDefaultSocketConfig(SocketConfig.custom().setSoTimeout(Timeout.ofSeconds(5)).build())
                .build();

            var httpClient = HttpClients.custom()
                .setConnectionManager(cm)
                .evictExpiredConnections()
                .build();

            var rf = new HttpComponentsClientHttpRequestFactory(httpClient);
            rf.setConnectionRequestTimeout(Duration.ofSeconds(1)); // time to resolve idle http connection from conn pool
            rf.setConnectTimeout(Duration.ofSeconds(2)); // time to establish tcp connection

            return new RestTemplate(rf);
        } catch (GeneralSecurityException e) {
            throw new org.springframework.beans.factory.BeanCreationException(
                "trustAllRestTemplate", "Failed to build SSL trust-all RestTemplate", e);
        }
    }
}
```

## trustAllRestTemplate Bean Exmaple httpclient5 5.5+

for org.apache.httpcomponents.client5:httpclient5:jar:5.5+

* RestTemplate is designed for blocking method if async request method is needed use `org.springframework.web.reactive.function.client.WebClient`
* HttpComponentsClientHttpRequestFactory designed by spring scope sets timeout settings per request of httpclient if default request config for HttpClient then use `RequestConfig`

```java
@Bean
@Primary
public RestTemplate trustAllRestTemplate() {
    try {
        var sslContext = SSLContexts.custom()
            .loadTrustMaterial(null, TrustAllStrategy.INSTANCE)
            .build();

        var tls = ClientTlsStrategyBuilder.create()
            .setSslContext(sslContext)
            .setHostnameVerifier(NoopHostnameVerifier.INSTANCE)
            .buildClassic();

        // TCP(Socket) level Connection Config
        var cc = ConnectionConfig.custom()
            .setConnectTimeout(Timeout.of(Duration.ofSeconds(2)))  // timeout while waiting for TCP connection establishment
            .setSocketTimeout(Timeout.of(Duration.ofSeconds(5)))   // inactivity timeout of inter-packet
            .build();

        // use connection pool
        var cm = PoolingHttpClientConnectionManagerBuilder.create()
            .setTlsSocketStrategy(tls)
            .setDefaultConnectionConfig(cc)
            .build();

        // use BasicHttpClientConnectionManager for one-time connection
        /*
            var cm = BasicHttpClientConnectionManager.create()
                .setTlsSocketStrategy(tls)   
                .build();
        */

        // Application(HTTP) level Connection Config
        var rc = RequestConfig.custom()
            .setConnectionRequestTimeout(Timeout.ofSeconds(1)) // timeout while waiting for lent a connection from connection pool
            .setResponseTimeout(Timeout.ofSeconds(5))          // timeout while waiting for server response
            .build();

        var httpClient = HttpClients.custom()
            .setConnectionManager(cm)
            .setDefaultRequestConfig(rc)
            .evictExpiredConnections() // Find, close and remove connections that are stored in the connection pool that are past the server-set validity (TTL)
            .evictIdleConnections(TimeValue.ofSeconds(10)) // evict(remove) connection has no sending request for more than 10 sec
            .build();

        /* 
            * set timeout setting per request in RestTemplate (Spring driven),
            * if timeouts set here then timeout config is not require at ConnectionConfig and RequestConfig
            */
        var rf = new HttpComponentsClientHttpRequestFactory(httpClient);
        rf.setConnectionRequestTimeout(Duration.ofSeconds(1)); // time to resolve idle http connection from conn pool
        rf.setConnectTimeout(Duration.ofSeconds(2));           // time to establish tcp connection
        rf.setReadTimeout(Duration.ofSeconds(5));              // time to wait response against http request from http server

        var template = new RestTemplate(rf);

        /*
            * RestCient can be made by RestTmeplate
            */
        // RestClient client = RestClient.create(template);

        return template;
    } catch (GeneralSecurityException e) {
        throw new org.springframework.beans.factory.BeanCreationException(
            "trustAllRestTemplate", "Failed to build SSL trust-all RestTemplate", e);
    }
}
```

### Example: How to use RestTemplate

#### Controller

```java
import java.util.Optional;
import org.apache.commons.lang3.StringUtils;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestAttribute;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@RestController
@RequiredArgsConstructor
@RequestMapping(value = "/api/v1",
        consumes = MediaType.APPLICATION_JSON_VALUE,
        produces = MediaType.APPLICATION_JSON_VALUE)
public class RestTemplateTestController {

    private final RestTemplateTestService service;

    @PostMapping("/rest/invoke")
    public String searchCve(@RequestBody Map<String, Object> requestJson) {
        log.info("## Controller::body : {}", requestJson);

        var paramMap = requestJson.getParamMap();
        var key = Optional.ofNullable(paramMap.get("param"))
                            .filter(String.class::isInstance)
                            .map(String.class::cast)
                            .orElse("");
        log.debug("requested param: {}", param);
        if (StringUtils.isEmpty(param)) {
            log.error("Error: {}", "invalid param");
        }

        return service.invoke(param);
    }
}
```

#### Service

```java
import java.net.URI;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

import org.apache.logging.log4j.util.Strings;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.client.ResourceAccessException;
import org.springframework.web.client.RestClientResponseException;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriComponentsBuilder;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.json.JsonMapper;
import com.fasterxml.jackson.databind.node.ArrayNode;
import com.fasterxml.jackson.databind.node.ObjectNode;

import lombok.extern.slf4j.Slf4j;
/*
 * RestTemplate GET Test
 */

@Slf4j
@SpringBootTest
public class RestTemplateSyncTests {

    @Autowired
    @Qualifier("trustAllRestTemplate")
    private RestTemplate restTemplate;

    @Autowired
    @Qualifier("restClientObjectMapper")
    private JsonMapper mapper;

    // https://jsonplaceholder.typicode.com/todos
    private static final String URL_DOMAIN_PORTAL = "jsonplaceholder.typicode.com";
    private static final String URL_API_PATH = "/todos";
    private static final String TEST_BEARER_AUTH_TOKEN = "";

    public List<Map<String, Object>> send(HttpMethod method, URI uri, HttpEntity<Void> httpEntity) {
        ResponseEntity<JsonNode> responseEntity;
        List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();;

        if (Strings.isEmpty(method.name())) {
            log.error("Error: No content");
            return list;
        }

        try {
            log.debug("## request uri : {}", uri);

            responseEntity = restTemplate.exchange(uri, method, httpEntity, JsonNode.class);

            log.debug("Response Body: " + responseEntity.getBody());
            log.debug("Status Code: " + responseEntity.getStatusCode());
            log.debug("Headers: " + responseEntity.getHeaders());
            log.debug("Response http code: {}, reason: {}",
                    responseEntity.getStatusCode().value(),
                    HttpStatus.valueOf(responseEntity.getStatusCode().value()).getReasonPhrase());

            if (responseEntity.getStatusCode().equals(HttpStatus.OK)) {
                var responseBody = responseEntity.getBody();

                if (responseBody == null || responseBody.isEmpty()) {
                    log.error("Error: Empty Body");
                }

                if (responseBody.isObject()) {
                    ObjectNode node = (ObjectNode) responseBody;
                    Map<String, Object> map = mapper.convertValue(node, new com.fasterxml.jackson.core.type.TypeReference<Map<String,Object>>() {});
                    // var map = mapper.treeToValue(node, Map.class);
                    list.add(map);
                } else if (responseBody.isArray()) {
                    ArrayNode node = (ArrayNode) responseBody;
                    list = mapper.convertValue(node, new com.fasterxml.jackson.core.type.TypeReference<List<Map<String,Object>>>() {});
                } else {
                    log.error("Error: Response content type is NOT applicable");
                }
            } else if (responseEntity.getStatusCode().equals(HttpStatus.NO_CONTENT)) {
                log.error("Error: No content");
            } else {
                log.error("Error: Http Server Internal Error");
            }
        } catch (RestClientResponseException ex) {
            log.error("Error: {}", ex.getMessage());

            var status = ex.getStatusCode();
            log.debug("Status Code: " + status);
            // log.debug("code value: " + status.value());
            // log.debug("status text: " + ex.getStatusText());
            log.debug("Response Body: " + ex.getResponseBodyAsString());
            log.debug("Headers: " + ex.getResponseHeaders());

            if (status.is4xxClientError()) {
                log.error("Error: {}", ex.getResponseBodyAsString());

                // if required handle with exact 4XX Error
                if (status.equals(HttpStatus.BAD_REQUEST)) {
                    log.error("Error: BAD_REQUEST: {}", ex.getResponseBodyAsString());
                } else if (status.equals(HttpStatus.UNAUTHORIZED)) {
                    log.error("Error: UNAUTHORIZED: {}", ex.getResponseBodyAsString());
                } else if (status.equals(HttpStatus.FORBIDDEN)) {
                    log.error("Error: FORBIDDEN: {}", ex.getResponseBodyAsString());
                } else if (status.equals(HttpStatus.NOT_FOUND)) {
                    log.error("Error: NOT_FOUND: {}", ex.getResponseBodyAsString());
                } else if (status.equals(HttpStatus.I_AM_A_TEAPOT)) {
                    log.error("Error: I_AM_A_TEAPOT: {}", ex.getResponseBodyAsString());
                } else if (status.equals(HttpStatus.TOO_MANY_REQUESTS)) {
                    log.error("Error: TOO_MANY_REQUESTS: {}", ex.getResponseBodyAsString());
                }
            } else if (status.is5xxServerError()) {
                log.error("Error: {}", ex.getResponseBodyAsString());
            } else {
                log.error("Error: {}", ex.getResponseBodyAsString());
            }
        } catch (ResourceAccessException ex) {
            log.error("Error: {}", ex.getMessage());
        } catch (Exception ex) {
            log.error("Error: {}", ex.getMessage());
        }

        return list;
    }

    @Test
    public void getTest() {
        var uri = UriComponentsBuilder.newInstance()
                    .scheme("https")
                    .host(URL_DOMAIN_PORTAL)
                    .path(URL_API_PATH)
                    //.queryParam("key", "value")
                    .encode().build().toUri();

        var headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);

        if (Strings.isNotEmpty(TEST_BEARER_AUTH_TOKEN))
            headers.setBearerAuth(TEST_BEARER_AUTH_TOKEN);

        var httpEntity = new HttpEntity<Void>(headers);
        var responseBody = send(HttpMethod.GET, uri, httpEntity);

        if (responseBody != null && responseBody.size() > 0) {
            try {
                log.debug("## response: {}",
                        mapper.writerWithDefaultPrettyPrinter().writeValueAsString(responseBody));
            } catch (JsonProcessingException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
                log.error("## invalid response");
            }
        } else {
            log.error("## empty response");
        }
    }
}
```

## WebClient (Async mechanism HttpClient for Spring)

- `WebClient` is Async HttpClient Wrapper, in case of `Apache httpclient5`, Async HttpClient is `CloseableHttpAsyncClient`
- `spring-boot-starter-webflux` package is required to use `WebClient`

```gradle
    implementation 'org.springframework.boot:spring-boot-starter-webflux'
```

```java
import java.security.GeneralSecurityException;
import java.time.Duration;

import org.apache.hc.client5.http.config.RequestConfig;
import org.apache.hc.client5.http.impl.async.CloseableHttpAsyncClient;
import org.apache.hc.client5.http.impl.async.HttpAsyncClients;
import org.apache.hc.client5.http.impl.async.PoolingAsyncClientConnectionManager;
import org.apache.hc.client5.http.impl.async.PoolingAsyncClientConnectionManagerBuilder;
import org.apache.hc.client5.http.ssl.ClientTlsStrategyBuilder;
import org.apache.hc.core5.ssl.SSLContexts;
import org.apache.hc.core5.util.Timeout;
import org.apache.hc.core5.http.nio.ssl.TlsStrategy;
import org.apache.hc.client5.http.ssl.NoopHostnameVerifier;
import org.apache.hc.client5.http.ssl.TrustAllStrategy;

import org.springframework.http.client.reactive.HttpComponentsClientHttpConnector;
import org.springframework.web.reactive.function.client.WebClient;

public WebClient trustAllWebClient() {
    try {
        var sslContext = SSLContexts.custom()
        .loadTrustMaterial(null, TrustAllStrategy.INSTANCE)
        .build();

        // TlsStrategy: async client method
        var tls = ClientTlsStrategyBuilder.create()
            .setSslContext(sslContext)
            .setHostnameVerifier(NoopHostnameVerifier.INSTANCE)
            .buildAsync();

        // TCP(Socket) level Connection Config
        var cc = ConnectionConfig.custom()
                .setConnectTimeout(Timeout.of(Duration.ofSeconds(2)))  // timeout while waiting for TCP connection establishment
                .setSocketTimeout(Timeout.of(Duration.ofSeconds(5)))  // inactivity timeout of inter-packet
                .build();

        // connection pool: async client method
        var cm = PoolingAsyncClientConnectionManagerBuilder.create()
                .setTlsStrategy(tls)
                .setDefaultConnectionConfig(cc)
                // modify to enhance performance, mainly Connection Per Route
                .setMaxConnTotal(25)    // default: 25
                .setMaxConnPerRoute(5)  // default: 5
                .build();

        // Application(HTTP) level Connection Config
        var rc = RequestConfig.custom()
                .setConnectionRequestTimeout(Timeout.ofSeconds(1)) // timeout while waiting for lent a connection from connection pool
                .setResponseTimeout(Timeout.ofSeconds(5))          // timeout while waiting for server response
                .build();

        // build CloseableHttpAsyncClient
        var asyncClient = HttpAsyncClients.custom()
            .setConnectionManager(cm)
            .setDefaultRequestConfig(rc)
            .evictExpiredConnections() // Find, close and remove connections that are stored in the connection pool that are past the server-set validity (TTL)
            .evictIdleConnections(TimeValue.ofSeconds(10)) // evict(remove) connection has no sending request for more than 10 sec
            .build();

        // assign WebClient to HttpConnector (make Spring control connection lifecycle)
        var connector = new HttpComponentsClientHttpConnector(asyncClient);

    return WebClient.builder()
        .clientConnector(connector)
        .build();

    } catch (GeneralSecurityException e) {
        throw new org.springframework.beans.factory.BeanCreationException(
            "trustAllWebClient", "Failed to build SSL trust-all WebClient", e);
    }
}
```
