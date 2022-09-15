# SqlSugar IOC依赖注入


SqlSugar使用IOC 有2种方式 ，2种方式不能混用，用一种就可以了

# SqlSugar.IOC
SqlSugar.IOC 用法简单，注入直接开箱就用

Nuget 安装 SqlSugar.Ioc 和 SqlSugarCore

.NET Core 3.0 + .NET 5

``` cs
// 10秒入门
SugarIocServices.AddSqlSugar(new IocConfig()
{
  // ConfigId="db01"  // 多租户用到
  ConnectionString = "server=.;uid=sa;pwd=123456;database=SQLSugarDemo",
  DbType = IocDbType.SqlServer,
  IsAutoCloseConnection = true //自动释放
}); // 多个库就传 List<IocConfig>
 
// 配置参数
SugarIocServices.ConfigurationSugar(db =>
{
    db.Aop.OnLogExecuting = (sql, p) =>
    {
      Console.WriteLine(sql);
    };
    // 设置更多连接参数
    // db.CurrentConnectionConfig.XXXX=XXXX
    // db.CurrentConnectionConfig.MoreSettings=new ConnMoreSettings(){}
    // 二级缓存设置
    // db.CurrentConnectionConfig.ConfigureExternalServices = new ConfigureExternalServices()
    // {
    //    DataInfoCacheService = myCache //配置我们创建的缓存类
    // }
    // 读写分离设置
    // laveConnectionConfigs = new List<SlaveConnectionConfig>(){...}
      
    /*多租户注意*/
    // 单库是db.CurrentConnectionConfig 
    // 多租户需要db.GetConnection(configId).CurrentConnectionConfig 
});

//注入后就能所有地方使用
DbScoped.SugarScope.Queryable<UserOrgMapping>().Where(it=>it.Id>0).ToList() //1.7版本支持
DbScoped.Sugar.Queryable<UserOrgMapping>().Where(it=>it.Id>0).ToList()

//DbScoped.SugarScope=SqlSugarScope (推荐)
//DbScoped.Sugar=SqlSugarClient
```

# .NET IOC
## 注入ISqlSugarClient
.NET 自带的 IOC 使用也很方便 

注意:

SqlSugarScope 用单例 AddSingleton 单例

SqlSugarClient 用 AddScoped 每次请求一个实例

2 选 1**只能用一种方式**

``` cs
public static void AddSqlsugarSetup(this IServiceCollection services, IConfiguration configuration)
{
    // 连接字符串
    var str = configuration.GetConnectionString("SqlServer");

    SqlSugarScope sqlSugar = new(new ConnectionConfig()
    {
        ConnectionString = str,
        DbType = DbType.SqlServer,
        IsAutoCloseConnection = true,
    },
        db =>
        {
            //单例参数配置，所有上下文生效
            db.Aop.OnLogExecuting = (sql, pars) =>
            {
                // Console.WriteLine(sql);//输出sql
            };
            // 技巧：拿到非ORM注入对象
            // services.GetService<注入对象>();
        });
    
    // 建库
    sqlSugar.DbMaintenance.CreateDatabase();
    // 建表，Users 是自己写的实体类
    sqlSugar.CodeFirst.InitTables(typeof(Users));

    // 这边是SqlSugarScope用AddSingleton
    services.AddSingleton<ISqlSugarClient>(sqlSugar);
}
```

在 Startup 文件添加下面代码
``` cs
services.AddSqlsugarSetup(Configuration);
```
