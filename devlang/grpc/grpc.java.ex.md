# grpc java example

Java에서 gRPC를 구현하려면 .proto 정의, 코드 생성(Stubs), 서버 구현, 클라이언트 호출의 4단계를 거칩니다. 주로 [Maven](https://narup.tistory.com/122)이나 [Gradle](https://tech.ktcloud.com/entry/gRPC%EB%A1%9C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-API-%EA%B0%9C%EB%B0%9C-%EC%B2%AB-%EB%B2%88%EC%A7%B8-%EC%84%9C%EB%B2%84%EC%99%80-%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8-%EA%B5%AC%ED%98%84) 프로젝트 환경에서 플러그인을 통해 자동화합니다.

## GRPC 서비스 개발 순서

1. protobuf 구현
1. protobuf 빌드 하여 클라이언트(Stub)과 서버(ImplBase) Java class 생성
1. 서버 로직 구현 ()
1. 클라이언트 로직 구현

## 서버 구현

Java와 Spring Boot를 사용해 gRPC 백엔드 API를 구현하는 전체 과정을 핵심만 정리해 드립니다.

### 1. 프로젝트 설정 (build.gradle)

gRPC 라이브러리와 Proto 파일을 Java 코드로 자동 변환해 주는 플러그인을 추가합니다.

```gradle
plugins {
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
    id 'java'
    id 'com.google.protobuf' version '0.9.4' // Protobuf 컴파일 플러그인
}

dependencies {
    // Spring Boot 환경에서 gRPC 서버를 쉽게 구동해 주는 스타터
    implementation 'net.devh:grpc-server-spring-boot-starter:2.15.0.RELEASE'
    implementation 'com.google.protobuf:protobuf-java-util:3.25.1'
}

protobuf {
    protoc { artifact = "com.google.protobuf:protoc:3.25.1" }
    plugins {
        grpc { artifact = "io.grpc:protoc-gen-grpc-java:1.60.0" }
    }
    generateProtoTasks {
        all()*.plugins { grpc {} }
    }
}
```

- Maven: protobuf-maven-plugin 또는 protoc-jar-maven-plugin을 pom.xml에 추가합니다.
- Gradle: com.google.protobuf 플러그인을 사용하여 build.gradle에 설정합니다.

### 2. protobuf API 규격 정의 (src/main/proto/user.proto)

서버와 클라이언트가 주고받을 데이터와 서비스 메서드를 정의합니다.

```protobuf
syntax = "proto3";

package user;
option java_multiple_files = true;
option java_package = "com.example.grpc.user";

// gRPC 서비스 정의
service UserService {
  rpc GetUserProfile (UserRequest) returns (UserResponse);
}
// 요청 데이터
message UserRequest {
  string user_id = 1;
}
// 응답 데이터
message UserResponse {
  string user_id = 1;
  string name = 2;
  map<string, string> metadata = 3;
}
```

작성 후 `./gradlew generateProto` 명령어를 실행하면 Java 클래스 파일들이 자동 생성됩니다.

#### message 정의 시 필드에 지정하는 정수 (필드 넘버)

```protobuf
message UserResponse {
  string user_id = 1;
  string name = 2;
  map<string, string> metadata = 3;
}
```

- proto3에서 필드 선언의 `= 1`, `= 2` 같은 숫자는 필드 번호(field number, tag) 입니다.
- 바이너리 직렬화에서 필드를 식별하는 ID입니다.  `user_id = 1`, `name = 2`는 전송 데이터 안에서 각각 1번/2번 필드로 인코딩됩니다.
- 순서가 아니라 “계약(API 스키마)”의 일부입니다. 코드에서 필드 순서를 바꿔도 번호가 같으면 호환성이 유지됩니다.
- 중복 불가입니다. 한 message 내부에서 번호는 유일해야 합니다.
  - 필드 번호는 message 내부에서 각각 유일해야 합니다.
  - 보통 삭제한 번호도 나중에 재사용하지 않도록 `reserved`로 막아 둡니다.
- 한번 배포한 번호는 가급적 재사용하면 안 됩니다. 필드를 삭제할 때는 보통 reserved로 막아 두어야, 예전 클라이언트와 충돌을 피할 수 있습니다.
- 번호 크기에 따라 인코딩 크기 효율이 달라집니다. 자주 쓰는 필드는 작은 번호(예: 1~15)를 주는 것이 유리합니다.

### 3. 비즈니스 로직 구현 (Java Service)

자동 생성된 베이스 클래스(UserServiceImplBase)를 상속받고, @GrpcService 어노테이션을 붙여 Spring Bean으로 등록합니다.

```java
package com.example.grpc.service;
import com.example.grpc.user.UserRequest;
import com.example.grpc.user.UserResponse;
import com.example.grpc.user.UserServiceGrpc;
import io.grpc.stub.StreamObserver;
import net.devh.boot.grpc.server.service.GrpcService;
import java.util.HashMap;
import java.util.Map;

@GrpcService // 이 클래스를 gRPC 서비스로 노출합니다.
public class UserServiceImpl extends UserServiceGrpc.UserServiceImplBase {

    @Override
    public void getUserProfile(UserRequest request, StreamObserver<UserResponse> responseObserver) {
        // 1. 요청 데이터 추출
        String userId = request.getUserId();

        // 2. 비즈니스 로직 처리 및 Map 데이터 생성
        Map<String, String> userMetadata = new HashMap<>();
        userMetadata.put("role", "developer");
        userMetadata.put("tier", "premium");

        // 3. Protobuf 빌더로 응답 객체 생성
        UserResponse response = UserResponse.newBuilder()
                .setUserId(userId)
                .setName("홍길동")
                .putAllMetadata(userMetadata)
                .build();

        // 4. 클라이언트에게 응답 전달 및 스트림 종료
        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
```

### 4. 서버 포트 설정 (application.yml)

gRPC 서버가 사용할 포트를 지정합니다. 기본값은 9090입니다.

```yaml
spring:
  grpc:
    server:
      port: 9090
```

### 5. 실행 및 테스트

   1. ./gradlew bootRun으로 스프링 부트 애플리케이션을 실행합니다.
   1. Postman 또는 BloomRPC 같은 테스트 툴을 켭니다.
   1. 작성한 user.proto 파일을 툴에 로드합니다.
   1. 주소창에 localhost:9090을 입력하고 {"user_id": "123"} 형식으로 요청을 보내 테스트합니다.

---

## 클라이언트 구현

Java 환경에서 서버를 호출할 gRPC 클라이언트를 구현하는 방법입니다.

### 1. 의존성 설정 추가 (build.gradle)

기존 서버 프로젝트에 클라이언트 기능을 추가하거나, 별도 프로젝트라면 아래 클라이언트 스타터 라이브러리를 의존성에 추가해야 합니다.

```gradle
dependencies {
    // gRPC Spring Boot 클라이언트 스타터
    implementation 'net.devh:grpc-client-spring-boot-starter:2.15.0.RELEASE'
}
```

### 2. 클라이언트 설정 (application.yml)

연결할 gRPC 서버의 주소와 포트를 설정합니다. user-server라는 임의의 채널 이름을 지정하여 관리합니다.

```yaml
grpc:
  client:
    user-server: # 클라이언트 채널 이름
      address: 'static://localhost:9090' # 서버 주소
      negotiation-type: plaintext # SSL/TLS 미사용 시 설정
```

- 클라이언트 채널 이름
  - 채널 이름은 클라이언트 로컬 식별자로써 클라이언트에서 특정 grpc 서버/포트와 맵핑 되는 ID 값이다. 다시 말해 클라이언트에서 연결해야 grpc 서버가 여러 개 있을 때 각 서버를 클라이언트에서 구분하는 이름이다. 클라이언트는 grpc 서버당 구분되는 채널 이름을 지정해서 사용한다.
  - 채널 이름은 grpc 클라이언트 내에서만 유효하며 관리된다. grpc 서버에서는 전혀 관련하지 않는다.
- 클라이언트에서 grpc 서버로의 실제 연결은 채널 이름 매칭으로 이루어짐
  - `@GrpcClient("channel-name")`에 적은 문자열이 클라이언트 설정(application.yml) 키와 같아야 합니다.

원하시면 다음 답변에서 “서버에서 확인해야 하는 것” 체크리스트(포트, 서비스명, 패키지, TLS 여부)만 1분 점검표로 짧게 정리해드리겠습니다.

### 3. 클라이언트 서비스 구현 (Java)

@GrpcClient 어노테이션을 사용하여 서버와 통신할 Stub(스텁)을 자동으로 주입받습니다.

```java
package com.example.grpc.client;
import com.example.grpc.user.UserRequest;
import com.example.grpc.user.UserResponse;
import com.example.grpc.user.UserServiceGrpc;
import net.devh.boot.grpc.client.inject.GrpcClient;
import org.springframework.stereotype.Service;

@Servicepublic class UserClientService {

    // application.yml에 설정한 채널 이름을 입력합니다.
    @GrpcClient("user-server")
    private UserServiceGrpc.UserServiceBlockingStub userServiceBlockingStub;

    public void callUserProfile(String userId) {
        // 1. 서버로 보낼 요청 객체 생성
        UserRequest request = UserRequest.newBuilder()
                .setUserId(userId)
                .build();

        try {
            // 2. 서버 호출 및 응답 수신 (동기 방식)
            UserResponse response = userServiceBlockingStub.getUserProfile(request);

            // 3. 결과 출력 및 검증
            System.out.println("=== gRPC 서버 응답 결과 ===");
            System.out.println("User ID: " + response.getUserId());
            System.out.println("User Name: " + response.getName());
            System.out.println("Metadata Map: " + response.getMetadataMap());
            
        } catch (Exception e) {
            System.err.println("gRPC 호출 실패: " + e.getMessage());
        }
    }
}
```

### 4. 테스트용 Controller 또는 Runner 작성

스프링 부트가 구동될 때 클라이언트를 바로 실행해 볼 수 있도록 테스트용 CommandLineRunner를 작성합니다.

```java
package com.example.grpc;
import com.example.grpc.client.UserClientService;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplicationpublic class GrpcClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(GrpcClientApplication.class, args);
    }

    @Bean
    public CommandLineRunner testRunner(UserClientService userClientService) {
        return args -> {
            System.out.println("gRPC 서버 호출 테스트를 시작합니다...");
            // 앞서 구현한 클라이언트 서비스 호출
            userClientService.callUserProfile("user_12345");
        };
    }
}
```
