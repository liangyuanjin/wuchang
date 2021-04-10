---
title: Job与Cronjob
date: 2021-04-10 13:56:46
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---
> 在日常开发中通常会遇到一下任务，或者按时间来进行调度的工作，`Kubernetes` 提供了 `Job` 和 `CronJob` 两种资源对西那个来完成我们这种需求，`Job` 负责处理任务，`仅执行一次的任务`, 而 `Cronjob` 则就是在 `Job` 上加上时间调度

## Job

> 利用`Job` 资源对象来创建一个任务，定义一个`YAML`文件如下，执行一个倒计时的任务

```yaml
---
apiVersion: batch/v1
kind: Job
metadata:
  name: job-demo
spec:
  template:
    metadata:
      name: job-demo
    spec:
      restartPolicy: Never
      containers:
      - name: counter
        image: busybox
        command:
        - "bin/sh"
        - "-c"
        - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done"
```

> `Job` 的 `RestartPolicy` 仅支持 `Never` 和 `OnFailure` 不支持 `Always`,`Job`就相当于来执行一个批处理任务，执行完就结束了，如果支持 `Always` 就表示陷入死循环了。

`创建Job`

```bash
kubectl apply -f job-demo.yaml
job "job-demo" created
```

`查看Job状态`

```bash
kubectl get  jobs
NAME       COMPLETIONS   DURATION   AGE
job-demo   1/1           4s         9m11s
```

`查看 Job 的描述信息`

```bash
kubectl describe job job-demo
.....
```

`查看Job的日志`

```bash
kubectl logs job-demo-52q2d
9
8
7
6
5
4
3
2
1
```

> 可以看到上面已经输出了任务的执行结果，这样一个 Job 资源对象就算完成了

## Cronjob

> `Cronjob` 其实就是在 `Job`基础上加上了时间调度，在给定的时间店运行一个任务，也可以周期性地在给定时间点运行。和`Linux`中的`crontab` 很相似，一个`CronJob` 对象就对应`crontab` 文件中那一行，它根据配置的时间格式周期的运行一个 `Job`, 格式和 `crontab`一样

### Cronjob 格式

> 分 时 日 月 周 要运行的命令 第1列分钟0～59 第2列小时0～23） 第3列日1～31 第4列月1～12 第5列星期0～7（0和7表示星期天）第6列要运行的命令

`CronJob 管理 Job`

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-demo
spec:
  successfulJobsHistoryLimit: 10
  failedJobsHistoryLimit: 5
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: hello
            image: busybox
            args:
            - "bin/sh"
            - "-c"
            - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done"
```

> `kind` 为 `CronJob`, 而且`.spec.schedule` 字段是必须填写的，用来指定任务的周期，格式和 `crontab` 格式一样 `.spec.jobTemplate`用来指定需要运行的的任务，格式和 `Job` 一样，除此之外，`.spec.successfulJobsHistoryLimit`和`.spec.failedJobsHistoryLimit` 表示历史限制，他们指定可以保留多少个完成和失败的`Job`, 默认没有限制，成功和失败的都会被保留下来，运行一个 `CronJob`时，`Job` 可以很快的就堆积起来，所以推荐设置这两个字段的值。

`创建 CornJob`

```bash
kubectl apply -f cronjob-dmeo.yaml
cronjob.batch/cronjob-demo created
```

`查看CronJob`

```bash
kubectl get cronjob
NAME           SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob-demo   */1 * * * *   False     0        18s             5m36
```

`查看 Job`

```bash
kubectl get jobs
NAME                      COMPLETIONS   DURATION   AGE
cronjob-demo-1618037340   1/1           3s         2m42s
cronjob-demo-1618037400   1/1           3s         102s
cronjob-demo-1618037460   1/1           5s         42s
```

> 可以看到 `Job` 是不断增加的，每一分钟执行一次

`根据YAML文件删除CronJob`

```bash
kubectl delete -f cronjob-demo.yaml
cronjob.batch "cronjob-demo" deleted
```

`根据CronJob资源对象删除`

```bash
kubectl delete cronjob cronjob-demo
cronjob.batch "cronjob-demo" deleted
```
