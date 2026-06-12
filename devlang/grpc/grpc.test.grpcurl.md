# grpcurl

- grpcurl을 사용하면 cli에서 grpc 호출을 할 수 있다.
- 

## 예제

```bash
grpcurl -plaintext \
  -import-path api/src/main/proto \
  -proto path/to/order_service.proto \
  -H "x-trace-id: springboot-test-id-001" \
  -H "authorization: Bearer jwt_token_here" \
  -d '{"item_id": "PROD-100", "amount": 50000}' \
  localhost:9090 \
  com.example.grpc.proto.OrderService/CreateOrder
```

###

gRPC는 HTTP/2 기반의 이진(Binary) 프로토콜을 사용합니다.
REST API와 달리 grpcurl은 포트 번호만 가지고는 서버에 어떤 서비스나 메서드, 필드들이 존재하는지 전혀 알지 못합니다.

`-import-path`와 `-proto` 옵션은 서버 측에 리플렉션(Reflection) 설정이 되어 있지 않을 때, 클라이언트(grpcurl)가 API 스펙을 직접 파싱할 수 있도록 소스 코드 파일의 위치를 알려주는 옵션입니다.

- -import-path (또는 축약형 -I)
  - 값 : 디렉토리 path
  - 역할: .proto 파일 내부에서 다른 proto 파일을 가져올 때(import) 기준이 되는 루트 디렉터리(폴더) 경로를 지정합니다.
  - 프로토콜 버퍼 파일 안에서 import "google/protobuf/timestamp.proto";나 import "console/s3/v1/common.proto";와 같은 구문을 만났을 때, grpcurl은 지정된 -import-path 폴더를 기준으로 해당 파일들을 찾기 시작합니다.

- -proto
  - 값: `.proto` 파일 path
  - 역할: .내가 호출하려는 서비스와 메서드가 정의되어 있는 실제 .proto 파일의 경로를 지정합니다.
  - 일반적으로 -import-path로 지정한 루트 디렉터리를 기준으로 한 상대 경로를 입력합니다.


## 서버 리플렉션 활성화 (application.yml)

서버 리플렉션을 활성화 하면 

`-import-path`, `-proto` 

```yaml
spring:
  grpc:
    server:
      reflection:
        enabled: true
```