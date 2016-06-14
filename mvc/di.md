# 依赖注入

依赖注入是一种广泛使用的设计模式，可以帮助你分离你的组件，使代码更模块化。

Play 是一个提供了Scala和Java两套API的Full Stack WEB应用框架，它从2.4版本开始推荐使用 Guice来作DI（依赖注入）。
而从2.5版本开始，路由定义从之前默认使用Scala object改为了使用class + DI的方式。

**动机**

依赖注入的目的：

1. 可以轻松地将不同的实现方式绑定到相同的组件。这个在测试时特别有用，那时可以手动实例化一个 Mock 组件。
0. 可以避免一些全局的状态及静态的object服务。虽然使用Scala的object来定义服务是可行的，但你需要加倍
小心来管理各个object之间的相互依赖关系。

在 [Guice Wiki](https://github.com/google/guice/wiki/Motivation) 上有对使用依赖注入更详细的阐述。

## 运行时注入

在Scala中使用依赖注入非常的简洁，只需要在主构造函数前加上 `@Inject()` 注解即可。

```scala
import javax.inject.{Inject, Singleton}

@Singleton
class NewsService @Inject()(newsData: NewsData)  {
  // ....
}
```

一般情况下，我都推荐加上：`@Singleton` 注解。这会告诉注入框架，这个组件在应用生命周期内是“单例”的。
特别对于一些“服务”组件，这样可以免去每次需要注入这个组件时都重新生成一个实例，这样性能上会更好。
而对于一此要重要的资源，比如：数据库连接池这样整个应用内只应该保持一份的资源。那包含它的组件就必需为“单例”了。
