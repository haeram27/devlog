# RDB row generator

## FOR문 이용한 ROW 생성

- main table에 PRIMARY KEY row_id가 있고 sub table에서 해당 키를 참조 하는 구조
- main table과 main의 PRIMARY KEY를 참조하는 sub table에 모두 row를 생성

```sql
DO $$
BEGIN
    -- generates 100 rows
    FOR i IN 1..100 LOOP
        WITH insert_main_table AS (
            INSERT INTO
                tb_main (
                    name
                    , description
                    , action
                    , auto_expiration
                    , expiration_time
                    , auto_remove
                    , admin_id
                    , is_use
                    , is_removed
                    , created_time
                    , modified_time)
            VALUES (
                    'test' || i    -- '||' is conacat string operator
                    , 'description ' || i
                    , 'block'::en_block_ip_rule_action
                    , false
                    , '2026-09-04 07:00:00.000'::timestamp
                    , false
                    , 'admin.id'
                    , true
                    , false
                    , NOW()
                    , NOW())
            RETURNING row_id
        )
        INSERT INTO
            tb_sub (
                row_id
                , data
                , modified_time)
        SELECT
            row_id
            , '[{"subdata":"hello"}]'::json
            , NOW()
        FROM
            insert_main_table;
    END LOOP;
END $$;
```
