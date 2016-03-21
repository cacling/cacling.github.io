---
layout: post
title:  "务实的RESTFUL API设计方式"
date:   2016-03-21 16:56:46 +0800
categories: 架构设计
---

## RESTFUL是API设计的银弹？

每一种风格的软件架构都有自己的优劣，永远不存在适用于任何运行环境的、包治百病的银弹式架构。所以RESTFUL相对传统的RPC风格的SOAP Webservice在不少应用场景下还是挺合适的。

###  RESTFUL与RPC的对比

SOAP的设计则是RCP风格基于action思想的设计方式。SOAP是根本不知道系统里有什么，只关心能通过方法达到什么目的 . 鼓励应用设计人员定义任意的词汇（动词和名词）设计接口，像getUsers()，saveOrder，getBookPrice等。任何需求都可以新开一个接口去处理，后端可以针对单一需求编写逻辑去满足业务。SOAP的有优点是直观， 但缺点也明显， 当接口不断膨胀时容易导致系统混乱，逻辑耦合度高，很难控制系统的复杂度。 
举个例子, 有一个箱子里装着一堆盒子， REST风格是一开始就把箱子和盒子定义好， 你首先找到箱子然后打开箱子找里面你需要的盒子。这是个很自然的导向过程，不需要看说明书你就可以自己去观察去做到。

而RPC风格则是像你先要看说明书，看看有哪些方法可以帮你找盒子，例如找到一个searchBoxBy(…)的方法， 你还得弄明白传什么参数给这个方法才能调用， 调用的过程中， 你不需要知道箱子的存在。问题是如果说明书写得不好，有很多方法而风格不统一，那你就想得头晕了。

###  RESTFUL的适用场景

我认为， 公开的open API以RESTFUL的形式展现是合适的，因为这些API会被很多不同的客户端调用， 一定要简单，让调用方容易明白， 而且一旦API发布后，就很难对它做很大的改动并保持像先前一样的正确性。对于Opan API来说， 必须是够友好好简单，开发者才会尝试去使用它，它才会有价值 。 
例如Amazon提供的服务进行图书查找, Twitter提供服务对微博进行操作。 
一旦你发布一个公开的API，你必须承诺”在没有通告的前提下，不会更改APIDe功能” .对于外部可见API的更新，文档必须包含任何将废弃的API的时间表和详情。应该通过博客(更新日志)或者邮件列表送达更新说明(最好两者都通知)。

但是对于企业内部SOA集成的接口来说， 具体还是要看企业的应用本身是面向资源还是面向活动的，才决定是以SOAP的形式还是RESTFUL的形式定义接口。如果应用要集中在访问信息资源的能力， 那么就REST。如果应用主要集中于被执行的活动（这些活动与所依赖的资源不相关），则应该利用 SOAP 样式的设计模式。 
数据模型已经稳定，这一点是比较困难的。

# 关于RESTFUL设计

###  RESTFUL API的URL设计

REST最重要的一点是先定义好Resource， 确定有哪些 actions 应用这这些resouce上，这些 action到resource的映射就是你的 API 。RESTful 原则提供了 HTTP methods（GET, POST, PUT, DELETE） 作为CRUD actions，如下：

方法	注释
GET /orders	获取order列表
GET /orders/12	获取一个order #12
POST /orders	创建一个新的 order
PUT /orders/12	更新 order #12
PATCH /orders/12	更新部分信息 order #12
DELETE /orders/12	删除 order #12
可以看到REST 利用现有的 HTTP 方法应用在resource上是很整洁干净的的, 没有什么方法命名约定需要去遵循

###  URL上的Resource应该以单数还是复数来表达？

虽然在语法层面看描述单个Resource时使用单数会更为接近语言习惯， 但实际上我们在设计API的时候常常会遇到各种英文的单复数形式问题， 例如单复数同形的goods、news ，还有非正规的复数形式如 children, foot等等。 所以为了避免不必要的思考和混淆， 统一使用复数形式是更好的



###  该如何处理resource间的父子关系呢？

RESTFUL提供了很好的指导原则。例如order下包含多个goods。那goods和order的逻辑关系通过URL描述如下：

方法	注释
GET /orders/12/goods	获取order #12下的消息列表
GET /orders/12/goods/5	获取order #12下的编号为5的商品
POST /orders/12/goods	为order #12添加一个商品
PUT /orders/12/goods/5	更新order #12下的编号为5的商品
PATCH /orders/12/goods/5	部分更新order #12下的编号为5的商品
DELETE /orders/12/goods/5	删除order #12下的编号为5的商品
如果Action不符合CRUD操作该怎么办？

