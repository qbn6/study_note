# 第02章_MySQL的数据目录

**Author**: `GaoFy`

**Reference**: `shk`

**Date**: `2022-06-04`

---

## 1.MySQL 8.0的主要目录结构

---

```shell
[root@MySQL8Learning ~]# find / -name mysql
```

安装好`MySQL 8.0`之后，我们查看如下的目录结构：

### 1.1 数据库文件的存放路径

**`MySQL`数据库文件的存放路径：`/var/lib/mysql/`**

`MySQL`服务器程序在启动时会到文件系统的某个目录下加载一些文件，之后在运行过程中产生的数据也都会存到这个目录下的某些文件中，这个目录就称为==数据目录==。

`MySQL`把数据都存到哪个路径下呢？其实==数据目录==对应着一个系统变量`datadir`，在使用客户端与服务器建立连接之后查看这个系统变量的值就可以了：

```mysql
mysql> show variables like 'datadir';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.04 sec)
```

从结果中可以看出，数据目录就是`/var/lib/mysql/`。

### 1.2 相关命令目录

**相关命令目录：`/usr/bin (mysqldadmin、mysqlbinlog、mysqldump等命令)` 和`/usr/sbin`**。

==安装目录==下非常重要的`bin`目录，它里面存储了许多关于控制客户端程序和服务器程序的命令（许多可执行文件，比如`mysql`、`mysqladmin`、`mysqldump`等）。而==数据目录==是用来存储`MySQL`在运行过程中产生的数据，注意区分开二者。

### 1.3 配置文件目录

**配置文件目录：`/usr/share/mysql-8.0 （命令及配置文件）`，`/etc`（如`my.cnf`）**

## 2.数据库和文件系统的关系

---

像`InnoDB`、`MyISAM`这样的存储引擎都是把表存储在磁盘上的，操作系统用来管理磁盘的结构被称为==文件系统==，所以用专业一点的话来表述就是：像`InnoDB`、`MyISAM`这样的存储引擎都是把==表存储在文件系统上==的。当我们想读取数据时，这些存储引擎会从文件系统中把数据读出来返回给我们，当我们想写数据时，这些存储引擎会把这些数据又写回文件系统。本章学习`InnoDB`和`MyISAM`这两个存储引擎的数据如何在文件系统中存储。

### 2.1 查看默认数据库

```mysql
mysql> SHOW DATABASES;
```

可以看到有`4`个数据库是属于`MySQL`自带的系统数据库。

- `mysql`

`MySQL`系统自带的核心数据库，它存储了`MySQL`的用户账户和权限信息，一些存储过程、事件的定义信息，一些运行过程中产生的日志信息，一些帮助信息以及时区信息等。

- `information_schema`

`MySQL`系统自带的数据库，这个数据库保存着`MySQL`服务器==维护的所有其他数据库的信息==，比如有哪些表、哪些视图、哪些触发器、哪些列、哪些索引。这些信息并不是真实的用户数据，而是一些描述性信息，有时也称之为==元数据==。在系统数据库`inforamtion_schema`中提供了一些以(~~innodb_sys~~)`innodb_`开头的表，用于表示内部系统表。

```mysql
mysql> USE information_schema;
Database changed

mysql> show tables like 'innodb_%';
+-----------------------------------------+
| Tables_in_information_schema (INNODB_%) |
+-----------------------------------------+
| INNODB_BUFFER_PAGE                      |
| INNODB_BUFFER_PAGE_LRU                  |
| INNODB_BUFFER_POOL_STATS                |
| INNODB_CACHED_INDEXES                   |
| INNODB_CMP                              |
| INNODB_CMPMEM                           |
| INNODB_CMPMEM_RESET                     |
| INNODB_CMP_PER_INDEX                    |
| INNODB_CMP_PER_INDEX_RESET              |
| INNODB_CMP_RESET                        |
| INNODB_COLUMNS                          |
| INNODB_DATAFILES                        |
| INNODB_FIELDS                           |
| INNODB_FOREIGN                          |
| INNODB_FOREIGN_COLS                     |
| INNODB_FT_BEING_DELETED                 |
| INNODB_FT_CONFIG                        |
| INNODB_FT_DEFAULT_STOPWORD              |
| INNODB_FT_DELETED                       |
| INNODB_FT_INDEX_CACHE                   |
| INNODB_FT_INDEX_TABLE                   |
| INNODB_INDEXES                          |
| INNODB_METRICS                          |
| INNODB_SESSION_TEMP_TABLESPACES         |
| INNODB_TABLES                           |
| INNODB_TABLESPACES                      |
| INNODB_TABLESPACES_BRIEF                |
| INNODB_TABLESTATS                       |
| INNODB_TEMP_TABLE_INFO                  |
| INNODB_TRX                              |
| INNODB_VIRTUAL                          |
+-----------------------------------------+
31 rows in set (0.00 sec)
```

