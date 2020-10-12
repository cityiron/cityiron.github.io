# 一拳搞定 Kubernetes | 玩转 Minikube

本文介绍如何在[本机]^(mac)运行 Minikube 来学习 kubernetes.

<!--more-->

琦玉：
杰诺斯，接下去要好好学习 kubernetes。

杰诺斯：
老师，我会的。你先教我如何搭建一个集群呗？

琦玉：
感慨万千，以前找三台旧电脑搭建一个 kubernetes 集群，现在在本机敲两下命令就可以跑起来，杰诺斯，今天就从 minikube 入手吧。


## 一、前言

本篇是整个 kubernetes 的第一篇，比较简单，完全走的官方路线。安装好了 kubernetes 集群，我们才能愉快的进入容器编排的海洋。

## 二、Minikube 操作

### 2-1. 暂停 Minikube

```bash
minikube stop
```

输出:

```bash
* Stopping "minikube" in hyperkit ...
* "minikube" 已停止
```

### 2-2. 启动

```bash
minikube start
```

输出:
```bash
* Darwin 10.15.6 上的 minikube v1.6.2
  - KUBECONFIG=/Users/tc/.kube/config.22
* Selecting 'hyperkit' driver from existing profile (alternates: [])
* Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
* Starting existing hyperkit VM for "minikube" ...
* Waiting for the host to be provisioned ...
^[* 正在 Docker '19.03.5' 中准备 Kubernetes v1.17.0…
* 正在启动 Kubernetes ...
* 完成！kubectl 已经配置至 "minikube"
```

### 2-3. 启动 dashboard

```bash
minikube dashboard
```

输出:
```bash
* Verifying dashboard health ...
* Launching proxy ...
* Verifying proxy health ...
* Opening http://127.0.0.1:58368/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

### 2-4. 查看 service 的 URL

```bash
minikube service hello-minikube --url
```

### 2-5. 删除 Minikube 集群

```bash
minikube delete
```

输出:

```bash
* Deleting "minikube" ...
* The "minikube" cluster has been deleted.
```

## 三、补充说明

### 3-1. dashboard 打不开了

在电脑从公司带回家再次来到公司的时候，出现了一些网络问题，比如 dashboard 无法打开，在浏览器回车 dashboard 地址(http://127.0.0.1:63999/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/service?namespace=default)后返回:

```json
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "no endpoints available for service \"http:kubernetes-dashboard:\"",
  "reason": "ServiceUnavailable",
  "code": 503
}
```

这时候建议 stop minikube 后再重新启动.

## 四、参考
https://kubernetes.io/zh/docs/setup/learning-environment/minikube/

