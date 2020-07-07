# API 文档编写建议

如无特殊说明，本文默认前后端使用 REST 风格的 HTTP 协议通讯。[GraphQL](https://graphql.org/) 不在讨论范围。

本文侧重于 app 专有的 API，部分内容（如 HATEOAS 部分）不适用于公用服务，如政府公开数据。

## 在 wiki 或其他平台撰写文档

你可以使用 GitLab 或 GitHub 自带的 wiki 撰写文档，我写了[一个示例](https://github.com/BB9z/API-Documentation-Sample/wiki/README)，你可以在此基础上创建你的文档。

本文的格式说明是基于 markdown 格式的。

> Markdown 编辑推荐使用 [VSCode](https://code.visualstudio.com)，插件推荐 [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint)

[OpenAPI](https://www.openapis.org) 也是不错的选择，标准的东西，迁移方便，很多工具支持，推荐使用。

也可以尝试使用其他第三方工具。一些我用过的：

* [Postman Documentation](https://www.getpostman.com/docs/postman/api_documentation/intro_to_api_documentation)
* [Swagger](https://swagger.io)
* [eoLinker.com](https://www.eolinker.com)
* [ShowDoc](https://www.showdoc.cc)
* [YApi](https://github.com/YMFE/yapi)

## 尽量符合 REST 及其他标准规范

REST 是一种架构风格，包含的内容很多。资源路径命名，HTTP 的请求方法、状态码的选用，无状态，除了这些经常被讨论的内容，围绕 HTTP 头有大量文章可做，比如常常被忽视的缓存（笔者参与的几十个项目有做服务端 cache 控制的屈指可数）。

符合规范的最大好处是减少开发，越规范，能利用的现有工具越多，需要的二次开发越少。其次是相对更安全，自己想出来的方案很难比不过时的标准考虑全面。但在实战中，不那么 RESTful 很多时候对项目的进行影响其实也不大。

下面是一些具体分析。

### 经典「三元素」

路径命名、请求方法、状态码的选用，基本上讲 REST 规范的文本都会讨论这仨。设计得好好处很明显——看着就清晰：

> 看 URL 就知道要什么
>
> 看 HTTP method 就知道干什么
>
> 看 HTTP status code 就知道结果如何
>
> 来自：[怎样用通俗的语言解释REST，以及RESTful？ - 徐磊的回答 - 知乎](https://www.zhihu.com/question/28557115/answer/41267240)

除了清晰外对服务器扩容有点好处。

## 提供一般约定

在列出接口列表并说明之前，应当对诸如鉴权、错误处理、如何分页、服务器地址等通用行为进行说明。

## JSON 响应

多数情况我们会使用 JSON 格式，注意此时 HTTP 响应（response）的 Content-Type 应为 `application/json`。

## HTTP 状态码和错误码

客户端首先应根据 HTTP 响应的状态码判定请求是否成功，后台应返回符合 HTTP 规范的状态码。当错误发生时，后台的响应中应包括业务上的错误码，毕竟 HTTP 有限的状态码无法完全覆盖业务上的需求。

后台返回的 HTTP 状态码建议细化但不必须，只要能区分大类即可：成功的请求应该返回 2xx，因为客户端的原因出错返回 4xx，因后台服务的原因呢出错返回 5xx。3xx 用于重定向，但 API 一般用不到。

当 4xx 错误时，后台应返回错误结构。这个错误结构中需要有一个业务上的错误码，用来指示客户端如何处理错误。5xx 错误建议也返回。

> **举例**
>
> 登录接口，账号未注册、密码错误和其他一般错误应该返回不同的错误码，因为客户端可能会根据这些情况作做不同的处理：未注册时弹窗提示注册；密码错误时高亮密码区域并提示；一般错误仅报出来即可。
>
> 那么文档可以写成
>
> ```md
> 错误码
> * 1101 账号不存在
> * 1102 账号与密码不匹配
> ```

错误码的自定义范围建议 1000-9999，不用负数。大于 1000 避免与 HTTP 状态码重复，但如果响应的 HTTP 状态码足以准确反应是何种错误且无需扩展，错误码应和 HTTP 状态码保持一致。

## token 的使用

多数项目是有用户系统的，需要使用 token 或类似的等价体进行权限管理。建议：

1. 不要引入 refresh token、expires in 的概念

    OAuth 中有 refresh token、expires in、grant 等概念，第三方 API 用合适，但不适合应用的 API 使用。

    遇到这种需要刷 token 的，绝大部分情况开发者会选择应用启动的时候强制刷新一下。因为 app 里请求是一般是并行的，如不在启动时强刷，而是等过期了需要时再刷，在刷的时候已经发出去的请求怎么办，刷新的过程中需要进新的请求怎么办？业务层不可能正确处理这些情况的，只能靠接口框架统一处理。而除了专门针对 OAuth 的，一般的通用框架是不会带这种处理的。

    而且就算启动时强刷也不完美，请求时网络不好超时怎么办，一直让用户等超时结束体验很差，要等几秒钟再进 app，还没完成的请求取不取消？就算取消了也可能后台已经刷 token 了，只是响应未到达；要不取消，有可能其他请求用旧的 token 正发着呢，刷新完成了，这些请求可能回收到 token 失效请登出的错误，这时怎么办？

    综上，几乎就没有 app 端开发者能严格意义上的正确实现这个机制。但第三方服务就不同了，因为使用场景单一、可控，可能串行队列就能满足需求，即使出错，影响也有限。

2. token 和基本用户信息一起返回

    这点一般会涉及到注册、登录接口，有的后台会只返回 token，app 需要再拉一个接口才能获取到用户信息。因为多数情况 app 需要用户的 ID 结合 token 才能算登录成功（建用户的数据库需要用户 ID），如果能一块返回的话会更快、网络成功率更高。

    综上，如果后台不是特殊架构，一起返回比较困难的话，还是一起返回吧。

3. 开一个特殊的登录接口，App 每次启动调用登录，在这个接口刷新 token 的有效期，并返回用户基本的信息

    token 的有效期一般还是需要的，app 启动的时候刷也比较稳妥；用户信息正常也需要至少获取一次的，另获也行，能一个接口完成最好。

## 单独定义 Model，避免响应示例中贴大段的代码

好处：

* 文档写起来更轻松，工作量小——不用造数据，贴数据了了，编写和阅读更具可读性；
* 统一模型，避免不同的接口返回同样的东西但是字段不一；
* 避免调整模型字段后需要到处更新文档。

## 在更新接口的响应中直接返回更新后的结构

如果返回的结构不是非常大的话。

这样做便于 model 更新后直接刷新 UI，无需重新拉取 moedel 数据。直接用发给服务器的值修改本地 model 可不可以呢？考虑一些情景：

* 更新的字段会影响其他字段；
* 因为一些原因服务器端未更新或者需要延迟才能更新。

还是用服务器返回的新 model 更统一、更简单。

若支持 [Prefer return](https://tools.ietf.org/html/rfc7240#section-4.2) 头更佳。

## Markdown 格式说明

### 目录

如果使用的是 wiki 的话，一般支持多个 page 并会有自动生成的 page 目录，单个 page 里推荐手写一个目录，列出这个 page 里的接口。

```md
## 目录

* [登录注册](#SectionLogin)
  * [发送登录注册短信](#SMSSend)

## <a name="SectionLogin"></a>登录注册

### <a name="SMSSend"></a>发送登录注册短信
```

### 一般的接口定义

一般需要包括：

* 接口名
* HTTP method + 地址，不需要带 host
* 参数说明，包括字段名、是否可选、字段内容的类型、说明
* HTTP header，可选，如果该接口有特殊的头的话
* 响应说明，响应示例
* 错误说明，可选，如果有针对该接口的特殊错误的话

## 反标准

### HATEOAS 或称 hypermedia

[Hypermedia as the Engine of Application State](https://en.wikipedia.org/wiki/HATEOAS)，虽然是 REST 规范的一部分，但我认为是低效、过时的，不适合 API 用，维护好文档才是最现实、有效的。

HATEOAS 字面意思是通过在返回的响应中附加信息（hypermedia）控制客户端的可达的状态（application state）。在实践中有两个方向：

  1. 添加相关链接，起资源发现或者列出可达状态
  2. 返回结构标准化，便于程序理解

对于一个应用来说，HATEOAS 化的接口提供的部分潜在好处是：

  1. 保持根接口不变，其他接口的地址、请求方法可以在服务器端调整；
  2. 一个资源的可用操作，通过服务器端控制；
  3. 结构标准化后，一些逻辑可以统一实现，跨接口、跨应用。

<!-- 待完善 -->

首先第一点，服务器下发请求配置文件就能做到的事，在接口里现去一个个发现效率低下。不只是响应变大了，原本能多个接口并行请求，变成需要一层层逐级展开。

第二点，直接返回一个自定义的可操作列表，比整这些简单快捷多了。

第三点，标准化方便的主要是爬虫……另外再看看标准方案感人的效果……

* [Web Link 标准](https://tools.ietf.org/html/rfc8288)

  在 HTTP 头添加带属性的链接，这玩意基本就是给搜索引擎爬虫和浏览器用的，看注册的 [Relation Types](https://www.iana.org/assignments/link-relations/link-relations.xhtml) 就知道。最有用的场景是帮助浏览器插件自动加载下一页，其他方面对用户体验帮助不大。

  标准里也说了 Relation Type 可以自定义，但 app 的接口用这个？明显不合适——功能弱，解析费劲。

* [JSON HAL 草案](https://tools.ietf.org/html/draft-kelly-json-hal-08) 和 [JSON-LD 提案](https://www.w3.org/TR/json-ld/)

  这俩都是返回结构标准化的尝试。除非你的接口就是让人抓的，看例子你觉得有必要用么？

一些非标准的实现供参考：

* [JSON:API](https://jsonapi.org/)
* [Siren: a hypermedia specification for representing entities](https://github.com/kevinswiber/siren)

想用随意。

----

## 参考

* [HTTP API Development Tools](https://github.com/yosriady/api-development-tools)
* [API Security Checklist](https://github.com/shieldfy/API-Security-Checklist)
* [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines)
* [Coinbase Digital Currency API](https://developers.coinbase.com/api/v2)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)
