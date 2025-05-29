# batch configuration
|Class|Members|
|---|---|
|TriggerFactory(Trigger)	  | JobDetail, StartTime, Delay, Interval, RepeatCount |
|JobDetailFactory(JobDetail)  | JobClass, Durability(bool), RequestRecovery(bool), JobData(Job, JobParameters) |
|Job	                      | JobParametersIncrementer, Step, Listener |
|Step	                      | {ItemReader, ItemProcessor, ItemWriter}, Tasklet, Listener |

----------
* JobDetail
  * JobClass(QuartzJobBean == job executor, launch jobs)
  * Durability(boolean)
  * RequestRecovery(boolean)
  * JobData(Job)

* JobClass
  * Job Executor 역할
  * jobExecutionContext에서 실행할 Job 정보(JobKey, JobParameters)를 획득
  * JobLocator 부터 실행할 Job 정보와 매치되는 Job Instance를 획득
  * JobLauncher로 Job Instance를 실행
  * JobLocator에 정보와 매치되는 Job Instance가 없으면 해당 Job과 연관된 scheduler에서 연관 정보를 삭제 필요

JobParameters
