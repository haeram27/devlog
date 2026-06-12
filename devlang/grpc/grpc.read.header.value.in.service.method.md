# spring grpc server에서 http header 읽기

수동으로 서버 빌더를 조작할 필요 없이, Spring의 의존성 주입(DI)과 애너테이션(`@GrpcGlobalInterceptor` 또는 `@GrpcInterceptor`)을 사용하여 헤더 파라미터를 서비스 메서드로 깔끔하게 전달할 수 있습니다.

Spring Boot 백엔드에 맞춘 상세 가이드와 구현 예제입니다.

## 핵심 개념 구조

   1. 글로벌 인터셉터(Global Interceptor): 들어오는 모든 gRPC 요청을 가로채어 Metadata(헤더)에서 원하는 키를 추출하고, 이를 스레드 로컬 기반의 io.grpc.Context에 바인딩합니다.
   2. 컨텍스트 키(Context Key): 데이터를 안전하게 저장하고 꺼내기 위한 스레드 세이프(Thread-safe)한 보관함 열쇠 역할을 합니다.
   3. gRPC 서비스 메서드: 인터셉터가 생성한 컨텍스트 내에서 실행되므로, 메서드 내부 어디서든 해당 헤더 파라미터를 동적으로 꺼내 쓸 수 있습니다.

### `@GrpcGlobalInterceptor`와 `@GrpcInterceptor`

