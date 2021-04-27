## Java动态代理
利用java的反射技术，在运行时创建一个实现某些给定接口的新类，及其实例，代理的是接口，不是类，也不是抽象类。在运行时才知道具体的实现。

重要方法：
```
/**
 *@param loader 用哪个类加载器去加载代理对象
 *@param interfaces 动态代理类需要实现的接口
 *@param h 动态代理方法在执行时，会调用h里面的invoke方法去执行
*/
Proxy:
public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h) throws IllegalArgumentException

InvocationHandler:
/**
 *@param proxy 代理对象，newProxyInstance方法的返回对象
 *@param method 调用的方法
 *@param args 方法中的参数
*/ 
public Object invoke(object proxy, Method method,Object[] args)
```
实现demo：
```
Tool:
public interface Tool {
    void run();

    void fly();
}


Car:
public class Car implements Tool {
    @Override
    public void run() {
        System.out.println("car run");
    }

    @Override
    public void fly() {
        System.out.println("car fly");
    }
}

ToolInvocationHandler:
public class ToolInvocationHandler implements InvocationHandler {

    private final Tool tool;

    public ToolInvocationHandler(Tool tool){
        this.tool = tool;
    }

    @Override
    public Object invoke(Object o, Method method, Object[] objects) throws Throwable {
        System.out.println("before");
        //这个地方的tool如果写成了o，因为o表示代理产生的新类，那么会递归调用自己导致内存溢出
        Object invoke = method.invoke(tool,objects);
        System.out.println("after");

        return invoke;
    }
}

main:
public class Main {
    public static void main(String[] args){
        Tool car = new Car();
        Tool newCar = (Tool) Proxy.newProxyInstance(car.getClass().getClassLoader(),car.getClass().getInterfaces(),new ToolInvocationHandler(car));
        newCar.run();
        newCar.fly();
    }
}
```



## 使用CGlib实现Bean拷贝
业务中经常会将查询出来的数据封装进专门对外对DTO/VO模型中来保证系统一个可扩展性。模型转化时，两个模型的结构都是类似的，而且转换需要写很多set/get，而使用CGlib的BeanCopier就可以实现转换且不需要写很多get/set方法。其性能也比Spring的BeanUtils,Apache的BeanUtils和PropertyUtils要好很多，尤其是数据量比较大的情况下。

具体的使用情况-举个🌰：
```
//三个模型（这里我用了lombok的@Data省去了get/set方法的声明
public class User {
    private int age;
    private String name;
}


public class UserDTO {
    private int age;
    private String name;

}

public class UserVO {
    private Integer age;
    private String name;
}
```
第一个模型到第二个模型（属性类型和属性名都相同）的转化：
```
User user = new User();
user.setAge(3);
user.setName("xx大帅比");
BeanCopier beanCopier = BeanCopier.create(User.class,UserDTO.class,false);
UserDTO userDTO = new UserDTO();
beanCopier.copy(user,userDTO,null);
System.out.println(userDTO.toString());
//转换过程中无数据丢失
```

第一个模型到第三个模型（属性类型不同，属性名相同）的转化：
```
User user = new User();
user.setAge(3);
user.setName("xx大帅比");
BeanCopier beanCopier = BeanCopier.create(User.class,UserVO.class,false);
UserVO userVO = new UserVO();
beanCopier.copy(user,userVO,null);
System.out.println(userVO.toString());
//类型不同的属性转化数据丢失
```

修改第三个模型中类型相同的属性的属性名并转换：属性名不同的属性转化数据丢失
* 综上，BeanCopier必须属性名和属性类型都相同才能转化。


## 定时任务执行
Java定时任务实现方式：