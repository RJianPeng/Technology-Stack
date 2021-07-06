* [Bean](#bean)
* [常见注解](#常见注解)

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


# 常见注解
### @ConditionalOnBean
仅仅在当前上下文中存在某个对象时，才会实例化一个bean

### @ConditionalOnClass
当某个class位于类路径上，才会实例化一个bean

### @ConditionalOnExpression
当表达式为true的时候，才会实例化一个bean

### @ConditionalOnMissingBean
在当前上下文中不存在某个bean时，才会实例化一个Bean

## spring切面相关
(参考文档：https://blog.csdn.net/w_linux/article/details/80230222)
### @Pointcut 
声明切点
```
JoinPoint相关方法： 

//获取封装了方法签名信息的对象，在该对象中可以获取到目标方法名，所属类的Class信息等
Signature getSignature();

//获取传入目标方法的参数对象
Object[] getArgs();

//获取被代理的对象
Object getTarget();

//获取代理对象
Object getThis();
```

ProceedingJoinPoint对象，是JoinPoint的子接口，该对象用在@Around的切面方法中
```
ProceedingJoinPoint常用方法：

//执行目标方法
Object proceed() throws Throwable

//传入的新的参数去执行目标方法
Object proceed(Object[] var1) throws Throwable
```

### @Before
前置通知，在方法执行之前执行，里面的参数可以是声明的切点，或者直接是切点指定的内容从而跳过切点的声明

### @After
后置通知，方法执行之后执行（不管是否发生异常）

### @AfterReturning
返回通知，方法正常执行完毕后执行

### @AfterThrowing 
异常通知，在方法抛出异常之后执行

### @Around
环绕通知，包裹了切点方法的整个执行周期

### @Transactional
spring的事物使用注解，通过aop的方式实现，提供的是非侵入式的实现方式
修饰方法，表示当前方法中对同一个库的操作打包为一个事物。
默认的connection是自动提交的，所以被该注解修饰的方法，在进入方法的时候就会获取链接，并且把这个链接放入ThreadLocal中，并且将自动提交设置为false，之后在这个方法中，对db的操作都会从ThreadLocal中获取这个连接。
spring事物参考文档：https://blog.csdn.net/fuqianming/article/details/100560200
ps:数据库底层的事物提交和回滚都是通过binglog和redolog实现的(//TODO 这里后面可以再复习下)

#### spring事务的传播性
就是同时存在多个事务的时候，Spring如何处理它们的问题


# 拦截器
## HandlerInterceptor
```
public interface HandlerInterceptor {
    //在业务处理器处理请求之前被调用
	boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception;

    //TODO 可以测一下
	//在业务处理器处请求执行完成后，返回之前处理
	void postHandle(
			HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
			throws Exception;

	//DispatcherServlet完全处理完请求后被调用，可以用于资源清理等
	void afterCompletion(
			HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception;

}
``` 

## ResponseBodyAdvice