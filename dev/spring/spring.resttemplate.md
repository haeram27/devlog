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
    import java.security.GeneralSecurityException;
    import java.time.Duration;
    import org.apache.hc.client5.http.impl.classic.HttpClients;
    import org.apache.hc.client5.http.impl.io.PoolingHttpClientConnectionManagerBuilder;
    import org.apache.hc.client5.http.ssl.ClientTlsStrategyBuilder;
    import org.apache.hc.client5.http.ssl.NoopHostnameVerifier;
    import org.apache.hc.client5.http.ssl.TrustAllStrategy;
    import org.apache.hc.core5.ssl.SSLContexts;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.ComponentScan;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
    import org.springframework.web.client.RestTemplate;

@Configuration
public class HttpClientConfig {

    // register RestTemplate as Spring Bean to be singleton
    @Bean
    public RestTemplate trustAllRestTemplate() {
        try {
            var sslContext = SSLContexts.custom()
                .loadTrustMaterial(null, TrustAllStrategy.INSTANCE)
                .build();

            var tls = ClientTlsStrategyBuilder.create()
                .setSslContext(sslContext)
                .setHostnameVerifier(NoopHostnameVerifier.INSTANCE)
                .buildClassic();

            // use connection pool
            var cm = PoolingHttpClientConnectionManagerBuilder.create()
                .setTlsSocketStrategy(tls)
                .build();

            // use BasicHttpClientConnectionManager for one-time connection
            /*
                var cm = BasicHttpClientConnectionManager.create()
                    .setTlsSocketStrategy(tls)   
                    .build();
            */

            var httpClient = HttpClients.custom()
                .setConnectionManager(cm)
                .evictExpiredConnections()
                .build();

            // set timeout setting per request in RestTemplate (Spring driven)
            var rf = new HttpComponentsClientHttpRequestFactory(httpClient);
            rf.setConnectionRequestTimeout(Duration.ofSeconds(1)); // time to resolve idle http connection from conn pool
            rf.setConnectTimeout(Duration.ofSeconds(2)); // time to establish tcp connection
            rf.setReadTimeout(Duration.ofSeconds(5)); // time to wait response against http request from http server

            return new RestTemplate(rf);
        } catch (GeneralSecurityException e) {
            throw new org.springframework.beans.factory.BeanCreationException(
                "trustAllRestTemplate", "Failed to build SSL trust-all RestTemplate", e);
        }
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

    @PostMapping("/cve/search")
    public String searchCve(@RequestBody Map<String, Object> requestJson) {
        log.info("## AtipController::searchCve - request : {}", requestJson);

        var paramMap = requestJson.getParamMap();
        var cveId = Optional.ofNullable(paramMap.get("cveId"))
                            .filter(String.class::isInstance)
                            .map(String.class::cast)
                            .orElse("");
        log.debug("requested cveId: {}", cveId);
        if (StringUtils.isEmpty(cveId)) {
            log.error("Error: {}", "invalid cveId");
        }

        return service.searchCve(cveId);
    }
}
```

#### Service

```java
import java.net.URI;
import java.security.GeneralSecurityException;
import java.time.Duration;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import org.apache.hc.client5.http.impl.classic.HttpClients;
import org.apache.hc.client5.http.impl.io.BasicHttpClientConnectionManager;
import org.apache.hc.client5.http.socket.ConnectionSocketFactory;
import org.apache.hc.client5.http.socket.PlainConnectionSocketFactory;
import org.apache.hc.client5.http.ssl.NoopHostnameVerifier;
import org.apache.hc.client5.http.ssl.SSLConnectionSocketFactory;
import org.apache.hc.client5.http.ssl.TrustAllStrategy;
import org.apache.hc.core5.http.config.RegistryBuilder;
import org.apache.hc.core5.ssl.SSLContexts;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.stereotype.Service;
import org.springframework.web.client.ResourceAccessException;
import org.springframework.web.client.RestClientResponseException;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriComponentsBuilder;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.json.JsonMapper;
import com.fasterxml.jackson.databind.node.ArrayNode;
import com.fasterxml.jackson.databind.node.ObjectNode;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
/*
 * RestAPI Client GET Test
 */

@Slf4j
@Service
@RequiredArgsConstructor
public class RestTemplateTestService {

    private final RestTemplate trustAllRestTemplate;

