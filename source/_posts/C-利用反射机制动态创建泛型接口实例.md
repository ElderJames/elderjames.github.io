---
title: 'C#利用反射机制动态创建泛型接口实例'
permalink: dong-tai-chuang-jian-fan-xing-jie-kou-shi-li
id: 12
updated: '2016-07-12 13:40:55'
date: 2016-03-21 19:19:10
tags:
---

我很懒，不希望写重复的copy代码，所以希望在写代码时能尽量简化和重用，一些设计上有但没必要的能不写就不写，但当然必须要实现。

最近学会用C#的反射机制，觉得非常好，可以通过类名实例化一个对应的对象。于是我就想到了在设计业务层时，一些通用的数据库操作是以泛型基类的形式供给各个模型使用的，只需替换传入的模型的类型参数，即刻操作相应的模型，而对于一些模型需要的特殊操作，则以这个基类派生出包含特殊方法的子类来实现。

但是这里会出现一些问题，就是在各层透明的情况下，其他层是不知道模型是否包含有特殊的方法，最初的解决方法是为每个模型都写一个继承业务基类的子类，就算它里面是空的（没有特殊方法）。而现在，通过反射，我们可以很好地实现统一的业务实例化操作，实现自动判断该模型是否拥有对应的业务子类。

先设计一个泛型接口`IService<T>`,声明两个方法：

```
namespace Service
{
   public Interface IService<T> where T : class
   {

      IRepository<T> _repo { get; }
      void SetRespository(IRepository<T> repo);
      T Add(T entity);
      bool Update(T entity);
   }
}
```
然后再写一个基础类，继承这个接口：

```
namespace Service
{
      public class BaseService<T> : IService<T> where T : class
    {
        IRepository<T> _repo ;
        public void SetRespository(IRepository<T> repo)
        {
            _repo=repo;
        }
        public T Add(T entity){}
        public bool Update(T entity){}
    }
}
```
设计业务工厂类，提供统一的实例化出口

```
    /// <summary>
    /// 业务工厂类
    /// <para>所有业务类都必须从这里产生</para>
    /// </summary>
    public class ServiceFactory
    {
        //获取当前程序集中的所有类的类型
        static Type[] ts = Assembly.GetExecutingAssembly().GetTypes();

        /// <summary>
        /// 动态映射创建实现了IService<T>接口的实例
        /// <para>如果字符集中已有继承了BaseService<T>的类，则返回该类的实例,
        /// 否则返回BaseService<T> 的实例</para>
        /// </summary>
        /// <typeparam name="T">数据模型</typeparam>
        /// <param name="repo">仓储实例</param>
        /// <returns>实现IService<T>接口的实例对象</returns>
        public static IService<T> GetService<T>(IRepository repo) where T : class
        {
            //获取传入类型的 System.Type 对象
            Type TType = typeof(T);
            //取当前线程内存块中可能已存储的Service对象
            var _service = CallContext.GetData($"BaseService<{TType.Name}>") as IService<T>;

            if (_service != null) return _service;
            //查询扩展业务类是否存在于程序集中
            Type _class = ts.FirstOrDefault(o => o.Name.Equals(TType.Name + "Service"));
            try
            {
                if (_class != null)
                {
                    _service = Activator.CreateInstance(_class) as IService<T>;
                }
                else
                {
                    _service = Activator.CreateInstance(typeof(BaseService<T>)) as IService<T>;
                }
            }
            catch (Exception ex)
            {
                throw new Exception($"无法实例化{typeof(BaseService<T>).Name}:{ex.Message}");
            }

            //获取泛型类定义的type类型
            var _repoType = repo.GetType().GetGenericTypeDefinition();
            //传入泛型类型参数
            _repoType = _repoType.MakeGenericType(typeof(T));
            //实例化仓储类
            var _repo = Activator.CreateInstance(_repoType) as IRepository<T>;
            //注入DbContext
            _repo.SetDbContext(repo.db);
            //注入仓储类
            _service.SetRespository(_repo);

            //将对象保存到当前线程的内存块中
            CallContext.SetData($"BaseService<{TType.Name}>", _service);
            return _service;
        }
    }
```