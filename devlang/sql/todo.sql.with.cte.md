# WITH CTE(Common Table Expression)

* WITH CTE(Common Table Expression)는 단일 SQL 문(statement) 에서만 유효하다. 달리 말하면 CTE가 사용되는 단일 SQL statement 이다.
* WITH CTE는 되도록 조회(SELECT) 용도로만 사용한다.
  * WITH CTE로 작성된 SELECT 문에 의해 생성된 임시 table은 연결된 다음 CTE나 최종 statement에서 참조 가능하다.
  * WITH CTE에 작성된 DATA 변경 지시자(INSERT/UPDATE/DELETE) 의한 실제 변경 사항은 연결된 다음 CTE나 최종 statement에서 참조가 불가능하다.
    * 단일 sql 문에서만 사용되므로 DATA 변경(update)는 DB가 statement를 처리하는 과정 중 하나의 페이즈에서 전부 처리되며, 이 때문에 WITH CTE로 데이터 변경문을 작성한다고 해도 연결된 다음 CTE나 최종 SQL문에서 

## WITH CTE문으로 생성된 읽기와 쓰기의 

WITH CTE에서 SELECT 문으로 읽을 수 있는 table 데이터는 트랜잭션 시작시 캡쳐된 스냅샷 상태의 내용이며 상위 CTE에서 사용한 데이터 변경 SQL문(INSERT/UPDATE/DELETE)이 적용된 table의 상태를 하위 CTE나 최종 SQL 문에서 참조(FROM 등)할 수 없다.

## MVCC (Multi Version Concurrency Control)

* RDB 제품(postgres 등)에서 트랜잭션 변경 사항에 대한 동시성 문제를 해결하기 위해서 사용하는 방식
* git의 동시 변경 문제에 대한 해결책과 동일한 방식으로 "first-commiter-win" 방식이라고 할 수 있다.
* 공유자원(data 값)에 대한 read/write lock을 사용하지 않는다.
* "first-commiter-win" 방식: 2개 이상의 transaction에서 동일한 데이터를 수정하면 먼저 commit된 트랜잭션이 적용되고 충돌이 발생한 다른 transaction은 에러와 함께 rollback 된다.
  * rdb는 트랜잭션 단위로 변경사항 적용 여부를 결정한다.
  * 트랜잭션의 최소 단위는 단일 sql statement이다.

### MVCC 방식에서 트랜잭션의 가시성 문제

* RDB는 트랜잭션을 시작할 때 SQL 문에서 참조되는 table 들의 스냅샷을 생성하고 트랜잭션 종료 시까지 스냅샷을 유지한다.
* RDB는 트랜잭션 내 SQL 처리을 할 때 데이터에 대해 READ(SELECT)는 실제 디스크가 아닌 스냅샷으로 부터 읽으며 변경(INSERT/UPDATE/DELETE) 사항은 따로 UP

## batch job clean

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
