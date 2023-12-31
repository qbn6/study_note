# 第07章_InnoDB数据存储结构

**Author**: `GaoFy`

**Reference**: `shk`

**Date**: `2022-06-12`

<h2 style="color:red">其余内容见 第07章_InnoDB数据存储结构.mmap （思维导图）</h2>

---

## 1.数据库的存储结构：页

---

索引结构给我们提供了高效的索引方式，不过索引信息以及数据记录都是保存在文件上的，确切说是存储在页结构中。另一方面，索引是在存储引擎中实现的，`MySQL`服务器上的==存储引擎==负责对表中数据的读取和写入工作。不同存储引擎中，==存放的格式==一般是不同的，甚至有的存储引擎比如`Memory`都不用磁盘来存储数据。

由于`InnoDB`是`MySQL`的==默认存储引擎==，所以本章剖析`InnoDB`存储引擎的数据存储结构。

### 1.1 磁盘与内存交互基本单位：页

`InnoDB`将数据划分为若干个页，`InnoDB`中页的大小默认为**`16KB`**。

以==页==作为磁盘和内存之间交互的==基本单位==，也就是一次最少从磁盘中读取`16KB`的内容到内存中，一次最少把内存中的`16KB`内容刷新到磁盘中。也就是说，**在数据库中，不论读一行，还是读多行，都是将这些行所在的页进行加载。也就是说，数据库管理存储空间的基本单位是页（Page），数据库`I/O`操作的最小单位是页**。一个页中可以存储多个行记录。

> 记录是按照行来存储的，但是数据库的读取并不以行为单位，否则一次读取（也就是一次`I/O`操作）只能处理一行数据，效率会非常低。

![image-20220609191813892](第07章_InnoDB数据存储结构.assets\image-20220609191813892.png)

### 1.2 页结构概述

页a、页b、页c...页n这些页可以==不在物理结构上相连==，只要通过==双向链表==相关联即可。每个数据页中的记录会按照主键值从小到大的顺序组成一个==单向链表==，每个数据页都会为存储在它里边的记录生成一个==页目录==，在通过主键查找某条记录的时候可以在页目录中==使用二分法==快速定位到对应的槽，然后再遍历该槽对应分组中的记录即可快速找到指定的记录。

### 1.3 页的大小

不同的数据库管理系统（简称`DBMS`）的页大小不同。比如在`MySQL`的`InnoDB`存储引擎中，默认页的大小是`16KB`，可以通过下面的命令来进行查看：

```mysql
mysql> show variables like '%innodb_page_size%';
```

![image-20220609192818249](第07章_InnoDB数据存储结构.assets\image-20220609192818249.png)

`SQL Server`中页的大小为`8KB`，而在`Oracle`中用术语"块"（Block）来代表"页"，`Oracle`支持的块大小为`2KB`，`4KB`，`8KB`，`16KB`，`32KB`，`64KB`。

### 1.4 页的上层结构

另外在数据库中，还存在着区（Extent）、段（Segment）和表空间（Tablespace）的概念。行、页、区、段、表空间的关系如下图所示：

![image-20220609193935528](第07章_InnoDB数据存储结构.assets\image-20220609193935528.png)

区（Extent）是比页大一级的存储结构，在`InnoDB`存储引擎中，一个区会分配==64个连续的页==。因为`InnoDB`中的页大小默认是`16KB`，所以一个区的大小是`64*16KB=1MB`。

段（Segment）由一个或多个区组成，区在文件系统是一个连续分配的空间（在`InnoDB`中是连续的`64`个页），不过在段中不要求区与区之间是相邻的。==段是数据库中的分配单位，不同类型的数据库对象以不同的段形式存在==。当我们创建数据表、索引的时候，就会相应创建对应的段，比如创建一张表时会创建一个表段，创建一个索引时会创建一个索引段。

