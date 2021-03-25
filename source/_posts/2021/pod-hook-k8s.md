---
title: Pod Hook
date: 2021-03-24 08:20:24
tags: K8s
categories: K8s
description: 容器编排
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: /images/k8s/k8s.jpeg
---
> `Pod` 是 `kubenetes` 集群中的最小单元， 而 `Pod` 是由容器组成的，`Pod` 的生命周期和`容器`的生命周期息息相关

> kubernetes 为容器提供了生命周期的钩子，称为 `Pod Hook`, `Pod Hook` 是由， kubectl 发起的，`当容器中的进程启动前或者容器中的进程终止之前运行`, 可以为 Pod 中的所有容器都配置 hook

## Pod Hook 分类

* `PostStart`: 钩子函数在容器创建后立即执行，但是并不一定是在容器 `ENTRYPOINT` 之前运行，主要用户资源部署，环境准备，需要注意如果钩子花费太长时间以至于不能运行或者挂起，容器不能达到 `Running` 状态
* `PreStop`: 钩子函数将在容器终止前立即被调用，特点是`阻塞`, 必须在删除容器的调用发出之前完成，主要用于优雅的关闭应用程序，通知其他系统等，在钩子执行期间挂起，Pod 阶段将停留在 `Running`状态，并且永远不会达到 `Failed` 状态.

> 如果 `PostStart` 或者 `PreStop` 钩子失败，他会杀死容器，所以我们应该让钩子函数尽可能的轻量。

## 示例PostStart

定义一个 `Nginx Pod`, 在文件中设置 `PostStart` 钩子函数，创建容器过后写入一句话到 `/usr/share/message` 文件中

### 创建 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hook-demo1
spec:
  containers:
  - name: hook-demo1
    image: nginx
    # 使用 PostStart 钩子函数
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
```

创建Pod

```bash
kubectl apply -f hook-pod1.yaml
```

### 进入Pod查看输出

进入 Pod 内部 可以用 `kubectl exec --help` 查看命令帮助

```bash
kubectl exec --help

Examples:
  # Get output from running 'date' command from pod mypod, using the first container by default
  kubectl exec mypod date

  # Get output from running 'date' command in ruby-container from pod mypod
  kubectl exec mypod -c ruby-container date

  # Switch to raw terminal mode, sends stdin to 'bash' in ruby-container from pod mypod
  # and sends stdout/stderr from 'bash' back to the client
  kubectl exec mypod -c ruby-container -i -t -- bash -il

  # List contents of /usr from the first container of pod mypod and sort by modification time.
  # If the command you want to execute in the pod has any flags in common (e.g. -i),
  # you must use two dashes (--) to separate your command's flags/arguments.
  # Also note, do not surround your command and its flags/arguments with quotes
  # unless that is how you would execute it normally (i.e., do ls -t /usr, not "ls -t /usr").
  kubectl exec mypod -i -t -- ls -t /usr

  # Get output from running 'date' command from the first pod of the deployment mydeployment, using
the first container by default
  kubectl exec deploy/mydeployment date

  # Get output from running 'date' command from the first pod of the service myservice, using the
first container by default
  kubectl exec svc/myservice date

Options:
  -c, --container='': Container name. If omitted, the first container in the pod will be chosen
      --pod-running-timeout=1m0s: The length of time (like 5s, 2m, or 3h, higher than zero) to wait
until at least one pod is running
  -i, --stdin=false: Pass stdin to the container
  -t, --tty=false: Stdin is a TTY
```

进入容器

```bash
kubectl exec hook-demo1 -i -t /bin/bash
```

> 然后去 `/usr/share/message` 查看输出

## 示例PreStop

> 用户请求删除含有 Pod 的资源对象时， kubernetes 为了让程序优雅的关闭(应用程序完成正在处理的请求后，再关闭软件)， kubernetes 提供了两种信息通知

* 默认: kubernetes 通知 node 执行 `docker stop` 命令，docker 会先想容器中的 `PID` 为 1 的进程发送系统信号 `SIGTERM`， 然后等待容器中的应用程序终止执行，如果等待达到设定的超时时间，会继续发送 `SIGKILL` 的系统信号强行 KILL 掉进程。

* 使用 pod 生命周期(利用 `RreStop` 回调函数), 它执行在发送终止信号之前

### 强制删除

> kubernetes 默认所有的优雅退出都是在 30 秒内，`kubectl delete ` 命令支持 `--grace-period=<seconds>` 选项，允许用户用他们自己指定的值覆盖默认值，值`0` 代表强制删除Pod, `kubectl1.5` 版本以上必须同时指定 `--force --grace-period=0`。

> 强制删除一个 pod 是从集群还有 etcd 中立刻删除这个 Pod, 在 Pod 被强制删除时， api 服务器不会等待来自 pod 所在节点上的 kubectl 确认信息： pod 已经被终止。

### PreStop 示例

定义了一个Nginx Pod，其中设置了PreStop钩子函数，即在容器退出之前，优雅的关闭 Nginx

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hook-demo2
spec:
  containers:
  - name: hook-demo2
    image: nginx
    lifecycle:
      preStop:
        exec:
          command: ["/usr/sbin/nginx","-s","quit"]

---
apiVersion: v1
kind: Pod
metadata:
  name: hook-demo2
  labels:
    app: hook
spec:
  containers:
  - name: hook-demo2
    image: nginx
    ports:
    - name: webport
      containerPort: 80
    # 将容器内的 /usr/share/目录挂载到宿主机上
    volumeMounts:
    - name: message
      mountPath: /usr/share/
    lifecycle:
      preStop:
        exec:
          command: ['/bin/sh', '-c', 'echo Hello from the preStop Handler > /usr/share/message']
  # 声明宿主机的挂载 path
  volumes:
  - name: message
    hostPath:
      path: /tmp
```