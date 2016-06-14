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

*（注：在Scala中，构造函数注入需要使用：`@Inject()`的形式，使用`@Inject`是无效的。）*

除了使用主构造函数注入，还可以使用属性注入，就像很多Java代码里作的那样：

```scala
@Singleton
class NewsService {
  @Inject
  private var newsData: NewsData = null

  // ....
}
```

相比属性注入，在Scala中使用主构造函数注入是一种更好的风格。它明确了注入对象是 `val` ，不可变的，
杜绝了可能的在代码中不小心修改了注入对象的可能。而且这也是一种更“函数式”的风格。

## 组件的生命周期

依赖注入系统管理着注入组件的生命周期，创建他们并在需要其它组件时注入需要的组件。组件的生命周期通常如下管理：

- 每次注入组件时都需要生成相应组件的一个新实例，这是默认情况。若你不需要这种特性，那就应该显示声明组件是“单例”的。
- 组件的实例都是按需创建的，它们都是“lazy”的。如果一个组件没有被其它组件所依赖，那它不会被创建，直到第一次需要它们的时候才会被实例化。但是，在某此特殊情况，需要在应用程序刚启动时就实例化一个组件。这时候可以使用 Guice 框架的 [eager binding](https://playframework.com/documentation/2.5.x/ScalaDependencyInjection#Eager-bindings) 功能来实现。
- 组件实例没有自动清理功能，当它们不再被引用时，组件本身会被垃圾回收。但是框架不会被任何特殊的事情来关闭组件持有的资源，比如说一个 `dispose` 方法。但是，Play 提供了一个特殊的组件来让你在退出应用程序时做一个清理工作：[shutdown when the application stops](https://playframework.com/documentation/2.5.x/ScalaDependencyInjection#Stopping/cleaning-up)。

## 自定义绑定

**使用注解绑定"

Guice 提供了一个简单的注解：[`@ImplementedBy`](https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/ImplementedBy.html)来绑定接口到具体的实现：

```scala
import com.google.inject.ImplementedBy

@ImplementedBy(classOf[EnglishHello])
trait Hello {
  def sayHello(name: String): String
}

class EnglishHello extends Hello {
  def sayHello(name: String) = "Hello " + name
}
```

**编程实现绑定**

对于一些复杂的情况，比如一个接口有多个实现。可以使用一个命名注解：[@Named](https://docs.oracle.com/javaee/8/api/javax/inject/Named.html)来区分不同的实现类。可以通过自定义 Guice [Module](https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/Module.html) 来在 Play 中实现这功能：

```scala
package me.yangbajing.playing.config

import com.google.inject.AbstractModule
import com.google.inject.name.Names

class HelloModule extends AbstractModule {
  def configure() = {
    bind(classOf[Hello])
      .annotatedWith(Names.named("en"))
      .to(classOf[EnglishHello])

    bind(classOf[Hello])
      .annotatedWith(Names.named("de"))
      .to(classOf[GermanHello])
  }
}
```

若将此自定义模块放到根包路径下，那它会被 Play 应用自动注册。但通常我们都会把它放到一个不同的包中，这时就可以通过完全限定名在 `application.conf` 的 `play.modules.enabled` 配置项中注册它：

```conf
play.modules.enabled += "package me.yangbajing.playing.config.HelloModule"
```

而对于不想使用的模块，也可以禁用它：

```conf
play.modules.disabled += "Module"
```
