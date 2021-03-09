# ASP.NET Core扩展库之日志

上一篇我们对Xfrogcn.AspNetCore.Extensions扩展库功能进行了简单的介绍，从这一篇文章开始，我将逐步介绍扩展库中的核心功能。

日志作为非业务的通用领域基础功能，有非常多的技术实现，这些第三方库避免了我们花费时间去重复实现，不过，很多日志库配置复杂，不易于使用，入手较难，而有些库可能与ASP.NET Core的结合并不好。

如果我们没有对所使用的日志库进行详细了解，日志库也可能产生严重的问题，在我的开发生涯中，曾经遇到过多次因为日志库而导致的生产事故。

扩展库日志模块致力于将日志相关的最佳实践进行封装，简化日志库的使用，让我们真正从非业务代码中解放出来。

## 简介

ASP.NET Core扩展库中日志功能是对Serilog的进一步封装，之所以选择Serilog，源于我们在开发工程中的实践，我们的日志库经历了自己开发、选择使用NLog，最后定格在使用Serilog库上。

Serilog日志库也并不是非常易于使用，而且可能也缺少一些必要功能，这就是我们需要进一步封装的原因。

日志功能默认提供了Console及File两种日志目标，他们都分别支持文本和Json格式。

我们也添加了日志的分类、日志记录层级的动态修改、本地文件日志的定时清理、本地日志文件的按目录存储、对容器化下EFK日志架构的支持、以及日志在测试中的支持功能等。

## 使用

日志库是随着扩展库一起启用的，最简单的情况是启用扩展库即可，默认配置将开启文件日志目标，日志存入应用下Logs目录，以日期为文件夹，以日志名称为文件名称。

开启扩展库有两种方式，可以在IHostBuilder上通过UseExtensions方法，或者在Startup启动类ConfigureServices方法中通过IServiceCollection的AddExtensions方法。

```c#
    // 通过IHostBuilder上的UseExtensions方法
    // Program.cs  .NET 5.0 
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseExtensions(args);
                    webBuilder.UseStartup<Startup>();
                });
    }
```

或者：

```c#
    // 在Startup类中
    public class Startup
    {

        public void ConfigureServices(IServiceCollection services)
        {
            services.AddExtensions(Configuration);
        }
    }
```

## 配置

日志的配置可以通过代码方式或者通过配置文件方式。

采用代码方式，在UseExtensions方法或者AddExtensions中传入配置对象委托即可：

```c#
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseExtensions(args, config=>
                    {
                        config.AppLogLevel = Serilog.Events.LogEventLevel.Verbose;
                        config.SystemLogLevel = Serilog.Events.LogEventLevel.Verbose;
                    });
                    webBuilder.UseStartup<Startup>();
                });
```

如果采用配置文件方式，只需在配置源（如appsettings.json）中设置相关的配置字段：

```appsetting.json
{
  "AppLogLevel": "Verbose",
  "AllowedHosts": "*"
}
```

如果都采用，代码方式将会覆盖配置文件方式。

除此之外，如果你需要对Serilog配置进行更详细的控制，那么可以直接在UseExtensions方法或者AddExtensions中传入Serilog的日志配置委托。此项委托的设置将覆盖上述的自动配置。

## 配置日志级别

为了简化配置，在扩展库中，我们根据日志名称将日志分为系统日志、应用日志以及EFCore日志，他们分别通过配置中的AppLogLevel、SystemLogLevel及EFCoreCommandLevel属性来控制。日志级别的配置都支持运行时动态修改，无需重启应用。

| 日志分类 | 对应日志名 | 对应配置字段 | 默认级别 |
| ---- | --- | --- | --- |
| 系统日志 | Microsoft.* 以及 System.* | SystemLogLevel | Warning |
| EFCore日志 | Microsoft.EntityFrameworkCore.Database.Command | EFCoreCommandLevel | Information |
| 应用日志 | 除开系统日志及EFCore日志之外的日志 | AppLogLevel | Information |

## 日志级别的动态修改

如果你是通过配置源来配置的日志级别，那么当配置源更新时（一般通过配置对象的Reload方法），日志级别将自动修改。

如果需要采用代码方式，你可以通过全局的WebApiConfig实例进行配置：

```c#
    // 动态修改日志级别
    var apiConfig = host.Services.GetRequiredService<WebApiConfig>();
    apiConfig.AppLogLevel = Serilog.Events.LogEventLevel.Error;
```

## 本地文件日志配置

针对本地日志的配置，包含日志文件的路径模板、日志文件的定时清理、日志的自动压缩等。

本地文件日志路径通过LogPathTemplate设置来配置，默认为LogPathTemplates.DayFolderAndLoggerNameFile，表示以每天作为子目录，以日志名称作为日志文件名。通过LogPathTemplates也内置了其他的路径模板：

|  路径模板名 | 说明 |
| ---------- | ---- |
| DayFolderAndLoggerNameFile | 以每天日期为目录，日志名称为文件名 |
| DayFile | 以每天日期为日志名称 |
| LoggerNameAndDayFile | 以[日志名称_每天日志]为日志文件名称 |
| LevelFile | 以日志级别缩写为日志文件名称 |
| DayFolderAndLevelFile | 以每天日期为目录，日志级别缩写为日志名称 |

由于LogPathTemplate未字符串配置，你也可以配置其他的路径模板。

关于日志的定时清理，可以通过MaxLogDays配置来指定日志保留的天数，如果设置为0，表示不清理，这是默认配置。

通过MaxLogFileSize以及RetainedFileCount配置可以设置日志文件的自动压缩策略，MaxLogFileSize默认设置为100mb，超过此大小后，日志将写到新的文件，RetainedFileCount为可旋转的日志文件数量，默认为31个，超过此数量后的日志将被自动压缩。

## 容器化支持

在容器化环境下，日志一般会采用EFK的架构，在k8s中，我们推荐F采用fluent-bit而不是filebeat。这种框架下，我们只需将日志输出到控制台，容器将控制台输出定位到Docker宿主机，然后通过fluent-bit扫描日志文件，进行解析处理，发送给ES。

在这种模式下，你需要将ConsoleJsonLog设置为true来开启JSON格式的控制台日志目标。同时，由于控制台单行字数有限制，可能导致日志被截取，故可能需要通过MaxLogLength来设置单条日志的长度限制，此设置默认为8kb，适合大多数场景。超出长度的日志并不会被忽略，而是会拆分成多条日志，以此来保证日志的完整性。

```c#
    // 要支持容器化EFK日志模式，一般只需要设置ConsoleJsonLog为true即可
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseExtensions(args, config=>
                {
                    config.ConsoleJsonLog = true;
                });
                webBuilder.UseStartup<Startup>();
            });
```

## 测试支持

有时我们可能需要在单元测试中检查日志的输出，这时，我们可以使用扩展库在ILoggingBuilder上的扩展方法来添加测试日志目标。随后，你可以通过IServiceProvider上的GetTestLogContent方法获取已记录的日志内容列表。

```c#
    IServiceCollection services = new ServiceCollection()
        .AddExtensions()
        .AddLogging(logBuilder =>
        {
            // 添加测试日志记录器
            logBuilder.AddTestLogger();
        });
        
    IServiceProvider provider = services.BuildServiceProvider();
    // 获取日志内容
    var logContent = provider.GetTestLogContent();
```

## 禁用Serilog

如果你觉得上述所有功能都不太适合你的场景，但是你又需要使用扩展库的其他功能，那怎么办呢？ 非常简单，你只需要将EnableSerilog设置为false，即可完全禁用上述日志功能。
