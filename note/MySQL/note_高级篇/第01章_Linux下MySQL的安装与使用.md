# 第01章_Linux下MySQL的安装与使用

**Author**: `GaoFy`

**Reference**: `shk`

**Date**: `2022-06-02`

---

## 1.安装前说明

---

### 1.1 Linux 系统及工具的准备

- 安装并启动好两台虚拟机：`CentOS 7`
  - 掌握克隆虚拟机的操作
    - `mac`地址
    - 主机名
    - `ip`地址
    - `UUID`
- 安装有`Xshell`和`Xftp`等访问`CentOS`系统的工具
- `CentOS6`和`CentOS7`在`MySQL`的使用中的区别

```
防火墙：6是 iptables，7是 firewalld

启动服务的命令：6是 service，7是 systemctl
```

### 1.2 查看是否安装过 MySQL

- 如果是用`rpm`安装，检查一下`RPM PACKAGE`：

```shell
rpm -qa | grep -i mysql # -i 忽略大小写
```

- 检查`mysql service`：

```shell
systemctl status mysqld.service
```

### 1.3 MySQL 的卸载

**1.关闭 mysql 服务**

```shell
systemctl stop mysqld.service
```

**2.查看当前 mysql 安装状况**

```shell
rpm -qa | grep -i mysql
# 或
yum list installed | grep mysql
```

**3.卸载上述命令查询出的已安装程序**

```shell
yum remove mysql-xxx mysql-xxx mysql-xxx
```

务必卸载干净，反复执行`rpm -qa | grep -i mysql`确认是否有卸载残留

**4.删除 mysql 相关文件**

- 查出相关文件

```shell
find / -name mysql
```

- 删除上述命令查找出的相关文件

```shell
rm -rf xxx
```

**5.删除 my.cnf**

```shell
rm -rf /etc/my.cnf
```

## 2. MySQL 的 Linux 版安装

---

### 2.1 MySQL 的 4 大版本

> - **MySQL Community Server 社区版本**，开源免费，自由下载，但不提供官方技术支持，适用于大多数普通用户。
> - **MySQL Enterprise Edition 企业版本**，需付费，不能在线下载，可以试用30天。提供了更多的功能和更完备的技术支持，更适合于对数据库的功能和可靠性要求较高的企业客户。
> - **MySQL Cluster 集群版**，开源免费。用于架设集群服务器，可将几个`MySQL Server`封装成一个`Server`。需要在社区版或企业版的基础上使用。
> - **MySQL Cluster CGE 高级集群版**，需付费。

- 截止目前，官方最新版本为`8.0.29`

此外，官方还提供了`MySQL Workbench`（GUITOOL）一款专为`MySQL`设计的==ER/数据库建模工具==。它是著名的数据库设计工具`DBDesigner4`的继任者。`MySQL Workbench`又分为两个版本，分别是==社区版==`(MySQL Workbench OSS)`、==商用版==(`MySQL WorkbenchSE`)。

### 2.2 下载 MySQL 指定版本

**1.下载地址**

官网：https://www.mysql.com

**2.打开官网，点击 DOWNLOADS**

然后，点击`MySQL Community(GPL) Downloads`

**3.点击 MySQL Community Server**

**4.在`General Availability(GA) Releases`中选择合适版本**

- 如果安装`Windows`系统下`MySQL`，推荐下载==MSI安装程序==；点击`Go to Download Page`进行下载即可

- `Windows`下的`MySQL`安装有两种安装程序
  - `mysql-installer-web-community-8.0.25.0.msi`下载程序大小：2.4M；安装时需要联网安装组件。
  - `mysql-installer-community-8.0.25.0.msi`下载程序大小：435.7M；安装时离线安装即可。**推荐**

**5. Linux系统安装 MySQL 的几种方式**

**5.1 Linux系统下安装软件的常用三种方式：**

**方式1： rpm 命令**

使用`rpm`命令安装扩展名为`.rpm`的软件包。

`.rpm`包的一般格式：

![image-20220602194532641](第01章_Linux下MySQL的安装与使用.assets\image-20220602194532641.png)

**方式2：`yum`命令**

需联网，从==互联网获取==的`yum`源，直接使用`yum`命令安装。

**方式3：编译安装源码包**

针对`.tar .gz`这样的压缩格式。要用`tar`命令来解压；如果是其他压缩格式，就使用其他命令。

**5.2 `Linux`系统下安装`MySQL`，官方给出多种安装方式**

| 安装方式       | 特点                                                 |
| -------------- | ---------------------------------------------------- |
| rpm            | 安装简单，灵活性差，无法灵活选择版本、升级           |
| rpm repository | 安装包极小，版本安装简单灵活，升级方便，需要联网安装 |
| 通用二进制包   | 安装比较复杂，灵活性高，平台通用性好                 |
| 源码包         | 安装最复杂，时间长，参数设置灵活，性能好             |

- 这不能直接选择`CentOS 7`系统的版本，所以选择与之对应的`Red Hat Enterprise Linux`
- http://downloads.mysql.com/archives/community/ 直接点`Download`下载`RPM Bundle`全量包。包括了所有下面的组件。不需要一个一个下载了。

![image-20220602195344653](第01章_Linux下MySQL的安装与使用.assets\image-20220602195344653.png)

**6.下载的`tar`包，用压缩工具打开**

![image-20220602195421928](第01章_Linux下MySQL的安装与使用.assets\image-20220602195421928.png)

- 解压后`rpm`安装包（红框为抽取出来的安装包）

![image-20220602195510341](第01章_Linux下MySQL的安装与使用.assets\image-20220602195510341.png)

### 2.3 CentOS7 下检查 MySQL 依赖

#### 1.检查`/tmp`临时目录权限（必不可少）

由于`MySQL`安装过程中，会通过`MySQL`用户在`/tmp`目录下新建`tmp_db`文件，所以请给`/tmp`较大的权限。执行：

```shell
chmod -R 777 /tmp
```

#### 2.安装前，检查依赖

```shell
rpm -qa | grep libaio
```

- 如果存在`libaio`包如下：

![image-20220602195806782](第01章_Linux下MySQL的安装与使用.assets\image-20220602195806782.png)

```shell
rpm -qa | grep net-tools
```

- 如果存在`net-tools`包如下：

