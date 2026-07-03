# Kafka Partition과 복제(Replication) 구조

## 1) Partition이란?

Kafka에서 **Partition**은 하나의 Topic을 여러 조각으로 나눈 **로그 단위 저장소**다.

- Topic은 논리 이름이고, 실제 데이터는 Partition에 순서대로 append 된다.
- 각 Partition 내부 메시지는 `offset`(0, 1, 2...)으로 식별된다.
- **순서 보장 범위는 Partition 내부**로 한정된다. (Topic 전체 순서 보장은 아님)
- Partition을 늘리면 Producer/Consumer가 병렬 처리할 수 있어 처리량(throughput)이 증가한다.

즉, Partition은 Kafka의 확장성과 성능을 만드는 핵심 단위다.

---

## 2) 메시지는 어떤 Partition으로 들어가나?

Producer는 아래 방식으로 Partition을 선택한다.

1. 메시지에 partition 번호를 직접 지정
2. key를 기반으로 해시 계산 (`hash(key) % partitionCount`)
3. key가 없으면 기본 분배 전략(라운드로빈/스티키 등)

실무에서는 **같은 key는 같은 Partition**으로 보내 순서 일관성을 유지하는 패턴을 가장 많이 쓴다.

---

## 3) 복제(Replication)란?

Kafka는 장애 허용을 위해 Partition을 여러 Broker에 복제한다.

- `replication.factor = 3`이면, 하나의 Partition 사본이 총 3개 생긴다.
- 이 중 1개는 **Leader**, 나머지는 **Follower**다.
- Producer/Consumer의 읽기/쓰기는 기본적으로 Leader를 통해 수행된다.
- Follower는 Leader의 로그를 복제하여 동일 상태를 맞춘다.

---

## 4) 복제 구성의 구조

복제 구조는 보통 아래 개념으로 설명한다.

### (1) Leader / Follower

- **Leader Replica**: 클라이언트 요청 처리(쓰기/읽기)
- **Follower Replica**: Leader 로그를 추적 복제

### (2) ISR (In-Sync Replicas)

- Leader와 충분히 동기화된 Replica 집합
- Leader 자신도 ISR에 포함
- 장애 시 **새 Leader는 ISR 중에서 선출**된다.

### (3) AR (Assigned Replicas)

- 해당 Partition에 할당된 전체 Replica 목록
- ISR은 AR의 부분집합이다.

예시:

- Partition `topicA-0`
- AR: `[Broker1, Broker2, Broker3]`
- Leader: `Broker1`
- ISR: `[Broker1, Broker2]` (Broker3는 지연으로 ISR 제외 가능)

---

## 5) 내구성 관련 핵심 설정

복제 구조의 안정성은 Producer acks / min.insync.replicas 조합으로 결정된다.

- `acks=all`: ISR의 확인을 받아야 쓰기 성공
- `min.insync.replicas=2` + `replication.factor=3`:
  - ISR이 2개 미만이면 쓰기 실패시켜 데이터 손실 위험을 줄임

권장 기본 조합(중요 데이터):

- Topic: `replication.factor=3`
- Producer: `acks=all`, `enable.idempotence=true`
- Broker/Topic: `min.insync.replicas=2`

---

## 6) 한눈에 보는 구조 요약

```text
Topic
 ├─ Partition 0: Leader(B1), Follower(B2, B3)
 ├─ Partition 1: Leader(B2), Follower(B1, B3)
 └─ Partition 2: Leader(B3), Follower(B1, B2)
```

- Partition: 병렬 처리/확장 단위
- Replication: 장애 대응/내구성 단위
- Leader-ISR 기반으로 쓰기 안정성과 가용성을 균형 있게 유지한다.

---

## 7) Partition · Consumer Group · Consumer 연결 관계 기본 원칙

이 세 가지의 관계를 한 문장으로 요약하면 다음과 같다.

> **Topic의 파티션을, Consumer Group이 나눠 맡고, Group 내부 Consumer 인스턴스가 실제로 처리한다.**

핵심은 "파티션 ↔ 컨슈머 1:1"이 아니라, **"같은 Group 내부에서 파티션 소유권이 배분"**된다는 점이다.

### (1) 가장 중요한 규칙: Group 내부의 독점 소비

- 같은 Consumer Group 안에서는 **하나의 Partition을 동시에 하나의 Consumer만** 읽는다.
- 따라서 Group 내부에서는 같은 메시지를 중복 처리하지 않는다. (정상 동작 기준)
- 이 특성 때문에 Group은 "확장(Scale-out) 단위"가 된다.

예시:

```text
Topic A (Partition 3개: P0, P1, P2)
Group G1 (Consumer 2개: C1, C2)

C1 -> P0, P1
C2 -> P2
```

### (2) Group이 달라지면 같은 메시지를 각각 소비

- Consumer Group이 다르면 offset도 별도 관리된다.
- 즉 같은 Topic 메시지를 여러 Group이 **각자 독립적으로** 읽을 수 있다.
- 그래서 "업무 기능별 소비"(예: 정산/알림/분석)를 구현할 때 Group을 분리한다.

예시:

```text
Topic A
 ├─ Group Billing  -> Topic A 전체를 Billing 관점으로 소비
 └─ Group Notify   -> Topic A 전체를 Notify 관점으로 소비
```

### (3) 파티션 수와 Consumer 수의 관계

같은 Group 기준으로 아래 규칙이 성립한다.

1. **Consumer < Partition**
   - 일부 Consumer가 여러 Partition을 맡는다.
2. **Consumer = Partition**
   - 이상적인 1:1 배분 가능 (대칭적 처리)
3. **Consumer > Partition**
   - 남는 Consumer는 유휴(idle) 상태가 된다.

즉, 같은 Group의 병렬 처리 상한은 본질적으로 **파티션 수**다.

### (4) 리밸런싱(Rebalancing)과 연결 관계 변화

- Group에 Consumer가 추가/제거되거나 장애가 나면 파티션 재할당이 발생한다.
- 이 과정을 리밸런싱이라 하며, 순간적으로 처리 지연이 생길 수 있다.
- 운영 관점에서는 "컨슈머 증감 빈도"와 "리밸런싱 비용"을 함께 고려해야 한다.

### (5) 순서 보장과 연결 관계의 의미

- 순서는 **파티션 단위로만** 보장된다.
- 같은 key를 같은 파티션으로 보내면, 그 key의 이벤트 순서를 보존하기 쉽다.
- 반대로 key 분배가 불균형하면 특정 Consumer에 부하가 몰릴 수 있다.

### (6) 흔한 오해 정리

- 오해 1: "파티션마다 Group을 하나씩 만든다"
  - 실제로는 일반적으로 **업무 단위로 Group을 나누고**, Group 내부에서 파티션을 분배한다.
- 오해 2: "Consumer를 늘리면 무조건 처리량이 오른다"
  - 같은 Group에서는 파티션 수를 초과하면 더 이상 병렬성이 늘지 않는다.
- 오해 3: "Topic 전체 순서가 보장된다"
  - 순서 보장은 Topic 전체가 아니라 Partition 내부 한정이다.

### (7) 관계를 한 번에 보는 그림

```text
Topic Orders (P0, P1, P2, P3)

Group A (정산)
  - Consumer A1 -> P0, P1
  - Consumer A2 -> P2, P3

Group B (알림)
  - Consumer B1 -> P0, P1, P2, P3

※ Group A와 Group B는 같은 메시지를 각각 독립 소비
※ Group 내부에서는 각 Partition을 한 Consumer만 담당
```