表空间（Tablespace）是一个逻辑容器，表空间存储的对象是段，在一个表空间中可以有一个或多个段，但是一个段只能属于一个表空间。数据库由一个或多个表空间组成，表空间从管理上可以划分为==系统表空间、用户表空间、撤销表空间、临时表空间==等。

## 2.页的内部结构

---

页如果按类型划分的话，场景的有==数据页（保存 B+Tree 节点）、系统页、Undo页和事务数据页==等。数据页是我们最常使用的页。

数据页的`16KB`大小的存储空间被划分为七个部分，分别是文件头（File Header）、页头（Page Header）、最大最小记录（infimum+supremum）、用户记录（User Records）、空闲空间（Free Space）、页目录（Page Directory）和文件尾（File Tailer）。

页结构的示意图如下所示：

![image-20220609195309599](第07章_InnoDB数据存储结构.assets\image-20220609195309599.png)

这7个部分作用分别如下，简单梳理如下表所示：

![image-20220609195458550](第07章_InnoDB数据存储结构.assets\image-20220609195458550.png)

### 2.3 从数据页的角度看`B+Tree`如何查询

一个`B+Tree`按照节点类型可以分成两部分：

1. 叶子节点，`B+Tree`最底层的节点，节点的高度为0，存储行记录。
2. 非叶子节点，节点的高度大于0，存储索引键和页面指针，并不存储行记录本身。

![image-20220612140146155](第07章_InnoDB数据存储结构.assets\image-20220612140146155.png)

当从页结构来理解`B+Tree`的结构时，可以帮我们理解一些通过索引进行检索的原理：

**1.`B+Tree`是如何进行记录检索的？**

如果通过`B+Tree`的索引查询行记录，首先是从`B+Tree`的根开始，逐层检索，直到找到叶子节点，也就是找到对应的数据页为止，将数据页加载到内存中，页目录中的槽（slot）采用==二分查找==的方式先找到一个粗略的记录分组，然后再在分组中通过==链表遍历==的方式查找记录。

**2.普通索引和唯一索引在查询效率上有什么不同？**

创建索引时可以是普通索引，也可以是唯一索引，那么这两个索引在查询效率上有什么不同呢？

唯一索引就是在普通索引上增加了约束性，也就是关键字唯一，找到了关键字就停止检索。而普通索引，可能会存在用户记录中的关键字相同的情况，根据页结构的原理，当我们读取一条记录时，不是单独将这条记录从磁盘中读出来，而是将这个记录所在的页加载到内存中进行读取，`InnoDB`存储引擎的页大小为`16KB`，在一个页中可能存储着上千个记录，因此在普通索引的字段上进行查找也就是在内存中多几次"判断下一条记录"的操作，对于`CPU`来说，这些操作所消耗的时间是可以忽略不计的，所以对一个索引字段进行检索，采用普通索引还是唯一索引在检索效率上基本上没有差别。

## 4.区、段与碎片区

---

### 4.1 为什么要有区？

`B+Tree`的每一层中的页都会形成一个双向链表，如果是以==页为单位==来分配存储空间的话，双向链表相邻的两个页之间的==物理位置可能离得非常远==。介绍`B+Tree`索引的适用场景的时候特别提到范围查询只需要定位到最左边的记录和最右边的记录，然后沿着双向链表一直扫描就可以了，而如果链表中相邻的两个页物理位置离得非常远，就是所谓的==随机I/O==。再一次强调，磁盘的速度和内存的速度差了好几个数量级，==随机I/O是非常慢==的，所以应该尽量让链表中相邻的页的物理位置也相邻，这样进行范围查询时才可以使用所谓的==顺序I/O==。

引入==区==的概念，一个区就是在物理位置上连续的==64个页==。因为`InnoDB`中的页大小默认是`16KB`，所以一个区的大小是`64*16KB=1MB`。在表中==数据量大==的时候，为某个索引分配空间时就不再按照页为单位分配了，而是按照==区为单位分配==，甚至在表中的数据特别多的时候，可以一次性分配多个连续的区。虽然可能造成==一点点空间的浪费==（数据不足以填充满整个区），但是从性能角度看，可以消除很多的随机`I/O`。

