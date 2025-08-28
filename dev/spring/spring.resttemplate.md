# RestTemplate: spring에서 http client 사용하기

## RestTemplate Example

```java
import java.net.URI;
import java.security.GeneralSecurityException;
import java.time.Duration;
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
import org.springframework.web.client.RestClientResponseException;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriComponentsBuilder;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

/*
 * RestAPI Client GET Test
 */

@Slf4j
@Service
@RequiredArgsConstructor
public class AtipService {

    // https://api.cvesearch.com/search?q=CVE-2023-1234
    private static final String URL_DOMAIN_PORTAL = "api.cvesearch.com";
    private static final String URL_API_PATH_CVE_SEARCH = "/search";

    private static final String TEST_BEARER_AUTH_TOKEN =
            "my-access-token";

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
        try {
            RestTemplate restTemplate = createRestTemplate();

            log.debug("## Request RestAPI GET Method : {}", uri);

            responseEntity = restTemplate.exchange(uri, HttpMethod.GET, httpEntity, String.class);

            // Access information from the ResponseEntity
            log.debug("Response Body: " + responseEntity.getBody());
            log.debug("Status Code: " + responseEntity.getStatusCode());
            log.debug("Headers: " + responseEntity.getHeaders());

            log.debug("ti response http code: {}, reason: {}",
                    responseEntity.getStatusCode().value(),
                    HttpStatus.valueOf(responseEntity.getStatusCode().value()).getReasonPhrase());

            if (responseEntity.getStatusCode().equals(HttpStatus.OK)) {
                return (String) responseEntity.getBody();
            }
        } catch (RestClientResponseException ex) {
            log.error("Error: {}", ex.getMessage());

            var status = ex.getStatusCode();
            log.debug("Status Code: " + status);
            log.debug("Status Code value: " + status.value());
            log.debug("Status Code: " + ex.getStatusText());
            log.debug("Response Body: " + ex.getResponseBodyAsString());
            log.debug("Headers: " + ex.getResponseHeaders());

            // used inappropriate access-key: try to exchange ti domain between portal and bundle  
            if (status.equals(HttpStatus.I_AM_A_TEAPOT)) {
                log.warn("inappropriate TI access key is used.");
                return "";
            }
        } catch (Exception e) {
            log.error("Error: {}", e.getMessage());
            return "";
        }

        return "";
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