很多的RESTFUL例子回避了这个问题， 在现实的应用中总会出现这样或那样的情形无法匹配CRUD操作， 有时候你实在是没有办法业务的Action对应RESTFUL现有的HTTP method中。 
这时候我们可以采用比较务实的方法， 利用RESTful原则像处理子资源一样处理它。 例如， Github的API让你通过PUT /gists/:id/star 来 star项目。 
还有经常遇到的搜索， 可以通过URL上加上/search来设计， 只要从调用方角度看起来是在做容易理解的，正确的事，并且写好文档就可以。




###  如何对结果进行处理（过滤、搜索、排序和分页）？

对返回结果进行处理时， 最好是尽量保持基本资源URL的简洁性。 对结果进行过滤、搜索、排序和分页时 ， 可以基于URL之上的查询参数来实现。

过滤: 通过resource的属性为参数， 对resource进行过滤。 例如：

GET /orders?order_state=canceled –获取所有已取消的订单 
GET /orders?pay_state=paid&pay_type=credicard –获取所有通过信用卡支付的订单
搜索: 有时基本的过滤不能满足需求，例如对resource的属性进行范围查询或者是正则查询。 这时我们可以通过在参数里加上”q” 来加上查询条件。例如:

GET /orders?q=order_date>20160101 – 查询1月1日后的订单 
GET /orders?q=address~广州市 – 查询属于广州市的订单 
GET /orders?q=price>100+count<10 –查询价格大于100且数目小于10的订单
排序: 跟过滤类似, 通过resouce的属性， 参数排序可以被用来描述排序的规则。为适应复杂排序需求，让排序参数采取逗号分隔的字段列表的形式，每一个字段前都可能有一个负号来表示按降序排序。例如：

GET /orders?sort=-order_date 
GET /orders?sort=-order_date,price
分页：更好的分页参数是使用 RFC 5988 中介绍的链接标头。例如：

GET /orders?page=3&per_page=100 
在API的返回里可以包含结果的总数， 按当前分页条数的分页的总数等信息。
把这些组合在一起，我们可以创建以下一些查询:

GET /orders?sort=-updated_at 
GET /orders?state=closed&sort=-updated_at 
GET /orders?q=return&state=open&sort=-priority,created_at
是否有必要限制由API返回哪些字段？

API的使用者并不总是需要一个资源的完整表示。选择返回字段的功能由来已久，它使得API使用者能够最小化网络阻塞，并加速他们对API的调用。

使用一个字段查询参数，它包含一个用逗号隔开的字段列表。例如，下列请求获得的信息将刚刚足够展示一个在售票的有序列表:

GET /orders?fields=id,subject,customer_name,updated_at&state=open&sort=-updated_at
API的版本是否应该包含在URL或者请求头中?

关于API的版本是否应该包含在URL或者请求头中 莫衷一是。从学术派的角度来讲，它应该出现在请求头中。然而版本信息出现在URL中必须保证不同版本资源的浏览器可浏览性（browser explorability），还记得文章开始提到的API要求吗？

我非常赞成 approach that Stripe has taken to API versioning - URL包含一个主版本号（比如http://shonzilla/api/v1/customers/1234） 
），但是API还包含基于日期的子版本（比如http://shonzilla/api/v1.2/customers/1234），可以通过配置HTTP请求头来进行选择。这种情况下，主版本确保API结构总体稳定性，而子版本会考虑细微的变化（field deprecation、接入点变化等）。

API不可能完全稳定。变更不可避免，重要的是变更是如何被控制的。维护良好的文档、公布未来数月的deprecation计划，这些对于很多API来说都是一些可行的举措。它归根结底是看对于业界和API的潜在消费者是否合理。



###  RESTFUL API的请求与返回值的设计

是否应该HATEOAS?

Hypermedia as the Engine of Application State (HATEOAS)超媒体作为应用程序状态引擎

对于API消费方是否应该创建链接，或者是否应该将链接提供给API，有许多混杂的观点。RESTful的设计原则指定了HATEOAS ，大致说明了与某个端点的交互应该定义在元数据(metadata)之中，这个元数据与输出结果一同到达，并不基于其他地方的信息。