### 4.2 为什么要有段

对于范围查询，其实是对`B+Tree`叶子节点中的记录进行顺序扫描，而如果不区分叶子节点和非叶子节点，统统把节点代表的页面放到申请到的区中的话，进行范围扫描的效果就大打折扣了。所以`InnoDB`对`B+Tree`的==叶子节点==和==非叶子节点==进行了区别对待，也就是说叶子节点有自己独有的区，非叶子节点也有自己独有的区。存放叶子节点的区的集合就算是一个==段（segment）==，存放非叶子节点的区的集合也算是一个段。也就是说一个索引会生成2个段，一个==叶子节点段==，一个==非叶子节点段==。

除了索引的叶子节点段和非叶子节点段之外，`InnoDB`中还有为存储一些特殊的数据而定义的段，比如回滚段。所以，常见的段有==数据段、索引段、回滚段==。数据段即为`B+Tree`的叶子节点，索引段即为`B+Tree`的非叶子节点。

在`InnoDB`存储引擎中，对段的管理都是由引擎自身所完成，DBA不能也没有必要对其进行控制。这从一定程度上简化了DBA对于段的管理。

段其实不对应表空间中某一个连续的物理区域，而是一个逻辑上的概念，由若干个零散的页面以及一些完整的区组成。

### 4.3 为什么要有碎片区？

默认情况下，一个使用`InnoDB`存储引擎的表只有一个聚簇索引，一个索引会生成2个段，而段是以区为单位申请存储空间的，一个区默认占用`1M(64*16KB=1024KB)`存储空间，所以**默认情况下一个只存了几条记录的小表需要`2M`的存储空间吗？**以后每次添加一个索引都要多申请`2M`的存储空间么？这对于存储记录比较少的表简直是天大的浪费。这个问题的症结在于到现在为止我们介绍的区都是非常==纯粹的==，也就是一个区被整个分配给某一个段，或者说区的所有页面都是为了存储同一个段的数据而存在的，即便段的数据填不满区中所有的页面，那余下的页面页不能挪作他用。

为了考虑以完整的区为单位分配给某个段对于==数据量较小==的表太浪费存储空间的这种情况，`InnoDB`提出了一个==碎片（fragment）区==的概念。在一个碎片区中，并不是所有的页都是为了存储同一个段的数据而存在的，而是碎片区中的页可以用于不同的目的，比如有些页用于段A，有些页用于段B，有些页甚至哪个段都不属于。==碎片区直属于表空间==，并不属于任何一个段。

所以此后为某个段分配存储空间的策略是这样的：

- 在刚开始向表中插入数据时，段是从某个碎片区以单个页面为单位来分配存储空间的。
- 当某个段已经占用了==32个碎片区==页面之后，就会申请以完整的区为单位来分配存储空间。

所以现在段不能仅定义为是某些区的集合，更精确的应该是==某些零散的页面==以及==一些完整的区==的集合。

### 4.4 区的分类

区大体上可以分为4种类型：

- ==空闲的区（FREE）==：现在还没有用到这个区中的任何页面。
- ==有剩余空间的碎片区（FREE_FRAG）==：表示碎片区中还有可用的页面。
- ==没有剩余空间的碎片区（FULL_FRAG）==：表示碎片区中的所有页面都被使用，没有空闲页面。
- ==附属于某个段的区（FSEG）==：每一个索引都可以分为叶子节点段和非叶子节点段。

处于`FREE`、`FREE_FRAG`以及`FULL_FRAG`这三种状态的区都是独立的，直属于表空间。而处于`FSEG`状态的区是附属于某个段的。

