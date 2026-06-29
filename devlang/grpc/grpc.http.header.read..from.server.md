# spring grpc server에서 request의 http header 읽기

`org.springframework.grpc:spring-grpc-spring-boot-starter`를 사용할 때, 클라이언트 요청의 Header(Metadata) 값을 백엔드 비즈니스 로직(gRPC ServiceImpl)에서 안전하게 가져다 쓰는 가장 표준적이고 적절한 방식은 **`io.grpc.ServerInterceptor`와 `io.grpc.Context`를 조합하는 방식**입니다.

gRPC는 HTTP/2 기반 위에서 작동하기 때문에 Thread-Local 개념만으로는 완벽하지 않으며, 비동기/스트리밍 환경에서도 안전하게 데이터를 전송할 수 있는 **gRPC 전용 `Context` 객체**를 지원합니다.

가장 적절한 구현 단계를 step-by-step으로 안내해 드리겠습니다.


## 1단계: Context.Key 정의하기

인터셉터와 서비스 클래스 양쪽에서 공유할 Key 객체를 생성합니다. 이 키를 통해 컨텍스트 내부의 특정 데이터에 접근할 수 있습니다.

```java
import io.grpc.Context;

public class GrpcAppContext {
    // Header에서 꺼낸 토큰이나 유저 ID를 저장할 키 정의 (Type-Safe)
    public static final Context.Key<String> USER_ID_KEY = Context.key("user-id");
    public static final Context.Key<String> TRACE_ID_KEY = Context.key("trace-id");
    public static final Context.Key<Integer> USER_ROLE_KEY = Context.key("user-role");
}
```

## 2단계: ServerInterceptor를 통해 Header 값을 Context에 바인딩

`org.springframework.grpc:spring-grpc-spring-boot-starter`에서 제공하는 `io.grpc.ServerInterceptor`를 상속하면 grpc request에 대한 전역 인터셉터를 구현할 수 있습니다.

요청이 들어오는 시점에 Header(Metadata)를 파싱하여 위에서 만든 `Context`에 담고, 해당 컨텍스트를 스레드 체인에 바인딩합니다.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.grpc.server.GlobalServerInterceptor;
import org.springframework.stereotype.Component;

import io.grpc.Context;
import io.grpc.Contexts;
import io.grpc.Metadata;
import io.grpc.ServerCall;
import io.grpc.ServerCallHandler;
import io.grpc.ServerInterceptor;

@Component
@GlobalServerInterceptor
public class HeaderLoggingInterceptor implements ServerInterceptor {

    private static final Logger log = LoggerFactory.getLogger(HeaderLoggingInterceptor.class);


