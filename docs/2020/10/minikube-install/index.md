# 一拳搞定 Kubernetes | 安装 Minikube

本文介绍如何在[本机]^(mac)运行 Minikube 来安装 kubernetes 集群.

<!--more-->

## 一、前言

本篇是整个 kubernetes 的开篇，只有安装好了 kubernetes 集群，我们才能愉快的在容器编排的海洋畅游。

为何选择 minikube？因为它只需要按照官方文档执行简单的命令就可以运行，上手简单，推荐使用。

{{< admonition tip "小贴士" >}}

自行备好 docker， linux 等常规知识

{{< /admonition >}}

## 二、Minikube 操作

> [安装参考](https://kubernetes.io/zh/docs/tasks/tools/install-minikube/)

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

会自动跳转到浏览器：

{{< figure src="dashboard.png" title="dashboard" >}}

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

### 3-2. kubeconfig

{{< admonition tip "小贴士" >}}

这份配置在后续 client-go 连接集群的时候也会讲到，是访问集群的身份认证信息配置文件。

{{< /admonition >}}

如上面启动时候的输出一份配置：

`- KUBECONFIG=/Users/tc/.kube/config.22`

这里 KUBECONFIG 是可以通过环境变量指定的，比如我本地：

`export KUBECONFIG=/Users/tc/.kube/config.22`

里面的内容如下：

```bash
cat /Users/tc/.kube/config.22

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURBVENDQWVtZ0F3SUJBZ0lKQU91U0IwYkU3VGIrTUEwR0NTcUdTSWIzRFFFQkN3VUFNQmN4RlRBVEJnTlYKQkFNTURERXdMakUxTWk0eE9ETXVNVEFlRncweE9URXlNekF3TmpVek1qTmFGdzAwTnpBMU1UY3dOalV6TWpOYQpNQmN4RlRBVEJnTlZCQU1NRERFd0xqRTFNaTR4T0RNdU1UQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQCkFEQ0NBUW9DZ2dFQkFNazV4T2tXb0RXUzdRdFZhOFFCN21DbFFCeC90N2VHdXpEdENYVXYyWU8wcVBJa3RHRVQKaDZkM2dFQ3kzK0ZVVGhDWTF1U2gzYkhDT2wybXJPWnJmN21DelVPdDJZSWQ3QWFVOEVZQzN6dDl6L2d3NTdBWgpWeHE4L3NYZUNQSlBQZXVpOTR0b252cTBSNXNVMm0veTB3UFJMdTFkS052cUV6K05YNFRPUnpCcVpNeVoycXU0CjNMazJCMkNycXJnOXQ1Wks0cU9wQTMwZUxYNHMwVlZUN042RkdFWlRFMTkzdXNLc2dkV2FGb1pJcnh6LzZDbUUKemlDektMcVZiNzAxdWJVZXlCTEU0OER4cVVyZTdtYkoxSWI0VUU5NjE1cDhuSml2UlRJdXZ5aUlxazFieEQ5RwpkaVNEMTdUV1NpTnUwelUyQkJWaEV4OTRYbDcwcENucnhxRUNBd0VBQWFOUU1FNHdIUVlEVlIwT0JCWUVGS29jClFUVE1RYTF1TzZMVk1nVEd0WndsMFNxek1COEdBMVVkSXdRWU1CYUFGS29jUVRUTVFhMXVPNkxWTWdUR3Rad2wKMFNxek1Bd0dBMVVkRXdRRk1BTUJBZjh3RFFZSktvWklodmNOQVFFTEJRQURnZ0VCQUZodlYwVkxGMlRYU0hTLwpGeXBQNG1jMy8vWG9RdWRrUUdNSTJQWDNmNk8xWlBteFVrOGpiMnhiUmJ0aGhTcU5BU0xPSGdSQnRqYjF1UnVaCi96NVliRnhpTEFXdWYyd3FoSzJtRkl4WHg2enF1bExremszRFMxRjZnVWxhYS9hRXoxWW5XdWhUdzArenRtYkkKQkdENnRaTUM3ZDA1YlhseTREQkNtZklPclNNUW14eURvN0IxV1RHWklpd2NwSzN5clJsUEFCc2w5dnJUcit0RQoydUwxaytoYnB3ZjRreVZrNzNRZWdWTGZaK3Q3ZUZRVUpCMDR2V1RSdmtQcVdoTWhqQXRTR1FObG80TVJ3eklWCit2TDUyVGcxZjNXb09LRldyQkQzM1MzOGhLWlpJbzVKUDlPMmxDUU9VRXhRVzZEcmN0UEp0bmhRbHdEUjQ0a04KL1FxNmFNZz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://30.11.44.31:16443
  name: microk8s-cluster
- cluster:
    certificate-authority: /Users/tc/.minikube/ca.crt
    server: https://192.168.64.3:8443
  name: minikube
contexts:
- context:
    cluster: microk8s-cluster
    user: admin
  name: microk8s
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: admin
  user:
    password: d0F2R1hDc2RYUEgxWFA4RGtpWVdFQldHM2xrU0Q2M3czZ3ovK3YzNXJxaz0K
    username: admin
- name: minikube
  user:
    client-certificate: /Users/tc/.minikube/client.crt
    client-key: /Users/tc/.minikube/client.key
```

也可以执行命令 `kubectl config view` 是一样的效果。

如果要访问多个 k8s 集群，可以通过在 `kubectl` 命令加上 `--kubeconfig=[位置]` 来实现。

## 四、参考
https://kubernetes.io/zh/docs/setup/learning-environment/minikube/

https://kubernetes.io/zh/docs/concepts/configuration/organize-cluster-access-kubeconfig/

