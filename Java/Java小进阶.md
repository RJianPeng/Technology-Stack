
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