> 如果把表空间比作是一个集团军，段就相当于师，区就相当于团。一般的团都是隶属于某个师的，就像是处于`FSEG`的区全都隶属于某个段，而处于`FREE`、`FREE_FRAG`以及`FULL_FRAG`这三种状态的区却直接隶属于表空间。就像独立团直接听命于军部一样。

## 5.表空间

---

表空间可以看作是`InnoDB`存储引擎逻辑结构的最高层，所有的数据都存放在表空间中。

表空间是一个==逻辑容器==，表空间存储的对象是段，在一个表空间中可以有一个或多个段，但是一个段只能属于一个表空间，表空间数据库由一个或多个表空间组成，表空间从管理上可以划分为==系统表空间==（System tablespace）、==独立表空间==（File-per-table tablespace）、==撤销表空间==（Undo Tablespace）和==临时表空间==（Temporary Tablespace）等。

### 5.1 独立表空间

独立表空间，即每张表有一个独立的表空间，也就是数据和索引信息都会保存在自己的表空间中。独立的表空间（即：单表）可以在不同的数据库之间进行==迁移==。

空间可以回收（`DROP TABLE`操作可自动回收表空间，其他情况，表空间不能自己回收）。如果对于统计分析或是日志表，删除大量数据后可以通过：`ALTER TABLE tableName engine=innodb;`回收不用的空间。对于使用独立表空间的表，不管怎么删除，表空间的碎片不会太严重的影响性能，而且还有机会处理。

**独立表空间结构**

独立表空间由段、区、页组成。

**真实表空间对应的文件大小**

一个新建的表对应的`.ibd`文件只占用了`96K`，才6个页面大小（MySQL 5.7中），这是因为一开始表空间占用的空间很小，因为表里面都没有数据。不过别忘了这些`.ibd`文件是==自扩展的==，随着表中数据的增多，表空间对应的文件也逐渐增大。

**查看`InnoDB`的表空间类型**

```mysql
mysql> show variables like 'innodb_file_per_table';
```

![image-20220612210325814](第07章_InnoDB数据存储结构.assets\image-20220612210325814.png)

这就意味着每张表都会单独保存一个`.ibd`文件。

### 5.2 系统表空间

系统表空间的结构和独立表空间基本类似，只不过由于整个`MySQL`进程只有一个系统表空间，在系统表空间中会额外记录一些有关整个系统信息的页面，这部分是独立表空间中没有的。

**`InnoDB`数据字典**

每当向一个表中插入一条记录的时候，`MySQL`检验过程如下：

先校验一下插入语句对应的表存不存在，插入的列和表中的列是否符合，如果语法没有问题的话，还需要知道该表的聚簇索引和所有二级索引对应的根页面是哪个表空间的哪个页面，然后把记录插入对应索引的`B+Tree`中。所以说，`MySQL`除了保存着我们插入的用户数据之外，还需要保存很多额外的信息，比方说：

> - 某个表属于哪个表空间，表里边有多少列
> - 表对应的每一个列的类型是什么
> - 该表有多少索引，每个索引对应哪几个字段，该索引对应的跟页面在哪个表空间的哪个页面
> - 该表有哪些外键，外键对应哪个表的哪些列
> - 某个表空间对应文件系统上文件路径是什么
> - ...

上述这些数据并不是我们使用`INSERT`语句插入的用户数据，实际上是为了更好的管理我们这些用户数据而不得已引入的一些额外数据，这些数据也称为==元数据==。`InnoDB`存储引擎特意定义了一些列的==内部系统表==（internal system table）来记录这些元数据：