![image-20220602195928521](第01章_Linux下MySQL的安装与使用.assets\image-20220602195928521.png)

- 如果不存在需要到`centos`安装盘里进行`rpm`安装。安装`Linux`如果带图形化界面，这些都是安装好的。

### 2.4 CentOS7 下 MySQL 安装过程

#### 1.将安装程序拷贝到`/opt/software`目录下

在`MySQL`的安装文件目录下执行：（必须按照顺序执行）

```shell
rpm -ivh mysql-community-common-8.0.25-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-plugins-8.0.25-1.el7.x86_64.rpm

rpm -ivh mysql-community-libs-8.0.25-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-8.0.25-1.el7.x86_64.rpm

rpm -ivh mysql-community-server-8.0.25-1.el7.x86_64.rpm
```

- 注意：如在检查工作时，没有检查`MySQL`依赖环境在安装`mysql-community-server`会报错
- `rpm`是`Redhat Package Manage`缩写，通过`RPM`的管理，用户可以把源代码包装成以`rpm`为扩展名的文件形式，易于安装。
- `-i`，`--install`安装软件包
- `-v`，`--verbose`提供更多的详细信息输出
- `-h`，`-hash`软件包安装时列出哈希标记（和`-v`一起使用效果更好），展示进度条

![image-20220602200733770](第01章_Linux下MySQL的安装与使用.assets\image-20220602200733770.png)

#### 2.安装过程截图

![image-20220602201055477](第01章_Linux下MySQL的安装与使用.assets\image-20220602201055477.png)

安装过程中可能的报错信息：

![image-20220602201134688](第01章_Linux下MySQL的安装与使用.assets\image-20220602201134688.png)

> 一个命令：`yum remove mysql-libs`解决，清除之前安装过的依赖即可

#### 3.查看MySQL版本

执行如下命令，如果成功表示安装`MySQL`成功。类似`java -version`如果打出版本等信息

```shell
mysql --version
# 或
mysqladmin --version
```

![image-20220602201324490](第01章_Linux下MySQL的安装与使用.assets\image-20220602201324490.png)

执行如下命令，查看是否安装成功。需要增加`-i`不用去区分大小写，否则搜索不到。

```shell
rpm -qa | grep -i mysql
```

![image-20220602201628100](第01章_Linux下MySQL的安装与使用.assets\image-20220602201628100.png)

#### 4.服务的初始化

为了保证数据库目录与文件的所有者为`MySQL`登录用户，如果你是以`root`身份运行`MySQL`服务，需要执行下面的命令初始化：

```shell
mysqld --initialize --user=mysql
```

说明：`--initialize`选项默认以"安全"模式来初始化，则会为`root`用户生成一个密码并将==该密码标记为过期==，登录后你需要设置一个新的密码。生成的==临时密码==会往日志中记录一份。

查看密码：

```shell
cat /var/log/mysqld.log
```

![image-20220602202132253](第01章_Linux下MySQL的安装与使用.assets\image-20220602202132253.png)

`root@localhost:`后面就是初始化的密码

#### 5.启动MySQL，查看状态

```shell
# 加不加 .service 后缀都可以

启动：systemctl start mysqld.service

关闭：systemctl stop mysqld.service

重启：systemctl restart mysqld.service

查看状态：systemctl status mysqld.service
```

> `mysqld`这个可执行文件就代表着`MySQL`服务器程序，运行这个可执行文件就可以直接启动一个服务器进程。

![image-20220602202533945](第01章_Linux下MySQL的安装与使用.assets\image-20220602202533945.png)

查看进程：

```shell
ps -ef | grep -i mysql
```

![image-20220602202614675](第01章_Linux下MySQL的安装与使用.assets\image-20220602202614675.png)

#### 6.查看MySQL服务是否自启动

```shell
systemctl list-unit-files | grep mysqld.service
```

![image-20220602202736224](第01章_Linux下MySQL的安装与使用.assets\image-20220602202736224.png)

默认是`enabled`。

- 如果不是`enabled`可以运行如下命令设置自启动

```shell
systemctl enable mysqld.service
```

- 如果希望不进行自启动，运行如下命令设置

```shell
systemctl disable mysqld.service
```

## 3.MySQL登录

---

### 3.1 首次登录

通过`mysql -hlocalhost -P3306 -uroot -p`进行登录，在`Enter password`：录入初始化密码

![image-20220602203326083](第01章_Linux下MySQL的安装与使用.assets\image-20220602203326083.png)

### 3.2 修改密码

- 因为初始化密码默认是过期的，所以查看数据库会报错
- 修改密码：

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```

- `5.7`版本之后（不含`5.7`），`MySQL`加入了全新的密码安全机制。设置新密码太简单会报错。

![image-20220602203534353](第01章_Linux下MySQL的安装与使用.assets\image-20220602203534353.png)

- 改成更复杂的密码规则之后，设置成功，可以正常使用数据库了

![image-20220602203835764](第01章_Linux下MySQL的安装与使用.assets\image-20220602203835764.png)

### 3.3 设置远程登录

#### 1.当前问题

在用`SQLyog`或`Navicat`中配置远程连接`MySQL`数据库时遇到如下报错信息，这是由于`MySQL`配置了不支持远程连接引起的。

![image-20220602205006073](第01章_Linux下MySQL的安装与使用.assets\image-20220602205006073.png)

#### 2.确认网络

1. 在远程机器上使用`ping ip`地址==保证网络畅通==
2. 在远程机器上使用`telnet`命令==保证端口号开放==访问

```shell
telnet ip地址 端口号
```

拓展：`telnet`命令开启：

![image-20220602205514177](第01章_Linux下MySQL的安装与使用.assets\image-20220602205514177.png)

#### 3.关闭防火墙或开放端口

**方式一：关闭防火墙**

- `CentOS6`

```shell
service iptables stop
```

- `CentOS7`

```shell
systemctl start firewalld.service

systemctl status firewalld.serivce

systemctl stop firewalld.service

# 设置开机启动防火墙
systemctl enable firewalld.service

# 设置开机禁用防火墙
systemctl disable firewalld.service
```

**方式二：开放端口**

- 查看开放的端口号

```shell
firewall-cmd --list-all
```

- 设置开放的端口号

```shell
firewall-cmd --add-service=http --permanent

