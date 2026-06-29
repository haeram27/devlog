# grpc protobuf keyword

Protobuf(특히 proto3) 문법에서 사용되는 주요 키워드와 식별자 목록입니다. [Language Specification](https://protobuf.com/docs/language-spec)에 따르면 약 43개의 키워드가 정의되어 있습니다.

## 1. 주요 예약 키워드 (Top-level & Structural)

구조 정의 및 파일 설정에 필수적인 키워드들입니다.

- syntax: .proto 파일의 버전을 지정합니다 (예: syntax = "proto3";).
- package: 메시지 타입 간의 이름 충돌을 방지하기 위한 네임스페이스를 정의합니다.
- import: 다른 .proto 파일의 정의를 가져올 때 사용합니다 (weak, public 옵션과 함께 사용 가능).
- option: 컴파일러나 런타임에 영향을 주는 설정을 정의합니다 (예: java_package).
- message: 데이터를 담는 기본 구조체를 정의합니다.
- enum: 열거형 타입을 정의합니다.
- service: RPC(Remote Procedure Call) 인터페이스를 정의합니다.
- rpc: 서비스 내의 메서드를 정의합니다.
- returns: RPC 메서드의 응답 타입을 지정합니다.

## 2. 필드 규칙 및 제어 키워드

메시지 내부 필드의 특성을 결정합니다.

- repeated: 해당 필드가 0개 이상의 값을 가질 수 있는 배열(List)임을 나타냅니다.
- optional: 필드 값의 존재 여부를 명시적으로 확인할 수 있게 합니다 (proto3에서 부활).
- oneof: 여러 필드 중 단 하나만 값을 가질 수 있는 그룹을 생성합니다.
- map: Key-Value 쌍의 데이터를 정의합니다.
- reserved: 삭제된 필드 번호나 이름을 예약하여 재사용을 방지합니다.
- extensions: (주로 proto2) 메시지 확장 지점을 지정합니다.

## 3. 데이터 타입 키워드 (Scalar Types)

각 언어의 기본 자료형으로 변환되는 키워드들입니다.

- 정수형: int32, int64, uint32, uint64, sint32, sint64, fixed32, fixed64, sfixed32, sfixed64
- 부동소수점: float, double
- 기타: bool, string, bytes

## 4. 기타 특수 식별자
문법 명세에 포함된 제어용 단어들입니다.

- to / max: reserved 2 to 10; 또는 reserved 10 to max;와 같이 범위를 지정할 때 사용합니다.
- stream: gRPC 스트리밍 통신을 정의할 때 사용합니다.
- inf / nan: 부동소수점 리터럴 값을 표현할 때 사용합니다.
- true / false: 불리언(bool) 값의 리터럴입니다.

주의사항: [Proto Best Practices](https://protobuf.dev/best-practices/dos-donts/)에 따르면 필드 이름으로 Java나 Python 같은 대상 언어의 키워드(예: class, data, import)를 사용하는 것은 피해야 합니다. 컴파일 시 이름이 강제로 변경되거나 접근 시 오류가 발생할 수 있기 때문입니다. [17, 18] 
더 구체적인 문법 규칙(EBNF)이나 특정 플랫폼별 매핑 테이블이 필요하신가요?

- [https://protobuf.com](https://protobuf.com/docs/language-spec)
- [https://www.tutorialspoint.com](https://www.tutorialspoint.com/protobuf/protobuf_quick_guide.htm)
- [https://victoriametrics.com](https://victoriametrics.com/blog/go-protobuf-basic/)
- [https://protobuf.dev](https://protobuf.dev/best-practices/dos-donts/)