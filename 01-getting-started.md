# SpinalHDL Run

## 0. 概要

  - SpinalHDL只是scala的库，因此无论是IDE还是命令行，只要配置可编译scala项目即可。
  - 国内使用主要的问题是第一次运行的访问速度，`mill`版本可以离线下载，源可以采用代理或配置为国内的源。
  - 第一次配置因网络原因速度较慢，后续编译使用速度很快。

## 1. 最简单的：终端直接使用Mill

最方便的开发方式是下载`SpinalTemplateMill`项目

查看版本：
```bash
~$ sudo pacman -S scala mill jre11-openjdk
~$ mill --version
~$ scala --version
```

设置环境变量：
```bash
~$ tail ~/.bashrc# scala

export SBT_OPTS="-Dsbt.override.build.repos=true"
export JAVA_HOME="/usr/lib/jvm/java-11-openjdk"
export PATH="${JAVA_HOME}/bin:${PATH}"
```

需要修改`SpinalTemplateMill`中的以下几个文件
```bash
~$ cat .mill-version
0.9.10

~$ cat build.sc
```

前面几行显示如下：
```scala
import mill._, scalalib._, publish._

trait CommonSpinalModule extends ScalaModule {
  def scalaVersion = "2.13.6"
  def spinalVersion = "1.6.1"

  def scalacOptions = Seq("-unchecked", "-deprecation", "-feature")

  def ivyDeps = Agg(
    ivy"com.github.spinalhdl::spinalhdl-core:$spinalVersion",
    ivy"com.github.spinalhdl::spinalhdl-lib:$spinalVersion",
  )
  def scalacPluginIvyDeps = Agg(ivy"com.github.spinalhdl::spinalhdl-idsl-plugin:$spinalVersion")
}
```

主要是需要根据软件版本，配置mill，SpinalHDL的版本以及scala的版本号（依赖的库）。

配置好后，运行（在代理后速度会快）
```bash
~$ mill simple.run

[39/39] simple.run 
[Runtime] SpinalHDL v1.6.1    git head : 3bf789d53b1b5a36974196e2d591342e15ddf28c
[Runtime] JVM max memory : 16048.0MiB
[Runtime] Current date : 2021.12.20 11:52:18
[Progress] at 0.000 : Elaborate components
[Progress] at 0.205 : Checks and transforms
[Progress] at 0.263 : Generate Verilog
[Done] at 0.304
[Progress] Simulation workspace in /home/users/scala/SpinalTemplateMill/./simWorkspace/MyTopLevel
[Progress] Verilator compilation started
[info] Found cached verilator binaries
[Progress] Verilator compilation done in 2461.470 ms
[Progress] Start MyTopLevel test simulation with seed 1359913771
[Done] Simulation done in 23.855 ms
```
会自动解决相关依赖。编译成功后会输出。

好，我们已经解决项目运行的问题。

## 2. VSCODE and Metals

使用VSCODE需安装Metals，配置：

```bash
Metals: Server Properties
-Dsbt.override.build.repos=true
```

第一次运行速度也很慢。可使用国内的源，
修改`~/.config/coursier/mirror.properties`，内容为
```text
central.from=https://repo1.maven.org/maven2
central.to=https://maven.aliyun.com/repository/central
```

在运行CODE后，需要从github下载Mill（即便你的系统已经安装了），速度较慢。可以提前下载与`.mill-version`对应的版本（例如从gitee），
放在这个位置（0.9.11是mill的可执行文件）
```text
./.cache/mill/download/0.9.11
```

在Metals安装完成后，可运行函数的上访会出现`run | debug`的图标，点击就可以运行。

**注意**，BSP使用Bloop，如果系统开全局代理（127.0.0.1）可能会有问题。

## 3. 配置SBT

后续决定使用Mill，不打算运行SBT。但是前期整环境折腾了不少，这里记录一下。

修改SBT的源（从国内镜像获取）：
```bash
~$ cat ~/.sbt/repositories

[repositories]
local
huaweicloud-maven: https://mirrors.huaweicloud.com/repository/maven/
aliyun: https://maven.aliyun.com/repository/central
maven-central: https://repo1.maven.org/maven2/[repositories]
```

注意还需配置`-Dsbt.override.build.repos=true`。