firewall-cmd --add-port=3306/tcp --permanent
```

- 重启防火墙

```shell
firewall-cmd --reload
```

#### 4.Linux下修改配置

在`Linux`系统`MySQL`下测试：

```mysql
use mysql;

SELECT Host, User FROM user;
```

![image-20220602210604888](第01章_Linux下MySQL的安装与使用.assets\image-20220602210604888.png)

可以看到`root`用户的当前主机配置信息为`localhost`。

- **修改`Host`为通配符`%`**

`Host`列指定了允许用户登录所使用的`IP`，比如`user=root Host=192.168.1.1`。这里的意思就是说`root`用户只能通过`192.168.1.1`的客户端去访问。`user=root Host=localhost`，表示只能通过本机客户端去访问。而`%`是个==通配符==，如果`Host=192.168.1.%`，那么就表示只要是`IP`地址前缀为"192.168.1."的客户端都可以连接。如果`Host=%`，表示所有`IP`都有连接权限。

注意：在生产环境下不能为了省事将`host`设置为`%`，这样做会存在安全问题，具体的设置可以根据生产环境的`IP`进行设置。

```mysql
update mysql.user set host = '%' where user = 'root';
```

`Host`设置了`%`后便可以允许远程访问。

![image-20220602211625418](第01章_Linux下MySQL的安装与使用.assets\image-20220602211625418.png)

`Host`修改完成后记得执行`flush privileges`使配置立即生效。

```mysql
flush privileges;
```

#### 5.测试

- 如果是`MySQL5.7`版本，接下来就可以使用`SQLyog`或`Navicat`成功连接至`MySQL`了。
- 如果是`MySQL8.0`版本，连接时还会出现如下问题：

![image-20220602211945435](第01章_Linux下MySQL的安装与使用.assets\image-20220602211945435.png)

配置新链接报错：错误号码`2058`，分析是`MySQL`密码加密方法变了。

**解决方法**：`Linux`下`mysql -uroot -p`登录你的`MySQL`数据库，然后执行这条`SQL`：

```mysql
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '密码';
```

然后再重新配置`SQLyog`的连接，则可连接成功了。

> 本人 Navicat15.0.12 - Premium 未出现以上问题

## 4.MySQL 8.0的密码强度评估（了解）

---

### 4.1 MySQL不同版本设置密码（可能出现）

- `MySQL5.7`中：成功

```mysql
mysql> ALTER USER 'root' IDENTIFIED BY 'root';
Query OK, 0 rows affected (0.00 sec)
```

- `MySQL8.0`中：失败

```mysql
mysql> ALTER USER 'root' IDENTIFIED BY 'abcd1234'; # HelloWorld_123
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

### 4.2 MySQL 8.0之前的安全策略

在`MySQL 8.0`之前，`MySQL`使用的是`validate_password`插件检测，验证帐号密码强度，保证帐号的安全性。

**安装/启用插件方式1：在参数文件`my.cnf`中添加参数**

```mysql
[mysqld]

plugin-load-add=validate_password.so

\#ON/OFF/FORCE/FORCE_PLUS_PERMANENT: 是否使用该插件（及强制/永久强制使用）

validate-password=FORCE_PLUS_PERMANENT
```

>说明1：
>
>`plugin library`中的`validate_password`文件名的后缀名根据平台不同有所差异。对于`Unix`和`Unix-like`系统而言，它的文件后缀名是`so`，对于`Windows`系统而言，它的文件后缀名是`.dll`。
>
>说明2：
>
>修改参数后必须重启`MySQL`服务才能生效。
>
>说明3：
>
>参数`FORCE_PLUS_PERMANENT`是为了防止插件在`MySQL`运行时的时候被卸载。当你卸载插件时就会报错。如下所示。

```mysql
mysql> SELECT PLUGIN_NAME, PLUGIN_LIBRARY, PLUGIN_STATUS, LOAD_OPTION
	-> FROM INFORMATION_SCHEMA.PLUGINS
	-> WHERE PLUGIN_NAME = 'validate_password';
+-------------------+----------------------+---------------+----------------------+
| PLUGIN_NAME       | PLUGIN_LIBRARY       | PLUGIN_STATUS | LOAD_OPTION          |
+-------------------+----------------------+---------------+----------------------+
| validate_password | validate_password.so | ACTIVE        | FORCE_PLUS_PERMANENT |
+-------------------+----------------------+---------------+----------------------+
1 row in set (0.00 sec)

mysql> UNINSTALL PLUGIN validate_password;
ERROR 1702 (HY000): Plugin 'validate_password' is force_plus_permanent and can not be unloaded
```

**安装/启用插件方式2：运行时命令安装**（推荐）

```mysql
mysql> INSTALL PLUGIN validate_password SONAME 'validate_password.so';
Query OK, 0 rows affected, 1 warning (0.11 sec)
```

此方法也会注册到元数据，也就是`mysql.plugin`表中，所以不用担心`MySQL`重启后插件会失效。

### 4.3 MySQL 8.0的安全策略

**1.`validate_password`说明**

`MySQL 8.0`，引入了服务器组件（Components）这个特性，`validate_password`插件已用服务器组件重新实现。`8.0.25`版本的数据库中，默认自动安装`validate_password`组件。

==未安装插件前，执行如下两个指令==，执行效果：

```mysql
mysql> show variables like 'validate_password%';
Empty set (0.04 sec)

mysql> SELECT * FROM mysql.component;
ERROR 1146 (42S02): Table 'mysql.component' doesn't exist
```

==安装插件后，执行如下两个命令==，执行效果：

```mysql
mysql> SELECT * FROM mysql.component;
+--------------+--------------------+------------------------------------+
| component_id | component_group_id | component_urn                      |
+--------------+--------------------+------------------------------------+
|            1 |                  1 | file://component_validate_password |
+--------------+--------------------+------------------------------------+
1 row in set (0.00 sec)

mysql> show variables like 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | ON     |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.01 sec)
```

关于`validate_password`组件对应的系统变量说明：

