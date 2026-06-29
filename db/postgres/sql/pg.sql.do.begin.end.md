# DO BEGIN END

SQL 블럭을 하나의 Transaction에서 실행

```sql
DO $$
BEGIN
    CREATE TABLE IF NOT EXISTS tb_server_alert_log_obj_id ();
    COMMENT ON TABLE tb_server_alert_log_obj_id IS '서버 이벤트 로그  obj_id 관리';

    ALTER TABLE IF EXISTS tb_server_alert_log_obj_id ADD COLUMN IF NOT EXISTS obj_id text;
    ALTER TABLE IF EXISTS tb_server_alert_log_obj_id ADD COLUMN IF NOT EXISTS row_number bigint;

    COMMENT ON COLUMN tb_server_alert_log_obj_id.obj_id IS 'MongoDB object ID';
    COMMENT ON COLUMN tb_server_alert_log_obj_id.row_number IS '순서';

    IF NOT EXISTS (SELECT 1 FROM information_schema.table_constraints WHERE table_name = 'tb_server_alert_log_obj_id' AND constraint_type = 'PRIMARY KEY') THEN
        ALTER TABLE tb_server_alert_log_obj_id ADD CONSTRAINT tb_server_alert_log_obj_id_pkey PRIMARY KEY (obj_id);
    END IF;

END $$ LANGUAGE 'plpgsql';
```