- `performance_schema`

`MySQL`系统自带的数据库，这个数据库里主要保存`MySQL`服务器运行过程中的一些状态信息，可以用来==监控 MySQL 服务的各类性能指标==。包括统计最近执行了哪些语句，在执行过程的每个阶段都花费了多长时间，内存的使用情况等信息。

- `sys`

`MySQL`系统自带数据库，这个数据库主要通过==视图==的形式把`information_schema`和`performance_schema`结合起来，帮助系统管理员和开发人员监控`MySQL`的技术性能。

### 2.2 数据库在文件系统中的表示

使用`CREATE DATABSE 数据库名`语句创建一个数据库时，在文件系统上实际发生了什么？其实很简单，每个数据库都对应数据目录下的一个子目录，或者说对应一个文件夹，每当新建一个数据库时，`MySQL`会帮我们做这两件事：

1. 在==数据目录==下创建一个和数据库名同名的子目录。
2. 在与该数据库名的子目录下创建一个名为`db.opt`的文件（仅限`MySQL 5.7`及之前版本），这个文件中包含了==该数据库的各种属性==，比如该数据库的默认字符集和比较规则。

### 2.3 表在文件系统中的表示

数据其实都是以==记录的形式==插入到表中的，每个表的信息其实可以分为两种：

1. 表结构的定义
2. 表中的数据

==表结构==就是该表的名称，表里边有多少列，每个列的数据类型，约束条件和索引，使用的字符集和比较规则等各种信息，这些信息都体现在我们的建表语句中了。

#### 2.3.1 `InnoDB`存储引擎模式

**1.表结构**

为了保存表结构，`InnoDB`在==数据目录==下对应的数据库子目录下创建了一个专门用于==描述表结构的文件==，文件名是这样的：

```file
表名.frm
```

比方说我们在`atguigu`数据库下创建一个名为`test`的表：

```mysql
mysql> USE atguigu;
Database changed

mysql> CREATE TABLE test(
	-> 	c1 INT
	-> );
Query OK, 0 rows affected (0.03 sec)
```

那在数据库`atguigu`对应的子目录下就会创建一个名为`test.frm`的用于描述表结构的文件。`.frm`文件的格式在不同的平台上都是相同的。这个后缀名为`.frm`是以==二进制格式==存储的，直接打开是乱码。

**2.表中数据和索引**

>储备知识：
>
>- `InnoDB`其实是使用==页==为基本单位来管理存储空间的，默认的==页==大小为`16KB`。
>- 对于`InnoDB`存储引擎来说，每个索引都对应着一颗`B+`树，该`B+`树的每个节点都是一个数据页，数据页之间不必要是物理连续的，因为数据页之间有==双向链表==来维护这些页的顺序。
>- `InnoDB`的聚簇索引的叶子节点存储了完整的用户记录，也就是所谓的索引即数据，数据即索引。

为了更好的管理这些页，`InnoDB`提出了一个==表空间==或者==文件空间==（英文名：`table space`或者`file space`）的概念，这个表空间是一个抽象的概念，它可以对应文件系统上一个或多个真实文件（不同表空间对应的文件数量可能不同）。每一个==表空间==可以被划分为很多个==页==，表数据就存放在某个==表空间==下的某些页里。这里表空间有几种不同的类型：

**① 系统表空间(`system tablespace`)**

默认情况下，`InnoDB`会在数据目录下创建一个名为`ibdata1`，大小为`12M`的文件，这个文件就是对应的==系统表空间==在文件系统上的表示。怎么才`12M`？注意这个文件是==自扩展文件==，当不够用时它会自己增加文件大小。

当然，如果你想让系统表空间对应文件系统上多个实际文件，或者仅仅觉得原来的`ibdata1`这个文件名难听，那可以在`MySQL`启动时配置对应的文件路径以及它们的大小，比如修改一下`my.cnf`配置文件：

```properties
[server]
innodb_data_file_path=data1:512M;data2:512M:autoextend
```

这样在`MySQL`启动之后就会创建这两个`512M`大小的文件作为==系统表空间==，其中的`autoextend`表明这两个文件如果不够用会自动扩展`data2`文件的大小。

