# Spring Batch RDB Read Job Config Example

## my.batch.app.job.JobConfig.java

```java
package my.batch.app.job;

import javax.sql.DataSource;
import org.springframework.batch.core.ExitStatus;
import org.springframework.batch.core.ItemProcessListener;
import org.springframework.batch.core.ItemReadListener;
import org.springframework.batch.core.ItemWriteListener;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.JobExecutionListener;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.JobParametersIncrementer;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.StepExecution;
import org.springframework.batch.core.StepExecutionListener;
import org.springframework.batch.core.configuration.annotation.JobScope;
import org.springframework.batch.core.configuration.annotation.StepScope;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.item.Chunk;
import org.springframework.batch.item.database.JdbcPagingItemReader;
import org.springframework.batch.item.database.PagingQueryProvider;
import org.springframework.batch.item.database.builder.JdbcPagingItemReaderBuilder;
import org.springframework.batch.item.database.support.SqlPagingQueryProviderFactoryBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.lang.Nullable;
import org.springframework.scheduling.quartz.JobDetailFactoryBean;
import org.springframework.scheduling.quartz.SimpleTriggerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import my.batch.app.helper.BatchJobHelper;
import my.batch.app.model.ProductLicenseUsageHourlyTimelineDto;
import my.batch.app.service.ProductLicenseUsageHourlyTimelineService;
import my.batch.app.writer.ProductLicenseUsageHourlyTimelineItemWriter;
import lombok.extern.slf4j.Slf4j;

@Configuration
@ConditionalOnProperty(name = "job.product.license.usage.hourly.timeline.enabled", havingValue = "true")
@Slf4j
public class ProductLicenseUsageHourlyTimelineJobConfig {
    private static final String ITEM_NAME_UPPER_CAMEL_CASE = "ProductLicenseUsageHourlyTimeline";
    private static final String ITEM_NAME_LOWER_CAMEL_CASE = "productLicenseUsageHourlyTimeline";
    private static final String ITEM_NAME_LOWER_KEBAB_CASE = "product-license-usage-hourly-timeline";

    private static final long JOB_INTERVAL_SECONDS = 30;
    private static final int CHUNK_SIZE = 1000;
    private static final int LIMIT_SIZE = 1000;

    private static final String BEAN_NAME_PREFIX = ITEM_NAME_LOWER_CAMEL_CASE;
    private static final String JOB_BEAN_NAME = BEAN_NAME_PREFIX + "Job";
    private static final String JOB_DETAIL_FACTORY_BEAN_NAME = BEAN_NAME_PREFIX + "JobDetail";
    private static final String TRIGGER_BEAN_NAME = BEAN_NAME_PREFIX + "Trigger";
    private static final String STEP_BEAN_NAME = BEAN_NAME_PREFIX + "Step";
    private static final String READER_BEAN_NAME = BEAN_NAME_PREFIX + "ItemReader";
    private static final String WRITER_BEAN_NAME = BEAN_NAME_PREFIX + "ItemWriter";

    private final JobRepository jobRepository;
    private final PlatformTransactionManager transactionManager;


    private String selectQuery = """
    SELECT
        now() AS measuremnet_time
        , license_no
        , product_id
        , register_time
        , expire_time
        , remaining_days
        , issue_quantity
        , usage_quantity
        , issue_quantity - usage_quantity AS available_quantity""";

    private String fromQuery = "tb_product_license";

    private String sortKey = "license_no";

    @Autowired
    public ProductLicenseUsageHourlyTimelineJobConfig(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
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

    @Bean(name = READER_BEAN_NAME)
    @StepScope
    public JdbcPagingItemReader<ProductLicenseUsageHourlyTimelineDto> getJdbcPagingItemReader(
            DataSource dataSource) throws Exception {

        return new JdbcPagingItemReaderBuilder<ProductLicenseUsageHourlyTimelineDto>()
                .name(ITEM_NAME_LOWER_KEBAB_CASE+"-jdbc-paging-item-reader")
                .dataSource(dataSource)
                .queryProvider(pagingQueryProvider(dataSource))
                .rowMapper(new BeanPropertyRowMapper<>(ProductLicenseUsageHourlyTimelineDto.class))
                .pageSize(CHUNK_SIZE)
                .fetchSize(CHUNK_SIZE)
                .build();
    }

    @Bean(name = WRITER_BEAN_NAME)
    @StepScope
    public ProductLicenseUsageHourlyTimelineItemWriter getItemWriter(ProductLicenseUsageHourlyTimelineService service) {
        return new ProductLicenseUsageHourlyTimelineItemWriter(service);
    }

    @Bean(name = STEP_BEAN_NAME)
    @JobScope
    public Step getStep(
        @Qualifier(READER_BEAN_NAME) JdbcPagingItemReader<ProductLicenseUsageHourlyTimelineDto> reader,
        @Qualifier(WRITER_BEAN_NAME) ProductLicenseUsageHourlyTimelineItemWriter writer) {
        return new StepBuilder(ITEM_NAME_LOWER_KEBAB_CASE+"-step", jobRepository)
                .<ProductLicenseUsageHourlyTimelineDto, ProductLicenseUsageHourlyTimelineDto>chunk(CHUNK_SIZE, transactionManager)
                .reader(reader)
                .writer(writer)
                .listener(getItemReadListener())
                .listener(getItemWriteListener())
                .listener(getItemProcessListener())
                .listener(getStepExecutionListener())
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

    private PagingQueryProvider pagingQueryProvider(DataSource dataSource) throws Exception {
        SqlPagingQueryProviderFactoryBean queryProvider = new SqlPagingQueryProviderFactoryBean();
        queryProvider.setDataSource(dataSource);
        queryProvider.setSelectClause(selectQuery);
        queryProvider.setFromClause(fromQuery);
        queryProvider.setSortKey(sortKey);

        return queryProvider.getObject();
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

        private StepExecutionListener getStepExecutionListener() {
        return new StepExecutionListener() {
            @Override
            public void beforeStep(StepExecution stepExecution) {
                log.debug("beforeJob - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }

            @Override
            public ExitStatus afterStep(StepExecution stepExecution) {
                log.debug("afterJob - {}", ITEM_NAME_UPPER_CAMEL_CASE);
                return null;
            }
        };
    }

    private ItemReadListener<ProductLicenseUsageHourlyTimelineDto> getItemReadListener() {
        return new ItemReadListener<ProductLicenseUsageHourlyTimelineDto>() {
            @Override
            public void beforeRead() {
                log.debug("beforeRead - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }

            @Override
            public void afterRead(ProductLicenseUsageHourlyTimelineDto item) {
                log.debug("afterRead - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }

            @Override
            public void onReadError(Exception e) {
                log.error("onReadError", e);
            }
        };
    }

    private ItemProcessListener<ProductLicenseUsageHourlyTimelineDto, Integer> getItemProcessListener() {
        return new ItemProcessListener<ProductLicenseUsageHourlyTimelineDto, Integer>() {
            @Override
            public void beforeProcess(ProductLicenseUsageHourlyTimelineDto item) {
                log.debug("beforeProcess - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }

            @Override
            public void afterProcess(ProductLicenseUsageHourlyTimelineDto item, @Nullable Integer result) {
                log.debug("afterProcess - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }

            @Override
            public void onProcessError(ProductLicenseUsageHourlyTimelineDto item, Exception e) {
                log.error("onProcessError", e);
            }
        };
    }

    private ItemWriteListener<ProductLicenseUsageHourlyTimelineDto> getItemWriteListener() {
        return new ItemWriteListener<ProductLicenseUsageHourlyTimelineDto>() {
            @Override
            public void beforeWrite(Chunk<? extends ProductLicenseUsageHourlyTimelineDto> items) {
                log.debug("beforeWrite - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }

            @Override
            public void afterWrite(Chunk<? extends ProductLicenseUsageHourlyTimelineDto> items) {
                log.debug("afterWrite - {}", ITEM_NAME_UPPER_CAMEL_CASE);
            }

            @Override
            public void onWriteError(Exception e,
                    Chunk<? extends ProductLicenseUsageHourlyTimelineDto> items) {
                log.error("onWriteError:", e);
            }
        };
    }
}
```

