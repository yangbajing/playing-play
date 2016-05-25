# 将WEB工程拆分到子项目

在日常开发中，我们一般会把应用逻辑分为几个部分。而这个划分很有可能从URI上体现出来，比如：

- /api: 普通用户访问接口
- /admin: 管理系统访问接口
- /sdk: 对外提供API服务
- /vendor: 接入第3方系统

若向上面这样系统很明晰的分为4大部分，我们就可以把这4块给拆分出来到子项目中。再加上一个`web-common`和`web`项目本身，
我们就可以把原来的一个`web`项目拆分成6个。

- web: 整个Play应用的入口项目
- web-common: 存入公共代码，并在此项目配置了反向路由映射
- web-api: API
- web-admin: 管理系统功能在此子项目
- web-sdk: 对外服务的代码在此子项目
- web-vendor: 接入的第3方应用在此子项目，比如：微信公众号、支付宝等

具体目录结构如下：

[include](_partial/app-dir.txt)

## 反向路由

当`web-api`，`web-admin`，`web-sdk`，`web-vendor`相互不依赖且`web`在依赖链的顶层时，若我们在`web-vendor`项目时向访问
`web`或`web-api`项目的路由，就需要使用反向路由配置了。

```scala
RoutesKeys.aggregateReverseRoutes := Seq(web, webApi, webAdmin, webSdk, webVendor)
```

这样，就可以在`web-vendor`项目的Play模版页面中使用`@routes.xxxx`语句访问`web`项目中定义的路由了。

[include](_partial/views/vendor/index.scala.html)

## 总结

把所有代码都写在一个`web`项目里不是一种好的实践，实际开发中我们应该尽量把不同功能的代码放到不同的子项目中。
这样做除了会有更快的编译速度以外，对工程模块化也是一种很好的促进。当我们把相同功能的代码放入同一个子项目以后，
团队成员可以更好的分工协作，程序员将可以更专注的在特定目录里实现自己那部分的功能。

而且，在对工程做项目拆分以后，我们会有时间来理顺整个工程的依赖关系。不会向把所以代码都放在一个项目里那样一团糟，因为拆分子项目后你
需要认真考虑整个依赖链的管理。一个好的实践方式是：单向依赖，从上向下传递依赖关系。
