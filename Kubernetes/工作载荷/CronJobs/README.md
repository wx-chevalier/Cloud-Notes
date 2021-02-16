# Cron Jobs

CronJob 服务不过是 crontab 文件中的一行或表 cron 中的同一文件。它按照给定的时间表定期安排和执行任务。但我们可以使用 Cron Job 来做什么呢？Cron 对于创建经常性和重复性任务非常有用，例如执行备份或发送电子邮件。

```yaml
$ vim primeiro-cron.yaml

apiVersion : batch/v1beta1
kind : CronJob
metadata :
   name : giropops-cron
spec :
   schedule : " * / 1 * * * * "
   jobTemplate :
     spec :
       template :
         spec :
           containers :
          - name : giropops-cron
            image : busybox
            args :
            - /bin/sh
            - -c
            - date; echo Welcome to Uncomplicating Kubernetes - LinuxTips VAIIII; sleep 30
          restartPolicy : OnFailure
```

我们前面的 CronJobs 例子每分钟打印当前时间和一条问候信息。让我们从清单中创建它的 CronJob。

```sh
$ kubectl create -f primeiro-cron.yaml

cronjob.batch/giropops-cron created

$ kubectl get cronjobs

NAME            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
giropops-cron   */1 * * * *   False     1        13s             2m

$ kubectl describe cronjobs.batch giropops-cron

Name:                       giropops-cron
Namespace:                  default
Labels:                     <none>
Annotations:                <none>
Schedule:                   */1 * * * *
Concurrency Policy:         Allow
Suspend:                    False
Starting Deadline Seconds:  <unset>
Selector:                   <unset>
Parallelism:                <unset>
Completions:                <unset>
Pod Template:
  Labels:  <none>
  Containers:
   giropops-cron:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Args:
      /bin/sh
      -c
      date; echo LinuxTips VAIIII ;sleep 30
    Environment:     <none>
    Mounts:          <none>
  Volumes:           <none>
Last Schedule Time:  Wed, 22 Aug 2018 22:33:00 +0000
Active Jobs:         <none>
Events:
  Type    Reason            Age   From                Message
  ----    ------            ----  ----                -------
  Normal  SuccessfulCreate  1m    cronjob-controller  Created job giropops-cron-1534977120
  Normal  SawCompletedJob   1m    cronjob-controller  Saw completed job: giropops-cron-1534977120
  Normal  SuccessfulCreate  41s   cronjob-controller  Created job giropops-cron-1534977180
  Normal  SawCompletedJob   1s    cronjob-controller  Saw completed job: giropops-cron-1534977180
  Normal  SuccessfulDelete  1s    cronjob-controller  Deleted job giropops-cron-1534977000
```

看看多酷，如果你看集群事件，你是 cron 已经在调度和执行任务了。现在我们就来看看这个 cron 工作通过命令`kubectl get`旁边的参数 --watch 来查看任务的输出，注意，这个任务会在 CronJob 创建后的一分钟左右被创建。

```sh
$ kubectl get jobs --watch

NAME                       DESIRED  SUCCESSFUL   AGE
giropops-cron-1534979640   1         1            2m
giropops-cron-1534979700   1         1            1m

$ kubectl get cronjob giropops-cron

NAME           SCHEDULE      SUSPEND   ACTIVE    LAST SCHEDULE   AGE
giropops-cron  */1 * * * *   False     1         26s             48m
```

我们可以看到，我们的 cronis 工作正常。要查看任务执行的命令输出将使用 kubectl 的命令日志。为此，我们将列出正在运行的 pods，然后从中获取日志。

```sh
$ kubectl get pods

NAME                            READY     STATUS      RESTARTS   AGE
giropops-cron-1534979940-vcwdg  1/1       Running     0          25s

$ kubectl logs giropops-cron-1534979940-vcwdg

Wed Aug 22 23:19:06 UTC 2018
LinuxTips VAIIII
```

cron 正确地执行任务，打印日期和我们在清单中创建的短语。如果 kubectl 得到 pods 我们运行一个，我们可以看到创建的 Pods 和用于执行任务的每分钟。

```sh
$ kubectl get pods

NAME                             READY    STATUS      RESTARTS   AGE
giropops-cron-1534980360-cc9ng   0/1      Completed   0          2m
giropops-cron-1534980420-6czgg   0/1      Completed   0          1m
giropops-cron-1534980480-4bwcc   1/1      Running     0          4s

$ kubectl delete cronjob giropops-cron

cronjob.batch "giropops-cron" deleted
```