    @Override
    public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(
            ServerCall<ReqT, RespT> call,
            Metadata headers,
            ServerCallHandler<ReqT, RespT> next) {

        // log whole headers for debugging
        log.info("[ServerInterceptor] Method: {} | Headers: {}", call.getMethodDescriptor().getFullMethodName(), headers);

        /* Set Header into Context */
        // Define Metadata keys for the headers you want to extract
        Metadata.Key<String> USER_ID_HEADER_KEY = Metadata.Key.of("x-user-id", Metadata.ASCII_STRING_MARSHALLER);
        Metadata.Key<String> TRACE_ID_HEADER_KEY = Metadata.Key.of("x-trace-id", Metadata.ASCII_STRING_MARSHALLER);
        Metadata.Key<String> USER_ROLE_HEADER_KEY = Metadata.Key.of("x-user-role", Metadata.ASCII_STRING_MARSHALLER);

        // extract the user ID from the incoming gRPC metadata headers
        String userId = headers.get(TRACE_ID_HEADER_KEY);
        String traceId = headers.get(USER_ID_HEADER_KEY);
        String userRole = headers.get(USER_ROLE_HEADER_KEY);

        if (userId != null) {
            // 현재 컨텍스트를 기반으로 새로운 값을 가진 컨텍스트 생성
            // Context는 불변(immutable)이며, withValue()를 호출할 때마다 새로운 Context 객체가 생성됩니다. 그러므로 Context 객체 하나당 하나의 key, value를 가질수 있습니다.
            // gRPC Context는 withValue()를 여러변 호출하면 Map처럼 한 바구니에 데이터를 다 담는 게 아니라, 연결 리스트(Linked List) 형태의 트리 구조로 연결됩니다. 그러므로 현재 context로 부터 모든 조상 Context의 key를 조회할 수 있게됩니다.
            // 만약 현재 Context에서 조상 Context의 키와 동일한 키로 값을 설정하면, 현재 Context에서 해당 키로 조회할 때는 현재(자식) Context의 값이 우선적으로 반환됩니다. (즉, 조상 Context의 값이 가려짐)
            // 여러 개의 Key,Value 쌍을 Context를 통해 사용하고자 하면 각 Key마다 Context를 새로 생성하지 말고, HasMap과 같은 자료구조를 Value로 사용하는 것이 권장됨(GC 성능:Multi-Context 디자인은 요청마다 Context LinkedList를 생성하게 됨, 탐색 성능) 
            Context context = Context.current().withValue(GrpcAppContext.USER_ID_KEY, userId)
                                        .withValue(GrpcAppContext.TRACE_ID_KEY, traceId)
                                        .withValue(GrpcAppContext.USER_ROLE_KEY, userRole != null ? Integer.parseInt(userRole) : null);
            
            // 3. propagate the new context with the user ID to the next interceptor or service implementation
            return Contexts.interceptCall(context, call, headers, next);
        }

        return next.startCall(call, headers);
    }
}
```

## 3단계: gRPC 서비스 레이어에서 Context 값 꺼내 쓰기

비즈니스 로직을 구현하는 `GrpcService` 클래스에서는 `GrpcAppContext.USER_ID_KEY.get()` 메서드를 통해 인터셉터가 넣어준 헤더 값에 안전하게 접근할 수 있습니다.

```java
import io.grpc.stub.StreamObserver;
import org.springframework.stereotype.Service;
// 컴파일된 proto 파일 예시 파일들
import com.example.grpc.UserServiceGrpc;
import com.example.grpc.UserRequest;
import com.example.grpc.UserResponse;

@Service
public class MyUserService extends UserServiceGrpc.UserServiceImplBase {

    @Override
    public void getUserProfile(UserRequest request, StreamObserver<UserResponse> responseObserver) {
        
        // 💡 핵심: 현재 실행 컨텍스트에서 인터셉터가 저장한 Header 값을 안전하게 추출
        String currentUserId = GrpcAppContext.USER_ID_KEY.get();
        
        System.out.println("현재 요청을 보낸 유저 ID: " + currentUserId);

        // 비즈니스 로직 수행...
        UserResponse response = UserResponse.newBuilder()
                .setMessage("Hello, User " + currentUserId)
                .build();

        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
```

## 실제 헤더 값 호출 테스트 하기

- `grpcurl`을 사용하여 Header 테스트를 한다.
- Header의 키 값이 정확히 맞아야 동작한다.

```bash
grpcurl -plaintext \
-H 'x-user-id: tenant' \
-d '{"message":"hello"}' \
localhost:51003 \
console.v1.MyGrpcService/Hello
```


## 💡 왜 이 방식이 가장 적절한가요?

1. **스레드 세이프(Thread-Safe) 보장:** gRPC는 대량의 비동기 요청과 양방향 스트리밍을 처리하기 때문에, 일반적인 Spring MVC의 `ThreadLocal`을 직접 쓰면 다른 요청과 데이터가 뒤섞이거나 유실되는 치명적인 문제가 발생할 수 있습니다. gRPC 내장 `Context`는 스레드가 바뀌어도 컨텍스트 전파 메커니즘(`Contexts.interceptCall`)을 통해 안전하게 데이터를 유지해 줍니다.
2. **비즈니스 로직의 순수성 유지:** 서비스 구현 클래스(`MyUserService`)가 복잡한 gRPC의 `Metadata` 객체나 HTTP 프로토콜 구조에 직접 의존하지 않게 되므로, 가독성이 좋아지고 유닛 테스트(Unit Test) 작성이 매우 용이해집니다.
3. **MDC(로깅) 확장성:** 만약 로그에 `userId`나 `traceId`를 남기고 싶다면, 위 인터셉터 안에서 `MDC.put()`을 해두고 응답이 끝나는 시점에 비워주는 방식으로 응용하기도 무척 편리합니다.