需要注意一点是，在==一个 MySQL 服务器中，系统表空间只有一份==。从`MySQL 5.5.7`到`MySQL 5.6.6`之间的各个版本中，**表中的数据都会被默认存储到这个系统表空间**。

**② 独立表空间(file-per-table tablespace)**

在`MySQL 5.6.6`以及之后的版本中，`InnoDB`并不会默认的把各个表的数据存储到系统表空间，而是为==每一个表建立一个独立表空间==，也就是说创建了多少个表，就有多少个独立表空间。使用==独立表空间==来存储表数据的话，会在该表所属数据库对应的子目录下创建一个表示该独立表空间的文件，文件名和表名相同，只不过添加了一个`.ibd`的扩展名而已，所以完整的文件名称长这样：

```file
表名.ibd
```

比如：使用==独立表空间==去存储`atguigu`数据库下的`test`表，那么在该表所在数据库对应的`atguigu`目录下会为`test`表创建者两个文件。

```file
test.frm
test.ibd
```

其中`test.ibd`文件就用来存储`test`表中的数据和索引。

**③ 系统表空间与独立表空间的设置**

指定使用==系统表空间==还是==独立表空间==来存储数据，这个功能由启动参数`innodb_file_per_table`控制，比如说刻意将数据存储到==系统表空间==时，可以在启动`MySQL`服务器时这样配置：

```properties
[server]
innodb_file_per_table=0 # 0：代表使用系统表空间。 1：表示使用独立表空间
```

默认情况：

```mysql
mysql> show variables like 'innodb_file_per_table';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
1 row in set (0.00 sec)
```

说明：`innodb_file_per_table`参数的修改只对新建的表起作用，对于已经分配了表空间的表并不起作用。

如果想把已经存在系统表空间中的表转移到独立表空间，可以使用下边的语句：

```mysql
ALTER TABLE 表名 TABLESPACE [=] innodb_file_per_table;
```

或者把已存在独立表空间的表转移到系统表空间，使用下边的语句：

```mysql
ALTER TABLE 表名 TABLESPACE [=] innodb_system;
```

其中中括号括起来的`=`可有可无。

**④ 其他类型的表空间**

随着`MySQL`的发展，除了上述两个老牌表空间之外，现在还新提出一些不同类型的表空间，比如通用表空间（`general tablespace`）、临时表空间（`temporary tablespace`）等。

**3.疑惑**

**`.frm`在`MySQL 8.0`中不存在了，哪里去了**

这就需要解析`ibd`文件。`Oracle`官方将`frm`文件的信息及更多信息移动到叫做序列化字典信息（`Serialized Dictionary Information, SDI`），`SDI`被写在`ibd`文件内部。`MySQL 8.0`属于`Oracle`公司的，同理。

为了从`ibd`文件中提取`SDI`信息，`Oracle`提供了一个应用程序`ibd2sdi`。

<span style="color:skyblue">== ibd2sdi 官方文档 ==</span>

这个工具不需要下载，`MySQL8`自带，只要配好环境变量随处用。

（1）查看表结构

到存储`ibd`文件的目录下，执行如下命令：

```shell
ibd2sdi --dump-file=student.txt student.ibd
```

结果如图所示

![image-20220604203325037](第02章_MySQL的数据目录.assets\image-20220604203325037.png)

这样`ibd2sdi`就会把`.ibd`里存储的表结构以`json`的格式保存在`student.txt`中。

![image-20220604203649232](第02章_MySQL的数据目录.assets\image-20220604203649232.png)

图中标记部分从上到下分别表示：

- 表名
- 列
- 列名
- 列的长度

通过上面的测试结果可以发现，`MySQL8`把之前版本的`frm`文件合并到`ibd`文件中了。

#### 2.3.2 `MyISAM`存储引擎模式

**1.表结构**

在存储表结构方面，`MyISAM`和`InnoDB`一样，也是在==数据目录==下对应的数据库子目录下创建一个专门用于描述表结构的文件：

```file
表名.frm
```

**2.表中数据和索引**

在`MyISAM`中的索引全部都是==二级索引==，该存储引擎的==数据和索引是分开存储==的。所以在文件系统中也是使用不同的文件来存储数据文件和索引文件，同时表数据都存放在对应的数据库子目录下。假如`test`表使用`MyISAM`存储引擎的话，那么在它所在数据库对应的`atguigu`目录下会为`test`表创建这三个文件：

```file
test.frm 存储表结构
test.MYD 存储数据（MYData）
test.MYI 存储索引（MYIndex）
```