    private final ObjectMapper mapper = JsonMapper.builder()
                                        .addModule(new JavaTimeModule())
                                        .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
                                        .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
                                        .build();

    // https://api.cvesearch.com/search?q=CVE-2023-1234
    private static final String URL_DOMAIN_PORTAL = "api.cvesearch.com";
    private static final String URL_API_PATH_CVE_SEARCH = "/search";

    private static final String TEST_BEARER_AUTH_TOKEN = "my-access-token";

    public List<Map<String, Object>> send(HttpMethod method, URI uri, HttpEntity<Void> httpEntity) {
        ResponseEntity<JsonNode> responseEntity;
        List<Map<String, Object>> list;

        if (StringUtils.isEmpty(method.name())) {
            log.error("Error: No content");
            return null;
        }

        try {
            log.debug("## Request to get CVE info. uri : {}", uri);

            responseEntity = trustAllRestTemplate.exchange(uri, method, httpEntity, JsonNode.class);

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
                    list = new ArrayList<Map<String, Object>>();
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
                log.error("Error: Atip Internal Error");
            }
        } catch (RestClientResponseException ex) {
            log.error("Error: {}", ex.getMessage());

            var status = ex.getStatusCode();
            log.debug("Status Code: " + status);
            // log.debug("code value: " + status.value());
            // log.debug("status text: " + ex.getStatusText());
            log.debug("Response Body: " + ex.getResponseBodyAsString());
            log.debug("Headers: " + ex.getResponseHeaders());

            // used inappropriate access-key: try to exchange ti domain between portal and bundle
            if (ex.getStatusCode().is4xxClientError()) {
                log.error("Error: {}", ex.getResponseBodyAsString());
            } else if (ex.getStatusCode().is5xxServerError()) {
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

    public void searchCve(String cveId, ResponseJsonObject responseJsonObject) {

        var uri = UriComponentsBuilder.newInstance()
                    .scheme("https")
                    .host(URL_DOMAIN_PORTAL)
                    .path(URL_API_PATH_CVE_SEARCH)
                    .queryParam("q", cveId)
                    .encode().build().toUri();

        var headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.setBearerAuth(TEST_BEARER_AUTH_TOKEN);

        var httpEntity = new HttpEntity<Void>(headers);
        var responseBody = send(HttpMethod.GET, uri, httpEntity);

        try {
            log.debug("## response: {}",
                    mapper.writerWithDefaultPrettyPrinter().writeValueAsString(responseBody));
        } catch (JsonProcessingException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            log.error("## invalid response");
        }
    }
}
```

## WebClient (Async mechanism HttpClient for Spring)

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

        // TLS 전략: 비동기용
        TlsStrategy tls = ClientTlsStrategyBuilder.create()
            .setSslContext(sslContext)
            .setHostnameVerifier(NoopHostnameVerifier.INSTANCE)
            .buildAsync();

        // 커넥션 풀: 비동기용
        PoolingAsyncClientConnectionManager cm =
            PoolingAsyncClientConnectionManagerBuilder.create()
                .setTlsStrategy(tls)
                // 필요 시 풀 한도 조정
                .setMaxConnTotal(200)    // default: 25
                .setMaxConnPerRoute(50)  // default: 5
                .build();

        // 요청/응답 타임아웃 등(비동기 클라이언트에도 적용 가능)
        RequestConfig requestConfig = RequestConfig.custom()
            .setConnectionRequestTimeout(Timeout.ofSeconds(1)) // 풀에서 커넥션 임대 대기
            .setConnectTimeout(Timeout.ofSeconds(2))           // TCP 연결 수립
            .setResponseTimeout(Timeout.ofSeconds(5))          // 응답 대기
            .build();

        CloseableHttpAsyncClient asyncClient = HttpAsyncClients.custom()
            .setConnectionManager(cm)
            .setDefaultRequestConfig(requestConfig)
            .evictExpiredConnections()
            .build();

        // WebClient 커넥터에 주입 (Spring이 lifecycle 관리)
        var connector = new HttpComponentsClientHttpConnector(asyncClient);

        return WebClient.builder()
            .clientConnector(connector)
            // 공통 타임아웃(선택): exchange 레벨에서 한번 더 안전망
            .build();

    } catch (GeneralSecurityException e) {
        throw new org.springframework.beans.factory.BeanCreationException(
            "trustAllWebClient", "Failed to build SSL trust-all WebClient", e);
    }
}
```