| **选项**                             | **默认值** | **参数描述**                                                 |
| ------------------------------------ | ---------- | ------------------------------------------------------------ |
| validate_password_check_user_name    | ON         | 设置为ON时，表示能将密码设置成当前用户名。                   |
| validate_password_dictionary_file    |            | 用于检查密码的字典文件的路径名，默认为空                     |
| **validate_password_length**         | 8          | 密码的最小长度，也就是说密码长度必须大于等于8                |
| validate_password_mixed_case_count   | 1          | 如果密码策略是中等或更强的，`validate_password`要求密码具有的小写和大写字符的最小数量，对于给定的这个值，密码必须有那么多小写字符和那么多大写字符。 |
| validate_password_number_count       | 1          | 密码必须包含的数字个数                                       |
| **validate_password_policy**         | MEDIUM     | 密码强度检验等级，可以使用数值`0`、`1`、`2`或相应的符号值`LOW`、`MEDIUM`、`STRONG`来指定。`0/LOW`：只检查长度。`1/MEDIUM`：检查长度、数字、大小写、特殊字符。`2/STRONG`：检查长度、数字、大小写、特殊字符、字典文件。 |
| validate_password_special_char_count | 1          | 密码必须包含的特殊字符个数                                   |

> 提示：
>
> 组件和插件的默认值可能有所不同。例如：`MySQL 5.7.validate_password_check_user_name`的默认值为`OFF`。

**2.修改安全策略**

修改密码验证安全强度

```mysql
SET GLOBAL validate_password_policy=LOW;

SET GLOBAL validate_password_policy=MEDIUM;

SET GLOBAL validate_password_policy=STRONG;

SET GLOBAL validate_password_policy=0; # FOR LOW

SET GLOBAL validate_password_policy=1; # FOR MEDIUM

SET GLOBAL validate_password_policy=2; # FOR HIGN

# 注意，如果是插件的话，SQL 为 set global validate_password_policy=LOW
```

此外，还可以修改密码中字符的长度

```mysql
set global validate_password_length = 1;
```

**3.密码强度测试**

如果创建密码是遇到"Your password does not satisfy the current policy requirements"，可以通过函数组件去检测密码是否满足条件：0-100。当评估在100时就是说明使用上了最基本的规则：大写+小写+特殊字符+数字组成的8位以上密码

```mysql
mysql> SELECT VALIDATE_PASSWORD_STRENGTH('medium');
+--------------------------------------+
| VALIDATE_PASSWORD_STRENGTH('medium') |
+--------------------------------------+
|                                   25 |
+--------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT VALIDATE_PASSWORD_STRENGTH('K354*45jKd5');
+-------------------------------------------+
| VALIDATE_PASSWORD_STRENGTH('K354*45jKd5') |
+-------------------------------------------+
|                                       100 |
+-------------------------------------------+
1 row in set (0.00 sec)
```

注意：如果没有安装`validate_password`组件或插件的话，那么这个函数永远都返回`0`。关于密码复杂度对应的密码复杂度策略。如下表格所示：

| **Password Test**                       | **Return Value** |
| --------------------------------------- | ---------------- |
| Length<4                                | 0                |
| Length>=4 and <validate_password_length | 25               |
| Satisfies policy 1(LOW)                 | 50               |
| Satisfies policy 2(MEDIUM)              | 75               |
| Satisfies policy 3(STRONG)              | 100              |

### 4.4 卸载插件、组件（了解）

**卸载插件**

```mysql
mysql> UNINSTALL PLUGIN validate_password;
Query OK, 0 rows affected, 1 warning (0.01 sec)
```

**卸载组件**

```mysql
mysql> UNINSTALL COMPONENT 'file://component_validate_password';
Query OK, 0 rows affected (0.02 sec)
```

## 5.字符集的设置

---

### 5.1 修改`MySQL 5.7`字符集

#### 1.修改步骤

在`MySQL 8.0`版本之前，默认字符集为`latin1`，`utf8`字符集指向的是`utf8mb3`。网站开发人员在数据库设计时往往会将编码修改为`utf8`字符集。如果遗忘修改默认的编码，就会出现乱码的问题。从`MySQL 8.0`开始，数据库的默认编码将改为`utf8mb4`，从而避免上述乱码的问题。

**操作1：查看默认使用的字符集**

```mysql
show variables like 'character_%';
# 或者
show variables like '%char%';
```

- `MySQL 8.0`中执行：

![image-20220602223307259](第01章_Linux下MySQL的安装与使用.assets\image-20220602223307259.png)

- `MySQL 5.7`中执行：

`MySQL 5.7`默认的客户端和服务器都用了`latin1`，不支持中文，保存中文会报错。`MySQL 5.7`截图如下：

![image-20220602224322453](第01章_Linux下MySQL的安装与使用.assets\image-20220602224322453.png)

在`MySQL 5.7`中添加中文数据时，报错：

![image-20220602224358961](第01章_Linux下MySQL的安装与使用.assets\image-20220602224358961.png)

因为默认情况下，创建表使用的是`latin1`。如下：

![image-20220602224435797](第01章_Linux下MySQL的安装与使用.assets\image-20220602224435797.png)

**操作2：修改字符集**

```shell
vim /etc/my.cnf
```

在`MySQL 5.7`或之前的版本中，在文件最后加上中文字符集配置：

```mysql
character_set_server=utf8
```

**操作3：重新启动`MySQL`服务**

```shell
systemctl restart mysqld
```

> 但是原库、原表的设定不会发生变化，参数修改只对新建的数据库生效。

#### 2.已有库&表字符集的变更

`MySQL 5.7`版本中，以前创建的库，创建的表字符集还是`latin1`。

![image-20220602224820838](第01章_Linux下MySQL的安装与使用.assets\image-20220602224820838.png)

修改已创建数据库的字符集

```mysql
ALTER DATABASE dbtest1 character set 'utf8';
```

修改已创建数据表的字符集

```mysql
ALTER TABLE t_emp convert to character set 'utf8';
```

> 注意：但是原有的数据如果是用非'utf8'编码的话，数据本身变化不会发生改变。已有数据需要导出或删除，然后重新插入。

### 5.2 各级别的字符集

`MySQL`有`4`个级别的字符集和比较规则，分别是：

- 服务器级别
- 数据库级别
- 表级别
- 列级别

执行如下`SQL`语句：

```mysql
show variables like 'character%';
```

![image-20220602223307259](第01章_Linux下MySQL的安装与使用.assets\image-20220602223307259.png)

