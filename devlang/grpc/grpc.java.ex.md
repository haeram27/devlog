# grpc java example

Java에서 gRPC를 구현하려면 .proto 정의, 코드 생성(Stubs), 서버 구현, 클라이언트 호출의 4단계를 거칩니다. 주로 [Maven](https://narup.tistory.com/122)이나 [Gradle](https://tech.ktcloud.com/entry/gRPC%EB%A1%9C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-API-%EA%B0%9C%EB%B0%9C-%EC%B2%AB-%EB%B2%88%EC%A7%B8-%EC%84%9C%EB%B2%84%EC%99%80-%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8-%EA%B5%AC%ED%98%84) 프로젝트 환경에서 플러그인을 통해 자동화합니다.

## 1. Protobuf 정의 (.proto 파일)

Java 프로젝트의 src/main/proto 디렉토리에 정의합니다. Java 관련 옵션을 추가하면 클래스 생성을 더 세밀하게 제어할 수 있습니다.

```protobuf
syntax = "proto3";
// Java 소스 생성을 위한 옵션
option java_multiple_files = true; // 각 메시지를 개별 파일로 생성
option java_package = "com.example.grpc"; // 생성될 Java 패키지 경로
option java_outer_classname = "HelloWorldProto"; // 외부 클래스 이름

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string name = 1;
}

message HelloResponse {
  string message = 1;
}
```

## 2. 환경 설정 및 코드 생성

플러그인을 통해 .proto 파일을 Java 클래스로 변환합니다.

- Maven: protobuf-maven-plugin 또는 protoc-jar-maven-plugin을 pom.xml에 추가합니다.
- Gradle: com.google.protobuf 플러그인을 사용하여 build.gradle에 설정합니다.
- 코드 생성: 빌드 도구에서 generate-sources 단계를 실행하면 HelloRequest.java, GreeterGrpc.java 등의 파일이 자동 생성됩니다.

## 3. 서버 구현 (Server Implementation)

생성된 ImplBase 클래스를 상속받아 실제 로직을 작성하고 서버를 실행합니다.

```java
public class HelloWorldServer {
    public static void main(String[] args) throws IOException, InterruptedException {
        // 1. 서비스 로직 구현
        GreeterGrpc.GreeterImplBase serviceImpl = new GreeterGrpc.GreeterImplBase() {
            @Override
            public void sayHello(HelloRequest request, StreamObserver<HelloResponse> responseObserver) {
                String greeting = "Hello, " + request.getName();
                HelloResponse response = HelloResponse.newBuilder().setMessage(greeting).build();
                responseObserver.onNext(response); // 응답 전송
                responseObserver.onCompleted(); // 스트림 종료
            }
        };

        // 2. 서버 실행
        Server server = ServerBuilder.forPort(8080)
                .addService(serviceImpl)
                .build();
        server.start();
        server.awaitTermination();
    }
}
```

## 4. 클라이언트 구현 (Client Invocation)

Stub을 생성하여 원격 메서드를 마치 로컬 함수처럼 호출합니다.

```java
public class HelloWorldClient {
    public static void main(String[] args) {
        // 1. 채널 생성 (서버 연결)
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 8080)
                .usePlaintext()
                .build();

        // 2. 스텁 생성 (Blocking 방식 예시)
        GreeterGrpc.GreeterBlockingStub stub = GreeterGrpc.newBlockingStub(channel);

        // 3. RPC 호출
        HelloRequest request = HelloRequest.newBuilder().setName("Java User").build();
        HelloResponse response = stub.sayHello(request);

        System.out.println("Response: " + response.getMessage());
        channel.shutdown();
    }
}
```