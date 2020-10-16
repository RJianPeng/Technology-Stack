- [Gradle是啥](#gradle是啥)
- [Gradle特性](#gradle特性)
- [Gradle和Maven的区别](#gradle和maven的区别)
- [Gradle解决依赖冲突](#gradle解决依赖冲突)
- [Gradle基础](#gradle基础)
- [Gradle使用中的问题](#gradle使用过程中的问题)






# Gradle是啥
完全开源的构建自动化系统 直白点就是像maven一样帮你处理项目所需要的包的一个工具。maven配置文件的编写使用的xml，gradle用的是groovy语言，语法类似于java，方便java程序员学习和使用。执行时同样是在jvm上执行。

## Groovy语言
DSL语言：domain specific language，即特定领域语言。Groovy即DSL语言

介绍：一种基于JVM的敏捷开发语言。可以与Java完美结合并且可以使用Java所有的库。

特性：
* 1.语法上支持动态类型、闭包等新一代语言特性。
* 2.无缝集成所有已经存在的Java类库。既支持面向对象也支持面向过程编程。（闭包：内部函数可以访问外部函数的局部变量，即使是外部函数的生命周期结束。闭包函数：声明在函数中的函数。）
* 3.可以不用分号结尾
* 4.使用def定义变量的时候可以不用指定类型。
* 5.返回时不用return 最后一句的结果就是返回值
* 6.单引号和双引号都可以声明一个字符串,但是双引号

优势：更便捷的语言，容易入门。既可以作为编程语言也可以作为脚本语言。




# Gradle特性
1.基于声明的构建和基于约定的构建
* Gradle采用了Groovy语言，这种声明式的语言是可以扩展的。可以添加新的或增强现有的语言元素。 因此，它提供了简明、可维护和易理解的构建。而且声明性语言优点在于通用任务图，你可以将其充分利用在构建中. 它提供了最大限度的灵活性，以让 Gradle 适应你的特殊需求。

2.构建结构化
* Gradle 的灵活和丰富性最终能够支持在你的构建中应用通用的设计模式。 例如，它可以很容易地将你的构建拆分为多个可重用的模块，最后再进行组装，但不要强制地进行模块的拆分。

3.多种方式管理依赖
* 不同的团队喜欢用不同的方式来管理他们的外部依赖。 从 Maven 和 Ivy 的远程仓库的传递依赖管理，到本地文件系统的 jar 包或目录，Gradle 对所有的管理策略都提供了方便的支持。

4.多项目构建
* Gradle 对多项目构建的支持非常出色。项目依赖是首先需要考虑的问题。 我们允许你在多项目构建当中对项目依赖关系进行建模，因为它们才是你真正的问题域。 Gradle 遵守你的布局。
Gradle 提供了局部构建的功能。 如果你在构建一个单独的子项目，Gradle 也会帮你构建它所依赖的所有子项目。 你也可以选择重新构建依赖于特定子项目的子项目。 这种增量构建将使得在大型构建任务中省下大量时间。

# Gradle和Maven的区别

## 性能
Gradle和Maven都采用某种形式的并行项目构建和并行依赖项解析。但是Gradle有三个特性导致其性能高于maven：
* 1.增量性 gradle会跟踪任务的输入和输出并仅运行必要的内容，并且仅在可能时处理已更改的文件，从而避免了不必要的工作量。
* 2.Build Cache（构建缓存） —重用具有相同输入的任何其他Gradle构建的构建输出，包括在机器之间。
* 3.Gradle Daemon —一个长期存在的过程，可将构建信息“热”存储在内存中。

Gradle几乎在所有情况下速度都至少是Maven的两倍

# Gradle解决依赖冲突
在Gradle中默认是传递依赖的。可以通过修改配置中transitive的值进行修改，为true则传递依赖，为false则不传递依赖，需要手动添加依赖。
依赖传递同样会造成一个问题那就是依赖冲突。可能项目中手动添加的依赖和现有的依赖所依赖的项目版本冲突，这样会导致系统运行出错。

## Gradle解决依赖冲突的方式
1.排除项目依赖
* 可以通过exclude手动排除依赖的项目中所依赖的发生冲突的项目

2.当发生依赖冲突时，制定一个版本号
```
Configuration configuration ->
configuration.resolutionStrategy.force(['org.slf4j:slf4j-api:1.6.1'])

//或者这样写
resolutionStrategy.setForcedModules(['org.slf4j:slf4j-api:1.6.1'])
```


Gradle学习参考资料：https://blog.csdn.net/u013700502/article/details/85231505


# Gradle基础
## hello world
一切的开始：hello world
```
task hello {
  print 'hello world'
}
```
一个简单的hello world代码如上，文件名为build.gradle——gradle默认的脚本构建文件。每个Gradle构建都包含三个基本构建块：project、task、property。

## gradlew
Gradle Wrapper即Gradlew的作用是简化Gradle本身的安装、部署。不同版本的项目可能需要不同版本的Gradle，手工部署的话比较麻烦，而且可能产生冲突，所以需要Gradle Wrapper帮你搞定这些事情。Gradle Wrapper是Gradle项目的一部分。

Gradle Wrapper的工作流程：
* 1.解析gradle-wrapper.properties文件，获取项目需要的 gradle 版本下载地址。
* 2.判断本地用户目录下的~/.gradle目录下是否存在该版本，不存在该版本，走第3点，存在走第4点。
* 3.下载gradle-wrapper.properties指定版本，并解压到用户目录的下 ~/.gradle文件下。
* 4.利用 ~/.gradle目录下对应的版本的 gradle 进行相应自动编译操作。

Gradle-Wrapper.properties文件中属性含义：
* distributionUrl：gradle下载的网址。
gradle文件有三种：-bin.zip（只包含可执行文件-二进制文件，可以用来编译）  -all.zip（包含所有文件包括二进制文件，源代码，文档）   -src.zip（只包含了源代码和文档，不能用于编译）
* zipStoreBase：它的值一般为GRADLE_USER_HOME，这个值所代表的路径在IDE配置上可以修改。和zipStorePath搭配，为gradle的zip文件下载到本地的位置。
* zipStorePath：和zipStoreBase搭配，为gradle的zip文件下载到本地的位置。
* distributionBase：和zipStoreBase类似，一般值为GRADLE_USER_HOME,和distributionPath搭配，为gradle的zip文件解压路径。
* distributionPath：和distributionBase搭配，为gradle的zip文件解压路径。

## Gradle工作流程
* 1.初始化阶段：首先解析settings.gradle
* 2.Configration阶段：解析每个Project中的build.gradle，解析过程中并不会执行各个build.gradle中的task。
* 3.经过Configration阶段，Project之间及内部Task之间的关系就确定了。一个 Project 包含很多 Task，每个 Task 之间有依赖关系。Configuration 会建立一个有向图来描述 Task 之间的依赖关系, 所有Project配置完成后，会有一个回调project.afterEvaluate，表示所有的模块都已经配置完了。
* 4.执行Task任务

## gradle部分指令
### compile和testCompile
compile和testCompile都是用于声明依赖，但是所声明的依赖使用范围不同，compile使用范围为src下面的代码，testCompile声明的依赖只能用于测试代码。

## Gradle依赖树 
打印gradle依赖树指令：gradle -q:项目名:dependencies --configuration compile

依赖树中的特殊符号解析：
（\*）表示这个依赖在依赖树中的其他地方也有出现
-> 指向依赖在版本冲突中最后选择的版本

# Gradle使用过程中的问题
## 初次使用Gradle执行hello world问题
代码如下
```
task hello << {
  print 'hello world'
}
```
通过gradle -q hello执行后的报错信息为：
```
A problem occurred evaluating root project 'bin'.
> Could not find method leftShift() for arguments [build_8r3b6whmgasaunaudchpimxjd$_run_closure1@96e948] on task ':hello' of type org.gradle.api.DefaultTask.
```
问题在于版本，gradle5.0及之后的版本中<<已经过时，删去或者调整版本即可。