- `character_set_server`：服务器级别的字符集
- `character_set_database`：当前数据库的字符集
- `character_set_client`：服务器解码请求时使用的字符集
- `character_set_connection`：服务器处理请求时会把请求字符串从`character_set_client`转为`character_set_connection`
- `character_set_results`：服务器向客户端返回数据时使用的字符集

#### 1.服务器级别

- `character_set_server`：服务器级别的字符集

可以在启动服务器程序时通过启动选项或者服务器程序运行过程中使用`SET`语句修改这两个变量的值。比如我们可以在配置文件中这样写：

```properties
[server]
character_set_server=gbk # 默认字符集
collation_server=gbk_chinese_ci # 对应的默认的比较规则
```

当服务器启动时读取这个配置文件后，这两个系统变量的值便修改了。

#### 2.数据库级别

- `character_set_database`：当前数据库的字符集

在创建和修改数据库时可以指定该数据库的字符集和比较规则，具体语法如下：

```mysql
CREATE DATABASE 数据库名
	[[DEFAULT] CHARACTER SET 字符集名称]
	[[DEFAULT] COLLATE 比较规则名称];
	
ALTER DATABASE 数据库名
	[[DEFAULT] CHARACTER SET 字符集名称]
	[[DEFAULT] COLLATE 比较规则名称];
```

其中的`DEFAULT`可以省略，并不影响语句的语义。比如：

```mysql
mysql> CREATE DATABASE charset_demo_db
	-> CHARACTER SET gb2312
	-> COLLATE gb2312_chinese_ci;
Query OK, 1 row affected (0.01 sec)
```

数据库的创建语句中也可以不指定字符集和比较规则，比如这样：

```mysql
CREATE DATABASE 数据库名;
```

**这样的话将使用服务器级别的字符集和比较规则作为数据库的字符集和比较规则**。

#### 3.表级别

也可以在创建和修改表的时候指定表的字符集和比较规则，语法如下：

```mysql
CREATE TABLE 表名(列信息)
	[[DEFAULT] CHARACTER SET 字符集名称]
	[COLLATE 比较规则的名称];
	
ALTER TABLE 表名
	[[DEFAULT] CHARACTER SET 字符集名称]
	[COLLATE 比较规则的名称];
```

比方在刚刚创建的`charset_demo_db`数据库中创建一个名为`t`的表，并指定这个表的字符集和比较规则：

```mysql
mysql> CREATE TABLE t(
	-> 	col VARCHAR(10)
	-> ) CHARACTER SET utf8 COLLATE utf8_general_ci;
Query OK, 0 rows affected (0.03 sec)
```

**如果创建和修改表的语句中没有指明字符集和比较规则，将使用该表所在数据库的字符集和比较规则作为该表的字符集和比较规则**。

#### 4.列级别

对于存储字符串的列，同一个表中的不同的列也可以有不同的字符集和比较规则。在创建和修改列定义的时候可以指定该列的字符集和比较规则，语法如下：

```mysql
CREATE TABLE 表名(
	列名 字符串类型 [CHARACTER SET 字符集名称] [COLLATE 比较规则名称],
    其他列 ...
);

ALTER TABLE 表名 MODIFY 列名 字符串类型 [CHARACTER SET 字符集名称] [COLLATE 比较规则名称];
```

比如修改一下表`t`中列`col`的字符集和比较规则可以这么写：

```mysql
mysql> ALTER TABLE t MODIFY col VARCHAR(10) CHARACTER SET gbk COLLATE gbk_chinese_ci;
Query OK, 0 rows affected (0.04 sec)
Records: 0 Duplicates: 0 Warnings: 0
```

**对于某个列来说，如果在创建和修改的语句中没有指明字符集和比较规则，将使用该列所在表的字符集和比较规则作为该列的字符集和比较规则**。

> 提示：
>
> 在转换列的字符集时需要注意，如果转换前，列中存储的数据不能用转换后的字符集进行表示会发生错误。比如说原先列使用的字符集是`utf8`，列中存储了一些汉字，现在把列的字符集转换为`ascii`的话就会出错，因为`ascii`字符集并不能表示汉字字符。

### 5.3 字符集与比较规则（了解）

#### 1.utf8与utf8mb4

`utf8`字符集表示一个字符需要使用`1~4`个字节，但是我们常用的一些字符使用`1~3`个字节就可以表示了。而字符集表示一个字符所用的最大字节长度，在某些方面会影响系统的存储和性能，所以设计`MySQL`的设计者偷偷的定义了两个概念：

- `utf8mb3`：阉割过的`utf8`字符集，只使用`1~3`个字节表示字符。
- `utf8mb4`：正宗的`utf8`字符集，使用`1~4`个字节表示字符。

在`MySQL`中`utf8`是`utf8mb3`的别名，所以之后在`MySQL`中提到`utf8`就意味着使用`1~3`个字节来表示一个字符。如果大家有使用`4`字节编码一个字符的情况，比如存储一些`emoji`表情，那使用`utf8mb4`。

此外，通过如下命令可以查看`MySQL`支持的字符集：

```mysql
SHOW CHARSET;
# 或者
SHOW CHARACTER SET;
```

![image-20220603081717676](第01章_Linux下MySQL的安装与使用.assets\image-20220603081717676.png)

#### 2.比较规则

上图中，`MySQL`版本一共支持`41`种字符集，其中的`Default collation`列表示这种字符集中一种默认的比较规则，里面包含着该比较规则主要作用于哪种语言。比如`utf8_polish_ci`表示以波兰语的规则比较，`utf8_spanish_ci`是以西班牙语的规则比较，`utf8_general_ci`是一种通用的比较规则。

| **后缀** | **英文释义**       | **描述**         |
| -------- | ------------------ | ---------------- |
| _ai      | accent insensitive | 不区分重音       |
| _as      | accent sensitive   | 区分重音         |
| _ci      | case insensitive   | 不区分大小写     |
| _cs      | case sensitive     | 区分大小写       |
| _bin     | binary             | 以二进制方式比较 |

最后一列`Maxlen`，它代表这种字符集表示一个字符最多需要几个字节。

这里把常见的字符集和对应的`Maxlen`显示如下：

