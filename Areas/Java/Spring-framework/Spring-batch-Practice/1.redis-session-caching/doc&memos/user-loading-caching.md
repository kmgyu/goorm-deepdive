https://s7won.tistory.com/8


먼저 배치작업을 위해 대량의 계정을 만들어준다.

![[spring-batch-usercreate.png]]

password encoder로 인해 엄청난 수준의 부하가 걸렸다.
기존에는 14초 내로 끝났는데 오버헤드가 굉장히 커진 것을 볼 수 있음.

```
2025-10-02T15:46:46.898+09:00  INFO 24388 --- [batchApplication] [nio-8080-exec-5] o.s.b.c.l.s.TaskExecutorJobLauncher      : Job: [SimpleJob: [name=userCreateJob]] launched with the following parameters: [{'requestId':'{value=1, type=class java.lang.String, identifying=true}','time':'{value=1759387606862, type=class java.lang.Long, identifying=true}'}]
2025-10-02T15:46:46.937+09:00  INFO 24388 --- [batchApplication] [nio-8080-exec-5] o.s.batch.core.job.SimpleStepHandler     : Executing step: [userStep]
2025-10-02T16:10:30.989+09:00  INFO 24388 --- [batchApplication] [nio-8080-exec-5] o.s.batch.core.step.AbstractStep         : Step: [userStep] executed in 23m44s51ms
```