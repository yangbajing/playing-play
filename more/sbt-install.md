# sbt安装

<a target="_blank" href="http://www.scala-sbt.org/0.13/docs/zh-cn/Setup.html">sbt官方安装文档</a>

下载sbt启动jar包：

```sh
mkdir ~/bin
wget https://repo.typesafe.com/typesafe/ivy-releases/org.scala-sbt/sbt-launch/0.13.11/sbt-launch.jar
```

创建`~/bin/sbt`脚本：

```sh
SBT_OPTS="-Xms512M -Xmx1536M -Xss1M -Dfile.encoding=UTF-8 -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=256M"
java $SBT_OPTS -jar `dirname $0`/sbt-launch.jar "$@"
```

**注：Java8开始不在需要`-XX:MaxPermSize`参数**

给脚本可执行权限：

```
chmod u+x ~/bin/sbt
```

## 使用`Repox`加速sbt

<a target="_blank" href="http://centaur.github.io/repox/">Repox（社区公服，若大家觉得好友的话希望能捐助）</a> 是 @老猪 开发的一款：“改善sbt解决依赖的速度”的开源软件。
我们可以使用它来解决下载依赖过慢和伟大的墙造成的很多资源不能访问问题。这里摘录官方WIKI的<a target="_blank" href="https://github.com/Centaur/repox/wiki/%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97">入门指南</a>

1. 配置 ~/.sbt/repositories 文件（如果文件还未创建过，则创建它），除了本地缓存外，仅使用repox作为仓库。文件内容如下：

```sbt
[repositories]
local
repox-maven: http://repox.gtan.com:8078/
repox-ivy: http://repox.gtan.com:8078/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
```

**请注意，repox-maven 与 repox-ivy 的次序是重要的，请将 repox-maven 写在 repox-ivy 的前面。**

2. 配置sbt，使它仅使用~/.sbt/repositories中的内容。

    - 如果你使用命令行，请在sbt命令行参数中添加 `-Dsbt.override.build.repos=true` 。例如我的sbt shell脚本的内容是这样的：

    ```sh
    #!/bin/sh
    export SBT_OPTS="-Dsbt.override.build.repos=true"
    exec java -Xmx512M ${SBT_OPTS} -jar $(dirname "$0")/sbt-launch.jar "$@"
    ```

    - 如果使用jetbrains IDEA，修改 `Preferences`->`SBT`->`JVM Options`->`VM parameters`，保证它包含

    ```sh
    -Dsbt.override.build.repos=true 
    ```

    - 如果使用 activator，请打开 ~/.activator/activatorconfig.txt 文件（如果此文件不存在，请创建它。很明显，配置文件满天飞也是 typesafe/sbt team的诸多恶趣味之一），在其中添加一行

    ```sh
    -Dsbt.override.build.repos=true
    ```
