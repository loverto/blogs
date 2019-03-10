---
author: "Mete Atamel"
translator: "loverto"
title: "上手Knative的 - 第2部分"
description: "上手Knative的 - 第2部分"
categories: "translation"
tags: ["Knative","kubernetes"]
date: "2019-03-05T20:18:57+08:00"
---
# 上手Knative的 - 第2部分

在我[之前的文章中](https://yinlongfei.com/posts/hands-on-knative-part-1/) ，我谈到了 [Knative Serving](https://github.com/knative/docs/tree/master/serving) ，用于快速部署和无服务器容器的自动扩展。 如果您希望HTTP呼叫同步触发您的服务，Knative Serving非常棒。 但是，在无服务器的微服务世界中，异步触发器更常见且更有用。 那是 [Knative Eventing](https://github.com/knative/docs/tree/master/eventing) 发挥作用的时候。

在Hands on Knative系列的第二部分中，我想介绍Knative Eventing并在我的 [Knative Tutorial](https://github.com/meteatamel/knative-tutorial) 中展示如何将其与各种服务集成的 一些示例 。

### 什么是Knative Eventing？

Knative Eventing与Knative Serving携手合作，为松散耦合的事件驱动服务提供原语。 典型的Knative Eventing架构如下所示：

![](https://ws1.sinaimg.cn/large/61411417ly1g0y4uthor3j20m8093q45.jpg)

有4个主要组成部分：

*   **Source** （aka Producer）从实际源读取事件并将下游转发到Channel或更不常见地直接转发到Service。
*   **频道** 从源接收事件，保存到其底层存储（稍后将详细介绍）并向所有订户扇出。
*   **订阅** 桥接通道和服务（或另一个通道）。
*   **服务** （又名消费者）是消费事件流的Knative服务。

让我们更详细地看一下这些。

### 来源，渠道和订阅

Knative Eventing的最终目标是将事件从源路由到服务，并使用我之前提到的原语来实现：Source，Channel和Subscription。

**Source** 从实际源读取事件并将其转发到下游。 截至今天，Knative支持从 [Kubernetes](https://github.com/knative/docs/tree/master/eventing#kuberneteseventsource) ， [GitHub](https://github.com/knative/docs/tree/master/eventing#githubsource) ， [Google Cloud Pub / Sub](https://github.com/knative/docs/tree/master/eventing#gcppubsubsource) ， [AWS SQS主题](https://github.com/knative/docs/tree/master/eventing#awssqssource) ， [容器](https://github.com/knative/docs/tree/master/eventing#awssqssource) 和 [CronJobs中](https://github.com/knative/docs/tree/master/eventing#cronjobsource) 读取事件 。

一旦事件被拉入Knative，它需要保存在内存中或更耐用的地方，如Kafka或Google Cloud Pub / Sub。 **通道** 发生了这种情况**。** 它有多种 [实现](https://github.com/knative/docs/tree/master/eventing/channels) 来支持不同的选项。

来自频道，该活动将传递给所有感兴趣的Knative Services或其他频道。 这可能是一对一或扇出。 **订阅** 确定了此传递的性质，并且充当了Channel和Knative服务之间的桥梁。

现在我们已经了解了Knative事件的基础知识，让我们来看一个具体的例子。

### Hello World Eventing

对于Hello World Eventing，让我们阅读来自Google Cloud Pub / Sub的消息，并将其记录在Knative Service中。 我的 [Hello World Eventing教程](https://github.com/meteatamel/knative-tutorial/blob/master/docs/06-helloworldeventing.md) 包含了所有细节，但回顾一下，这就是我们需要设置的内容：

1.  一个 **GcpPubSubSource** 阅读从谷歌Cloud发布/订阅消息。
2.  甲 **频道** 保存在内存中的该消息。
3.  一个 **订阅** 频道链接到Knative服务。
4.  一个 **Knative服务** 接收消息并注销。

`gcp-pubsub-source.yaml` 定义GcpPubSubSource。 它指向一个名为Pub / Sub的主题 `testing` ，它具有访问Pub / Sub的凭据，还指定应转发的Channel事件，如下所示：

```yaml
apiVersion: sources.eventing.knative.dev/v1alpha1
kind: GcpPubSubSource
metadata:
  name: testing-source
spec:
  gcpCredsSecret:  # A secret in the knative-sources namespace
    name: google-cloud-key
    key: key.json
  googleCloudProject: knative-atamel  # Replace this
  topic: testing
  sink:
    apiVersion: eventing.knative.dev/v1alpha1
    kind: Channel
    name: pubsub-test
```

接下来，我们定义Channel `channel.yaml` 。 在这种情况下，我们只是将消息保存在内存中：

```yaml
apiVersion: eventing.knative.dev/v1alpha1
kind: Channel
metadata:
  name: pubsub-test
spec:
  provisioner:
    apiVersion: eventing.knative.dev/v1alpha1
    kind: ClusterChannelProvisioner
    name: in-memory-channel
```

继续创建源和频道：

```bash
kubectl apply -f gcp-pubsub-source.yaml
kubectl apply -f channel.yaml
```

您可以看到创建了源和通道，并且还创建了一个源窗格：

```bash
kubectl get gcppubsubsource
kubectl get gcppubsubsource
NAME             AGE
testing-source   1m
kubectl get channel
NAME          AGE
pubsub-test   1m
kubectl get pods
NAME                                              READY     STATUS    
gcppubsub-testing-source-qjvnk-64fd74df6b-ffzmt   2/2       Running
```

最后，我们可以创建Knative服务并通过 `subscriber.yaml` 文件中 的订阅将其链接到Channel ：

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: message-dumper-csharp
spec:
  runLatest:
    configuration:
      revisionTemplate:
        spec:
          container:
            # Replace {username} with your actual DockerHub
            image: docker.io/{username}/message-dumper-csharp:v1
---
apiVersion: eventing.knative.dev/v1alpha1
kind: Subscription
metadata:
  name: gcppubsub-source-sample-csharp
spec:
  channel:
    apiVersion: eventing.knative.dev/v1alpha1
    kind: Channel
    name: pubsub-test
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1alpha1
      kind: Service
      name: message-dumper-csharp
```

正如您所看到的， `message-dumper-csharp` 它只是一个常规的Knative服务，但它通过Knative Eventing与其订阅异步触发。

```bash
kubectl apply -f subscriber.yaml
service.serving.knative.dev "message-dumper-csharp" created
subscription.eventing.knative.dev "gcppubsub-source-sample-csharp" configured
```

完成 `kubectl apply` 所有yaml文件后，您可以使用gcloud向Pub / Sub主题发送消息：

```bash
gcloud pubsub topics publish testing --message="Hello World"
```

您应该能够看到为该服务创建的pod：

```bash
kubectl get pods
NAME                                                      READY
gcppubsub-testing-source-qjvnk-64fd74df6b-ffzmt           2/2       Running   0          3m
message-dumper-csharp-00001-deployment-568cdd4bbb-grnzq   3/3       Running   0          30s
```

该服务将Base64编码消息记录在 `Data` ：

```log
info: message_dumper_csharp.Startup[0]
      C# Message Dumper received message: {"ID":"198012587785403","Data":"SGVsbG8gV29ybGQ=","Attributes":null,"PublishTime":"2019-01-21T15:25:58.25Z"}
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 29.9881ms 200
```

查看我的 [Hello World Eventing教程](https://github.com/meteatamel/knative-tutorial/blob/master/docs/06-helloworldeventing.md) ，了解有关步骤和实际代码的更多详细信息。

### 与云存储和Vision API集成

当您尝试以无缝方式连接完全不相关的服务时，Knative Eventing真正闪耀。 在我的 [Integrate with Vision API教程中](https://github.com/meteatamel/knative-tutorial/blob/master/docs/08-visioneventing.md) ，我将展示如何使用Knative Eventing连接Google Cloud Storage和Google Cloud Vision API。

云存储是一种全球可用的数据存储服务。 可以将存储桶配置为在保存图像时发出发布/订阅消息。 然后我们可以使用Knative Eventing监听这些Pub / Sub消息并将它们传递给Knative Service。 在服务中，我们使用图像进行Vision API调用，并使用机器学习从中提取标签。 所有细节都在教程中解释，但我想在这里指出一些事情。

首先，Knative默认阻止所有出站流量。 这意味着您默认情况下甚至无法通过Knative Service进行Vision API调用。 这最初让我感到惊讶，因此请确保 [配置网络出站访问](https://github.com/meteatamel/knative-tutorial/blob/master/docs/06-helloworldeventing.md#configuring-outbound-network-access) 。

其次，每当图像保存到云存储时，它都会发出 [CloudEvents](https://github.com/meteatamel/knative-tutorial/blob/master/docs/08-visioneventing.md#define-cloud-events) 。 Knative Eventing通常适用于CloudEvents。 您需要将传入的请求解析为CloudEvents并提取所需的信息，例如事件类型和图像文件的位置：

```js
var cloudEvent = JsonConvert.DeserializeObject<CloudEvent>(content);
var eventType = cloudEvent.Attributes["eventType"];
var storageUrl = ConstructStorageUrl(cloudEvent);
```

有了这些信息，可以很容易地为图像构建存储URL并使用该URL进行Vision API调用。 [这里](https://github.com/meteatamel/knative-tutorial/blob/master/docs/08-visioneventing.md#add-vision-api) 解释 [了](https://github.com/meteatamel/knative-tutorial/blob/master/docs/08-visioneventing.md#add-vision-api) 完整的源代码 [，](https://github.com/meteatamel/knative-tutorial/blob/master/docs/08-visioneventing.md#add-vision-api) 但这里是相关部分：

```js
var visionClient = ImageAnnotatorClient.Create();
var labels = await visionClient.DetectLabelsAsync(Image.FromUri(storageUrl), maxResults: 10);
```

代码准备好后，我们可以通过定义一个来将我们的服务挂钩到Knative Eventing `subscriber.yaml` 。 它与之前非常相似。 我们正在重新使用现有的Source和Channel，因此我们不必重新创建它们。 我们只是使用Vision API容器创建一个新的订阅指向我们的新Knative服务：

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: vision-csharp
spec:
  runLatest:
    configuration:
      revisionTemplate:
        spec:
          container:
            # Replace {username} with your actual DockerHub
            image: docker.io/{username}/vision-csharp:v1
---
apiVersion: eventing.knative.dev/v1alpha1
kind: Subscription
metadata:
  name: gcppubsub-source-vision-csharp
spec:
  channel:
    apiVersion: eventing.knative.dev/v1alpha1
    kind: Channel
    name: pubsub-test
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1alpha1
      kind: Service
      name: vision-csharp
```

创建所有内容后 `kubectl apply` ，无论何时将图像保存到云存储桶，都应该看到该图像的Knative服务日志标签。

例如，我有一张来自我最喜欢的地方的照片：

![](https://ws1.sinaimg.cn/large/61411417ly1g0y4vh3ccrj20m80t4hdt.jpg)

里约热内卢的伊帕内玛海滩

当我将该图像保存到存储桶时，我可以在日志中看到Vision API的以下标签：

```log
info: vision_csharp.Startup[0]
      This picture is labelled: Sea,Coast,Water,Sunset,Horizon
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 1948.3204ms 200
```

如您所见，我们使用Knative Eventing将一个服务（云存储）连接到另一个服务（Vision API）。 这只是一个例子，但可能性是无限的。 在本教程的 [Integrate with Translation API](https://github.com/meteatamel/knative-tutorial/blob/master/docs/07-translationeventing.md) 部分中，我将展示如何将Pub / Sub连接到Translation API。

---

这就是Knative Eventing。 在本系列的下一篇和最后一篇文章中，我将讨论Knative Build。