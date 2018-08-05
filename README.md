# API 文档编写建议

如无特殊说明，本文默认前后端使用 HTTP 协议通讯。

## 在 wiki 或其他平台撰写文档

你可以使用 GitLab 或 GitHub 自带的 wiki 撰写文档。推荐使用 markdown 格式，本文后面的格式说明是基于 markdown 格式的。

> Markdown 编辑推荐使用 [VSCode](https://code.visualstudio.com)，插件推荐 [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint)

也可以尝试使用其他第三方工具。一些我所知的：

* [Postman Documentation](https://www.getpostman.com/docs/postman/api_documentation/intro_to_api_documentation)
* [eoLinker.com](https://www.eolinker.com)
* [ShowDoc](https://www.showdoc.cc)

## 提供一般约定

在列出接口列表并说明之前，应当对诸如鉴权、错误处理、如何分页、服务器地址等通用行为进行说明。

## JSON 响应

多数情况我们会使用 JSON 格式，注意此时 HTTP 响应（response）的 Content-Type 应为 `application/json`。

## HTTP 状态码和错误码

客户端首先会根据 HTTP 响应的状态码判定请求是否成功，后台应根据情况返回正确的状态码：成功的请求应该返回 2xx，因为客户端的原因出错返回 4xx，因为服务器的原因呢出错返回 5xx。3xx 用于重定向，但 API 一般用不到。

当 4xx 错误时，服务器一般应返回错误结构。这个错误结构中应当有一个业务上的错误码，用来指示客户端如何处理错误。

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



## Markdown 格式说明

### 目录

如果使用的是 wiki 的话，一般支持多个 page 并会有自动生成的 page 目录，单个 page 里推荐手写一个目录，列出这个 page 里的接口。

```md
## 目录

* [登录注册](#SectionLogin)


```

### 一般的接口定义

一般需要包括：

* 接口名
* HTTP method + 地址，不需要带 host
* 参数说明，包括字段名、是否可选、字段内容的类型、说明
* HTTP header，可选，如果该接口有特殊的头的话
* 响应说明，响应示例
* 错误说明，可选，如果有针对该接口的特殊错误的话

----

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)
