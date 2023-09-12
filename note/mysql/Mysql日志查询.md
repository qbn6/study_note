## MySQL日志查询

## 1.  错误日志

![image-20230912163222123](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230912163222123.png)

```java
Windows：SHOW VARIABLES LIKE 'general_log%';
```

![image-20230912164449803](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230912164449803.png)



## 2.  二进制日志 binlog

### 2.1 在windows中开启Mysql数据库日志

https://blog.csdn.net/qzy_yyds/article/details/131396736 

### 2.2 如何看懂binlog

```
#判断MySQL是否已经开启binlog 
SHOW VARIABLES LIKE 'log_bin';  ||	show global variables like'log_bin';
#查看MySQL的binlog模式
SHOW VARIABLES LIKE '%log_bin%';
```

![image-20230912171845312](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230912171845312.png)

开启binlog后：

==SHOW VARIABLES LIKE '%log_bin%';==

![image-20230912180901409](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230912180901409.png)

1. ![image-20230912181248879](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230912181248879.png)
2. 解析：show binlog events in 'mysql-bin.000001';

查询结果：

![image-20230912181330266](C:\Users\Tmac1\AppData\Roaming\Typora\typora-user-images\image-20230912181330266.png)



## 3. 查询日志

开启general_log才能看到查询日志

## 4. 慢查询日志

