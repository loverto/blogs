---
date: "2016-02-25T20:18:57+08:00"
title: 设计实用RESTful API的最佳实践
---
# [设计实用RESTful API的最佳实践]

您的数据模型已开始稳定，您可以为您的网络应用创建公共API。您意识到，一旦API发布并且希望尽可能正确地获得API，很难对其进行重大更改。现在，互联网上对API设计的看法并不缺乏。但是，由于没有一个广泛采用的标准适用于所有情况，因此您有许多选择：您应该接受哪些格式？ 你应该如何认证？ 你的API应该被版本化吗？

在为 [Enchant](http://www.enchant.com) （ [Zendesk Alternative](http://www.enchant.com/zendesk-alternative) ） 设计API时 ，我试图为这些问题提出实用的答案。我的目标是为Enchant设计的[API](http://dev.enchant.com/api/v1) 是易于使用，易于采用，并具有足够的灵活性，以[内部测试](http://en.wikipedia.org/wiki/Eating_your_own_dog_food) 为我们自己的用户界面。

## 目录

* [API是开发人员的用户界面 \- 所以要付出一些努力让它变得愉快](#requirements)
* [使用RESTful URL和操作](#restful)
* [在任何地方使用SSL，没有例外](#ssl)
* [API只有它的文档一样好 \- 所以有很好的文档](#docs)
* [版本通过URL，而不是通过标题](#versioning)
* [使用查询参数进行高级筛选，排序和搜索](#advanced-queries)
* [提供一种方法来限制从API返回的字段](#limiting-fields)
* [从POST，PATCH和PUT请求中返回一些有用的东西](#useful-post-responses)
* [HATEOAS还不实用](#hateoas)
* [尽可能使用JSON，只有在必要时才使用XML](#json-responses)
* [你应该使用带有JSON的camelCase，但是snake\_case更容易阅读20％](#snake-vs-camel)
* [默认打印漂亮并确保支持gzip](#pretty-print-gzip)
* [默认情况下不要使用响应封装](#envelope)
* [考虑将JSON用于POST，PUT和PATCH请求主体](#json-requests)
* [使用链接标头进行分页](#pagination)
* [提供自动加载相关资源表示的方法](#autoloading)
* [提供一种覆盖HTTP方法的方法](#method-override)
* [为速率限制提供有用的响应标头](#rate-limiting)
* [使用基于令牌的身份验证，通过需要委派的OAuth2进行传输](#authentication)
* [包括便于缓存的响应标头](#caching)
* [定义消耗品错误有效负载](#errors)
* [有效使用HTTP状态代码](#http-status)

## API的关键要求{#requirements}

网上发现的许多API设计观点都是围绕模糊标准的主观解释而不是在现实世界中有意义的学术讨论。我在这篇文章中的目标是描述为当今的Web应用程序设计的实用API的最佳实践。如果感觉不对，我不会尝试满足标准。为了帮助指导决策制定过程，我已经写下了API必须要求的一些要求：

* 它应该使用*有意义的* Web标准
* 它应该对开发人员友好，并可通过浏览器地址栏进行探索
* 它应该简单，直观和一致，以使采用不仅容易而且令人愉快
* 它应该提供足够的灵活性来为大多数 [enchant](http://www.enchant.com) UI 提供动力
* 它应该是有效的，同时保持与其他要求的平衡

API是开发人员的UI \- 就像任何UI一样，确保仔细考虑用户体验非常重要！

## 使用RESTful URL和操作

如果有一件事被广泛采用，那就是RESTful原则。这些是 [Roy Fielding](http://roy.gbiv.com/) 在他关于基于[网络的软件架构的](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)论文的[第5章](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)中首次介绍的。

[REST](https://en.wikipedia.org/wiki/Representational_state_transfer) 的关键原则涉及将API分离为逻辑资源。使用HTTP请求操纵这些资源，其中方法（GET，POST，PUT，PATCH，DELETE）具有特定含义。

**但是我可以创造什么资源呢？**嗯，这些应该是从API消费者的角度来看有意义的[名词（不是动词！）](https://blog.apigee.com/detail/restful_api_design_nouns_are_good_verbs_are_bad)。虽然您的内部模型可能整齐地映射到资源，但它不一定是一对一映射。这里的关键是不要将不相关的实现细节泄露给您的API！一些enchant的名词将是*门票*，*用户*和*团体*。

定义资源后，您需要确定哪些操作适用于它们以及这些操作将如何映射到您的API。RESTful原则提供了使用HTTP方法处理 [CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) 操作的策略，映射如下：

* GET /ticket \- 检索故障单列表
* GET /tickets/12 \- 检索特定故障单
* POST /tickets \- 创建新票证
* PUT /tickets/12 \- 更新门票＃12
* PATCH /tickets/12 \- 部分更新机票＃12
* DELETE /tickets/12 \- 删除12号门票

REST的优点在于您利用现有的HTTP方法在单个/ticket 端点上实现重要功能。没有方法命名约定，URL结构清晰明了。*REST FTW！*

**端点名称应该是单数还是复数？** 保持简单的规则适用于此处。尽管您的内部语法学家会告诉您使用复数描述资源的单个实例是错误的，但实用的答案是保持URL格式一致并始终使用复数。不必处理奇怪的复数化（person/people，goose/geese）使API消费者的生活变得更好并且API提供者更容易实现（因为大多数现代框架本身将处理 /ticket 和 /ticket/12 下共同控制者）。

**但是你如何处理关系？** 如果关系只能存在于另一个资源中，则RESTful原则可提供有用的指导。我们来看一个例子吧。[Enchant](http://www.enchant.com)中的票证包含许多消息。这些消息可以逻辑映射到 /ticket 端点，如下所示：

* GET /tickets/12/messages \- 检索故障单＃12的消息列表
* GET /tickets/12/messages/5 \- 检索＃12票据的消息＃5
* POST /tickets/12/messages \- 在故障单＃12中创建一条新消息
* PUT /tickets/12/messages/5 \- 更新机票＃12的消息＃5
* PATCH /tickets/12/messages/5 \- 部分更新机票＃12的消息＃5
* DELETE /tickets/12/messages/5 \- 删除＃12票据的消息＃5

或者，如果关系可以独立于资源而存在，则仅在资源的输出表示中包含其标识符是有意义的。然后，API使用者必须达到关系的端点。但是，如果通常与资源一起请求关系，则API可以提供自动嵌入关系表示的功能，并避免对API的第二次命中。

**那些不适合CRUD操作世界的行为呢？**

这是事情变得模糊的地方。有很多方法：

1. 将操作重组为显示为资源字段。如果操作不采用参数，则此方法有效。例如，*激活* 动作可以映射到布尔激活的字段，并通过PATCH更新到资源。
2. 将其视为具有RESTful原则的子资源。例如，GitHub的API可以让你 [添加星号标记](http://developer.github.com/v3/gists/#star-a-gist) 与 PUT /gists/:id/star 和 [取消星号标记](http://developer.github.com/v3/gists/#unstar-a-gist) 与 DELETE /gists/:id/star。
3. 有时你真的无法将动作映射到合理的RESTful结构。例如，多资源搜索实际上没有意义应用于特定资源的端点。在这种情况下，即使它不是资源 ，/search 也会最有意义。这没关系 \- 只需从API使用者的角度做正确的事情，并确保明确记录以避免混淆。

## SSL无处不在 \- 始终如一

始终使用SSL。没有例外。今天，您的Web API可以从任何有互联网的地方访问（如图书馆，咖啡馆，机场等）。并非所有这些都是安全的。许多人根本不加密通信，如果身份验证凭据被劫持，则允许轻松窃听或模拟。

始终使用SSL的另一个好处是保证加密通信简化了身份验证工作 \- 您可以使用简单的访问令牌，而不必签署每个API请求。

需要注意的一点是对API URL的非SSL访问。难道 **不是** 这些重定向到SSL同行。反而投掷一个硬错误！ 您想要的最后一件事是配置不当的客户端将请求发送到未加密的端点，只是为了静默地重定向到实际的加密端点。

## 文档

API只能与其文档一样好。文档应该易于查找和公开访问。大多数开发人员会在尝试任何集成工作之前检查文档。当文档隐藏在PDF文件中或需要登录时，它们不仅难以找到而且不易搜索。

文档应该显示完整的请求/响应周期的示例。优选地，请求应该是可接受的示例 \- 可以粘贴到浏览器中的链接或可以粘贴到终端中的卷曲示例。[GitHub](http://developer.github.com/v3/gists/#list-gists) 和 [Stripe](https://stripe.com/docs/api) 做得很好。

一旦您发布了公共API，您就承诺不会在不事先通知的情况下破坏它。文档必须包含任何弃用计划和有关外部可见API更新的详细信息。更新应通过博客（即更改日志）或邮件列表（最好是两者！）进行。

## 版本

始终为您的API提供版本。版本控制可帮助您更快地进行迭代，并防止无效请求命中更新的端点。它还有助于平滑任何主要的API版本转换，因为您可以继续提供一段时间的旧API版本。

关于 [API版本是应该包含在URL还是标题中，](http://stackoverflow.com/questions/389169/best-practices-for-api-versioning) 有不一致的看法。从学术上讲，它可能应该在标题中。但是，版本需要在URL中以确保浏览器跨版本的资源可用性（请记住本文顶部指定的API要求？）。

我非常喜欢 [Stripe对API版本化的方法](https://stripe.com/docs/api#versioning) \- URL具有主版本号（v1），但API具有基于日期的子版本，可以使用自定义HTTP请求头来选择。在这种情况下，主要版本提供API整体的结构稳定性，而子版本则考虑较小的更改（字段弃用，端点更改等）。

API永远不会完全稳定。变化是不可避免的。重要的是如何管理这种变化。对于许多API而言，记录良好且公布的多月弃用计划可能是可接受的做法。这取决于行业和API的可能消费者的合理性。

## 结果过滤，排序和搜索

最好保持基本资源URL尽可能精简。复杂的结果过滤器，排序要求和高级搜索（当限制为单一类型的资源时）都可以轻松实现为基本URL之上的查询参数。让我们更详细地看一下这些：

**过滤** ：对实现过滤的每个字段使用唯一查询参数。例如，从 /tickets 端点 请求票证列表时 ，您可能希望将这些仅限于处于打开状态 的票证列表。这可以通过 GET /ticket?state=open 等请求来完成。这里，state 是一个实现过滤器的查询参数。

**排序** ：与过滤类似，通用参数 排序 可用于描述排序规则。通过让sort参数包含逗号分隔字段列表来容纳复杂的排序要求，每个字段都有一个可能的一元否定以暗示降序排序。我们来看一些例子：

* GET `/ticket?sort=-priority` \- 按优先级降序检索故障单列表
* GET `/tickets?sort=-priority,created_at` \- 按优先级降序检索故障单列表。在特定优先级内，首先订购旧票

**搜索** ：有时基本的过滤器是不够的，你需要全文搜索的力量。也许您已经在使用 [ElasticSearch](http://www.elasticsearch.org) 或其他基于 [Lucene](http://lucene.apache.org) 的搜索技术。当全文搜索用作检索特定类型资源的资源实例的机制时，它可以作为资源端点上的查询参数在API上公开。我们说 q。搜索查询应直接传递给搜索引擎，API输出的格式应与普通列表结果相同。

将这些组合在一起，我们可以构建如下查询：

* GET `/tickets?sort=-updated_at` \- 检索最近更新的票证
* GET `/tickets?state=closed&sort=-updated_at` \- 检索最近关闭的票证
* GET `/tickets?q=return&state=open&sort=-priority,created_at` \- 检索提到“return”一词的最高优先级开放票

**常见查询的别名**

为了使普通消费者的API体验更加愉快，可以考虑将一组条件打包成易于访问的RESTful路径。例如，上面最近关闭的票证查询可以打包为 GET `/tickets/recently_closed`

## 限制API返回哪些字段

API使用者并不总是需要资源的完整表示。选择和选择返回字段的能力在允许API使用者最小化网络流量并加速他们自己的API使用方面有很长的路要走。

使用 字段 查询参数，该参数采用逗号分隔的字段列表来包含。例如，以下请求将检索足够的信息以显示打开的票证的已排序列表：

GET `/tickets?fields=id,subject,customer_name,updated_at&state=open&sort=-updated_at`

## 更新和创建应返回资源表示

PUT，POST或PATCH调用可以对基础资源的字段进行修改，这些字段不是所提供参数的一部分（例如：created\_at或updated\_at timestamps）。为了防止API使用者必须再次访问API以获得更新的表示，请让API返回更新（或创建）的表示作为响应的一部分。

如果POST导致创建，请使用 [HTTP 201状态代码](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.5) 并包含指向新资源的URL的 [Location标头](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30)。

## 你应该HATEOAS吗？

关于API使用者是否应该创建链接或者是否应该向API提供链接，有很多不一致的意见。RESTful设计原则指定 [HATEOAS](https://blog.apigee.com/detail/hateoas_101_introduction_to_a_rest_api_style_video_slides) ，它粗略地声明应该在输出表示附带的元数据中定义与端点的交互，而不是基于带外信息。

虽然网络通常适用于HATEOAS类型的原则（我们进入网站的首页并根据我们在页面上看到的链接进行链接），但我认为我们还没准备好在API上使用HATEOAS。浏览网站时，会在运行时决定点击哪些链接。但是，使用API​​时，会在编写API集成代码时（而不是在运行时）决定将发送哪些请求。决定是否可以推迟到运行时间？ 当然，由于代码仍然无法在不中断的情况下处理重要的API更改，因此没有太多可以获得这条路线。也就是说，我认为HATEOAS很有希望但尚未准备好迎接黄金时段。必须付出更多努力来围绕这些原则定义标准和工具，以充分实现其潜力。

目前，最好假设用户可以访问文档并在输出表示中包含资源标识符，API使用者在制作链接时将使用这些标识符。坚持使用标识符有一些优点 \- 通过网络流动的数据被最小化，API使用者存储的数据也被最小化（因为它们存储小标识符而不是包含标识符的URL）。

此外，鉴于此帖子提倡URL中的版本号，从长远来看，API消费者存储资源标识符而不是URL更有意义。毕竟，标识符在不同版本中是稳定的，但代表它的URL不是！

## 只响应JSON

是时候将XML抛弃在API中了。它冗长，难以解析，难以阅读，其数据模型与大多数编程语言建模数据的方式不兼容，当您的输出表示的主要需求是从内部表示序列化时，其可扩展性优势无关紧要。

我不会花费太多精力来解释上述情况，因为看起来其他人（ [YouTube](http://apiblog.youtube.com/2012/12/the-simpler-yet-more-powerful-new.html) ，[Twitter](https://dev.twitter.com/docs/api/1.1/overview#JSON_support_only) 和 [Box](http://developers.blog.box.com/2012/12/14/v2_api/) ）已经开始放弃了XML。

我将给您留下以下Google趋势图表（ [XML API与JSON API](http://www.google.com/trends/explore?q=xml+api#q=xml%20api%2C%20json%20api&cmpt=q) ）作为思考的食物：

![](https://www.vinaysahni.com/images/201305-xml-vs-json-api.png)

但是，如果您的客户群由大量企业客户组成，您可能会发现自己无论如何都必须支持XML。如果你必须这样做，你会发现自己有一个新问题：

**媒体类型是否应根据Accept标头或基于URL进行更改？** 为确保浏览器可利用性，它应该在URL中。这里最明智的选择是将 .json 或 .xml 扩展名 附加 到端点URL。

## snake\_case vs camelCase用于字段名称

如果您使用JSON（ *JavaScript* Object Notation）作为主要表示格式，那么“正确”的做法是遵循JavaScript命名约定 \- 这意味着camelCase用于字段名称！ 如果你再以各种语言构建客户端库的路线，最好在其中使用惯用的命名约定 \- 用于C＃和Java的camelCase，用于python和ruby的snake\_case。

深思熟虑：我一直觉得 [snake\_case](http://en.wikipedia.org/wiki/Snake_case) 比JavaScript的 [camelCase](http://en.wikipedia.org/wiki/CamelCase) 惯例更容易阅读。直到现在，我还没有任何证据来支持我的直觉。基于 2010年 [对camelCase和snake\_case](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?tp=&arnumber=5521745) （ [PDF](http://www.cs.kent.edu/~jmaletic/papers/ICPC2010-CamelCaseUnderScoreClouds.pdf) ） 的 [眼动追踪研究](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?tp=&arnumber=5521745) ，**snake\_case比camelCase更容易阅读20％** ！ 这对可读性的影响会影响API可探索性和文档中的示例。

许多流行的JSON API使用snake\_case。我怀疑这是由于序列化库遵循他们使用的基础语言的命名约定。也许我们需要让JSON序列化库处理命名约定转换。

## 默认打印漂亮并确保支持gzip

从浏览器中查看，提供空白空间压缩输出的API并不是很有趣。虽然可以提供某种查询参数（例如 `?pretty=true` ）来启用漂亮打印，但默认情况下相当打印的API更加平易近人。额外数据传输的成本可以忽略不计，特别是当您与不实现gzip的成本进行比较时。

考虑一些用例：如果API使用者正在调试并且他们的代码打印出从API接收的数据该怎么办 \- 默认情况下它是可读的。或者，如果消费者抓住他们的代码生成的URL并直接从浏览器点击它 \- 默认情况下它是可读的。这些都是小事。使API变得愉快的小东西！

**但是所有额外的数据传输呢？**

让我们用一个真实世界的例子看看这个。我 [从GitHub的API中](https://api.github.com/users/veesahni) 提取了一些 [数据](https://api.github.com/users/veesahni) ，默认使用漂亮的打印。我还将进行一些gzip比较：

```bash

$ curl https://api.github.com/users/veesahni > with-whitespace.txt
$ ruby -r json -e 'puts JSON JSON.parse(STDIN.read)' < with-whitespace.txt > without-whitespace.txt
$ gzip -c with-whitespace.txt > with-whitespace.txt.gz
$ gzip -c without-whitespace.txt > without-whitespace.txt.gz

```

输出文件具有以下大小：

* without\-whitespace.txt \- 1252个字节
* with\-whitespace.txt \- 1369个字节
* without\-whitespace.txt.gz \- 496个字节
* with\-whitespace.txt.gz \- 509个字节

在这个例子中，当gzip不在播放时，空格将输出大小增加了8.5％，当gzip处于播放状态时，输出大小增加了2.6％。另一方面，**gzipping本身** 的行为**提供了超过60％的带宽节省**。由于漂亮打印的成本相对较小，因此最好默认打印，并确保支持gzip压缩！

为了进一步说明这一点，Twitter发现在其 [Streaming API](https://dev.twitter.com/docs/streaming-apis) 上启用gzip压缩时[节省](https://dev.twitter.com/blog/announcing-gzip-compression-streaming-apis) 了 [80％（在某些情况下）](https://dev.twitter.com/blog/announcing-gzip-compression-streaming-apis)。Stack Exchange甚至 [永远不会返回未压缩的响应](https://api.stackexchange.com/docs/compression) ！[](https://dev.twitter.com/docs/streaming-apis)[](https://api.stackexchange.com/docs/compression)

## 默认情况下不要使用封装，但在需要时可以使用

许多API将其响应包装在封装中，如下所示：

```json

{
 "data" : {
  "id" : 123,
  "name" : "John"
 }
}

```

这样做有几个理由 \- 它可以很容易地包含额外的元数据或分页信息，一些REST客户端不允许轻松访问HTTP标头，[JSONP](http://en.wikipedia.org/wiki/JSONP) 请求无法访问HTTP标头。但是，随着 [CORS](http://www.w3.org/TR/cors/) 和 [RFC 5988](http://tools.ietf.org/html/rfc5988#page-6) 的 [链接头](http://tools.ietf.org/html/rfc5988#page-6) 快速采用的标准，[封装](http://tools.ietf.org/html/rfc5988#page-6) 开始变得不必要了。

我们可以通过默认保留封装并仅在特殊情况下包络来证明API。

**如何在特殊情况下使用封装？**

有两种情况需要封装 \- 如果API需要通过JSONP支持跨域请求，或者客户端无法使用HTTP标头。

JSONP请求带有一个额外的查询参数（通常名为 callback 或 jsonp ），表示回调函数的名称。如果存在此参数，则API应切换到完整封装模式，它始终以200 HTTP状态代码响应并传递JSON有效内容中的实际状态代码。与响应一起传递的任何其他HTTP标头应映射到JSON字段，如下所示：

```javascript

callback_function({
 status_code: 200,
 next_page: "https://..",
 response: {
  ... actual JSON response body ...
 }
})

```

同样，为了支持有限的HTTP客户端，允许一个特殊的查询参数 ？envelope = true ，它将触发完全包络（没有JSONP回调函数）。

## JSON编码POST，PUT和PATCH主体

如果您正在阅读本文中的方法，那么您已经为所有API输出采用了JSON。我们考虑使用JSON进行API输入。

许多API在其API请求正文中使用URL编码。URL编码正是它的声音 \- 请求使用与用于在URL查询参数中编码数据的约定相同的约定来编码键值对的实体。这很简单，得到广泛支持并完成工作。

但是，URL编码有一些问题会导致问题。它没有数据类型的概念。这会强制API解析字符串中的整数和布尔值。此外，它没有层次结构的真实概念。虽然有一些约定可以用键值对构建一些结构（比如将\[\]附加到表示数组的键），但这与JSON的本机层次结构无法比较。

如果API很简单，URL编码就足够了。但是，复杂的API应该坚持使用JSON来获取API输入。无论哪种方式，选择一个并在整个API中保持一致。

接受JSON编码的POST，PUT和PATCH请求的API还应该要求将 `Content-Type` 标头设置为 application/json 或抛出415 Unsupported Media Type HTTP状态代码。

## 分页

爱好封装的API通常包括封装本身的分页数据。而且我不怪他们 \- 直到最近，没有更多更好的选择。今天包含分页细节的正确方法是使用 [RFC 5988引入](http://tools.ietf.org/html/rfc5988#page-6) 的 [链接头](http://tools.ietf.org/html/rfc5988#page-6)。

使用Link头的API可以返回一组现成的链接，因此API使用者不必自己构建链接。当分页 [基于游标](https://developers.facebook.com/docs/reference/api/pagination/) 时，这尤其重要。这是一个正确使用的链接头的示例，从 [GitHub](http://developer.github.com/v3/#pagination) 的文档中获取：

```http

Link: <https://api.github.com/user/repos?page=3&per_page=100>; rel="next", <https://api.github.com/user/repos?page=50&per_page=100>; rel="last"

```

但这不是一个完整的解决方案，因为许多API都希望返回额外的分页信息，例如可用结果总数的计数。需要发送计数的API可以使用自定义HTTP标头，如 X\-Total\-Count。

## 自动加载相关的资源表示

在许多情况下，API使用者需要从所请求的资源加载与（或引用）相关的数据。不要求消费者针对该信息重复地访问API，而是允许相关数据与原始资源一起按需返回和加载，从而显着提高效率。

但是，由于这 [违反了一些RESTful原则](http://idbentley.com/blog/2013/03/14/should-restful-apis-include-relationships/) ，我们可以通过仅基于嵌入（或扩展）查询参数 来最小化我们的偏差。

在这种情况下，embed 将是一个逗号分隔的要嵌入的字段列表。点符号可用于指代子字段。例如：

GET `/tickets/12?embed=customer.name,assigned_user`

这将返回包含嵌入其他详细信息的票证，例如：

```json

{
 "id" : 12,
 "subject" : "I have a question!",
 "summary" : "Hi, ....",
 "customer" : {
  "name" : "Bob"
 },
 assigned_user: {
  "id" : 42,
  "name" : "Jim",
 }
}

```

当然，实现这样的事情的能力实际上取决于内部复杂性。这种嵌入很容易导致 [N + 1选择问题](http://stackoverflow.com/questions/97197/what-is-the-n1-selects-issue)。

## 覆盖HTTP方法

某些HTTP客户端只能使用简单的GET和POST请求。为了提高对这些有限客户端的可访问性，API需要一种覆盖HTTP方法的方法。虽然这里没有任何硬标准，但流行的约定是接受一个请求头 `X-HTTP-Method-Override` ，其字符串值包含PUT，PATCH或DELETE之一。

请注意，**只** 应在POST请求中接受 覆盖标头。GET请求永远不应该 [更改服务器上的数据](http://programmers.stackexchange.com/questions/188860/why-shouldnt-a-get-request-change-data-on-the-server) ！

## 限速

为了防止滥用，标准做法是为API添加某种速率限制。[RFC 6585](http://tools.ietf.org/html/rfc6585) 引入了HTTP状态代码 [429 Too Many Requests](http://tools.ietf.org/html/rfc6585#section-4) 以适应这种情况。

但是，在消费者真正达到限制之前通知他们的限制是非常有用的。这是一个目前缺乏标准但是有许多 [使用HTTP响应头](http://stackoverflow.com/questions/16022624/examples-of-http-api-rate-limiting-http-response-headers) 的 [流行约定的领域](http://stackoverflow.com/questions/16022624/examples-of-http-api-rate-limiting-http-response-headers)。

至少包括以下标题（使用Twitter的 [命名约定，](https://dev.twitter.com/docs/rate-limiting/1.1) 因为标题通常没有中间词大小写）：

* `X-Rate-Limit-Limit` \- 当前期间允许的请求数
* `X-Rate-Limit-Remaining` \- 当前时间段内剩余请求的数量
* `X-Rate-Limit-Reset` \- 当前时间段内剩余的秒数

**为什么还剩下秒数而不是X\-Rate\-Limit\-Reset的时间戳？**

时间戳包含各种有用但不必要的信息，如日期和可能的时区。API消费者真的只想知道他们什么时候可以再次发送请求，并且秒数回答这个问题，最后只需要额外的处理。它还避免了与 [时钟偏差](http://en.wikipedia.org/wiki/Clock_skew) 相关的问题。

某些API使用UNIX时间戳（自纪元以来的秒数）进行X速率限制重置。不要这样做！

**为什么使用UNIX时间戳进行X\-Rate\-Limit\-Reset是不好的做法？**

在 [HTTP规范](http://www.w3.org/Protocols/rfc2616/rfc2616.txt) 已经 [指定](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.3) 使用 [RFC 1123的日期格式](http://www.ietf.org/rfc/rfc1123.txt) （目前正在使用的 [日期](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.18) ，[如果\-Modified\-Since的](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.25) 和 [上次修改](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.29) 的HTTP头）。如果我们要指定一个采用某种时间戳的新HTTP标头，它应该遵循RFC 1123约定而不是使用UNIX时间戳。

## 认证

RESTful API应该是无状态的。这意味着请求身份验证不应依赖于cookie或会话。相反，每个请求都应附带一些排序身份验证凭据。

通过始终使用SSL，可以将身份验证凭据简化为随机生成的访问令牌，该令牌在HTTP Basic Auth的用户名字段中提供。关于这一点的好处是它完全可以浏览浏览器 \- 如果 从服务器 收到 401 Unauthorized 状态代码，浏览器将弹出一个提示凭据的提示。

但是，这种基于身份验证令牌的身份验证方法只有在用户将令牌从管理界面复制到API使用者环境的情况下才可接受。如果无法做到这一点，则应使用 [OAuth 2](http://oauth.net/2/) 向第三方提供安全令牌转移。OAuth 2使用 [承载令牌](http://tools.ietf.org/html/rfc6750) ，并且还依赖于SSL进行底层传输加密。

需要支持JSONP的API需要第三种身份验证方法，因为JSONP请求无法发送HTTP Basic Auth凭证或承载令牌。在这种情况下，可以使用 特殊查询参数 access\_token。注意：使用查询参数作为令牌时存在固有的安全问题，因为大多数Web服务器在服务器日志中存储查询参数。

对于它的价值，上述所有三种方法都只是通过API边界传输令牌的方法。实际的底层令牌本身可能是相同的。

## 高速缓存

HTTP提供了一个内置的缓存框架！ 您所要做的就是包含一些额外的出站响应标头，并在收到一些入站请求标头时进行一些验证。

有两种方法： [ETag](http://en.wikipedia.org/wiki/HTTP_ETag) 和 [Last\-Modified](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.29)

**ETag** ：生成响应时，包含一个HTTP头标记ETag，其中包含表示的哈希或校验和。只要输出表示发生更改，此值就会更改。现在，如果入站HTTP请求包含 具有匹配ETag值 的 If\-None\-Match 标头，则API应返回 304 Not Modified 状态代码而不是资源的输出表示。

**Last\-Modified** ：这基本上与ETag类似，只是它使用时间戳。响应头 Last\-Modified 包含 [RFC 1123](http://www.ietf.org/rfc/rfc1123.txt) 格式 的时间戳，该时间戳 根据 If\-Modified\-Since进行 验证。请注意，HTTP规范有 [3种不同的可接受日期格式](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.3) ，服务器应准备接受其中任何一种。

## 错误

就像HTML错误页面向访问者显示有用的错误消息一样，API应该以已知的可消费格式提供有用的错误消息。错误的表示应该与任何资源的表示没有区别，只有它自己的字段集。

API应始终返回合理的HTTP状态代码。API错误通常分为两类：客户端问题的400系列状态代码和服务器问题的500系列状态代码。API应该至少标准化所有400系列错误都带有可消耗的JSON错误表示。如果可能（即，如果负载平衡器和反向代理可以创建自定义错误主体），则应扩展到500系列状态代码。

JSON错误正文应该为开发人员提供一些东西 \- 一个有用的错误消息，一个唯一的错误代码（可以在文档中查找更多详细信息）以及可能的详细描述。像这样的东西的JSON输出表示如下所示：

```json

{
 "code" : 1234,
 "message" : "Something bad happened :(",
 "description" : "More details about the error here"
}

```

PUT，PATCH和POST请求的验证错误需要字段细分。最好通过使用固定的顶级错误代码进行验证失败并在其他 错误 字段中 提供详细错误来建模 ，如下所示：

```json

{
 "code" : 1024,
 "message" : "Validation Failed",
 "errors" : [
  {
   "code" : 5432,
   "field" : "first_name",
   "message" : "First name cannot have fancy characters"
  },
  {
    "code" : 5622,
    "field" : "password",
    "message" : "Password cannot be blank"
  }
 ]
}

```

## HTTP状态代码

HTTP定义了一堆 可以从API返回 的 [有意义的状态代码](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes)。可以利用这些来帮助API消费者相应地路由他们的响应。我已经策划了一份你肯定应该使用的短名单：

* 200 OK \- 响应成功的GET，PUT，PATCH或DELETE。也可以用于不会导致创建的POST。
* 201 Created \- 对POST的响应，导致创建。应与 指向新资源位置的 [Location标头](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30) 结合使用
* 204 No Content \- 对不会返回正文的成功请求的响应（如DELETE请求）
* 304 Not Modified \- 在HTTP缓存标头播放时使用
* 400错误请求 \- 请求格式错误，例如正文无法解析
* 401 Unauthorized \- 未提供或无效的身份验证详细信息时。如果从浏览器使用API​​，也可以触发auth弹出窗口
* 403 Forbidden \- 身份验证成功但经过身份验证的用户无权访问资源
* 404 Not Found \- 请求不存在的资源时
* 405不允许的方法 \- 当请求的HTTP方法不允许经过身份验证的用户时
* 410 Gone \- 表示此端点的资源不再可用。有用作旧API版本的一揽子响应
* 415不支持的媒体类型 \- 如果作为请求的一部分提供了错误的内容类型
* 422不可处理的实体 \- 用于验证错误
* 429请求过多 \- 请求因速率限制而被拒绝

## 综上所述

API是开发人员的用户界面。努力确保它不仅功能齐全，而且使用愉快。