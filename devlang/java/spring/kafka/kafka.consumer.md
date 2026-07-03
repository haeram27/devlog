# Kafka Consumer와 Consumer Group

## 1) Kafka Consumer란?

**Kafka Consumer**는 Kafka Topic의 메시지를 읽어 처리하는 애플리케이션(또는 인스턴스)이다.

- Producer가 기록한 메시지를 Partition Leader에서 읽는다.
- 읽은 위치는 `offset`으로 식별된다.
- Consumer는 보통 poll 루프로 메시지를 가져와 비즈니스 로직을 수행한다.

즉, Consumer는 Kafka의 "읽기/처리" 주체다.

---

## 2) Consumer Group이란?

**Consumer Group**은 같은 `group.id`를 사용하는 Consumer들의 집합이다.

- 같은 Group 내부에서는 Topic의 Partition을 나눠서 병렬 처리한다.
- 같은 Group 내부에서는 **한 Partition을 동시에 한 Consumer만** 담당한다.
- Group이 다르면 같은 Topic 메시지를 각각 독립적으로 소비할 수 있다.

이 구조 때문에 Kafka는 "확장(스케일 아웃)"과 "기능 분리(여러 그룹 독립 소비)"를 동시에 지원한다.

---

## 3) Consumer Group과 Offset의 관계 (핵심)

Kafka에서 offset은 **Consumer 개인이 아니라 Consumer Group 기준**으로 관리된다.

- 같은 Topic/Partition이라도 Group이 다르면 offset도 다르다.
- Group 내부에서 Consumer가 바뀌어도(재시작/장애/리밸런싱) Group offset을 이어받아 계속 처리한다.
- 따라서 "어디까지 읽었는가"는 `group.id + topic + partition` 조합으로 결정된다.

예시:

```text
Topic orders, Partition 0

Group billing  -> offset 1200
Group notify   -> offset 980

같은 메시지 스트림을 읽지만,
각 Group은 서로 다른 진도(offset)를 가진다.
```

이게 중요한 이유:

1. **독립 배포/장애 복구**
   - `billing` 그룹 문제로 지연돼도 `notify` 그룹은 영향 없이 진행 가능
2. **독립 재처리**
   - 특정 Group만 offset rewind(오프셋 되감기)해서 재처리 가능
3. **역할 분리**
   - 같은 이벤트를 정산/알림/분석이 각자 다른 속도로 처리 가능

---

## 4) 파티션 수와 Group 내 Consumer 수 관계

같은 Group 기준 기본 규칙:

- Consumer 수 < Partition 수: 일부 Consumer가 여러 Partition 담당
- Consumer 수 = Partition 수: 보통 가장 균등한 병렬 처리
- Consumer 수 > Partition 수: 남는 Consumer는 idle

즉 Group 내 처리 병렬성의 상한은 Partition 수다.

---

## 5) 리밸런싱과 실무 포인트

Group 구성원이 변하면(증설/축소/장애) 파티션 재할당이 일어나는데, 이를 **리밸런싱**이라 한다.

- 리밸런싱 동안 일시적 처리 지연이 생길 수 있다.
- 안정적인 운영을 위해 Consumer 증감 빈도와 배포 전략을 고려해야 한다.
- offset commit 전략(자동/수동, 커밋 시점)은 중복 처리/유실 위험과 직접 연결된다.

---

## 6) 한눈에 요약

- Consumer: 메시지를 읽고 처리하는 실행 주체
- Consumer Group: Consumer들의 협업 단위(파티션 분담)
- Offset: **Group 단위 소비 진도**
- 핵심 관계: 같은 Topic이라도 Group이 다르면 offset도 다르고, 처리 상태도 완전히 독립적이다.

---

## 7) 하나의 Topic(다중 Partition)에서 Group/Consumer 배치 전략

예를 들어 Topic이 12 partitions라면, 전략의 핵심은 아래 두 가지다.

1. **Consumer Group을 기능(업무 책임) 단위로 분리**
2. **각 Group 내부 Consumer 수를 파티션 수와 처리 특성에 맞춰 배치**

### (1) Group 배치 전략: "기능별 Group 분리"

같은 Topic을 여러 기능이 소비한다면 Group을 분리한다.

- `orders-billing-group` (정산)
- `orders-notify-group` (알림)
- `orders-analytics-group` (분석)

이유:

- Group마다 offset이 독립이므로 장애/지연/재처리 영향이 분리된다.
- 한 기능에서 rewind가 필요해도 다른 기능 처리에는 영향이 없다.

### (2) Group 내부 Consumer 수 배치 전략

각 Group에서 Consumer 개수는 아래 순서로 정한다.

1. **상한 확인**: 같은 Group의 병렬 처리 상한은 파티션 수
2. **처리 특성 확인**:
   - CPU 바운드면 코어/처리시간 기준으로 점진 확장
   - I/O 바운드면 외부 API/DB 병목을 먼저 확인
3. **초기 배치**: 보통 `Consumer 수 <= Partition 수`로 시작
4. **모니터링 기반 조정**: lag, 처리시간, 실패율 보고 증감

실무 기본값:

- 꼭 필요한 만큼만 Consumer를 늘리고,
- 파티션 수보다 과도하게 큰 Consumer 수는 피한다(유휴 인스턴스 증가).

### (3) 균형 잡힌 예시 (Topic: 12 partitions)

- Billing Group: 6 consumers (무거운 처리지만 트래픽 중간)
- Notify Group: 3 consumers (가벼운 처리)
- Analytics Group: 12 consumers (대량 병렬 집계)

이때 각 Group은 같은 Topic을 읽어도 offset이 다르므로 서로 완전히 독립 운영된다.

### (4) 확장/운영 시 주의점

- Consumer 증설/축소 시 리밸런싱이 발생하므로 빈번한 스케일 변동은 지양
- 키 분배가 치우치면 특정 파티션만 lag가 쌓일 수 있음(핫 파티션)
- 처리량 부족을 Consumer 수만으로 해결하지 말고, 필요 시 파티션 증설도 함께 검토

### (5) 빠른 의사결정 규칙

- "기능이 다르다" → Group 분리
- "같은 기능 처리량이 부족하다" → 해당 Group Consumer 증설(파티션 수 이하 우선)
- "증설했는데도 느리다" → 외부 병목 또는 파티션/키 분배 문제 점검
