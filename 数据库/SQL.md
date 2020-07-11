### concat()、concat_ws和group_concat用法
concat(str1,str2....strn)，返回结果为参数拼接得来的字符串
concat_ws(str1,str2str3,.....strn)，该函数第一个参数为分隔符，将后面的参数拼接起来。
group_concat(exp1,exp2...expn)，该参数要搭配group by使用，将一个组数据中中同一属性的字段拼接在一起，expn必须为属性名，缺省的间隔符为‘，’，可以自定义间隔符。


### 开窗函数
开窗函数，可以理解为聚合函数的一个加强版。使用聚合函数的时候（不使用自查询），只能返回聚合列的值，不能返回基础行的值，而开窗函数就可以既返回聚合列的值又返回基础行的值。开窗函数用于为行提供一个窗口：将要进行聚合操作的行的集合。

demo：
```
select count(stu_name) from student //获取学生的人数
```

然后我们想查询数学60分以上的学生列表，并且每行都带有数学大于60分的学生数量：
```
select stu_name,sex,count(stu_name)
from student 
where math > 60
```
但是这种写法会报错student.sex无效，因为sex不在聚合函数中或者group by字句中

正确的写法：
```
select stu_name,sex,count(stu_name) over()
from student 
where math > 60
```

开窗函数的调用格式为：函数（） over（）

#### 聚合开窗函数
聚合函数（） over（选项） 这里的选项可以是partition by，不能是order by



#### 排序开窗函数
排序函数(列) OVER(选项)，这里的选项可以是ORDER BY子句，也可以是　OVER（PARTITION BY子句　ORDER BY子句），但不可以是PARTITION BY子句


### join
#### semi join
当一张表在另一张表找到匹配的记录之后，半连接（semi-jion）返回第一张表中的记录。
```
SELECT ... FROM outer_tables WHERE expr IN (SELECT ... FROM inner_tables ...) AND ...
```

#### inner join
内连接，也叫等值连接。要两边数据都符合条件才会拼接返回

#### left join 
左连接，理解为左边的表为基础表，右边的表有符合条件的即可拼接到左表对应行的位置，若无右表中的行符合条件的，左表对应行后面的属性为null
```
select * from a left join b where a.id = b.id
```

还可以获取只存在左表中而不在右表中的数据
```
select * from a left join b where a.id = b.id where b.id = null
```
还可以获取左表和右表的差集
```
select * from a left join b where a.id = b.id where b.id = null
union
select * from a right join b where a.id = b.id where a.id = null
```


#### right join
同上左连接，左右表对换即可。

#### cross join 
交叉连接，得到的结果是两个表的乘积，即笛卡尔积

#### full join
全连接产生的所有记录（双方匹配记录）在表A和表B。如果没有匹配,则对面将包含null。


### union
union操作符用于合并两个或多个 SELECT 语句的结果集。请注意，UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。
* union的结果集中不存在重复的数据 如果需要保留重复数据，请使用union all命令
#### union all

### case
简单的case函数
```
CASE sex
WHEN '1' THEN '男'
WHEN '2' THEN '女'
ELSE '其他' END
```

case搜索函数：
```
CASE WHEN sex = '1' THEN '男' 
WHEN sex = '2' THEN '女' 
ELSE '其他' END 
```






