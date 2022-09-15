# SqlSugar CRUD

最基础且最重要的 CRUD
<!-- more -->

这里只演示一个简单的 CRUD  
具体靠自己深入了解

首先创建一个仓储  
不了解仓储的小伙伴请看 [SqlSugar-仓储](./SqlSugar-%E4%BB%93%E5%82%A8.md) 或 [官方文档](https://www.donet5.com/Home/Doc?typeId=1228)
``` cs
public class Repository<T> : SimpleClient<T> where T : class, new()
{
    public Repository(ISqlSugarClient context = null) : base(context)
    {
        if (context == null)
        {
            base.Context = new SqlSugarClient(new ConnectionConfig()
            {
                DbType = SqlSugar.DbType.SqlServer,
                InitKeyType = InitKeyType.Attribute,
                IsAutoCloseConnection = true,
                ConnectionString = "server=.;database=SqlSugarDemo;uid=sa;pwd=123456;"
            });
        }
    }
}
```

如果是个人开发可以不用写 interface  
如果是团队开发建议写上，有利于团队间的合作  
无论是个人还是团队，最好还是写上因为以后工作是会用到接口编程  
``` cs
public class UserRepository : Repository<Users>, IUserRepository
{
    public IEnumerable<Users> GetAllUser()
    {
        return base.GetList();
    }

    public Users GetUserById(int id)
    {
        return base.GetById(id);
    }
}
```
## IOC 注入
在 Startup 文件添加下面代码
``` cs
services.AddScoped<IUserRepository,UserRepository>();
```

# C (create)
``` cs
[HttpPost]
public string AddUser(Users entity)
{
    _user.AddUser(entity);
    return new
    {
        Code = 200,
        Msg = "添加用户成功！"
    }.SerializeObject();
}
```

# R (retrieve)
``` cs
[HttpGet]
public string GetUser()
{
    var tmp = _user.GetAllUser();

    return new
    {
        Code = 200,
        Msg = "获取所有用户成功！",
        Data = tmp,
    }.SerializeObject();
}
```

# U (update)
``` cs
[HttpPut]
[Route("{id}")]
public string UpdateUser(Users entity,int id)
{
    _user.UpdateUser(entity,id);
    return new
    {
        Code = 200,
        Msg = "修改用户成功！"
    }.SerializeObject();
}
```


# D (delete)
``` cs
[HttpDelete]
public string DeleteUser(int id)
{
    _user.DeleteUser(id);
    return new
    {
        Code = 200,
        Msg = "删除用户成功！"
    }.SerializeObject();
}
```

SerializeObject 方法
``` cs
public static string SerializeObject(this object obj)
{
    var options =
        new JsonSerializerOptions
        {
            Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping,
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
            WriteIndented = true
        };
    return JsonSerializer.Serialize(obj, options);
}
```