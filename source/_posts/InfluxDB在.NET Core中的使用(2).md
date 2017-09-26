---
layout: post
title: 时序数据库InfluxDB在.NET Core中的使用——（2）IoC和仓储模式实现事件存储
date: 2017-09-26 00:44:01
permalink: influxdb-in-dotNEt-Core-2-ioc-and-repository-pattern
tags:
- .NET Core
- InfluxDB
categories:
- .NET Core
description:
thumbnail: https://www.msolution.io/wp-content/uploads/2015/12/influxdb.png
---

我上一篇文章[《InfluxDB在.NET Core中的使用——（1）InfluxDB简介》](/influxdb-in-dotNEt-Core-1-Introduction.html)中简单介绍了InfluxDB的相关概念，本文将继续介绍下半部分，就是在.NET Core中的使用方法。

## .NET 客户端

正因为InfluxDB是通过http请求来操作数据库，所以在.NET Core中使用很简单，只需要用HttpClient封装一个客户端、并且按照InfluxDB的数据结构去解析数据就行了。而且，已经有大神开发出了一个开源的可用于.NET Core的InfluxDB客户端项目 [InfluxData.Net](https://github.com/pootzko/InfluxData.Net) ，本文将利用这个客户端项目结合.NET Core自带的依赖注入框架和仓储模式，去封装一个事件存储的InfluxDB实现的类库。

## 如何设计

1. 把配置信息和InfluxDB客户端实例封装成一个服务。
1. 用这个服务去实现原来定义的事件仓储接口。
1. 遵循.NET Core的IoC风格，用`.Addxxx(option=>{})`的方法配置数据库并且注册服务。

## Show Code

#### InfluxDBOptions.cs

`InfluxDBOptions`类是一个配置类，主要有主机名`Host`( IP +端口号)，用户名 `UserName`，密码 `Password`，数据库名 `DatabaseName`。

```csharp
/// <summary>
/// InfluxDB设置
/// </summary>
public class InfluxDBOptions
{
    public string Host { get; set; }

    public string UserName { get; set; }

    public string Password { get; set; }

    public string DatabaseName { get; set; }
}
```

#### InfluxDbContext.cs

`InfluxDbContext` 类是InfluxDB上下文的意思，里面封装了 InfluxDbClient 的一些基本组成部分，数据库对象`Database`，客户端对象`Client`(不知道为啥要在`InfluxDbClient`内定义一个Client对象)，以及我们上面设计的配置对象`InfluxDBOptions`。

另外还有`EnsureDbCreated`、`WriteAsync`和`QueryAsync`，其中`EnsureDbCreated`是确保数据库已创建，而`WriteAsync`和`QueryAsync`是对原来客户端读写方法的简化。

```csharp
public class InfluxDbContext
{
    public InfluxDbContext(InfluxDBOptions options)
    {
        Options = options;
        var client = new InfluxDbClient(options.Host, options.UserName, options.Password, InfluxDbVersion.v_1_0_0);
        this.Client = client.Client;
        this.Database = client.Database;
        EnsureDbCreated().Wait();
    }

    public IDatabaseClientModule Database { get; }

    public IBasicClientModule Client { get; }

    public InfluxDBOptions Options { get; }

    public async Task EnsureDbCreated()
    {
        var dbs = await Database.GetDatabasesAsync();

        if (dbs.All(x => x.Name != Options.DatabaseName))
            await Database.CreateDatabaseAsync(Options.DatabaseName);
    }

    /// <summary>
    /// 查询一个Serie
    /// </summary>
    /// <param name="query"></param>
    /// <returns></returns>
    public async Task<Serie> QueryAsync(string query)
    {
        var result = await Client.QueryAsync(query, Options.DatabaseName);
        return result?.FirstOrDefault();
    }

    /// <summary>
    /// 写入一个Point
    /// </summary>
    /// <param name="point"></param>
    /// <returns></returns>
    public async Task<IInfluxDataApiResponse> WriteAsync(Point point)
    {
        return await Client.WriteAsync(point, Options.DatabaseName);
    }
}
```

#### IEventStorageRepository.cs

`IEventStorageRepository`事件仓储接口是我设计的，只有两个方法，一个存储方法`Store`，一个查询方法`GetEvents`。

```csharp
public interface IEventStorageRepository
{
    void Store(StoredEvent theEvent);

    IEnumerable<StoredEvent> GetEvents(Guid aggregateId, int afterVersion = 0);
}
```

其中`StoredEvent`是一个继承了Event的特殊类，用来保存事件的相关属性。关于这个类的设计在这不是重点，可以到我的项目[【Shriek】](https://github.com/ElderJames/shriek-fx/blob/master/src/Shriek/Storage/StoredEvent.cs)中一窥究竟。

```csharp
public class StoredEvent : Event
{
    public StoredEvent(Event @event, string data)
    {
        AggregateId = @event.AggregateId;
        Data = data;
        Version = @event.Version;
    }

    public StoredEvent() { }

    [Key]
    public int Id { get; protected set; }

    //事件的序列化json字符串
    public string Data { get; set; }
}
```
#### EventStorageRepository.cs

仓储实现类

```csharp
public class EventStorageRepository : IEventStorageRepository
{
    private readonly InfluxDbContext _dbContext;

    private const string TableName = "events";

    public EventStorageRepository(InfluxDbContext dbContext)
    {
        this._dbContext = dbContext;
    }

    public IEnumerable<StoredEvent> GetEvents(Guid aggregateId, int afterVersion = 0)
    {
        var query = $"SELECT * FROM {TableName} WHERE AggregateId='{aggregateId}' AND Version >= {afterVersion}";
        var result = _dbContext.QueryAsync(query).Result;

        return result == null ? new StoredEvent[] { } : SerieToStoredEvent(result);
    }

    public void Store(StoredEvent theEvent)
    {
        var point = new Point()
        {
            Name = TableName,
            Tags = new Dictionary<string, object>()
            {
                {"AggregateId", theEvent.AggregateId}
            },
            Fields = new Dictionary<string, object>()
            {
                {"Data", theEvent.Data},
                {"EventType", theEvent.MessageType},
                {"User", theEvent.User},
                {"Version", theEvent.Version}
            },
            Timestamp = theEvent.Timestamp
        };

        var result = _dbContext.WriteAsync(point).Result;

        if (!result.Success)
            throw new InfluxDataException("事件插入失败");
    }

    private static IEnumerable<StoredEvent> SerieToStoredEvent(Serie serie)
    {
        return serie.Values.Select(item => new StoredEvent
        {
            AggregateId = Guid.Parse(item[serie.Columns.IndexOf("AggregateId")].ToString()),
            Version = int.Parse(item[serie.Columns.IndexOf("Version")].ToString()),
            Data = item[serie.Columns.IndexOf("Data")].ToString().Replace(@"\", string.Empty),
            User = item[serie.Columns.IndexOf("User")].ToString(),
            MessageType = item[serie.Columns.IndexOf("EventType")].ToString().Replace(@"\", string.Empty),
            Timestamp = DateTime.Parse(item[0].ToString())
        });
    }
}
```

#### EventStorageInfluxDBExtensions.cs

IoC注册及配置扩展类

```csharp
public static class EventStorageInfluxDBExtensions
{
    public static void AddInfluxDbEventStorage(this IServiceCollection services, Action<InfluxDBOptions> optionAction)
    {
        var options = new InfluxDBOptions();
        optionAction(options);

        services.AddScoped(x => options);
        services.AddScoped<InfluxDbContext>();
        services.AddScoped<IEventStorageRepository, EventStorageRepository>();
        services.AddScoped<IMementoRepository, MementoRepository>();
        services.AddScoped<IEventStorage, SqlEventStorage>();
    }
}
```

#### 使用方法

```csharp
    var services = new ServiceCollection();
    services.AddInfluxDbEventStorage(options =>
    {
        options.Host = configuration.GetSection("InfluxDBConnection:Host").Value;
        options.Password = configuration.GetSection("InfluxDBConnection:Password").Value;
        options.UserName = configuration.GetSection("InfluxDBConnection:UserName").Value;
        options.DatabaseName = configuration.GetSection("InfluxDBConnection:DatabaseName").Value;
    });
```

配置文件`appsettings.json`中的配置

```json
  "InfluxDBConnection": {
    "Host": "http://localhost:8086",
    "DatabaseName": "event_store",
    "UserName": "admin",
    "Password": "admin"
  }
```

大功告成~由于太晚了，我先贴上代码，有空再补上说明咯！