虽然web逐渐依照HATEOAS类型的原则运作（我们打开一个网站首页并随着我们看到的页面中的链接浏览），我不认为我们已经准备好API的HATEOAS了。当浏览一个网站的时候，决定点击哪个链接是运行时做出的。然而，对于API，决定哪个请求被发送是在写API集成代码时做出的，并不是运行时。这个决定可以移交到运行时吗？当然可以，不过顺着这条路没有太多好处，因为代码仍然不能不中断的处理重大的API变化。也就是说，我认为HATEOAS做出了承诺，但是还没有准备好迎接它的黄金时间。为了完全实现它的潜能，需要付出更多的努力去定义围绕着这些原则的标准和工具。



###  自动装载相关的资源描述

在很多种情况下，API的使用者需要加载和被请求资源相关的数据（或被请求资源引用的数据）。与要求使用者反复访问API来获取这些信息相比，允许在请求原始资源的同时一并返回和装载相关资源，将会带来明显的效率提升。

然而, 由于这样确实 有悖于一些RESTful原则, 所以我们可以只使用一个内置的（或扩展）的查询参数来实现这一功能，来最小化与原则的背离。

这种情况下，“embed”将是一个逗号隔开的需要被内置的字段列表。点号可以用来表示子字段。例如:

GET /ticket/12?embed=customer.name,assigned_user
这将返回一个附带有详细内置信息的票据，如下:

```json
print:'Hello world'
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




###  字段名称书写格式是 snake_case 还是 camelCase?

如果你在使用JSON (JavaScript Object Notation) 作为你的主要表示格式，正确的方法就是遵守JavaScript命名约定——对字段名称使用camelCase！如果你要走用各种语言建设客户端库的路线，最好使用它们惯用的命名约定—— C# & Java 使用camelCase, python & ruby 使用snake_case。

深思：我一直认为snake_case比JavaScript的camelCase约定更容易阅读。我没有任何证据来支持我的直觉，直到现在，基于从2010年的camelCase 和 snake_case的眼动追踪研究 (PDF)，snake_case比驼峰更容易阅读20％！这种阅读上的影响会影响API的可勘探性和文档中的示例。

许多流行的JSON API使用snake_case。我怀疑这是由于序列化库遵从它们所使用的底层语言的命名约定。也许我们需要有JSON序列库来处理命名约定转换。


###  使用JSON 编码的 POST, PUT & PATCH 请求体

如果你正在跟随本文中讲述的开发过程，那么你肯定已经接受JSON作为API的输出。下面让我们考虑使用JSON作为API的输入。

许多API在他们的API请求体中使用URL编码。URL编码正如它们听起来那样 - 将使用和编码URL查询参数时一样的约定，对请求体中的键值对进行编码。这很简单，被广泛支持而且实用。

然而，有几个问题使得URL编码不太好用。首先，它没有数据类型的概念。这迫使API从字符串中转换整数和布尔值。而且，它并没有真正的层次结构的概念。尽管有一些约定，可以用键值对构造出一些结构（比如给一个键增加“[]”来表示一个数组），但还是不能跟JSON原生的层次结构相比。

如果API很简单，URL编码可以满足需要。然而，复杂API应当严格对待他们的JSON格式的输入。不论哪种方式，选定一个并且整套API要保持一致。

一个能接受JSON编码的POST, PUT 和 PATCH请求的API，应当也需要把Content-Type头信息设置为application/json，或者抛出一个415不支持的媒体类型（Unsupported Media Type）的HTTP状态码。

###  关于返回错误信息

就像一个HTML错误页面给访问者展示了有用的错误信息一样，一个API应当以一种已知的可使用的格式来提供有用的错误信息。 错误的表示形式应当和其它任何资源没有区别，只是有一套自己的字段。

API应当总是返回有意义的HTTP状态代码。API错误通常被分成两种类型: 代表客户端问题的400系列状态码和代表服务器问题的500系列状态码。最简情况下，API应当把便于使用的JSON格式作为400系列错误的标准化表示。如果可能(意思是，如果负载均衡和反向代理能创建自定义的错误实体), 这也适用于500系列错误代码。

一个JSON格式的错误信息体应当为开发者提供几样东西 - 一个有用的错误信息，一个唯一的错误代码 (能够用来在文档中查询详细的错误信息) 和可能的详细描述。这样一个JSON格式的输出可能会像下面这样:

```json
{
  "code" : 1234,
  "message" : "Something bad happened :(",
  "description" : "More details about the error here"
}
对PUT, PATCH和POST请求进行错误验证将需要一个字段分解。下面可能是最好的模式：使用一个固定的顶层错误代码来验证错误，并在额外的字段中提供详细错误信息，就像这样:

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

总结

一个API是一个给开发者使用的用户接口。要努力确保它不仅功能上可用，更要用起来愉快