| **字符集名称** | **Maxlen** |
| -------------- | ---------- |
| ascii          | 1          |
| latin1         | 1          |
| gb2312         | 2          |
| gbk            | 2          |
| utf8           | 3          |
| utf8mb4        | 4          |

**常用操作1**：

```mysql
# 查看 GBK 字符集的比较规则
SHOW COLLATION LIKE 'gbk%';

# 查看 UTF8 字符集的比较规则
SHOW COLLATION LIKE 'utf8%';
```

**常用操作2**：

```mysql
# 查看服务器的字符集和比较规则
SHOW VARIABLES LIKE '%_server';

# 查看数据库的字符集和比较规则
SHOW VARIABLES LIKE '%_database';

# 查看具体数据库的字符集
SHOW CREATE DATABASE dbtest1;

# 修改具体数据库的字符集
ALTER DATABASE dbtest1 DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
```

说明1：

> `utf8_unicode_ci`和`utf8_general_ci`对中、英文来说没有实质的差别。
>
> `utf8_general_ci`校对速度快，但准确度稍差。
>
> `utf8_unicode_ci`准确度高，但校对速度稍慢。
>
> 一般情况，用`utf8_general_ci`就够了，但如果应用有德语、法语或俄语，一定使用`utf8_unicode_ci`。

说明2：

> 修改了数据库的默认字符集和比较规则后，原来已经创建的数据表的字符集和比较规则并不会改变，如果需要，那么需单独修改。

**常用操作3**：

```mysql
# 查看表的字符集
SHOW CREATE TABLE employees;

# 查看表的比较规则
SHOW TABLE STATUS FROM atguigudb LIKE 'employees';

# 修改表的字符集和比较规则
ALTER TABLE emp1 DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
```

### 5.4 请求到响应过程中字符集的变化

我们知道从客户端发往服务器的请求本质上就是一个==字符串==，服务器向客户端返回的结果本质上也是一个字符串，而字符串其实是使用某个字符集编码的==二进制数据==。这个字符串可不是使用一种字符集的编码方式一条道走到黑的，从发送请求到返回结果这个过程中伴随着==多次字符集的转换==，在这个过程中会用到`3`个系统变量，我们先把它们写出来看一下：

| **系统变量**             | **描述**                                                     |
| ------------------------ | ------------------------------------------------------------ |
| character_set_client     | 服务器解码请求时使用的字符集                                 |
| character_set_connection | 服务器处理请求时会把请求字符串从`character_set_client`转为`character_set_connection` |
| character_set_results    | 服务器向客户端返回数据时使用的字符集                         |

![image-20220603084325074](第01章_Linux下MySQL的安装与使用.assets\image-20220603084325074.png)

为了体现出字符集在请求处理过程中的变化，我们这里特意修改一个系统变量的值：

```mysql
mysql> set character_set_connection = gbk;
Query OK, 0 rows affected (0.00 sec)
```

现在假设客户端发送的请求是下边这个字符串：

```mysql
SELECT * FROM t WHERE s = '我';
```

为了方便大家理解这个过程，只分析字符==我==在这个过程中字符集的转换。

现在看一下在请求从发送到结果返回过程中字符集的变化：

1. 客户端发送请求所使用的字符集

- 一般情况下客户端所使用的字符集和当前操作系统一致，不同操作系统使用的字符集可能不一样，如下：
  - 类`Unix`系统使用的是`utf8`
  - `Windows`使用的是`gbk`

​		当客户端使用的是`utf8`字符集，字符==我==在发送给服务器的请求中的字节形式就是：`0xE68891`

> 提示：
>
> 如果使用的是可视化工具，比如`navicat`之类的，这些工具可能会使用自定义的字符集来编码发送到服务器的字符串，而不采用操作系统默认的字符集（所以在学习时还是尽量用命令行窗口）。

2. 服务器接收到客户端发送来的请求其实是一串二进制的字节，它会认为这串字节采用的字符集是`character_set_client`，然后把这串字节转换为`character_set_connection`字符集编码的字符。由于本计算机上`character_set_client`的值是`utf8mb3`，首先会按照`utf8mb3`字符集对字符串`0xE68891`进行解码，得到的字符串就是==我==，然后按照`character_set_connection`代表的字符集，也就是`gbk`进行编码，得到的结果就是字节串`0xCED2`。
3. 因为表`t`的列`col`采用的是`gbk`字符集，与`character_set_connection`一致，所以直接到列中找字节值为`0xCED2`的记录，最后找到了一条记录。

> 提示：
>
> 如果某个列使用的字符集和`character_set_connection`代表的字符集不一致的话，还需要进行一次字符集转换。

4. 上一步骤找到的记录中`col`列其实是一个字符串`0xCED2`，`col`列是采用`gbk`进行编码的，所以首先会将这个字节串使用`gbk`进行解码，得到字符串==我==，然后再把这个字符串使用`character_set_results`代表的字符集，也就是`utf8mb3`进行编码，得到了新的字符串：`0xE68891`，然后发送给客户端。
5. 由于客户端是用的字符集是`utf8`，所以可以顺利的将`0xE68891`解释成字符==我==。

总结图示如下：

![image-20220603090728379](第01章_Linux下MySQL的安装与使用.assets\image-20220603090728379.png)

从这个分析中可以得出这么几点需要注意的地方：

- 服务器认为客户端发送过来的请求是用`character_set_client`编码的。

假设你的客户端采用的字符集和`character_set_client`不一样的话，这就会出现识别不准确的情况。比如我的客户端使用的是`utf8`字符集，如果把系统变量`character_set_client`的值设置为`ascii`的话，服务器可能无法理解我们发送的请求，更别谈处理这个请求了。

- 服务器将把得到的结果集使用`character_set_results`编码后发送给客户端。

假设你的客户端采用的字符集和`character_set_results`不一样的话，这就可能会出现客户端无法解码结果集的情况，结果就是在你的屏幕上出现乱码。比如我的客户端使用的是`utf8`字符集，如果把系统变量`character_set_results`的值设置为`ascii`的话，可能会出现乱码。

- `character_set_connection`只是服务器在将请求的字节串从`character_set_clinet`转换为`character_set_connection`时使用，一定要注意，该字符集包含的字符范围一定涵盖请求中的字符，要不然会导致有的字符无法使用`character_set_connection`代表的字符集进行编码。

