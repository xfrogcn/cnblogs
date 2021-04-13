# ASP.NET Core扩展库之Http通用扩展

本文将介绍Xfrogcn.AspNetCore.Extensions扩展库对于Http相关的其他功能扩展，这些功能旨在处理一些常见需求, 包括请求缓冲以及针对HttpClient与HttpRequestMessage、HttpResponseMessage的扩展方法。

## 开启服务端请求缓冲

ASP.NET Core 中请求体是不能多次读取的，由于在MVC中，框架已经读取过请求体，如果你在控制器中再次读取，将会引发异常，如下示例：

```c#
    [ApiController]
    [Route("[controller]")]
    public class TestController : ControllerBase
    {
 
        public TestController()
        {

        }

        [HttpPost]
        public async Task<WeatherForecast> Save([FromBody]WeatherForecast enttiy)
        {
            using (StreamReader reader = new StreamReader(Request.Body))
            {
                Request.Body.Position = 0;
                string response = await reader.ReadToEndAsync();
            }
            return enttiy;
        }
    }
```

当通过Post请求/test接口时，语句 Request.Body.Position 将触发异常：

```text
System.NotSupportedException: Specified method is not supported.
   at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.Http.HttpRequestStream.set_Position(Int64 value)
```

当然，实际中可能不会像示例这样处理请求，但在业务需求中，的确可能会有多次读取请求体的情况出现。

通过开启请求缓冲可以解决多次读取请求体的问题，Xfrogcn.AspNetCore.Extensions扩展库提供了EnableBufferingAttribute特性用于开启请求缓冲，你可以将此特性用于控制器或者Action方法。

以上示例，只需在Save方法上添加EnableBuffering特性：

```c#
    [HttpPost]
    [EnableBuffering]
    public async Task<WeatherForecast> Save([FromBody]WeatherForecast enttiy)
    {
        ....
    }
```

## Http消息上的扩展方法

扩展库在HttpRequestMessage上提供了GetObjectAsync、WriteObjectAsync扩展方法，以便于对请求消息的读写。 在HttpResponseMessage上提供了GetObjectAsync、WriteObjectAsync扩展方法，以便于对应答消息的读写。这些方法都采用json格式。

示例：

```c# 
    public class WeatherForecast
    {
        public DateTime Date { get; set; }

        public int TemperatureC { get; set; }

        public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);

        public string Summary { get; set; }
    }
```

```c#
    static async Task Main(string[] args)
    {
        IServiceCollection services = new ServiceCollection()
            .AddExtensions(null, config =>
            {
            });

        IServiceProvider serviceProvider = services.BuildServiceProvider();

        IHttpClientFactory factory = serviceProvider.GetRequiredService<IHttpClientFactory>();
        HttpClient client = factory.CreateClient();

        HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Post, "http://localhost:5000/test");
        
        // 写入请求对象
        await request.WriteObjectAsync(new WeatherForecast()
        {
            Date = DateTime.Now
        });
        request.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");

        // 读取请求对象
        var entity = await request.GetObjectAsync<WeatherForecast>();

        HttpResponseMessage response = await client.SendAsync(request);

        // 读取应答对象
        entity = await response.GetObjectAsync<WeatherForecast>();

        Console.ReadLine();
    }
```

## HttpClient上的扩展方法

为了更方便快捷地使用HttpClient，扩展库在HttpClient上增加了多个扩展方法：

- PostAsync&lt;TResponse&gt;: 发送对象到服务端，并获取指定类型的应答
- PostAsync: 发送对象到服务端，并获取应答字符串
- GetAsync&lt;TResponse&gt;: 发送Get请求，并获取TResponse类型的应答
- GetAsync: 发送Get请求，并获取String类型的应答
- SubmitFormAsync&lt;TResponse&gt;: 向服务器提交表单数据，并获取TResponse类型的应答
- SubmitFormAsync: 向服务器提交表单数据，并获取String类型的应答
- UploadFileAsync&lt;TResponse&gt;: 上次本地文件
- UploadStreamAsync&lt;TResponse&gt;: 上传流数据到服务器

有关这些扩展方法的详细说明，可参考文档 [GitHub](https://github.com/xfrogcn/Xfrogcn.AspNetCore.Extensions/blob/master/doc/HttpClient.md) [Gitee](https://gitee.com/WuYeCai/Xfrogcn.AspNetCore.Extensions/blob/master/doc/HttpClient.md)

**Xfrogcn.AspNetCore.Extensions地址：**[GitHub](https://github.com/xfrogcn/Xfrogcn.AspNetCore.Extensions) [Gitee](https://gitee.com/WuYeCai/Xfrogcn.AspNetCore.Extensions)