# spring batch - purge metadata in db

## 요약

- spring batch는 실행을 위해서 RDB에 다음 6개의 meta data table을 생성
  - batch_job_instance (key: job_instance_id)
    - batch_job_execution (key: job_execution_id)
      - batch_job_execution_params
      - batch_job_execution_context
      - batch_step_execution (key: step_execution_id)
        - batch_step_execution_context

- spring batch는 job을 생성하고 실행할 때마다 job과 step 메타 정보 데이터를 table에 누적한다.
- 별도의 삭제 처리를 하지 않으면 데이터가 무한히 누적 되므로, spring batch를 RDB를 연동하여 사용하고 있다면 반드시 더이상 보관이 필요하지 않은 메타 데이터를 주기적으로 삭제 처리해야 한다.

## meta data 삭제 방법

- spring batch 메타 데이터를 삭제하려면 DELETE 또는 TRUNCATE 문을 사용해야 한다.

### 제약 조건

- batch_job_instance로 부터 각 하위 테이블은 상위 테이블의 primary 키를 외래키로 참조하고 있다. 외래키 참조 구조의 테이블에서 특정 튜플을 삭제하는 방법은 다음 세가지 이다.
  - 외래키 참조 체인의 가장 최하위 테이블 부터 최상위 테이블 순서로 차례로 관련 외래키의 데이터를 삭제 (추천)
  - `DELETE CASCADE` 구문 사용. 단, 이 경우 테이블을 생성할 때 각 table에 별도로 `ON DELETE CASCADE` 제약조건을 명시해야 함.
  - `TRUNCATE CASCADE` 구문 사용. `TRUNCATE`의 경우 `CASCADE` 사용시 별도로 table의 제약조건이 필요하진 않지만 테이블을 완전히 비울 수만 있고, 일부 데이터만 삭제 하는 것은 불가능  
- spring은 metadata용 테이블을 생성할 때 각 table에 별도로 `ON DELETE CASCADE` 제약조건을 명시하지 않는다. 그러므로 특정 tuple을 삭제할 때 `DELETE CASCADE` 문법을 사용할 수 없다.

## sql - purge batch job metadata (mybatis)

- retentionInterval day 기간 이전에 실행이 완료된 job의 metadata를 삭제한다.
- 삭제 방식은 외래키 참조 체인의 역순으로 조건에 맞는 table내 데이터들을 삭제함 (step -> execution -> jab table 순서)
- 삭제 로직
  - 삭제 대상이 되는 `job_execution_id`를 구함 (target_job_execution_ids)
  - `target_job_execution_ids`를 이용하여 삭제 후 살아남게 될 `job_instance_id`를 구함 (survivor_job_instance_ids)
  - `target_job_execution_ids`에 연관된 step_execution_id를 구함 (target_step_execution_ids)
  - `target_step_execution_ids`를 이용하여 step table을 삭제 (del_sec, del_se)
  - `target_job_execution_ids`를 이용하여 execution table을 삭제 (del_jec, del_jep, del_je)
  - 마지막으로 `batch_job_instance` table에서 survivor가 아닌 모든 tuple 삭제

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

@Mapper
public interface BatchJobExecutionDao {
    void purgeBatchMeta(@Param("retentionInterval") Integer interval);
}
```

```xml
<update id="purgeBatchMeta" parameterType="int">
    WITH
    target_job_execution_ids AS (
        SELECT je.job_execution_id, je.job_instance_id
        FROM batch_job_execution je
        WHERE je.end_time <![CDATA[<]]> (NOW() - (INTERVAL '1 day' * #{retentionInterval}))
    ),
    survivor_job_instance_ids AS (
        SELECT DISTINCT je.job_instance_id
        FROM batch_job_execution je
        LEFT JOIN target_job_execution_ids t
        ON t.job_execution_id = je.job_execution_id
        WHERE t.job_execution_id IS NULL
    ),
    target_step_execution_ids AS (
        SELECT se.step_execution_id, se.job_execution_id
        FROM batch_step_execution se
        JOIN target_job_execution_ids t ON t.job_execution_id = se.job_execution_id
    ),
    del_sec AS (
        DELETE FROM batch_step_execution_context sec
        USING target_step_execution_ids t
        WHERE sec.step_execution_id = t.step_execution_id
    ),
    del_se AS (
        DELETE FROM batch_step_execution se
        USING target_step_execution_ids t
        WHERE se.step_execution_id = t.step_execution_id
    ),
    del_jec AS (
        DELETE FROM batch_job_execution_context jec
        USING target_job_execution_ids t
        WHERE jec.job_execution_id = t.job_execution_id
    ),
    del_jep AS (
        DELETE FROM batch_job_execution_params jep
        USING target_job_execution_ids t
        WHERE jep.job_execution_id = t.job_execution_id
    ),
    del_je AS (
        DELETE FROM batch_job_execution je
        USING target_job_execution_ids t
        WHERE je.job_execution_id = t.job_execution_id
    )
    DELETE FROM batch_job_instance ji
    WHERE NOT EXISTS (
        SELECT 1 FROM survivor_job_instance_ids s
        WHERE s.job_instance_id = ji.job_instance_id
    )
</update>
```