其中`test.MYD`代表表的数据文件，也就是我们插入的用户记录。采用独立表存储模式，每个表对应一个`MYD`文件；`test.MYI`代表表的索引文件，为该表创建的索引都会放在这个文件中。

举例：创建一个`MyISAM`表，使用`ENGINE`选项显式指定引擎。因为`InnoDB`是默认引擎。

```mysql
CREATE TABLE student_myisam(
	id BIGINT NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(64) DEFAULT NULL,
    age INT DEFAULT NULL,
    sex VARCHAR(2) DEFAULT NULL,
    PRIMARY KEY(id)
)ENGINE=MYISAM AUTO_INCREMENT=0 DEFAULT CHARSET=utf8mb3;
```

**MySQL8 版本中**

（1）进入数据库目录

![image-20220604204935104](第02章_MySQL的数据目录.assets\image-20220604204935104.png)

包含三个文件

```file
student_myisam_361.sdi -- 存储元数据
student_myisam.MYD -- 存储数据
student_myisam.MYI -- 存储索引
```

对于`InnoDB`表，`SDI`与`InnoDB`用户表空间中的数据一起存储。对于`MyISAM`和其他存储引擎，它被写入数据目录中的`.sdi`文件。

在`MySQL 8.0`中，`MyISAM`存储引擎不提供分区支持。在以前版本的`MySQL`中创建的分区`MyISAM`表不能在`MySQL 8.0`中使用。

**在`MySQL 5.7`中**：

（1）查看文件目录

![image-20220604205306557](第02章_MySQL的数据目录.assets\image-20220604205306557.png)

包含三个文件

```file
student_myisam.frm -- 存储元数据
student_myisam.MYD -- 存储数据
student_myisam.MYI -- 存储索引
```

可以发现，在之前的数据库版本中，`MyISAM`引擎已存在`frm`文件，但是在`MySQL8`之后也和`InnoDB`引擎一样去掉了，放入`sdi`文件中。

### 2.4 小结

举例：==数据库a==，==表b==。

1. 如果表`b`采用`InnoDB`，`data\a`中会产生`1`个或者`2`个文件；
   - `b.frm`：描述表结构文件，字段长度等
   - 如果采用==系统表空间==模式，数据信息和索引信息都存储在`ibdata1`中
   - 如果采用==独立表空间==存储模式，`data\a`中还会产生`b.ibd`文件（存储数据信息和索引信息）

此外：

① `MySQL 5.7`中会在`data\a`的目录下生成`db.opt`文件用于保存数据库的相关配置。比如：字符集、比较规则，而`MySQL 8.0`不再提供`db.opt`文件。

② `MySQL 8.0`中不再单独提供`b.frm`，而是合并在`b.ibd`文件中。

2. 如果表`b`采用`MyISAM`，`data\a`中会产生`3`个文件：

   - `MySQL 5.7`中：`b.frm`：描述表结构文件，字段长度等。

     `MySQL 8.0`中：`b.sdi`：描述表结构文件，字段长度等。

   - `b.MYD`：数据信息文件，存储数据信息（如果采用独立表存储模式）

   - `b.MYI`：存放索引信息文件

### 2.5 视图在文件系统中的表示

`MySQL`中的==视图==其实是==虚拟表==，也就是某个查询语句的一个别名而已，所以在存储视图时不需要存储真实的数据的，只需要把它的结构存储起来就行了。和表一样，描述视图结构的文件也会被存储到所属数据库对应的子目录下面，只会存储一个`视图名.frm`的文件，如下图的：`emp_details_view.frm`

![image-20220604210620410](第02章_MySQL的数据目录.assets\image-20220604210620410.png)

> <span style="color:red">MySQL 8.0的情况</span>：
>
> 没有在目标数据库子目录下找到关于视图的文件
>
> 猜测也是保存到所使用物理表的 ibd 文件中

### 2.6 其他的文件

除了上边这些用户自己存储的数据以外，==数据目录==下还包括为了更好运行程序的一些额外文件，主要包括这几种类型的文件：

- **服务器进程文件**

每运行一个`MySQL`服务器程序，都意味着启动一个进程。`MySQL`服务器会把自己的进程`ID`写入到一个文件中。

- **服务器日志文件**

在服务器运行过程中，会产生各种各样的日志，比如常规的查询日志、错误日志、二进制日志、`redo`日志等。这些日志各有各的用途。

- **默认/自动生成的`SSL`和`RSA`证书和密钥文件**

主要是为了客户端和服务器安全通信而创建的一些文件。