# Gradle是啥
完全开源的构建自动化系统 直白点就是像maven一样帮你处理项目所需要的包的一个工具。maven配置文件的编写使用的xml，gradle用的是groovy语言，语法类似于java，方便java程序员学习和使用。

## Groovy语言
DSL语言：domain specific language，即特定领域语言。Groovy即DSL语言

介绍：一种基于JVM的敏捷开发语言。可以与Java完美结合并且可以使用Java所有的库。

特性：语法上支持动态类型、闭包等新一代语言特性。无缝集成所有已经存在的Java类库。既支持面向对象也支持面向过程编程。（闭包：内部函数可以访问外部函数的局部变量，即使是外部函数的生命周期结束。闭包函数：声明在函数中的函数。）

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