- [grpc-spring-boot-starter](https://github.com/yidongnan/grpc-spring-boot-starter) 에서 제공하는 grpc Interceptor
- grpc 전역 서비스 또는 단일 서비스의 rpc가 호출될 때 Intercept를 설정해 줄 수 있다.
- `@GrpcGlobalInterceptor`와 `@GrpcInterceptor`는 둘 다 Spring Boot 환경에서 gRPC 인터셉터를 빈(Bean)으로 등록할 때 사용하는 애너테이션이지만, 적용되는 범위(Scope)에서 명확한 차이가 있습니다.

빌드 dependency 추가

```gradle
dependencies {
    implementation 'net.devh:grpc-server-spring-boot-starter:3.1.0.RELEASE'
}
```

- 패키지 경로: net.devh.boot.grpc.server.interceptor.GrpcGlobalInterceptor
- 특징: Spring Boot 진영에서 가장 활발하게 메인터넌스되는 라이브러리로, 별도의 설정 없이 빈(Bean) 등록만 하면 인터셉터가 자동으로 동작합니다.

#### 1. `@GrpcGlobalInterceptor` (전역 적용)

이 애너테이션이 붙은 인터셉터는 프로젝트 내의 모든 gRPC 서비스와 메서드에 자동으로 적용됩니다.

- 적용 범위: 전체 서비스 (Global)
- 주요 용도: 서비스 구분 없이 공통으로 처리해야 하는 전역 로직에 사용합니다.
- 서버 전체 로깅 및 추적 ID (Trace ID, MDC) 주입
  - 전역 예외 처리 (Global Exception Handling)
  - 모니터링을 위한 메트릭 수집 (Prometheus 등)
- 특징: 한 번 등록하면 새로 추가되는 gRPC 서비스에도 별도 설정 없이 자동으로 결합되므로 관리가 편리합니다.

```java
@GrpcGlobalInterceptorpublic class GlobalLoggingInterceptor implements ServerInterceptor {
    // 모든 gRPC 호출 시 이 로직을 통과함
}
```

#### 2. @GrpcInterceptor (선택적/지역 적용)

이 애너테이션은 특정 gRPC 서비스 클래스에만 선택적으로 인터셉터를 지정하고 싶을 때 사용합니다.

- 적용 범위: 특정 서비스 클래스 단위 (Local)
- 주요 용도: 특정 API 그룹이나 서비스에만 특화된 로직이 필요할 때 사용합니다.
- 특정 서비스 전용 권한 검증 (예: 관리자 전용 서비스에만 인증 인터셉터 적용)
  - 입력 데이터 검증 (Validation)
  - 특정 메서드 그룹에 대한 속도 제한 (Rate Limiting)
- 특징: @GrpcInterceptor를 인터셉터 클래스에 붙여 빈으로 등록한 후, 적용 대상이 될 gRPC 서비스 클래스 상단에 해당 인터셉터의 클래스 타입을 명시해 주어야 작동합니다.

[구현 및 사용 예시]

```java
// 1. 특정 서비스용 인터셉터 정의 및 빈 등록
@GrpcInterceptorpublic class AdminAuthInterceptor implements ServerInterceptor {
    // 관리자 서비스 전용 권한 체크 로직
}
// 2. 적용하고 싶은 특정 gRPC 서비스에만 명시적으로 결합
@GrpcService
@GrpcProfileInterceptor(interceptors = AdminAuthInterceptor.class)
// yidongnan 버전에 따라 문법 차이가 있을 수 있음
// (일반적으로 yidongnan 스타터에서는 @GrpcService(interceptors = AdminAuthInterceptor.class) 형태로 지정합니다)
public class AdminServiceImpl extends AdminServiceGrpc.AdminServiceImplBase {
    // AdminAuthInterceptor가 이 서비스에만 적용됨
}
```

#### 요약 비교

| 구분 | @GrpcGlobalInterceptor | @GrpcInterceptor |
|---|---|---|
| 적용 범위 | 애플리케이션 내 모든 gRPC 서비스 | @GrpcService에 명시적으로 지정한 서비스만 |
| 자동 결합 | O (빈 등록만 하면 끝) | X (서비스 클래스에 수동 지정 필요) |
| 권장 용도 | 로깅, 트레이싱(Trace ID), 전역 모니터링 | 서비스별 권한 검증, 데이터 검증 |

두 애너테이션을 동시에 사용할 경우 @GrpcGlobalInterceptor가 먼저 실행된 후, 서비스에 지정된 @GrpcInterceptor가 실행됩니다.
현재 구현 중이신 헤더(예: 인증 토큰, Trace ID 등)의 특성에 맞춰 방식을 선택하시면 됩니다. 혹시 이 인터셉터들의 실행 순서(Order)를 세부적으로 제어하는 방법이 필요하신가요?

## 단계별 구현 방법

### 1. 공유할 컨텍스트 키(Context Key) 정의

인터셉터와 서비스가 함께 사용할 키 설정 클래스를 생성합니다.

```java
package com.example.grpc.config;
import io.grpc.Context;
public class GrpcHeaderContext {
    // 'X-Trace-Id'나 커스텀 파라미터를 안전하게 전달하기 위한 스레드 세이프 키
    public static final Context.Key<String> TRACE_ID_KEY = Context.key("trace-id");
}
```

### 2. Spring Boot용 gRPC 인터셉터 구현

인터셉터 클래스에 `@GrpcGlobalInterceptor` 애너테이션을 붙입니다. 이렇게 하면 Spring이 lifecycle 관리 시 자동으로 모든 서비스에 인터셉터를 등록합니다.

```java
package com.example.grpc.interceptor;
import com.example.grpc.config.GrpcHeaderContext;
import io.grpc.Context;
import io.grpc.Contexts;
import io.grpc.Metadata;
import io.grpc.ServerCall;
import io.grpc.ServerCallHandler;
import io.grpc.ServerInterceptor;
import net.devh.boot.grpc.server.interceptor.GrpcGlobalInterceptor;

@GrpcGlobalInterceptor // 자동으로 모든 gRPC 서비스에 이 인터셉터를 적용합니다.
public class HeaderExtractionInterceptor implements ServerInterceptor {

    // 클라이언트가 보낼 헤더 이름과 일치하는 메타데이터 키 정의
    private static final Metadata.Key<String> TRACE_ID_HEADER = 
            Metadata.Key.of("x-trace-id", Metadata.ASCII_STRING_MARSHALLER);

    @Override
    public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(
            ServerCall<ReqT, RespT> call, 
            Metadata headers, 
            ServerCallHandler<ReqT, RespT> next) {
        
        // 1. 메타데이터(헤더)에서 파라미터 추출 (값이 없으면 null 반환)
        String traceId = headers.get(TRACE_ID_HEADER);
        
        if (traceId == null) {
            traceId = "UNKNOWN-GENERATE-NEW-ID-" + java.util.UUID.randomUUID();
        }

        // 2. 현재 스레드/요청에 종속된 gRPC Context에 값 바인딩
        Context context = Context.current().withValue(GrpcHeaderContext.TRACE_ID_KEY, traceId);
        
        // 3. 새롭게 값이 세팅된 컨텍스트를 아래 실행 체인으로 전달
        return Contexts.interceptCall(context, call, headers, next);
    }
}
```

### 3. @GrpcService 메서드에서 헤더 값 참조하기

`@GrpcService`가 선언된 서비스 클래스 내부에서 static으로 정의했던 컨텍스트 키의 .get() 메서드를 호출하여 값을 꺼냅니다.

```java
package com.example.grpc.service;
import com.example.grpc.config.GrpcHeaderContext;
import com.example.grpc.proto.OrderRequest;
import com.example.grpc.proto.OrderResponse;
import com.example.grpc.proto.OrderServiceGrpc;
import io.grpc.stub.StreamObserver;
import lombok.extern.slf4j.Slf4j;
import net.devh.boot.grpc.server.service.GrpcService;

@Slf4j
@GrpcService // Spring Boot가 이 클래스를 gRPC 서비스로 자동 등록합니다
public class OrderServiceImpl extends OrderServiceGrpc.OrderServiceImplBase {

    @Override
    public void createOrder(OrderRequest request, StreamObserver<OrderResponse> responseObserver) {
        // 1. 인터셉터가 현재 스레드 컨텍스트에 담아둔 헤더 파라미터를 안전하게 추출
        String traceId = GrpcHeaderContext.TRACE_ID_KEY.get();
        
        // 2. 로그 출력이나 비즈니스 로직에 파라미터 활용
        log.info("[TraceID: {}] 주문 생성 중 - 아이템 ID: {}", traceId, request.getItemId());
        
        OrderResponse response = OrderResponse.newBuilder()
                .setOrderId("ORD-12345")
                .setStatus("SUCCESS")
                .build();
                
        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
```

## Spring Boot 핵심 요약 Checklist

- 수동 설정 없음: 별도의 수동 코드 작성 없이 `@GrpcGlobalInterceptor`만으로 모든 서비스에 연동됩니다.
- 격리성 보장(Scope Safety): gRPC Java는 요청별 스레드(Thread-per-request) 패러다임 기반(혹은 명시적 컨텍스트 전파)으로 동작하므로, io.grpc.Context를 사용하면 동시 요청 간에 메타데이터가 섞이거나 유출되지 않습니다.