**经验**

开发中通常把`character_set_client`、`character_set_connection`、`character_set_results`这三个系统变量设置成和客户端使用的字符集一致的情况，这样减少了很多无谓的字符集转换。为了方便我们设置，`MySQL`提供了一条非常简便的语句：

```mysql
SET NAMES 字符集名;
```

这一条语句产生的效果和我们执行这`3`条的效果是一样的：

```mysql
SET character_set_client = 字符集名;
SET character_set_connection = 字符集名;
SET character_set_results = 字符集名;
```

比方说本机使用的是`utf8`字符集，所以需要把这几个系统变量的值都设置为`utf8`：

```mysql
mysql> SET NAMES utf8;
```

另外，如果想在启动客户端时就把`character_set_client`、`character_set_connection`、`character_set_results`这三个系统变量的值设置成一样的，那我们可以在启动客户端时指定一个叫`default-character-set`的启动选项，比如在配置文件里这么写：

```mysql
[client]
default-character-set=utf8
```

它起得效果和执行一遍`SET NAMES utf8`是一样的，都会把那三个系统变量值设置成`utf8`。

## 6.SQL大小写规范

---

### 6.1 Windows和Linux平台区别

在`SQL`中，关键字和函数名是不用区分字母大小写的，比如`SELECT`、`WHERE`、`ORDER`、`GROUP BY`等关键字，以及`ABS`、`MOD`、`ROUND`、`MAX`等函数名。

不过在`SQL`中，你还是要确定大小写的规范，因为在`Linux`和`Windows`环境下，你可能会遇到不同的大小写问题。`windows系统默认大小写不敏感`，但是`linux系统是大小写敏感的`。

通过如下命令查看：

```mysql
SHOW VARIABLES LIKE '%lower_case_table_names%';
```

- `Windows`系统下：

![image-20220603180654007](第01章_Linux下MySQL的安装与使用.assets\image-20220603180654007.png)

- `Linux`系统下：

![image-20220603183357452](第01章_Linux下MySQL的安装与使用.assets\image-20220603183357452.png)

- `lower_case_table_names`参数值的设置：
  - ==默认为0，大小写敏感==。
  - 设置`1`，大小写不敏感。创建的表，数据库都是以小写形式存放在磁盘上，对于`sql`语句都是转换为小写对表和数据库进行查找。
  - 设置`2`，创建的表和数据库依据语句上格式存放，凡是查找都是转换为小写进行。
- 两个平台上`SQL`大小写的区别具体来说：

> `MySQL`在`Linux`下数据库名、表名、列名、别名大小写规则是这样的：
>
> 1、数据库名、表名、表的别名、变量名是严格区分大小写的；
>
> 2、关键字、函数名称在`SQL`中不分区大小写；
>
> 3、列名（或字段名）与列的别名（或字段别名）在所有的情况下均是忽略大小写
>
> 
>
> **`MySQL`在`Windows`的环境下全部不区分大小写**

### 6.2 Linux下大小写规则设置

当想设置为大小写不敏感时，要在`my.cnf`这个配置文件[mysqld]中加入`lower_case_table_names=1`，然后重启服务器。

- 但是要在重启数据库实例之前就需要将原来的数据库和表转换成小写，否则将找不到数据库名。
- 此参数适用于`MySQL5.7`。在`MySQL 8.0`下禁止在重新启动`MySQL`服务时将`lower_case_table_names`设置成不同于初始化`MySQL`服务时设置的`lower_case_table_names`值。如果非要将`MySQL 8.0`设置为大小写不敏感，具体步骤为：

```
1.停止 MySQL 服务
2.删除数据目录，即删除 /var/lib/mysql 目录
3.在 MySQL 配置文件（/etc/my.cnf）中添加 lower_case_table_names=1
4.启动 MySQL 服务
```

> 注意：在进行数据库参数设置之前，需要掌握这个参数带来的影响，切不可盲目设置。

### 6.3 SQL编写建议

如果变量名命名规范没有统一，就可能产生错误。这里有一个有关命名规范的建议：

> 1.关键字和函数名称全部大写
>
> 2.数据库名、表名、表别名、字段名、字段别名等全部小写
>
> 3.`SQL`语句必须以分号结尾

数据库名、表名和字段名在`Linux MySQL`环境下是区分大小写的，因此建议统一这些字段的命名规则，比如全部采用小写的方式。

虽然关键字和函数名称在`SQL`中不区分大小写，也就是如果小写的话同样可以执行。但是同时将关键词和函数名称全部大写，以便于区分数据库名、表名、字段名。

## 7.sql_mode的合理设置

---

### 7.1 介绍

`sql_mode`会影响`MySQL`支持的`SQL`语法以及它执行的==数据验证检查==。通过设置`sql_mode`，可以完成不同严格程度的数据校验，有效地保障数据准确性。

`MySQL`服务器可以在不同的`SQL`模式下运行，并且可以针对不同的客户端以不同的方式应用这些模式，具体取决于`sql_mode`系统变量的值。

`MySQL 5.6`和`MySQL 5.7`默认的`sql_mode`模式参数是不一样的：

- `5.6`的`mode`默认值为空（即：`NO_ENGINE_SUBSTITUTION`），其实表示的是一个空值，相当于没有什么模式设置，可以理解为==宽松模式==。在这种设置下是可以允许一些非法操作的，比如允许一些非法数据的插入。
- `5.7`的`mode`是`STRICT_TRANS_TABLES`，也就是==严格模式==。用于进行数据的严格校验，错误数据不能插入，报`error`（错误），并且事务回滚。

### 7.2 宽松模式 vs 严格模式

**宽松模式**：

如果设置的是宽松模式，那么我们在插入数据时，即便是给了一个错误的数据，也可能会被接受，并且不报错。

==举例==：在创建一个表时，该表中有一个字段`name`，给`name`设置的字段类型时`char(10)`，如果在插入数据时，其中`name`这个字段对应的有一条数据的==长度超过了10==，例如'1234567890abc'，超过了设定的字段长度`10`，那么不会报错，并且取前`10`个字符存上，也就是说这个数据被存为'1234567890'，而'abc’就没有了。但是，给的这条数据是错误的，因为超过了字段长度，但是并没有报错，并且`MySQL`自行处理并接受了，这就是宽松模式的效果。