| 表名             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| `SYS_TABLES`     | 整个`InnoDB`存储引擎中所有的表的信息                         |
| `SYS_COLUMNS`    | 整个`InnoDB`存储引擎中所有的列的信息                         |
| `SYS_INDEXES`    | 整个`InnoDB`存储引擎中所有的索引的信息                       |
| `SYS_FIELDS`     | 整个`InnoDB`存储引擎中所有的索引对应的列的信息               |
| SYS_FOREIGN      | 整个`InnoDB`存储引擎中所有的外键的信息                       |
| SYS_FOREIGN_COLS | 整个`InnoDB`存储引擎中所有的外键对应列的信息                 |
| SYS_TABLESPACES  | 整个`InnoDB`存储引擎中所有的表空间信息                       |
| SYS_DATAFILES    | 整个`InnoDB`存储引擎中所有的表空间对应文件系统的文件路径信息 |
| SYS_VIRTUAL      | 整个`InnoDB`存储引擎中所有的虚拟生成列的信息                 |

这些系统表也被称为==数据字典==，他们都是以`B+Tree`的形式保存在系统表空间的某些页面中，其中`SYS_TABLES`、`SYS_COLUMNS`、`SYS_INDEXES`、`SYS_FIELDS`这四张表尤其重要，称之为基本系统表（basic system tables），这4张表的结构：

**`SYS_TABLES`表结构**

| 列名       | 描述                                                 |
| ---------- | ---------------------------------------------------- |
| NAME       | 表的名称。主键                                       |
| ID         | InnoDB存储引擎中每张表都有一个唯一的ID。（二级索引） |
| N_COLS     | 该表拥有列的个数                                     |
| TYPE       | 表的类型，记录了一些文件格式、行格式、压缩等信息     |
| MIX_ID     | 已过时，忽略                                         |
| MIX_LEN    | 表的一些额外的属性                                   |
| CLUSTER_ID | 未使用，忽略                                         |
| SPACE      | 该表所属表空间的ID                                   |

**`SYS_COLUMNS`表结构**

| 列名     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| TABLE_ID | 该列所属表对应的ID。（与`POS`一起构成联合主键）              |
| POS      | 该列在表中是第几列                                           |
| NAME     | 该列的名称                                                   |
| MTYPE    | main data type，主数据类型，就是那堆INT、CHAR、VARCHAR、FLOAT、DOUBLE之类的东西 |
| PRTYPE   | precise type，精确数据类型，就是修饰主数据类型的那堆东西，比如是否允许NULL值，是否允许负数什么的。 |
| LEN      | 该列最多占用存储空间的字节数                                 |
| PREC     | 该列的精度，不过这列貌似都没有使用，默认值都是0              |

**`SYS_INDEXES`表结构**

| 列名           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| TABLE_ID       | 该索引所属表对应的ID。（与`ID`一起构成联合主键）             |
| ID             | `InnoDB`存储引擎中每个索引都有一个唯一的ID                   |
| NAME           | 该索引的名称                                                 |
| N_FIELDS       | 该索引包含列的个数                                           |
| TYPE           | 该索引的类型，比如聚簇索引、唯一索引、更改缓冲区的索引、全文索引、普通的二级索引等等各种类型 |
| SPACE          | 该索引根页面所在的表空间ID                                   |
| PAGE_NO        | 该索引根页面所在的页面号                                     |
| MGRE_THRESHOLD | 如果页面中的记录被删除到某个比例，就把该页面和相邻页面合并，这个值就是这个比例 |

**`SYS_FIELDS`表结构**

| 表名     | 描述                                              |
| -------- | ------------------------------------------------- |
| INDEX_ID | 该索引列所属的索引的ID。(与`POS`一起构成联合主键) |
| POS      | 该索引列在某个索引中是第几列                      |
| COL_NAME | 该索引列的名称                                    |

注意：用户是==不能直接访问==`InnoDB`的这些内部系统表，除非直接去解析系统表空间对应文件系统上的文件。不过考虑到查看这些表的内容可能有助于大家分析问题，所以在系统数据库`information_schema`中提供了一些以`innodb_`开头的表：

