# 一拳搞定 dubbo-go | 不想通过标准的配置文件来启动

本文介绍如何玩转 dubbogo 的配置文件

<!--more-->

## 一、前言

dubbogo 是 dubbo 这个 RPC 框架在 go 语言下的延伸，也是各位 Java 开发人员上手 go 的一个优秀开源项目。启动 dubbogo 经常需要配置环境变量，如 `CONF_PROVIDER_FILE_PATH`，`CONF_CONSUMER_FILE_PATH` 两个最典型的例子。在 idea 中启动的时候，每次都需要加这个还是比较麻烦的，其实大部分跑例子的时候只需要关心注册中心的相关信息。

欢迎━(*｀∀´*)ノ亻!点 star (dubbogo)[https://github.com/apache/dubbo-go]

## 二、实战分析

### 2.1 去掉 client 端的配置文件

#### 2.1.1 泛化调用 consumer 端

**应用配置**

你可以定义一个默认的应用配置，如下：

```go
defaultApplication = &dg.ApplicationConfig{
  Organization: "dubbo-go-proxy",
  Name:         "Dubbogo Proxy",
  Module:       "dubbogo proxy",
  Version:      "0.1.0",
  Owner:        "Dubbogo Proxy",
  Environment:  "dev",
}
```

实际使用的时候，应用会有系统环境变量指明应用名称和环境等，上述的几个标准参数应该从业务约定的系统环境变量读取。

**构建 ConsumerConfig**

```go
// dubbogo comsumer config
dgCfg = dg.ConsumerConfig{
  Check:      new(bool),
  Registries: make(map[string]*dg.RegistryConfig, 4),
}
```

**设置默认 ClientConfig**

```go
dubbo.SetClientConf(dubbo.GetDefaultClientConfig())
```

**调用 dubbogo 加载**

```go
dg.Load()
```

## 三、参考

