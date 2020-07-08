### concat()、concat_ws和group_concat用法
concat(str1,str2....strn)，返回结果为参数拼接得来的字符串
concat_ws(str1,str2str3,.....strn)，该函数第一个参数为分隔符，将后面的参数拼接起来。
group_concat(exp1,exp2...expn)，该参数要搭配group by使用，将一个组数据中中同一属性的字段拼接在一起，expn必须为属性名，缺省的间隔符为‘，’，可以自定义间隔符。


