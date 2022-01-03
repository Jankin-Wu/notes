# MyBatis

## 拦截器（Interceptor）

> Mybatis核心对象介绍

+ Configuration 初始化基础配置，比如MyBatis的别名等，一些重要的类型对象，如，插件，映射器，ObjectFactory和typeHandler对象，MyBatis所有的配置信息都维持在Configuration对象之中
+ SqlSessionFactory  SqlSession工厂
+ SqlSession 作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能
+ Executor MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护
+ StatementHandler   封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集合。
+ ParameterHandler   负责对用户传递的参数转换成JDBC Statement 所需要的参数，
+ ResultSetHandler    负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；
+ TypeHandler          负责java数据类型和jdbc数据类型之间的映射和转换
+ MappedStatement   MappedStatement维护了一条<select|update|delete|insert>节点的封装， 
+ SqlSource            负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回
+ BoundSql 表示动态生成的SQL语句以及相应的参数信息

### Mybatis执行概要图

![](http://oss.jankinwu.com/img/20190612154305870.png)

### 拦截器注解的规则

``` java
@Intercepts({
    @Signature(type = StatementHandler.class, method = "query", args = {Statement.class, ResultHandler.class}),
    @Signature(type = StatementHandler.class, method = "update", args = {Statement.class}),
    @Signature(type = StatementHandler.class, method = "batch", args = {Statement.class})
})
```

+ @Intercepts：标识该类是一个拦截器；

+ @Signature：指明自定义拦截器需要拦截哪一个类型，哪一个方法；
  + type：对应四种类型中的一种；
  + method：对应接口中的哪类方法（因为可能存在重载方法）；
  + args：对应哪一个方法；
  
|     拦截的类     |                          拦截的方法                          |
| :--------------: | :----------------------------------------------------------: |
|     Executor     | update, query, flushStatements, commit, rollback,getTransaction, close, isClosed |
| ParameterHandler |              getParameterObject, setParameters               |
| StatementHandler |         prepare, parameterize, batch, update, query          |
| ResultSetHandler |           handleResultSets, handleOutputParameters           |