## my.batch.app.helper.BatchJobHelper.java

```java
package my.batch.app.helper;

import java.util.Arrays;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import org.quartz.JobDataMap;
import org.quartz.JobExecutionContext;
import org.quartz.SchedulerException;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobParameter;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.scheduling.quartz.JobDetailFactoryBean;
import org.springframework.scheduling.quartz.SimpleTriggerFactoryBean;
import my.batch.app.launcher.BatchJobExecutor;

public class BatchJobHelper {

    private static final String JOB_NAME_KEY = "job";
    private static final String JOB_PARAMETERS_KEY = "job.parameters";
    private static final String JOB_PARAMETERS_INSTANCE_ID_KEY = "instance.id";
    private static final String JOB_PARAMETERS_TIMESTAMP_KEY = "timestamp";

    private static final List<String> KEYWORDS = Arrays.asList(JOB_NAME_KEY, JOB_PARAMETERS_KEY);

    public static JobDetailFactoryBeanBuilder jobDetailFactoryBeanBuilder() {
        return new JobDetailFactoryBeanBuilder();
    }

    public static SimpleTriggerFactoryBeanBuilder simpleTriggerFactoryBeanBuilder() {
        return new SimpleTriggerFactoryBeanBuilder();
    }

    public static String getJobName(JobDataMap jobDataMap) {
        return (String) jobDataMap.get(JOB_NAME_KEY);
    }

    public static JobParameters getJobParameters(JobExecutionContext context)
            throws SchedulerException {
        JobDataMap jobDataMap = context.getMergedJobDataMap();

        return new JobParametersBuilder((JobParameters) jobDataMap.get(JOB_PARAMETERS_KEY))
                .addString(JOB_PARAMETERS_INSTANCE_ID_KEY,
                        context.getScheduler().getSchedulerInstanceId())
                .addLong(JOB_PARAMETERS_TIMESTAMP_KEY, System.currentTimeMillis())
                .toJobParameters();
    }

    public static class JobDetailFactoryBeanBuilder {
        private final Map<String, Object> map;
        private final JobParametersBuilder jobParametersBuilder;

        private boolean durability = true;
        private boolean requestsRecovery = true;

        JobDetailFactoryBeanBuilder() {
            this.map = new HashMap<>();
            this.jobParametersBuilder = new JobParametersBuilder();
        }

        public JobDetailFactoryBeanBuilder job(Job job) {
            this.map.put(JOB_NAME_KEY, job.getName());
            return this;
        }

        public JobDetailFactoryBeanBuilder durability(boolean durability) {
            this.durability = durability;
            return this;
        }

        public JobDetailFactoryBeanBuilder requestsRecovery(boolean requestsRecovery) {
            this.requestsRecovery = requestsRecovery;
            return this;
        }

        public JobDetailFactoryBeanBuilder parameters(String key, Object value) {
            if (KEYWORDS.contains(key)) {
                throw new RuntimeException("Invalid Parameter.");
            }

            this.addParameter(key, value);
            return this;
        }

        public JobDetailFactoryBean build() {
            if (!map.containsKey(JOB_NAME_KEY)) {
                throw new RuntimeException("Not Found Job Name.");
            }

            this.map.put(JOB_PARAMETERS_KEY, this.jobParametersBuilder.toJobParameters());

            JobDetailFactoryBean jobDetailFactoryBean = new JobDetailFactoryBean();
            jobDetailFactoryBean.setJobClass(BatchJobExecutor.class);
            jobDetailFactoryBean.setDurability(this.durability);
            jobDetailFactoryBean.setRequestsRecovery(this.requestsRecovery);
            jobDetailFactoryBean.setJobDataAsMap(this.map);

            return jobDetailFactoryBean;
        }

        private void addParameter(String key, Object value) {
            if (value instanceof String) {
                this.jobParametersBuilder.addString(key, (String) value);
                return;
            } else if (value instanceof Float || value instanceof Double) {
                this.jobParametersBuilder.addDouble(key, ((Number) value).doubleValue());
                return;
            } else if (value instanceof Integer || value instanceof Long) {
                this.jobParametersBuilder.addLong(key, ((Number) value).longValue());
                return;
            } else if (value instanceof Date) {
                this.jobParametersBuilder.addDate(key, (Date) value);
                return;
            } else if (value instanceof JobParameter) {
                this.jobParametersBuilder.addParameter(key, (JobParameter) value);
                return;
            }

            throw new RuntimeException("Not Supported Parameter Type.");
        }
    }

    public static class SimpleTriggerFactoryBeanBuilder {
        private String name;
        private long interval;
        private JobDetailFactoryBean jobDetailFactoryBean;

        public SimpleTriggerFactoryBeanBuilder name(String name) {
            this.name = name;
            return this;
        }

        public SimpleTriggerFactoryBeanBuilder interval(long interval) {
            this.interval = interval;
            return this;
        }

        public SimpleTriggerFactoryBeanBuilder jobDetailFactoryBean(
                JobDetailFactoryBean jobDetailFactoryBean) {
            this.jobDetailFactoryBean = jobDetailFactoryBean;
            return this;
        }

        public SimpleTriggerFactoryBean build() {
            if (this.interval < 1 || this.jobDetailFactoryBean == null) {
                throw new RuntimeException(
                        "SimpleTriggerFactoryBean is required interval and JobDetailFactoryBean.");
            }

            SimpleTriggerFactoryBean simpleTriggerFactoryBean = new SimpleTriggerFactoryBean();
            simpleTriggerFactoryBean.setName(this.name);
            simpleTriggerFactoryBean.setRepeatInterval(this.interval * 1000L);
            simpleTriggerFactoryBean.setJobDetail(this.jobDetailFactoryBean.getObject());

            return simpleTriggerFactoryBean;
        }
    }
}
```

