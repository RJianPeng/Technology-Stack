* [Bean](#bean)

# Bean
Bean有两个最常见的属性，即id和name，两个属性都是起到标识实例的作用，推荐使用id。

* 1.id属性的命名必须满足XML的命名规范，不能以数字、符号打头，不能有空格。
* 2.name属性则没有这些规定，你可以使用几乎任何的名称。此时，虽然初始化时不会报错，但在getBean()则会报出
```
org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'a b' is defined
```

* 3.配置文件中不允许出现两个id相同的bean，否则在初始化时即会报错。
* 4.但配置文件中允许出现两个name相同的bean，在用getBean()返回实例时，后面一个Bean被返回，应该是前面那个bean被后面同名的bean覆盖了。
* 5.name属性可以用逗号（，）隔开指定多个名字，如<bean name = "b1,b2,b3">，相当于多个别名，这时通过getBean("b1"),getBean("b2"),getBean("b3")返回的都是同一个实例。
* 6.如果id和name都没有指定，则用类全名作为name



上面既然多次提到了getBean()方法，那这里讲一下这个方法的用法 //TODO
