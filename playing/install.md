# 安装

这里我不使用官方的 `active` 来创建 Play 项目，使用手动创建的方式。这样创建的项目会更轻量，读者也可以更明析的了解 Play 项目结构。

## 简版

```
mkdir first-play
cd first-play
mkdir -p app/controllers conf test/controllers project

echo 'addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.5.2")
' >> project/plugins.sbt

echo 'sbt.version=0.13.11
' >> project/build.properties

echo 'name := """first-play"""

version := "1.0.0"

scalaVersion := "2.11.8"

lazy val root = project.in(file("."))
  .enablePlugins(PlayScala)
  .settings(
    libraryDependencies ++= Seq(
      ws,
      "org.scalatestplus.play" %% "scalatestplus-play" % "1.5.1" % "test"
    )
  )
' >> build.sbt

echo 'package controllers

import play.api.mvc._

class AppController extends Controller {
  def index = Action {
    Ok("Hello, Play!")
  }
}
' >> app/controllers/AppController.scala

echo 'GET  /   controllers.AppController.index
' >> conf/routes

touch conf/application.conf
```

接下来在命令行执行 **`sbt run`** 即可启动 Play 程序了。（ sbt 在Mac下可以使用brew安装，`brew install sbt`）

通过浏览器访问：[http://localhost:9000](http://localhost:9000) 可以看到页面输出了“Hello, Play!”。OK，这就是一个 Play 程序，它是如此的简洁、优美。

**测试**

现代程序开发，测试驱动是一个比较好的方式。Play 基于 [scalatest](http://scalatest.org) 开发了一套很优秀的测试套件。我们来试着写一个测试代码。

```
echo 'package controllers

import org.scalatestplus.play.PlaySpec
import play.api.mvc.Results
import play.api.test.FakeRequest
import play.api.test.Helpers._

class AppControllerTest extends PlaySpec with Results {

  "AppControllerTest" should {
    "index" in {
      val appController = new AppController()
      val result = appController.index(FakeRequest())
      val bodyTest = contentAsString(result)
      bodyTest mustBe "Hello, Play!"
    }
  }

}
' >> test/controllers/AppControllerTest.scala
```

执行 **`sbt test`** 执行单元测试，我们可以看到如下测试通过的输出：

```
$ sbt test
[info] AppControllerTest:
[info] AppControllerTest
[info] - should index
[info] ScalaTest
[info] Run completed in 1 second, 438 milliseconds.
[info] Total number of tests run: 1
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 1, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[info] Passed: Total 1, Failed 0, Errors 0, Passed 1
[success] Total time: 3 s, completed 2016-4-26 17:50:44
```

## Play 目录结构

上一小节通过几个简单的命令尝试了一个“最简化”的 Play 应用，接下来介绍下 Play 应用的目录结构：

```
.
├── app
│   └── controllers
│       └── AppController.scala
├── build.sbt
├── conf
│   └── routes
├── project
│   ├── build.properties
│   └── plugins.sbt
└── test
```

- app: 放置应用代码的地方，除了 `controllers/`（控制器）外，一般还有：`models/`（模型类），`views/`（Play页面模板代码）等
- build.sbt: 项目主配置文件，这里可以设置第3方库依赖等
- project/build.properties: 指定使用 sbt 的版本
- project/plugins.sbt: sbt插件设置目录，我们在这里设置 Play 的版本
- test: 放置测试代码的地方

