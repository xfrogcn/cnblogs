# ASP.NET Core扩展库

亲爱的.neter们，在我们日复一日的编码过程中是不是会遇到一些让人烦恼的事情：

- 日志配置太过复杂，各种模板、参数也搞不清楚，每次都要去查看日志库的文档，还需要复制粘贴一些重复代码，好无赖
- 当需要类型转换时，使用AutoMapper时感觉配置又复杂，自己写人肉转换代码又冗长，又枯燥，好无聊
- 当调用其他服务时，总是不放心，于是在调用前、调用后总是不断重复地记录请求和应答日志？
- 当其他服务需要令牌时，我们不得不管理令牌的生命周期，而且不同第三方服务令牌的认证、维护过程还不一样，有时调用每一个接口时都要手动传入token，好麻烦
- 作为应用开发的你，你编写的服务和很多其他服务交互，经常因为其他服务的问题影响你的开发进度，同时你的服务由于依赖于其他服务，导致调试测试困难
- 在微服务模式下，需要请求链路跟踪，于是，你又在调用其他服务时，不断第重复传递链路跟踪的请求头
- 作为APIer的你，为了快速查找问题，不得不记录每一个接口的请求和应答内容，于是，你就在控制器里面增加了一堆的日志，你知道这不科学，但时间紧，任务重，就先这样吧
- ......

也许，以上这些问题，都有相应的库或者示例代码来解决，但这实在是太零散了，我们没有精力或不想去做这些，所以结果是常常我们采用了最“笨”的办法。

现在，解决这些问题的综合库来了，它就是Xfrogcn.AspNetCore.Extensions扩展库，它深度融合ASP.NET Core的设计模式，使用方式与ASP.NET Core完全一致。

## 简介

ASP.NET Core扩展库是针对.NET Core常用功能的扩展，包含日志、Token提供器、并行队列处理、HttpClient扩展、轻量级的DTO类型映射等功能。

源码地址：[GitHub](https://github.com/xfrogcn/Xfrogcn.AspNetCore.Extensions) [Gitee](https://gitee.com/WuYeCai/Xfrogcn.AspNetCore.Extensions)  
包地址：[NuGet](https://www.nuget.org/packages/Xfrogcn.AspNetCore.Extensions)

## 日志扩展

扩展库中，我们对Serilog日志库进行了简单的封装使其更加容易配置，同时也增强了本地文件日志Sink，使其支持更复杂的日志目录结构。

## 轻量级实体映射

在分层设计模式中，各层之间的数据通常通过数据传输对象(DTO)来进行数据的传递，而大多数情况下，各层数据的定义结构大同小异，如何在这些定义结构中相互转换，之前我们通过使用AutoMapper库，但AutoMapper功能庞大，在很多场景下，可能我们只需要一些基础功能，那么此时你可以选择扩展库中的轻量级AutoMapper实现。

## AspNetCore Http服务端的扩展

针对AspNetCore Http服务端，扩展库提供了以下功能：

- 请求与应答详细日志记录
- EnableBufferingAttribute特性，开启请求的Buffer（可重复读取）

## HttpClient扩展

.NET Core扩展库中通过HttpFactory及HttpClient来执行HTTP请求调用，HttpClient扩展在此基础上进行了更多功能的扩展，增加易用性、可测试性。

HttpClient包含以下功能：

- 针对HttpClient的相关扩展方法
- 针对HttpRequestMessage及HttpResponseMessage的扩展方法
- 请求日志记录
- 请求头的自动传递（请求链路跟踪）
- Http请求模拟（用于测试或模拟第三方服务）
- Http受限请求中，可自动获取及管理访问令牌

## 令牌提供器

令牌提供器用于应用的相关访问令牌的生命周期管理，包含令牌的自动获取、缓存、失效判断、自动重试等，主要由HttpClient扩展使用。当然你也可以单独使用。

## 并行队列处理

并行队列处理可以将一个大的队列，拆分到多个子队列进行并行处理，以提高处理效率。同时，在每个子队列处理中实现了处理管道，可灵活扩展。

![Img](并行队列.png)

以上定义即为扩展库所支持的功能，后面会有相关的文章进行详细介绍。