# JobDetail

## durability와 recovery

`Spring Batch`는 내부적으로 Quartz를 사용할 수 있으며, 이때 `Quartz`의 `JobDetail` 설정 중 `durability`와 `requestRecovery`는 **Job의 생명주기와 장애 복구에 관련된 중요한 속성**입니다. 아래에 상세히 설명드릴게요.

---

### 종합 요약

| 속성                | 기본값   | 설명                             |
| ----------------- | ----- | ------------------------------ |
| `durability`      | false | Trigger 없이도 JobDetail을 유지할지 여부 |
| `requestRecovery` | false | 스케줄러 장애 후 Job 복구 실행 여부         |

---

### 1. `durability` (내구성)

* `JobDetail`이 **Trigger와 연결되지 않아도** 스케줄러에 **등록된 상태로 남아있을지를 결정**합니다.
특정 Job을 **프로그래밍적으로 수동 실행**하거나, **트리거를 나중에 등록할 예정**이라면 `durability = true`로 설정해야 합니다.

* true (내구성 있음)
  * 트리거 없이도 `JobDetail`이 스케줄러에 남아있습니다.
  * 수동 트리거나 다른 트리거와 나중에 연결 가능.

* false (기본값)
  * `Trigger`가 연결되어 있지 않으면 해당 `JobDetail`은 삭제됩니다.

---

### 2. `requestRecovery` (복구 요청)

* 만약 스케줄러가 **비정상 종료**되었고, 그 순간 실행 중이던 Job이 있었다면, 스케줄러가 재시작될 때 그 Job을 **자동 복구 실행할지를 결정**합니다.
실행 중 Job이 **중요한 작업**이고, 중단 시 **반드시 다시 실행되어야** 한다면 `requestRecovery = true`가 권장됩니다.

* true (복구 요청 O)
  * 이전 실행 중이던 Job을 **복구해서 재실행**합니다.
* false (기본값)
  * 비정상 종료 중 실행된 Job은 **복구되지 않음**.

---

### 실무 팁

Spring Batch에서 Quartz를 사용할 경우 보통 다음과 같이 설정하는 것이 안전합니다:

```java
JobDetailFactoryBean jobDetailFactory = new JobDetailFactoryBean();
jobDetailFactory.setDurability(true);
jobDetailFactory.setRequestRecovery(true);
```

이렇게 설정하면:

* Job은 트리거가 없더라도 사라지지 않고,
* 시스템 장애 시 실행 중이던 Job이 복구 실행됩니다.