==应用场景==：通过设置`sql_mode`为宽松模式，来保证大多数`sql`符合标准的`sql`语法，这样应用在不同数据表之间进行==迁移==时，则不需要对业务`sql`进行较大的修改。

**严格模式**：

出现上面宽松模式的错误，应该报错才对。所以`MySQL 5.7`版本就将`sql_mode`默认值改为了严格模式。所以在==生产等环境==中，我们必须采用的是严格模式，进而==开发==。==测试环境==的数据库也必须要设置，这样在开发测试阶段就可以发现问题。并且即便是用的`MySQL 5.6`，也应该自行将其改为严格模式。

==开发经验==：`MySQL`等数据库总想把关于数据的所有操作都自己包揽下来，包括数据的校验，其实开发中，应该在自己==开发的项目程序级别将这些校验给做了==，虽然写项目时麻烦了一些步骤，但是这样做之后，我们在进行数据库迁移或在项目的迁移时，就会方便很多。

改为严格模式后可能会存在的问题：

若设置模式中包含了`NO_ZERO_DATE`，那么`MySQL`数据库不允许插入零日期，插入零日期会抛出错误而不是警告。例如，表中包含字段`TIMESTAMP`列（如果未声明为`NULL`或显示`DEFAULT`子句）将自动分配`DEFAULT '0000-00-00 00:00:00'`（零时间戳），这显然是不满足`sql_mode`中的`NO_ZERO_DATE`而报错。

### 7.3 宽松模式再举例

**宽松模式举例1**：

```mysql
SELECT * FROM emp GROUP BY ename LIMIT 10;

SET sql_mode = ONLY_FULL_GROUP_BY;

SELECT * FROM emp GROUP BY ename LIMIT 10;
```

![image-20220603193823259](第01章_Linux下MySQL的安装与使用.assets\image-20220603193823259.png)

**宽松模式举例2**：

```mysql
mysql> desc t1;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   | PRI | NULL    |       |
| name  | varchar(2)  | YES  |     | NULL    |       |
| age   | smallint(6) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
1 rows in set (0.01 sec)

mysql> set sql_mode = '';
Query OK, 0 rows affected (0.00 sec)

mysql> insert into t1 values(1,'aa','aaa');
Query OK, 1 rows affected, 1 warning (0.01 sec)

mysql> show warnings;
+---------+------+----------------------------------------------------------+
| Level   | Code | Message                                                  |
+---------+------+----------------------------------------------------------+
| Warning | 1366 | Incorrect integer value; 'aaa' for column 'age' at row 1 |
+---------+------+----------------------------------------------------------+
1 rows in set (0.00 sec)

mysql> select * from t1;
+----+------+-----+
| id | name | age |
+----+------+-----+
| 1  | aa   |   0 |
+----+------+-----+
1 rows in set (0.00 sec)
```

设置`sql_mode`模式为`STRICT_TRANS_TABLES`，然后插入数据：

```mysql
mysql> set sql_mode = 'STRICT_TRANS_TABLES';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> insert into t1 values(2,'bb','bbb');
ERROR 1366 (HY000): Incorrect integer value: 'bbb' for column 'age' at row 1
```

### 7.4 模式查看和设置

- **查看当前的`sql_mode`**

```mysql
select @@session.sql_mode

select @@global.sql_mode

# 或者

show variables like 'sql_mode';
```

![image-20220603195110294](第01章_Linux下MySQL的安装与使用.assets\image-20220603195110294.png)

- **临时设置方式：设置当前窗口中设置`sql_mode`**

```mysql
SET GLOBAL sql_mode = 'modes...'; # 全局

SET SESSION sql_mode = 'modes...'; # 当前会话
```

举例：

```mysql
# 改为严格模式。此方法只在当前会话中生效，关闭当前会话就不生效了。
set SESSION sql_mode = 'STRICT_TRANS_TABLES';

# 改为严格模式。此方法在当前服务中生效，重启 MySQL 服务后失效。
SET GLOBAL sql_mode = 'STRICT_TRANS_TABLES';
```

- **永久设置方式：在`/etc/my.cnf`中配置`sql_mode`**

在`my.cnf`文件（`windows`系统是`my.ini`文件），新增：

```properties
[mysqld]
sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

然后==重启 MySQL==

当然生产环境上是禁止重启`MySQL`服务的，所以采用==临时设置方式 + 永久设置方式==来解决线上的问题，那么即便是有一天真的重启了`MySQL`服务，也会永久生效了。

### 7.5 sql_mode常用值

下面列出`MySQL`中最重要的`3`种模式：

| sql_mode                   | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ONLY_FALL_GROUP_BY         | 对于`GROUP BY`聚合操作，如果在`SELECT`中的列，没有在`GROUP BY`中出现，那么这个`SQL`是不合法的，因为列不在`GROUP BY`从句中。 |
| NO_AUTO_VALUE_ON_ZERO      | 该值影响自增长列的插入。默认设置下，插入`0`或`NULL`代表生成下一个自增长值。如果用户希望插入的值为`0`，而该列又是自增长的，那么这个选项就有用了。 |
| `STRICT_TRANS_TABLES`      | 在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做限制。 |
| NO_ZERO_IN_DATE            | 在严格模式下，不允许日期和月份为零。                         |
| NO_ZERO_DATE               | 设置该值，`MySQL`数据库不允许插入零日期，插入零日期会抛出错误而不是警告。 |
| ERROR_FOR_DIVISION_BY_ZERO | 在`INSERT`或`UPDATE`过程中，如果数据被零除，则产生错误而非警告。如果未给出该模式，那么数据库被零除时`MySQL`返回`NULL` |
| NO_AUTO_CREATE_USER        | 禁止`GRANT`创建密码为空的用户                                |
| NO_ENGINE_SUBSTITUTION     | 如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常。 |
| PIPES_AS_CONCAT            | 将“\|\|”视为字符串的连续操作符而非或运算符，这和`Oracle`数据库是一样的，夜和字符串的拼接函数`Concat`相类似 |
| ANSI_QUOTES                | 启用`ANSI_QUOTES`后，不能用双引号来引用字符串，因为它被解释为识别符。 |