```mysql
mysql> use information_schema;
Database changed

mysql> show tables like 'innodb%';
+----------------------------------------+
| Tables_in_information_schema (INNODB%) |
+----------------------------------------+
| INNODB_BUFFER_PAGE                     |
| INNODB_BUFFER_PAGE_LRU                 |
| INNODB_BUFFER_POOL_STATS               |
| INNODB_CACHED_INDEXES                  |
| INNODB_CMP                             |
| INNODB_CMP_PER_INDEX                   |
| INNODB_CMP_PER_INDEX_RESET             |
| INNODB_CMP_RESET                       |
| INNODB_CMPMEM                          |
| INNODB_CMPMEM_RESET                    |
| INNODB_COLUMNS                         |
| INNODB_DATAFILES                       |
| INNODB_FIELDS                          |
| INNODB_FOREIGN                         |
| INNODB_FOREIGN_COLS                    |
| INNODB_FT_BEING_DELETED                |
| INNODB_FT_CONFIG                       |
| INNODB_FT_DEFAULT_STOPWORD             |
| INNODB_FT_DELETED                      |
| INNODB_FT_INDEX_CACHE                  |
| INNODB_FT_INDEX_TABLE                  |
| INNODB_INDEXES                         |
| INNODB_METRICS                         |
| INNODB_SESSION_TEMP_TABLESPACES        |
| INNODB_TABLES                          |
| INNODB_TABLESPACES                     |
| INNODB_TABLESPACES_BRIEF               |
| INNODB_TABLESTATS                      |
| INNODB_TEMP_TABLE_INFO                 |
| INNODB_TRX                             |
| INNODB_VIRTUAL                         |
+----------------------------------------+
31 rows in set (0.00 sec)
```

在`information_schema`数据块中的这些以`INNODB_`开头的表并不是真正的内部系统表（内部系统表就是我们上面以`SYS`开头那些表），而是在存储引擎启动时读取这些以`SYS`开头的系统表，然后填充到这些以`INNODB_`开头的表中。以`INNODB_`开头的表和以`SYS`开头的表中的字段并不完全一样，提供大家参考已经足矣。

## 附录：数据页加载的三种方式

`InnoDB`从磁盘中读取数据的==最小单位==是数据页，而你想得到的`id=xxx`的数据，就是这个数据页众多行中的一行。

对于`MySQL`存放的数据，逻辑概念上称为表，存磁盘等物理层面而言是按照==数据页==形式进行存储的，当其加载到`MySQL`中我们称之为==缓存页==。

如果缓冲池中没有该页的数据，那么缓冲池有以下三种读取数据的方式，每种方式的读取效率都是不同的：

**1.内存读取**

如果该数据都存在于内存中，基本上指向时间在`1ms`左右，效率还是很高的。

![image-20220612201230651](第07章_InnoDB数据存储结构.assets\image-20220612201230651.png)

**2.随机读取**

如果数据没有在内存中，就需要磁盘上对该页进行查找，整体时间预估在`10ms`左右，这`10ms`中有`6ms`是磁盘在实际繁忙时间（包括了==寻道和半圈旋转时间==），有`3ms`是对可能发生的排队时间的预估值，另外还有`1ms`的传输时间，将页从磁盘服务器缓冲区传输到数据库缓冲区中。这`10ms`看起来很快，但实际上对于数据库来说消耗的时间已经非常长了，因为这还只是一个页的读取时间。

![image-20220612201614511](第07章_InnoDB数据存储结构.assets\image-20220612201614511.png)

**3.顺序读取**

顺序读取其实是一种批量读取的方式，因为我们请求的==数据在磁盘上往往都是相邻存储的==，顺序读取可以帮我们批量读取页面，这样的话，一次性加载到缓冲池中就不需要再对其他页面单独进行磁盘`I/O`操作了。如果一个磁盘的吞吐量是`40MB/S`，那么对于一个`16KB`大小的页来说，一次可以顺序读取`2560(40MB/16KB)`个页，相当于一个页的读取时间为`0.4ms`。采用批量读取的方式，即便是从磁盘上进行读取，效率也比从内存中只单独读取一个页的效率更高。

