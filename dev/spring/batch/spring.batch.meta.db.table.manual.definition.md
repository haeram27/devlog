
# spring batch meta db

```sql
-----------------------------------------------------------------------------------
--- Spring Batch meta table > Batch
--- https://docs.spring.io/spring-batch/reference/schema-appendix.html
--- https://github.com/spring-projects/spring-batch/blob/main/spring-batch-core/src/main/resources/org/springframework/batch/core/schema-drop-postgresql.sql
--- https://github.com/spring-projects/spring-batch/blob/main/spring-batch-core/src/main/resources/org/springframework/batch/core/schema-postgresql.sql
-----------------------------------------------------------------------------------
DROP TABLE IF EXISTS BATCH_JOB_INSTANCE CASCADE;
DROP TABLE IF EXISTS BATCH_JOB_EXECUTION CASCADE;
DROP TABLE IF EXISTS BATCH_JOB_EXECUTION_PARAMS CASCADE;
DROP TABLE IF EXISTS BATCH_STEP_EXECUTION CASCADE;
DROP TABLE IF EXISTS BATCH_STEP_EXECUTION_CONTEXT CASCADE;
DROP TABLE IF EXISTS BATCH_JOB_EXECUTION_CONTEXT CASCADE;

DROP SEQUENCE BATCH_STEP_EXECUTION_SEQ;
DROP SEQUENCE BATCH_JOB_EXECUTION_SEQ;
DROP SEQUENCE BATCH_JOB_SEQ;

CREATE TABLE BATCH_JOB_INSTANCE  (
    JOB_INSTANCE_ID BIGINT  NOT NULL PRIMARY KEY ,
    VERSION BIGINT ,
    JOB_NAME VARCHAR(100) NOT NULL,
    JOB_KEY VARCHAR(32) NOT NULL,
    constraint JOB_INST_UN unique (JOB_NAME, JOB_KEY)
) ;

CREATE TABLE BATCH_JOB_EXECUTION  (
    JOB_EXECUTION_ID BIGINT  NOT NULL PRIMARY KEY ,
    VERSION BIGINT  ,
    JOB_INSTANCE_ID BIGINT NOT NULL,
    CREATE_TIME TIMESTAMP NOT NULL,
    START_TIME TIMESTAMP DEFAULT NULL ,
    END_TIME TIMESTAMP DEFAULT NULL ,
    STATUS VARCHAR(10) ,
    EXIT_CODE VARCHAR(2500) ,
    EXIT_MESSAGE VARCHAR(2500) ,
    LAST_UPDATED TIMESTAMP,
    constraint JOB_INST_EXEC_FK foreign key (JOB_INSTANCE_ID)
    references BATCH_JOB_INSTANCE(JOB_INSTANCE_ID)
) ;

CREATE TABLE BATCH_JOB_EXECUTION_PARAMS  (
    JOB_EXECUTION_ID BIGINT NOT NULL ,
    PARAMETER_NAME VARCHAR(100) NOT NULL ,
    PARAMETER_TYPE VARCHAR(100) NOT NULL ,
    PARAMETER_VALUE VARCHAR(2500) ,
    IDENTIFYING CHAR(1) NOT NULL ,
    constraint JOB_EXEC_PARAMS_FK foreign key (JOB_EXECUTION_ID)
    references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
) ;

CREATE TABLE BATCH_STEP_EXECUTION  (
    STEP_EXECUTION_ID BIGINT  NOT NULL PRIMARY KEY ,
    VERSION BIGINT NOT NULL,
    STEP_NAME VARCHAR(100) NOT NULL,
    JOB_EXECUTION_ID BIGINT NOT NULL,
    CREATE_TIME TIMESTAMP NOT NULL,
    START_TIME TIMESTAMP DEFAULT NULL ,
    END_TIME TIMESTAMP DEFAULT NULL ,
    STATUS VARCHAR(10) ,
    COMMIT_COUNT BIGINT ,
    READ_COUNT BIGINT ,
    FILTER_COUNT BIGINT ,
    WRITE_COUNT BIGINT ,
    READ_SKIP_COUNT BIGINT ,
    WRITE_SKIP_COUNT BIGINT ,
    PROCESS_SKIP_COUNT BIGINT ,
    ROLLBACK_COUNT BIGINT ,
    EXIT_CODE VARCHAR(2500) ,
    EXIT_MESSAGE VARCHAR(2500) ,
    LAST_UPDATED TIMESTAMP,
    constraint JOB_EXEC_STEP_FK foreign key (JOB_EXECUTION_ID)
    references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
) ;

CREATE TABLE BATCH_STEP_EXECUTION_CONTEXT  (
    STEP_EXECUTION_ID BIGINT NOT NULL PRIMARY KEY,
    SHORT_CONTEXT VARCHAR(2500) NOT NULL,
    SERIALIZED_CONTEXT TEXT ,
    constraint STEP_EXEC_CTX_FK foreign key (STEP_EXECUTION_ID)
    references BATCH_STEP_EXECUTION(STEP_EXECUTION_ID)
) ;

CREATE TABLE BATCH_JOB_EXECUTION_CONTEXT  (
    JOB_EXECUTION_ID BIGINT NOT NULL PRIMARY KEY,
    SHORT_CONTEXT VARCHAR(2500) NOT NULL,
    SERIALIZED_CONTEXT TEXT ,
    constraint JOB_EXEC_CTX_FK foreign key (JOB_EXECUTION_ID)
    references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
) ;

CREATE SEQUENCE BATCH_STEP_EXECUTION_SEQ MAXVALUE 9223372036854775807 CYCLE;
CREATE SEQUENCE BATCH_JOB_EXECUTION_SEQ MAXVALUE 9223372036854775807 CYCLE;
CREATE SEQUENCE BATCH_JOB_SEQ MAXVALUE 9223372036854775807 CYCLE;

CREATE INDEX IF NOT EXISTS job_instance_idx_1 ON batch_job_instance(job_name, job_key);
CREATE INDEX IF NOT EXISTS job_execution_idx_1 ON batch_job_execution(job_instance_id);
CREATE INDEX IF NOT EXISTS job_execution_params_idx_1 ON batch_job_execution_params(job_execution_id);
CREATE INDEX IF NOT EXISTS step_execution_idx_1 ON batch_step_execution(job_execution_id);
CREATE INDEX IF NOT EXISTS step_execution_idx_2 ON batch_step_execution(step_execution_id, version);
CREATE INDEX IF NOT EXISTS step_execution_context_idx_1 ON batch_step_execution_context(step_execution_id);

-----------------------------------------------------------------------------------
--- Spring Quartz-scheduler meta table > Quartz
--- https://github.com/quartz-scheduler/quartz/blob/main/quartz/src/main/resources/org/quartz/impl/jdbcjobstore/tables_postgres.sql
-----------------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS qrtz_calendars (
    sched_name varchar(120) NOT NULL,
    calendar_name varchar(200) NOT NULL,
    calendar bytea NOT NULL,
    CONSTRAINT qrtz_calendars_pkey PRIMARY KEY (sched_name, calendar_name)
);

CREATE TABLE IF NOT EXISTS qrtz_fired_triggers (
    sched_name varchar(120) NOT NULL,
    entry_id varchar(95) NOT NULL,
    trigger_name varchar(200) NOT NULL,
    trigger_group varchar(200) NOT NULL,
    instance_name varchar(200) NOT NULL,
    fired_time int8 NOT NULL,
    sched_time int8 NOT NULL,
    priority int4 NOT NULL,
    state varchar(16) NOT NULL,
    job_name varchar(200) NULL,
    job_group varchar(200) NULL,
    is_nonconcurrent bool NULL,
    requests_recovery bool NULL,
    CONSTRAINT qrtz_fired_triggers_pkey PRIMARY KEY (sched_name, entry_id)
);

CREATE TABLE IF NOT EXISTS qrtz_job_details (
    sched_name varchar(120) NOT NULL,
    job_name varchar(200) NOT NULL,
    job_group varchar(200) NOT NULL,
    description varchar(250) NULL,
    job_class_name varchar(250) NOT NULL,
    is_durable bool NOT NULL,
    is_nonconcurrent bool NOT NULL,
    is_update_data bool NOT NULL,
    requests_recovery bool NOT NULL,
    job_data bytea NULL,
    CONSTRAINT qrtz_job_details_pkey PRIMARY KEY (sched_name, job_name, job_group)
);

CREATE TABLE IF NOT EXISTS qrtz_locks (
    sched_name varchar(120) NOT NULL,
    lock_name varchar(40) NOT NULL,
    CONSTRAINT qrtz_locks_pkey PRIMARY KEY (sched_name, lock_name)
);

CREATE TABLE IF NOT EXISTS qrtz_paused_trigger_grps (
    sched_name varchar(120) NOT NULL,
    trigger_group varchar(200) NOT NULL,
    CONSTRAINT qrtz_paused_trigger_grps_pkey PRIMARY KEY (sched_name, trigger_group)
);

CREATE TABLE IF NOT EXISTS qrtz_scheduler_state (
    sched_name varchar(120) NOT NULL,
    instance_name varchar(200) NOT NULL,
    last_checkin_time int8 NOT NULL,
    checkin_interval int8 NOT NULL,
    CONSTRAINT qrtz_scheduler_state_pkey PRIMARY KEY (sched_name, instance_name)
);

CREATE TABLE IF NOT EXISTS qrtz_triggers (
    sched_name varchar(120) NOT NULL,
    trigger_name varchar(200) NOT NULL,
    trigger_group varchar(200) NOT NULL,
    job_name varchar(200) NOT NULL,
    job_group varchar(200) NOT NULL,
    description varchar(250) NULL,
    next_fire_time int8 NULL,
    prev_fire_time int8 NULL,
    priority int4 NULL,
    trigger_state varchar(16) NOT NULL,
    trigger_type varchar(8) NOT NULL,
    start_time int8 NOT NULL,
    end_time int8 NULL,
    calendar_name varchar(200) NULL,
    misfire_instr int2 NULL,
    job_data bytea NULL,
    CONSTRAINT qrtz_triggers_pkey PRIMARY KEY (sched_name, trigger_name, trigger_group),
    CONSTRAINT qrtz_triggers_sched_name_fkey FOREIGN KEY (sched_name,job_name,job_group) REFERENCES qrtz_job_details(sched_name,job_name,job_group)
);

CREATE TABLE IF NOT EXISTS qrtz_simple_triggers (
    sched_name varchar(120) NOT NULL,
    trigger_name varchar(200) NOT NULL,
    trigger_group varchar(200) NOT NULL,
    repeat_count int8 NOT NULL,
    repeat_interval int8 NOT NULL,
    times_triggered int8 NOT NULL,
    CONSTRAINT qrtz_simple_triggers_pkey PRIMARY KEY (sched_name, trigger_name, trigger_group),
    CONSTRAINT qrtz_simple_triggers_sched_name_fkey FOREIGN KEY (sched_name,trigger_name,trigger_group) REFERENCES qrtz_triggers(sched_name,trigger_name,trigger_group)
);

CREATE TABLE IF NOT EXISTS qrtz_simprop_triggers (
    sched_name varchar(120) NOT NULL,
    trigger_name varchar(200) NOT NULL,
    trigger_group varchar(200) NOT NULL,
    str_prop_1 varchar(512) NULL,
    str_prop_2 varchar(512) NULL,
    str_prop_3 varchar(512) NULL,
    int_prop_1 int4 NULL,
    int_prop_2 int4 NULL,
    long_prop_1 int8 NULL,
    long_prop_2 int8 NULL,
    dec_prop_1 numeric(13, 4) NULL,
    dec_prop_2 numeric(13, 4) NULL,
    bool_prop_1 bool NULL,
    bool_prop_2 bool NULL,
    CONSTRAINT qrtz_simprop_triggers_pkey PRIMARY KEY (sched_name, trigger_name, trigger_group),
    CONSTRAINT qrtz_simprop_triggers_sched_name_fkey FOREIGN KEY (sched_name,trigger_name,trigger_group) REFERENCES qrtz_triggers(sched_name,trigger_name,trigger_group)
);

CREATE TABLE IF NOT EXISTS qrtz_blob_triggers (
    sched_name varchar(120) NOT NULL,
    trigger_name varchar(200) NOT NULL,
    trigger_group varchar(200) NOT NULL,
    blob_data bytea NULL,
    CONSTRAINT qrtz_blob_triggers_pkey PRIMARY KEY (sched_name, trigger_name, trigger_group),
    CONSTRAINT qrtz_blob_triggers_sched_name_fkey FOREIGN KEY (sched_name,trigger_name,trigger_group) REFERENCES qrtz_triggers(sched_name,trigger_name,trigger_group)
);

CREATE TABLE IF NOT EXISTS qrtz_cron_triggers (
    sched_name varchar(120) NOT NULL,
    trigger_name varchar(200) NOT NULL,
    trigger_group varchar(200) NOT NULL,
    cron_expression varchar(120) NOT NULL,
    time_zone_id varchar(80) NULL,
    CONSTRAINT qrtz_cron_triggers_pkey PRIMARY KEY (sched_name, trigger_name, trigger_group),
    CONSTRAINT qrtz_cron_triggers_sched_name_fkey FOREIGN KEY (sched_name,trigger_name,trigger_group) REFERENCES qrtz_triggers(sched_name,trigger_name,trigger_group)
);


CREATE INDEX IF NOT EXISTS idx_qrtz_ft_inst_job_req_rcvry ON qrtz_fired_triggers USING btree (sched_name, instance_name, requests_recovery);
CREATE INDEX IF NOT EXISTS idx_qrtz_ft_j_g ON qrtz_fired_triggers USING btree (sched_name, job_name, job_group);
CREATE INDEX IF NOT EXISTS idx_qrtz_ft_jg ON qrtz_fired_triggers USING btree (sched_name, job_group);
CREATE INDEX IF NOT EXISTS idx_qrtz_ft_t_g ON qrtz_fired_triggers USING btree (sched_name, trigger_name, trigger_group);
CREATE INDEX IF NOT EXISTS idx_qrtz_ft_tg ON qrtz_fired_triggers USING btree (sched_name, trigger_group);
CREATE INDEX IF NOT EXISTS idx_qrtz_ft_trig_inst_name ON qrtz_fired_triggers USING btree (sched_name, instance_name);
CREATE INDEX IF NOT EXISTS idx_qrtz_j_grp ON qrtz_job_details USING btree (sched_name, job_group);
CREATE INDEX IF NOT EXISTS idx_qrtz_j_req_recovery ON qrtz_job_details USING btree (sched_name, requests_recovery);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_c ON qrtz_triggers USING btree (sched_name, calendar_name);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_g ON qrtz_triggers USING btree (sched_name, trigger_group);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_j ON qrtz_triggers USING btree (sched_name, job_name, job_group);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_jg ON qrtz_triggers USING btree (sched_name, job_group);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_n_g_state ON qrtz_triggers USING btree (sched_name, trigger_group, trigger_state);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_n_state ON qrtz_triggers USING btree (sched_name, trigger_name, trigger_group, trigger_state);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_next_fire_time ON qrtz_triggers USING btree (sched_name, next_fire_time);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_nft_misfire ON qrtz_triggers USING btree (sched_name, misfire_instr, next_fire_time);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_nft_st ON qrtz_triggers USING btree (sched_name, trigger_state, next_fire_time);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_nft_st_misfire ON qrtz_triggers USING btree (sched_name, misfire_instr, next_fire_time, trigger_state);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_nft_st_misfire_grp ON qrtz_triggers USING btree (sched_name, misfire_instr, next_fire_time, trigger_group, trigger_state);
CREATE INDEX IF NOT EXISTS idx_qrtz_t_state ON qrtz_triggers USING btree (sched_name, trigger_state);
```
