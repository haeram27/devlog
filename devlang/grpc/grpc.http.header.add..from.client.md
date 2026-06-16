# grpc 클라이언트에서 rpc request에 http header(metadata) 추가하기

gRPC에서 "HTTP Header"라고 부르는 값은 실제 코드에서는 대부분 `Metadata`로 다룹니다.
즉, 클라이언트는 RPC 호출 전에 `Metadata`에 값을 넣고, 이를 Stub 호출에 붙여서 전송합니다.

아래는 Java/Spring 환경에서 가장 많이 쓰는 방법입니다.

## 1) 기본 개념

- 헤더 키는 `Metadata.Key<T>`로 정의한다.
- 일반 문자열 헤더는 `Metadata.ASCII_STRING_MARSHALLER`를 사용한다.
- `authorization`, `x-user-id`, `x-request-id` 같은 값을 여기에 담아 보낸다.

예시 키:

```java
private static final Metadata.Key<String> AUTHORIZATION_KEY =
        Metadata.Key.of("authorization", Metadata.ASCII_STRING_MARSHALLER);

private static final Metadata.Key<String> USER_ID_KEY =
        Metadata.Key.of("x-user-id", Metadata.ASCII_STRING_MARSHALLER);
```

## 2) 가장 표준적인 방법: ClientInterceptor로 Stub에 헤더 부착

gRPC Java에서 가장 일반적으로 쓰는 방식은 `MetadataUtils.newAttachHeadersInterceptor(...)`를 이용해 Stub에 인터셉터를 붙이는 방법입니다.

```java
import io.grpc.Channel;
import io.grpc.ClientInterceptor;
import io.grpc.Metadata;
import io.grpc.stub.MetadataUtils;

// generated stub
import com.example.grpc.UserServiceGrpc;

public class UserGrpcClient {

    private static final Metadata.Key<String> AUTHORIZATION_KEY =
            Metadata.Key.of("authorization", Metadata.ASCII_STRING_MARSHALLER);
    private static final Metadata.Key<String> USER_ID_KEY =
            Metadata.Key.of("x-user-id", Metadata.ASCII_STRING_MARSHALLER);

    public UserServiceGrpc.UserServiceBlockingStub createStub(Channel channel) {
        Metadata headers = new Metadata();
        headers.put(AUTHORIZATION_KEY, "Bearer sample-jwt-token");
        headers.put(USER_ID_KEY, "user-1001");

        ClientInterceptor headerInterceptor = MetadataUtils.newAttachHeadersInterceptor(headers);

        return UserServiceGrpc.newBlockingStub(channel)
                .withInterceptors(headerInterceptor);
    }
}
```

이 방식은 다음 상황에 적합합니다.

- 여러 RPC 호출에 동일한 고정 헤더를 반복해서 붙여야 할 때
- 토큰/테넌트 ID/공통 추적 ID 등을 쉽게 공통화하고 싶을 때

## 3) Spring gRPC에서 채널 Bean에 전역 적용하기

Spring Boot에서 gRPC 채널 Bean을 만들어 쓰는 구조라면, 채널 생성 시점에 인터셉터를 한 번 붙여두는 방식이 관리하기 좋습니다.

```java
import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;
import io.grpc.ClientInterceptor;
import io.grpc.Metadata;
import io.grpc.stub.MetadataUtils;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GrpcClientConfig {

    private static final Metadata.Key<String> AUTHORIZATION_KEY =
            Metadata.Key.of("authorization", Metadata.ASCII_STRING_MARSHALLER);

    @Bean
    public ManagedChannel userServiceChannel() {
        Metadata headers = new Metadata();
        headers.put(AUTHORIZATION_KEY, "Bearer fixed-token");

        ClientInterceptor interceptor = MetadataUtils.newAttachHeadersInterceptor(headers);

        return ManagedChannelBuilder.forAddress("localhost", 9090)
                .usePlaintext()
                .intercept(interceptor)
                .build();
    }
}
```

주의할 점:

- 토큰이 요청마다 달라지는 경우(사용자별 JWT 등)에는 고정 헤더 방식만으로는 부족하다.
- 이 경우 4번처럼 호출 시점에 동적으로 값을 넣는 방식을 사용한다.

## 4) 호출마다 다른 헤더를 넣어야 할 때

요청마다 값이 바뀌면, 호출 직전에 `Metadata`를 새로 만들고 해당 호출용 Stub에만 붙이는 방식이 안전합니다.

```java
import io.grpc.Metadata;
import io.grpc.stub.MetadataUtils;

import com.example.grpc.UserRequest;
import com.example.grpc.UserResponse;
import com.example.grpc.UserServiceGrpc;

public class DynamicHeaderClient {

    private static final Metadata.Key<String> AUTHORIZATION_KEY =
            Metadata.Key.of("authorization", Metadata.ASCII_STRING_MARSHALLER);

    public UserResponse getProfile(
            UserServiceGrpc.UserServiceBlockingStub baseStub,
            String jwtToken,
            String requestId
    ) {
        Metadata headers = new Metadata();
        headers.put(AUTHORIZATION_KEY, "Bearer " + jwtToken);
        headers.put(Metadata.Key.of("x-request-id", Metadata.ASCII_STRING_MARSHALLER), requestId);

        UserServiceGrpc.UserServiceBlockingStub callScopedStub =
                baseStub.withInterceptors(MetadataUtils.newAttachHeadersInterceptor(headers));

        return callScopedStub.getUserProfile(
                UserRequest.newBuilder().setUserId("user-1001").build()
        );
    }
}
```

## 5) 서버에서 읽기와 연결

클라이언트에서 보낸 `Metadata`는 서버에서 `ServerInterceptor + Context` 패턴으로 읽는 것이 안전합니다.
이 부분은 아래 문서를 참고하면 바로 연결해서 구현할 수 있습니다.

- `grpc.read.header.value.in.service.method.md`

## 6) 자주 하는 실수 체크

- 헤더 키 이름 오타 (`x-user-id` vs `x-userid`)
- `-bin` 접미사 규칙 누락
  - 바이너리 값은 반드시 `key-bin` + `BINARY_BYTE_MARSHALLER`를 사용해야 함
- 요청별 동적 토큰인데 고정 인터셉터로만 처리하는 설계

정리하면, gRPC 클라이언트에서 헤더를 추가하는 핵심은 `Metadata`를 만들고, `ClientInterceptor`를 통해 Stub/Channel에 붙여 전송하는 것입니다.
