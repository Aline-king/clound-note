# Job和Cron Job

## Job

### 资源属性

```yaml
apiVersion: batch/v1  			# API群组及版本
kind: Job  				# 资源类型特有标识
metadata:
  name <string>  			# 资源名称，在作用域中要唯一
  namespace <string>  			# 名称空间；Job资源隶属名称空间级别
spec:
  selector <object> 			# 标签选择器，必须匹配template字段中Pod模板中的标签
  template <object>  			# Pod模板对象
  completions <integer> 		# 期望的成功完成的作业次数，成功运行结束的Pod数量
  ttlSecondsAfterFinished  <integer> 	# 终止状态作业的生存时长，超期将被删除
  parallelism  <integer>  		# 作业的最大并行度，默认为1
  backoffLimit <integer>  		# 将作业标记为Failed之前的重试次数，默认为6
  activeDeadlineSeconds  <integer> 	# 作业启动后可处于活动状态的时长
```

## cron job

CronJob其实就是在Job的基础上加上了时间调度，

我们可以：在给定的时间点运行一个任务，也可以周期性地在给定时间点运行。

其效果与linux中的crontab效果非常类似，一个CronJob对象其实就对应中crontab文件中的一行，它根据配置的时间格式周期性地运行一个Job，格式和crontab也是一样的。

{% hint style="info" %}
crontab的格式如下：

&#x20;      分          时          日        月         周&#x20;

(0～59) (0～23) (1～31) (1～12) (0～7)
{% endhint %}

### 资源属性

```yaml
apiVersion: batch/v1  			# API群组及版本
kind: CronJob  				# 资源类型特有标识
metadata:
  name <string>  			# 资源名称，在作用域中要唯一
  namespace <string>  			# 名称空间；CronJob资源隶属名称空间级别
spec:
  jobTemplate  <Object>  		# job作业模板，必选字段
    metadata <object>  			# 模板元数据
    spec <object> 				# 作业的期望状态
  schedule <string>  			# 调度时间设定，必选字段
  concurrencyPolicy  <string> 		# 并发策略，可用值有Allow、Forbid和Replace
  failedJobsHistoryLimit <integer> 	# 失败作业的历史记录数，默认为1
  successfulJobsHistoryLimit  <integer> # 成功作业的历史记录数，默认为3
  startingDeadlineSeconds  <integer> 	# 因错过时间点而未执行的作业的可超期时长
  suspend  <boolean> 			# 是否挂起后续的作业，不影响当前作业，默认为false
```

### 例子

{% tabs %}
{% tab title="创建资源清单文件" %}
111

```
[root@kubernetes-master1 /data/kubernetes/controller]# cat 06_kubernetes_cronjob_test.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: superopsmsb-cronjob
spec:
  schedule: "* * * * *"
  successfulJobsHistoryLimit: 100
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: cronjob
            image: kubernetes-register.superopsmsb.com/superopsmsb/busybox:1.28
            command: ["/bin/sh","-c","echo cronjob-test"]
```

应用配置文件

```
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl  apply -f 06_kubernetes_cronjob_test.yaml
cronjob.batch/superopsmsb-cronjob created
```
{% endtab %}

{% tab title="查看效果" %}
{% hint style="danger" %}
注意

这是6分钟过后才进行查看的效果
{% endhint %}

```
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl get cronjobs.batch
NAME                  SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
superopsmsb-cronjob   * * * * *   False     0        38s             50s
​
等待8分钟后查看job任务执行效果
[root@kubernetes-master1 /data/kubernetes/controller]# kubectl get jobs.batch
NAME                           COMPLETIONS   DURATION   AGE
superopsmsb-cronjob-27638376   1/1           4s         7m55s
superopsmsb-cronjob-27638377   1/1           3s         6m55s
superopsmsb-cronjob-27638378   1/1           3s         5m55s
superopsmsb-cronjob-27638379   1/1           3s         4m55s
superopsmsb-cronjob-27638380   1/1           3s         3m55s
superopsmsb-cronjob-27638381   1/1           4s         2m55s
superopsmsb-cronjob-27638382   1/1           3s         115s
superopsmsb-cronjob-27638383   1/1           3s         55s
```

{% code title="查看日志效果" %}
```
[root@kubernetes-master1 /data/kubernetes/controller]# cornjob_list=$(kubectl get pod | grep cronjob | awk '{print $1}')
[root@kubernetes-master1 /data/kubernetes/controller]# for i in $cornjob_list ;do kubectl logs $i --timestamps=true; done
2022-07-20T07:36:00.998466094Z cronjob-test
2022-07-20T07:37:00.992842042Z cronjob-test
2022-07-20T07:38:00.996314468Z cronjob-test
2022-07-20T07:39:01.039271449Z cronjob-test
2022-07-20T07:40:00.999991261Z cronjob-test
2022-07-20T07:41:01.032948311Z cronjob-test
2022-07-20T07:42:01.073678317Z cronjob-test
2022-07-20T07:43:01.076927560Z cronjob-test
2022-07-20T07:44:01.082994524Z cronjob-test
```
{% endcode %}
{% endtab %}
{% endtabs %}

\
