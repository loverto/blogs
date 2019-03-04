---
author: "Mete Atamel"
translator: "loverto"
title: "上手Knative的 - 第1部分"
description: "上手Knative的 - 第1部分"
categories: "translation"
tags: ["Knative""kubernetes"]
date: "2019-04-04T20:18:57+08:00"
---

# 上手Knative的 \- 第1部分

![](https://ws1.sinaimg.cn/large/61411417ly1g0r1zpj6jjj205k05kaa3.jpg)

我最近一直在研究 [Knative](https://github.com/knative/docs) 。这个博客系列由3部分组成，我想解释一下我的学习内容并展示我在GitHub上发布的[Knative Tutorial](https://github.com/meteatamel/knative-tutorial) 中的示例 。

### 什么是Knative？

[Knative](https://github.com/knative/docs) 是一个开源构建块的集合，用于在Kubernetes上运行的serverless容器。

在这一点上，你可能想知道：“Kubernetes，serverless，发生了什么？”但是，当你想到它时，它是有道理的。Kubernetes是非常受欢迎的容器管理平台。serverless是应用程序开发人员想要运行其代码的方式。Knative将两个世界与一组构建块结合在一起。

谈到构建块，它由3个主要组件组成：

*   [Knative Serving](https://github.com/knative/docs/tree/master/serving) 用于快速部署和serverless容器的自动扩展。
*   [Knative Eventing](https://github.com/knative/docs/tree/master/eventing)针对松散耦合的事件驱动服务的。
*   [Knative Build](https://github.com/knative/docs/tree/master/build) 用于无痛的代码到容器的注册表工作流程。

让我们从Knative Serving开始吧。

### 什么是Knative Serving？

简而言之，Knative Serving允许serverless容器的快速部署和自动扩展。您只需指定要部署的容器，Knative将详细说明如何创建该容器并将流量路由到该容器。将serverless容器部署为Knative服务后，您将获得自动扩展，每个配置更改的revision，不同revision之间的流量分配等功能。

### 你好世界服务

要将代码部署为Knative服务，您需要：

1.  包含您的代码并将镜像推送到公共注册表。
2.  创建一个服务yaml文件，告诉Knative在哪里可以找到容器镜像及其具有的任何配置。

在我的Knative教程的 [Hello World服务](https://github.com/meteatamel/knative-tutorial/blob/master/docs/01-helloworldserving.md) 中，我详细描述了这些步骤，但回顾一下，这是最小的Knative服务定义的 `service-v1.yaml` 示例：

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: helloworld-csharp
  namespace: default
spec:
  runLatest:
    configuration:
      revisionTemplate:
        spec:
          container:
            # replace {username} with your DockerHub 
            image: docker.io/{username}/helloworld-csharp:v1
            env:
              - name: TARGET
                value: "C# Sample v1"
```

`runLatest` 意味着我们希望使用指定的容器和配置立即部署最新版本的代码。部署服务：

```bash
kubectl apply -f service-v1.yaml
```

此时，您将看到许多内容已创建。首先，创建Knative服务及其pod。其次，创建配置以捕获Knative服务的当前配置。第三，创建revisions作为当前配置的快照。最后，创建一个路由以将流量定向到新创建的Knative服务：

```bash
kubectl get pod,ksvc,configuration,revision,route
NAME                                                      READY     STATUS    RESTARTS   
pod/helloworld-csharp-00001-deployment-7fdb5c5dc9-wf2bp   3/3       Running   0          

NAME                                            
service.serving.knative.dev/helloworld-csharp   

NAME                                                  
configuration.serving.knative.dev/helloworld-csharp   

NAME                                                   
revision.serving.knative.dev/helloworld-csharp-00001   

NAME                                          
route.serving.knative.dev/helloworld-csharp
```

### 更改配置

在Knative Serving中，每当您更改 [服务](https://github.com/knative/serving/blob/master/docs/spec/spec.md#service) [配置](https://github.com/knative/serving/blob/master/docs/spec/spec.md#configuration) 时 ，它都会创建一个新的 [Revision](https://github.com/knative/serving/blob/master/docs/spec/spec.md#revision) ，它是代码的时间点快照。它还会创建一个新 [路由](https://github.com/knative/serving/blob/master/docs/spec/spec.md#route) ，新版本将开始接收流量。

![](https://ws1.sinaimg.cn/large/61411417ly1g0r207h0mqj20f00bs3z2.jpg)

在我的Knative Tutorial的[更改配置](https://github.com/meteatamel/knative-tutorial/blob/master/docs/03-changeconfig.md) 部分中，您可以看到更改环境变量或Knative服务的容器镜像如何触发创建新revision。

### 流量拆分

您可以在Knative中非常轻松地在服务的不同revision版之间拆分流量。例如，如果要推出新revision版（0004）并将20％的流量路由到该revision版，则可以执行以下操作：

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: helloworld-csharp
  namespace: default
spec:
  release:
    # Ordered list of 1 or 2 revisions. 
    # First revision is traffic target "current"
    # Second revision is traffic target "candidate"
    revisions: ["helloworld-csharp-00001", "helloworld-csharp-00004"]
    rolloutPercent: 20 # Percent [0-99] of traffic to route to "candidate" revision
    configuration:
      revisionTemplate:
        spec:
          container:
            # Replace {username} with your actual DockerHub
            image: docker.io/{username}/helloworld-csharp:v1
            env:
              - name: TARGET
                value: "C# Sample v4"
```

请注意，我们已从 `runLatest` 模式更改为 `release` 模式，以便为我们的服务分割流量。

我的Knative Tutorial的 [Traffic Splitting](https://github.com/meteatamel/knative-tutorial/blob/master/docs/04-trafficsplitting.md) 部分有更多示例，例如如何在现有revision版之间拆分流量。

### 与其他服务集成

Knative Serving非常适合与其他服务整合。例如，您可以将Knative服务用作Twilio等外部服务的webhook。如果您有Twilio号码，您可以回复从Knative服务发送到该号码的SMS消息。

与我的Knative Tutorial的 [Twilio](https://github.com/meteatamel/knative-tutorial/blob/master/docs/05-twiliointegration.md) 部分集成有 细的步骤，但它基本上归结为创建处理Twilio消息的代码：

```
[Route("[controller]")]
public class SmsController : TwilioController
{
    [HttpGet]
    public TwiMLResult Index(SmsRequest incomingMessage)
    {
        var messagingResponse = new MessagingResponse();
        messagingResponse.Message("The Knative copy cat says: " + incomingMessage.Body);
        return TwiML(messagingResponse);
    }
}
```

从中定义Knative服务：

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: twilio-csharp
  namespace: default
spec:
  runLatest:
    configuration:
      revisionTemplate:
        spec:
          container:
            # Replace {username} with your actual DockerHub
            image: docker.io/{username}/twilio-csharp:v1
```

然后将Knative服务指定为Twilio SMS消息的webhook：

![](https://ws1.sinaimg.cn/large/61411417ly1g0r20t7iz5j20m8044t98.jpg)

---

这就是Knative Serving。在下一篇文章中，我将讨论 [Knative Eventing](https://github.com/knative/docs/tree/master/eventing) 。