# 依赖注入

Play 是一个提供了Scala和Java两套API的Full Stack WEB应用框架，它从2.4版本开始推荐使用 Guice来作DI（依赖注入）。
而从2.5版本开始，路由定义从之前默认使用Scala object改为了使用class + DI的方式。
