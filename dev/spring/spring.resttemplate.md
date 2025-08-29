# RestTemplate: spring에서 http client 사용하기

## RestTemplate Example

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

    private final ObjectMapper mapper = JsonMapper.builder()
                                        .addModule(new JavaTimeModule())
                                        .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
                                        .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
                                        .build();

    // https://api.cvesearch.com/search?q=CVE-2023-1234
    private static final String URL_DOMAIN_PORTAL = "api.cvesearch.com";
    private static final String URL_API_PATH_CVE_SEARCH = "/search";

    private static final String TEST_BEARER_AUTH_TOKEN = "my-access-token";

    public String searchCve(String cveId) {

        URI uri = UriComponentsBuilder.newInstance()
                    .scheme("https")
                    .host(URL_DOMAIN_PORTAL)
                    .path(URL_API_PATH_CVE_SEARCH)
                    .queryParam("q", cveId)
                    .encode().build().toUri();

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.setBearerAuth(TEST_BEARER_AUTH_TOKEN);

        HttpEntity<Void> httpEntity = new HttpEntity<>(headers);
        ResponseEntity<String> responseEntity;
        String result;
        ObjectNode responseJsonObject = mapper.createObjectNode();

        try {
            RestTemplate restTemplate = createRestTemplate();

            log.debug("## Request RestAPI GET Method : {}", uri);

            responseEntity = restTemplate.exchange(uri, HttpMethod.GET, httpEntity, String.class);

            // ResponseEntity
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

                    // ObjectNode to Map<String, Object>
                    Map<String, Object> map = mapper.convertValue(node,
                        new com.fasterxml.jackson.core.type.TypeReference<Map<String,Object>>() {});
                    // var map = mapper.treeToValue(node, Map.class);
                    responseJsonObject.set("responseJsonObject", map);

                    // ObjectNode to String
                    result = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(node);
                } else if (responseBody.isArray()) {
                    ArrayNode node = (ArrayNode) responseBody;

                    // ArrayNode to List<Map<String, Object>>
                    List<Map<String, Object>> list = mapper.convertValue(node,
                        new com.fasterxml.jackson.core.type.TypeReference<List<Map<String,Object>>>() {});
                    responseJsonObject.set("responseJsonObject", list);

                    // ArrayNode to String
                    result = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(node);
                } else {
                    log.error("Error: Response content type is NOT applicable");
                }
            } else {
                // TODO: Http response OK, but there is NO information
                log.error("Error: Response have NO information");
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

        return result;
    }

    // for org.apache.httpcomponents.client5:httpclient5:jar:5.3
    private RestTemplate createRestTemplate() throws GeneralSecurityException {
        var sslContext = SSLContexts.custom()
            .loadTrustMaterial(null, TrustAllStrategy.INSTANCE)
            .build();

        var socketFactoryRegistry = RegistryBuilder.<ConnectionSocketFactory> create()
            .register("https", new SSLConnectionSocketFactory(sslContext, NoopHostnameVerifier.INSTANCE))
            .register("http", new PlainConnectionSocketFactory())
            .build();

        var cm = new BasicHttpClientConnectionManager(socketFactoryRegistry);
        // if it needs to use http connection pool then
        // use PoolingHttpClientConnectionManager instead of BasicHttpClientConnectionManager
        // and register RestTemplate as Spring Bean to be singleton
        // var cm = new PoolingHttpClientConnectionManager(socketFactoryRegistry);

        var httpClient = HttpClients.custom()
            .setConnectionManager(cm)
            .evictExpiredConnections()
            .build();

        var rf = new HttpComponentsClientHttpRequestFactory(httpClient);
        rf.setConnectTimeout(Duration.ofSeconds(5));
        rf.setConnectionRequestTimeout(Duration.ofSeconds(5));

        return new RestTemplate(rf);
    }
}
```

## RestTemplate for apache httpclient5 5.4+

```java
    import java.security.GeneralSecurityException;
    import java.time.Duration;
    import org.apache.hc.client5.http.impl.classic.HttpClients;
    import org.apache.hc.client5.http.impl.io.BasicHttpClientConnectionManager;
    import org.apache.hc.client5.http.ssl.ClientTlsStrategyBuilder;
    import org.apache.hc.client5.http.ssl.NoopHostnameVerifier;
    import org.apache.hc.client5.http.ssl.TrustAllStrategy;
    import org.apache.hc.core5.ssl.SSLContexts;
    import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
    import org.springframework.web.client.RestTemplate;

    // for org.apache.httpcomponents.client5:httpclient5:jar:5.4+
    private RestTemplate createRestTemplate() throws GeneralSecurityException {
        var sslContext = SSLContexts.custom()
            .loadTrustMaterial(null, TrustAllStrategy.INSTANCE)
            .build();

        var tls = ClientTlsStrategyBuilder.create()
            .setSslContext(sslContext)
            .setHostnameVerifier(NoopHostnameVerifier.INSTANCE)
            .build();


        var cm = BasicHttpClientConnectionManagerBuilder.create()
            .setTlsSocketStrategy(tls)   
            .build();

        // if it needs to use http connection pool then
        // use PoolingHttpClientConnectionManager instead of BasicHttpClientConnectionManager
        // and register RestTemplate as Spring Bean to be singleton
        /*
            var cm = PoolingHttpClientConnectionManagerBuilder.create()
                .setTlsSocketStrategy(tls)   
                .build();
        */

        var httpClient = HttpClients.custom()
            .setConnectionManager(cm)
            .evictExpiredConnections()
            .build();

        var rf = new HttpComponentsClientHttpRequestFactory(httpClient);
        rf.setConnectTimeout(Duration.ofSeconds(5));
        rf.setConnectionRequestTimeout(Duration.ofSeconds(5));

        return new RestTemplate(rf);
    }
```