## my.batch.app.launcher.BatchJobExecutor

```java
package my.batch.app.launcher;

import my.batch.app.helper.BatchJobHelper;
import lombok.extern.slf4j.Slf4j;
import org.quartz.DisallowConcurrentExecution;
import org.quartz.InterruptableJob;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.quartz.JobKey;
import org.quartz.Scheduler;
import org.quartz.SchedulerException;
import org.quartz.UnableToInterruptJobException;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.configuration.JobLocator;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.batch.core.launch.NoSuchJobException;
import org.springframework.scheduling.quartz.QuartzJobBean;
import org.springframework.scheduling.quartz.SchedulerFactoryBean;

@DisallowConcurrentExecution // prevent executing job concurrently
@Slf4j
public class BatchJobExecutor extends QuartzJobBean implements InterruptableJob {
    private final SchedulerFactoryBean schedulerFactoryBean;
    private final JobLocator jobLocator;
    private final JobLauncher jobLauncher;

    private boolean isJobInterrupted = false;
    private String jobName;

    public BatchJobExecutor(SchedulerFactoryBean schedulerFactoryBean, JobLocator jobLocator,
            JobLauncher jobLauncher) {
        this.schedulerFactoryBean = schedulerFactoryBean;
        this.jobLocator = jobLocator;
        this.jobLauncher = jobLauncher;
    }

    @Override
    public void interrupt() throws UnableToInterruptJobException {
        log.info("Job is interrupting... job.name : {}", jobName);

        isJobInterrupted = true;
    }

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext)
            throws JobExecutionException {
        JobKey jobKey = jobExecutionContext.getJobDetail().getKey();

        try {
            jobName = BatchJobHelper.getJobName(jobExecutionContext.getMergedJobDataMap());
            JobParameters jobParameters = BatchJobHelper.getJobParameters(jobExecutionContext);

            // when job interrupting
            if (isJobInterrupted) {
                log.warn("Job is interrupted. [job.name : {}, job.parameters : {}]", jobName, jobParameters);

                return;
            }

            log.info("Starting Job. [job.name : {}, job.parameters : {}]", jobName, jobParameters);

            Job job = jobLocator.getJob(jobName);
            jobLauncher.run(job, jobParameters);
        } catch (NoSuchJobException noSuchJobException) {
            log.error("Error occurred while finding job", noSuchJobException);

            Scheduler scheduler = schedulerFactoryBean.getScheduler();

            try {
                scheduler.deleteJob(jobKey);
                log.info("delete job. job.key : {}", jobKey);
            } catch (SchedulerException e) {
                log.error("deleting schedule job", e);
            }
        } catch (JobExecutionException jobException) {
            log.error("Error occurred while running job", jobException);

            throw jobException;
        } catch (Exception e) {
            log.error(String.format("Error occurred while execute batch.job : [job.name : %s]", jobName), e);
        }
    }
}
```
