# 谈：基于约束条件的SQL攻击
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;今天读了几篇如题的文章，[原文见](https://dhavalkapil.com/blogs/SQL-Attack-Constraint-Based/)，[译文见](http://www.freebuf.com/articles/web/124537.html)，[参考文章](http://goodwaf.com/2016/12/30/%E5%9F%BA%E4%BA%8E%E7%BA%A6%E6%9D%9F%E6%9D%A1%E4%BB%B6%E7%9A%84SQL%E6%94%BB%E5%87%BB/)。下面谈下自己的理解。   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先，漏洞是利用数据库的两个特点：  
1. 数据库字符串比较：在数据库对字符串进行比较时，如果两个字符串的长度不一样，则会将较短的字符串末尾填充空格，使两个字符串的长度一致，比如，字符串A:[String]和字符串B:[String2]进行比较时，由于String2比String多了一个字符2，这时MySQL会将字符串A填充为[String ]，即在原来字符串后面加了一个空格，使两个字符串长度一致。看如下两条查询语句：

```
select * from users where username='Dumb'
select * from users where username='Dumb    '
```


>>它们的查询结果是一致的，即第二条查询语句中Dumb后面的空格并没有对查询有任何影响。因为在MySQL把查询语句里的username和数据库里的username值进行比较时，它们就是一个字符串的比较操作，符合上述特征。

2. INSERT截断：这是数据库的另一个特性，当设计一个字段时，我们都必须对其设定一个最大长度，比如CHAR(10)，VARCHAR(20)等等。但是当实际插入数据的长度超过限制时，数据库就会将其进行截断，只保留限定的长度。  
这个截断得有个前提：就是数据入库之前，代码里没有做参数的长度判断。如果代码里做了长度判断，不允许长于数据库里的列长度，那就不会产生截断了（lengh(name_parameter)<=length(name)）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;比如,数据表admin_user有列name(varchar(5))和列password(varchar(20))

```
insert into admin_user(name,passoword) values('shen    1','passwd1')
```
==>>这个时候数据表admin_user里只有"shen ",其余字符被截掉==  
**重点来了**：  
如果用户名不是唯一的，即没有约束（unique）,这个时候就可以使网站拥有两个相同的用户名。而如果网站业务逻辑是根据用户名进行判断的，后果可想而知，我就可以仿造任何用户。如果仿造的同权限用户，实现的就是水平越权，如果仿造的是高权限用户，实现的就是垂直越权。