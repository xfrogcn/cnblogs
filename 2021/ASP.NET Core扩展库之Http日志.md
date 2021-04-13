# ASP.NET扩展库之Http日志

最佳实践都告诉我们不要记录请求的详细日志，因为这有安全问题，但在实际开发中，请求的详细内容对于快速定位问题却是非常重要的，有时也是系统的强力证据。Xfrogcn.AspNetCore.Extensions扩展库提供了服务端和客户端的详细日志功能，通过配置可以开启。

服务端日志通过请求中间件来完成，中间件会以Trace级别记录请求和应答详情，以Debug级别记录请求耗时。服务的请求日志的名称为ServerRequest.Logger

要开启服务端详情日志，只需将扩展库配置中的ServerRequestLevel属性设置为Verbose级别，该配置默认是Information，故不会记录请求详情及请求耗时。

开启请求详情后，由于需要读取请求和应答的详细内容，对性能将有所影响。同时，由于要读取请求体，将自动开启请求的缓冲。只有在需要记录详细日志时，才会读取详情，故关闭后对于性能不会产生太大影响。

客服端的请求详细日志，是基于IHttpClientFactory以及HttpClient框架，在客户端请求管道处理中加入了日志记录管道。请求处理管道会以Trace级别记录请求和应答详情，另外，如果请求发生异常，将以Error级别记录异常详情。客户端请求日志的名称为ClientRequest.Logger

要开启客户端请求详细日志，只需将扩展库配置中的EnableClientRequestLog设置为true，同时将ClientRequestLevel设置为Verbose，该设置的默认值为Information。与服务端一样，只有在符合条件时才会记录请求与应答详情，故如果未开启，对性能不会产生影响。注意，当EnableClientRequestLog设置为false时，扩展库不会将日志请求管道插入客户端请求管道中。该设置默认为true。

## 开启服务端请求日志

要在服务端开启请求详细日志，只需引用Xfrogcn.AspNetCore.Extensions库，然后在Startup类中，配置服务请求级别为Verbose：

```c#
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddExtensions(Configuration, config=>
        {
            config.FileLog = true;
            config.ConsoleLog = true;
            // 设置服务端请求日志级别为Verbose
            config.ServerRequestLevel = Serilog.Events.LogEventLevel.Verbose;
        });
        services.AddControllers();
    }
```

## 开启客户端请求日志

要开启客户端日志，只需引用Xfrogcn.AspNetCore.Extensions库，然后在Startup类中，配置ClientRequestLevel为Verbose, EnableClientRequestLog设置为true。

```c#
    class Program
    {
        static async Task Main(string[] args)
        {
            IServiceCollection services = new ServiceCollection()
                .AddExtensions(null, config =>
                {
                    config.EnableClientRequestLog = true;
                    config.ClientRequestLevel = Serilog.Events.LogEventLevel.Verbose;
                    config.ConsoleLog = true;
                });

            IServiceProvider provider = services.BuildServiceProvider();
            var clientFactory = provider.GetRequiredService<IHttpClientFactory>();
            HttpClient client = clientFactory.CreateClient();

            var response = await client.GetAsync("http://localhost:5000/weatherforecast");

            Console.ReadLine();
        }
    }
```

## 示例

详细示例请参考[GitHub](https://github.com/xfrogcn/Xfrogcn.AspNetCore.Extensions/tree/master/examples/Http/RequestLog)

Xfrogcn.AspNetCore.Extensions地址：[GitHub](https://github.com/xfrogcn/Xfrogcn.AspNetCore.Extensions) [Gitee](https://gitee.com/WuYeCai/Xfrogcn.AspNetCore.Extensions)
