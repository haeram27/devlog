# Spring Batch Tasklet executtion Job Config Example

## my.batch.app.job.JobConfig.java

```java
package my.batch.app.job;

import org.springframework.batch.core.ChunkListener;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.JobExecutionListener;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.JobParametersIncrementer;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.configuration.annotation.JobScope;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.scheduling.quartz.JobDetailFactoryBean;
import org.springframework.scheduling.quartz.SimpleTriggerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import my.batch.app.helper.BatchJobHelper;
import lombok.extern.slf4j.Slf4j;

@Configuration
@ConditionalOnProperty(name = "job.my.tasklet.enabled", havingValue = "true")
@Slf4j
public class MyTaskletJobConfig {
    private static final String ITEM_NAME_UPPER_CAMEL_CASE = "MyTaskletJob";
    private static final String ITEM_NAME_LOWER_CAMEL_CASE = "myTaskletJob";
    private static final String ITEM_NAME_LOWER_KEBAB_CASE = "my-tasklet-job";

    private static final long JOB_INTERVAL_SECONDS = 30;

    private static final String BEAN_NAME_PREFIX = ITEM_NAME_LOWER_CAMEL_CASE;
    private static final String JOB_BEAN_NAME = BEAN_NAME_PREFIX + "Job";
    private static final String JOB_DETAIL_FACTORY_BEAN_NAME = BEAN_NAME_PREFIX + "JobDetail";
    private static final String TRIGGER_BEAN_NAME = BEAN_NAME_PREFIX + "Trigger";
    private static final String STEP_BEAN_NAME = BEAN_NAME_PREFIX + "Step";

    private final JobRepository jobRepository;
    private final PlatformTransactionManager transactionManager;


    private String upsertQuery = """
INSERT INTO tb_my_test (
    measuring_time
)
SELECT
    now() AT TIME ZONE 'UTC' AS measuring_time_utc
ON CONFLICT (measuring_time_utc) DO UPDATE
SET
    measuring_time_utc = EXCLUDED.measuring_time_utc;
""";

    @Autowired
    public MyTaskletJobConfig(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
        this.jobRepository = jobRepository;
        this.transactionManager = transactionManager;
    }

    @Bean(name = TRIGGER_BEAN_NAME)
    public SimpleTriggerFactoryBean getJobTrigger(
        @Qualifier(JOB_DETAIL_FACTORY_BEAN_NAME) JobDetailFactoryBean jobDetailFactory) {

        return BatchJobHelper.simpleTriggerFactoryBeanBuilder()
                .jobDetailFactoryBean(jobDetailFactory)
                .interval(JOB_INTERVAL_SECONDS)
                .build();
    }

    @Bean(name = JOB_DETAIL_FACTORY_BEAN_NAME)
    public JobDetailFactoryBean getJobDetailFactory(@Qualifier(JOB_BEAN_NAME) Job job) {
        return BatchJobHelper.jobDetailFactoryBeanBuilder()
                .job(job)
                .build();
    }

    @Bean(name = STEP_BEAN_NAME)
    @JobScope
    public Step getStep(JdbcTemplate jdbcTemplate) {
        return new StepBuilder(ITEM_NAME_LOWER_KEBAB_CASE+"-step", jobRepository)
                .tasklet((StepContribution contribution, ChunkContext chunkContext) -> {
                    jdbcTemplate.update(upsertQuery);
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .listener(getStepTaskletChunkListener())
                .build();
    }

    @Bean(name = JOB_BEAN_NAME)
    public Job getJob(
        @Qualifier(STEP_BEAN_NAME) Step step) {
        return new JobBuilder(ITEM_NAME_LOWER_KEBAB_CASE+"-job", jobRepository)
                .incrementer(getUniqueJobParametersIncrementer())
                .listener(getJobExecutionListener())
                .start(step)
                .build();
    }

    private JobParametersIncrementer getUniqueJobParametersIncrementer() {
        return new JobParametersIncrementer() {
            private static final String RUN_ID = "run.id";

            @Override
            public JobParameters getNext(JobParameters jobParameters) {
                if (jobParameters == null || jobParameters.isEmpty()) {
                    return new JobParametersBuilder().addLong(RUN_ID, 1L).toJobParameters();
                }

                long id = jobParameters.getLong(RUN_ID, 0L) + 1;

                return new JobParametersBuilder().addLong(RUN_ID, id).toJobParameters();
            }
        };
    }

    private JobExecutionListener getJobExecutionListener() {
        return new JobExecutionListener() {
            @Override
            public void beforeJob(JobExecution jobExecution) {
                log.debug("beforeJob - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }

            @Override
            public void afterJob(JobExecution jobExecution) {
                log.debug("afterJob - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }
        };
    }

        private ChunkListener getStepTaskletChunkListener() {
        return new ChunkListener() {
            @Override
            public void beforeChunk(ChunkContext context ) {
                log.debug("beforeChunk - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }

            @Override
            public void afterChunk(ChunkContext context ) {
                log.debug("afterChunk - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }

            @Override
            public void afterChunkError(ChunkContext context) {
                log.debug("afterChunkError - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }
        };
    }
}
```