### 第01章 Linux下MySQL的安装与使用

#### **1.** **安装前说明**

##### **1.1** **查看是否安装过MySQL**

- 如果你是用rpm安装, 检查一下RPM PACKAGE：

```shell
rpm -qa | grep -i mysql # -i 忽略大小写
```

- 检查mysql service：

```shell
systemctl status mysqld.service
```

##### **1.2 MySQL的卸载**

**1.** **关闭** **mysql** **服务**

```shell
systemctl stop mysqld.service
```

**2.** **查看当前** **mysql** **安装状况**

```shell
rpm -qa | grep -i mysql
# 或
yum list installed | grep mysql
```

**3.** **卸载上述命令查询出的已安装程序**

```shell
yum remove mysql-xxx mysql-xxx mysql-xxx mysqk-xxxx
```

务必卸载干净，反复执行`rpm -qa | grep -i mysql`确认是否有卸载残留

**4.** **删除** **mysql** **相关文件**

- 查找相关文件

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

#### **2. MySQL的Linux版安装**

##### **2.1 CentOS7下检查MySQL依赖** 

**1.** **检查/tmp临时目录权限（必不可少）**

由于mysql安装过程中，会通过mysql用户在/tmp目录下新建tmp_db文件，所以请给/tmp较大的权限。执行 ：

```shell
chmod -R 777 /tmp
```

**2.** **安装前，检查依赖**

```shell
rpm -qa|grep libaio
rpm -qa|grep net-tools
```

##### **2.2 CentOS7下MySQL安装过程** 

**1.** **将安装程序拷贝到/opt目录下**

在mysql的安装文件目录下执行：（必须按照顺序执行）

```shell
rpm -ivh mysql-community-common-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-client-plugins-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-libs-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-client-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-server-8.0.25-1.el7.x86_64.rpm
```

- `rpm`是Redhat Package Manage缩写，通过RPM的管理，用户可以把源代码包装成以rpm为扩展名的文件形式，易于安装。
- `-i`, --install 安装软件包
- `-v`, --verbose 提供更多的详细信息输出
- `-h`, --hash 软件包安装的时候列出哈希标记 (和 -v 一起使用效果更好)，展示进度条

> 若存在mariadb-libs问题，则执行**yum remove mysql-libs**即可

##### **2.3** **查看MySQL版本**

```shell
mysql --version 
#或
mysqladmin --version
```

##### **2.4** **服务的初始化**

为了保证数据库目录与文件的所有者为 mysql 登录用户，如果你是以 root 身份运行 mysql 服务，需要执行下面的命令初始化：

```shell
mysqld --initialize --user=mysql
```

说明： --initialize 选项默认以“安全”模式来初始化，则会为 root 用户生成一个密码并将`该密码标记为过期`，登录后你需要设置一个新的密码。生成的`临时密码`会往日志中记录一份。

查看密码：

```shell
cat /var/log/mysqld.log
```

root@localhost: 后面就是初始化的密码

##### **2.5** **启动MySQL，查看状态** 

```shell
#加不加.service后缀都可以 
启动：systemctl start mysqld.service 
关闭：systemctl stop mysqld.service 
重启：systemctl restart mysqld.service 
查看状态：systemctl status mysqld.service
```

##### **2.6** **查看MySQL服务是否自启动**

```shell
systemctl list-unit-files|grep mysqld.service
```

- 如不是enabled可以运行如下命令设置自启动

```shell
systemctl enable mysqld.service
```

- 如果希望不进行自启动，运行如下命令设置

```shell
systemctl disable mysqld.service
```

#### **3. MySQL登录**

##### **3.1** **首次登录**

通过`mysql -hlocalhost -P3306 -uroot -p`进行登录，在Enter password：录入初始化密码

##### **3.2** **修改密码**

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```

##### **3.3** **设置远程登录**

**1.** **确认网络** 

1.在远程机器上使用ping ip地址`保证网络畅通`

2.在远程机器上使用telnet命令`保证端口号开放`访问

**2.** **关闭防火墙或开放端口**

**方式一：关闭防火墙**

- CentOS6 ：

```shell
service iptables stop
```

- CentOS7：

```shell
#开启防火墙
systemctl start firewalld.service
#查看防火墙状态
systemctl status firewalld.service
#关闭防火墙
systemctl stop firewalld.service
#设置开机启用防火墙 
systemctl enable firewalld.service 
#设置开机禁用防火墙 
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

#### **4. Linux下修改配置**

- 修改允许远程登陆

```mysql
use mysql;
select Host,User from user;
update user set host = '%' where user ='root';
flush privileges;
```

> `%`是个 通配符 ，如果Host=192.168.1.%，那么就表示只要是IP地址前缀为“192.168.1.”的客户端都可以连接。如果`Host=%`，表示所有IP都有连接权限。
>
> 注意：在生产环境下不能为了省事将host设置为%，这样做会存在安全问题，具体的设置可以根据生产环境的IP进行设置。

配置新连接报错：错误号码 2058，分析是 mysql 密码加密方法变了。

**解决方法一：**升级远程连接工具版本

**解决方法二：**

```mysql
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'abc123';
```

#### **5.** **字符集的相关操作**

##### **5.1** **各级别的字符集**

MySQL有4个级别的字符集和比较规则，分别是：

+ 服务器级别

  + ```sql
    我们可以在启动服务器程序时通过启动选项或者在服务器程序运行过程中使用$ET语句修改这两个变量的值。比如我们可以在配置文件中这样写：
    [server]
    character_set_server=gbk#默认字符集
    collation_server=gbk_chinese_ci#对应的默认的比较规则
    ```

+ 数据库级别

  + ```sql
    # 创建时指明
    mysq1>CREATE DATABASE charset_demo_db
    CHARACTER SET gb2312 #字符集
    COLLATE gb2312_chinese_ci; #比较规则
    # 创建后修改
    mysq1>alter database dbtest1 character set 'utf8'; 
    ```

+ 表级别

  + ```sql
    # 创建时声明
    mysql>CREATE TABLE t(
    					col VARCHAR(10)
    					)
    CHARACTER SET utf8 
    COLLATE utf8_general_ci;
    
    # 创建后修改
    alter table t_emp convert to character set utf8';
    ```

+ 列级别

  + ```sql
    # 创建时指明
    CREATE TABLE t(
    			vol varchar(10) character set 'utf8'
    				)
    # 创建后修改
    ALTER TABLE t 
    MODIFY col VARCHAR(10) 
    CHARACTER SET gbk COLLATE gbk_chinese_ci;
    ```

```mysql
show variables like 'character%';
```

- character_set_server：服务器级别的字符集
- character_set_database：当前数据库的字符集
- character_set_client：服务器解码请求时使用的字符集
- character_set_connection：服务器处理请求时会把请求字符串从character_set_client转为character_set_connection 
- character_set_results：服务器向客户端返回数据时使用的字符集

##### 5.2 修改字符集

`vim /etc/my.cnf`	  

在mysql5.7或者之前的版本中，文件最后加上中文字符集

```
character_set_server=utf8
```

 重启mysqld服务，就更改好了

```
systemctl restart mysqld
```

已有库表字符集的变更

```sql
修改已创建数据库的字符集
alter database dbtest1 character set 'utf8';
修改已创建数据表的字符集
alter table t_emp convert to character set utf8';
```

**小结**

- 如果`创建或修改列`时没有显式的指定字符集和比较规则，则该列`默认用表的`字符集和比较规则
- 如果`创建表时`没有显式的指定字符集和比较规则，则该表`默认用数据库的`字符集和比较规则
- 如果`创建数据库时`没有显式的指定字符集和比较规则，则该数据库`默认用服务器的`字符集和比较规则

##### **5.2** **请求到响应过程中字符集的变化**

```mermaid
graph TB
A(客户端) --> |"使用操作系统的字符集编码请求字符串"| B(从character_set_client转换为character_set_connection)
B --> C(从character_set_connection转换为具体的列使用的字符集)
C --> D(将查询结果从具体的列上使用的字符集转换为character_set_results)
D --> |"使用操作系统的字符集解码响应的字符串"| A

```

### 

##### 5.3字符集与比较规则（了解）

###### 1.utf8与utf8mb4

utf8字符集表示一个字符需要使用1~4个字节，但是我们常用的一些字符使用1~3个字节就可以表示了。而字符集表示一个字符所用的最大字节长度，在某些方面会影响系统的存储和性能，所以设计MySQL的设计者偷偷的
定义了两个概念：

+ utf8mb3:阉割过的utf8字符集，只使用1~3个字节表示字符

+ utf8mb4:正宗的utf8字符集，使用1~4个字节表示字符。在MySQL中utf8是utf8mb3的别名，所以之后在MySQL中提到utf8就意味着使用1~3个字节来表示一个字符。如果大家有使用4字节编码一个字符的情况，比如存储一些emoji表情，那请使用utf8mb4。

  ```sql
  #查看MySQL支持的字符集：
  SHOW CHARSET;
  #或者
  SHOW CHARACTER SET;
  ```

###### 2. 比较规则

常用操作1：
```sql
#查看GBK字符集的比较规则
SHOW COLLATION LIKE 'gbk%';
#查看UTF-8字符集的比较规则
SHOW COLLATION LIKE 'utf8%';
```

常用操作2：

```sql
#查看服务器的字符集和比较规则
SHOW VARIABLES LIKE '%_server'
#查看数据库的字符集和比较规则
SHOW VARIABLES LIKE '%_database';
#查看具体数据库的字符集
SHOW CREATE DATABASE dbtest1
#修改具体数据库的字符集
ALTER DATABASE dbtest1 DEFAULT CHARACTER SET 'utf8' COLLATE 'utf8-general_ci';
```

说明1：**中英**都可以，但是**德法俄**用utf8_unicode_ci

```
utf8_unicode_ci和utf8_general_ci对中、英文来说没有实质的差别。
utf8_general_ci校对速度快，但准确度稍差。
utf8_unicode_ci准确度高，但校对速度稍慢。
一般情况，用utf8_general_ci就够了，但如果你的应用有德语、法语或者俄语，请一定使用utf8_unicode_ci。
```

说明2：

```
修改了数据库的默认字符集和比较规则后，原来已经创建的表格的字符集和比较规侧并不会改变，如果需要，那么需单独修改。
```

常用操作3

```sql
#查看表的字符集
show create table employees;
#查看表的比较规则
show table status from atguigudb like employees';
#修改表的字符集和比较规则
ALTER TABLE emp1 DEFAULT CHARACTER SET utf8 'COLLATE utf8_general_ci';
```

##### 5.4请求到响应过程中字符集的变化

​       我们知道从客户端发往服务器的请求本质上就是一个字符串，服务器向客户端返回的结果本质上也是一个字符串，而字符串其实是使用某种字符集编码的二进制数据。这个字符串可不是使用一种字符集的编码方式一条道走到黑的，从发送请求到返回结果这个过程中伴随着多次字符集的转换，在这个过程中会用到3个系统变量，我们先把它们写出来看一下

| 系统变量                 | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| character_set_client     | 服务器解码请求时使用的字符集                                 |
| character_set_connection | 服务器处理请求时会把请求字符串从character_set_client转为character_set_connection |
| character_set_results    | 服务器向客户端返回数据时使用的字符集                         |

现在看一下在请求从发送到结果返回过程中字符集的变化：

1. **客户端发送请求所使用的字符集**

   一般情况下客户端所使用的字符集和当前操作系统一致，不同操作系统使用的字符集可能不一样，如下：

+ 类Unix系统使用的是utf8
+ Windows使用的是gbk

当客户端使用的是utf8字符集，字符'我'在发送给服务器的请求中的字节形式就是：BxE68891

```
如果你使用的是可视化工具，比如navicat之类的，这些工具可能会使用自定义的字符集来编码发送到服务器的字符串，而不采用操作系统默认的字符集（所以在学习的时候还是尽量用命令行窗口）。
```

2. **服务器接收到客户端发送来的请求**其实是一串二进制的字节，它会认为这串字节采用的字符集是character_set_client,然后把这串字节转换为character.set_connection字符集编码的字符。
   由于我的计算机上character.set_client的值是utf8,首先会按照utf8字符集对字节串gxE68891进行解码，得到的字符串就是'我'，然后按照character_set_connection代表的字符集，也就是gbk进行编码，得到的结果就是字节串0xCED2。

3. 因为表t的列col采用的是gbk字符集，与character_set_connection一致，所以直接到列中找字节值为0xCED2的记录，最后找到了一条记录。

   > 提示
   > 如果某个列使用的字符集和character__set_connection代表的字符集不一致的话，还需要进行一次字符集转换。

4. 上一步骤找到的记录中的co1列其实是一个字节串xCED2,co1列是采用gbk进行编码的，所以首先会将这个字节串使用gbk进行解码，得到字符串'我'，然后再把这个字符串使用character_set_results代
   表的字符集，也就是utf8进行编码，得到了新的字节串：BxE68891,然后发送给客户端。

5. 由于客户端是用的字符集是utf8,所以可以顺利的将xE68891解释成字符我，从而显示到我们的显示器上，所以我们人类也读懂了返回的结果。

![image-20221124211824476](C:\Users\gaoji\AppData\Roaming\Typora\typora-user-images\image-20221124211824476.png)

从这个分析中我们可以得出这么几点需要注意的地方：

+ 服务器认为客户端发送过来的请求是用character._set_client编码的.
  假设你的客户端采用的字符集和character-set_client不一样的话，这就会出现识别不准确的情况。比如我的客户端使用的是utf8字符集，如果把系统变量character_set_client的值设置为ascii的话，服
  务器可能无法理解我们发送的请求，更别谈处理这个请求了。
+ 服务器将把得到的结果集使用character_set_results编码后发送给客户端。假设你的客户端采用的字符集和character_set_results不一样的话，这就可能会出现客户端无法解码结果集的情况，结果就是在你的屏幕上出现乱码。比如我的客户端使用的是utf8字符集，如果把系统变量
  character_set_results的值设置为ascii的话，可能会产生乱码。
+ character_set_connection只是服务器在将请求的字节串从character_set_client转换为character_set_connection时使用，一定要注意，该字符集包含的字符范围一定涵盖请求中的字符，要不
  然会导致有的字符无法使用character_set_connection代表的字符集进行编码。

###### 经验

​       开发中通常把character_set_client、character_set_connection、character_set_results这三个系统变量设置成和客户端使用的字符集一致的情况，这样减少了很多无谓的字符集转换。为了方便我们设置，MySQL提供了一条非常简便的语句：

```SQL
SET NAMES 字符集名;

#这一条语句产生的效果和我们执行这3条的效果是一样的：
SET character_set_client=字符集名；
SET character_set_connection=字符集名；
SET character_set_results=字符集名；

#比方说我的客户端使用的是utf8字符集，所以需要把这几个系统变量的值都设置为utf8:
SET NAMES utf8;
```


​       另外，如果你想在启动客户端的时候就把character_set_client、character_set_connection、character_set_results,这三个系统变量的值设置成一样的，那我们可以在启动客户端的时候指定一个叫default-character-set的启动选项，比如在配置文件里可以这么写：

```SQL
[client]
default-character-set=utf8
```

它起到的效果和执行一遍SET NAMES utf:8是一样一样的，都会将那三个系统变量的值设置成utf8。

#### 6.SQL大小写规范

##### 6.1 windows和linux平台区别 

在SQL中，**关键字和函数名**是不用区分字母大小写的，比如SELECT、WHERE、ORDER、GROUP BY等关键字，以及ABS、MOD、ROUND、MAX等函数名。
不过在SQL中，你还是要确定大小写的规范，因为在Linux和Windows环境下，你可能会遇到不同的大小写问题。windows系统默认大小写不敏感，但是linux系统是大小写敏感的。
通过如下命令查看：

```sql
SHOW VARIABLES LIKE '%lower_case_table_names%';
```

**lower_case_table_names**参数值的设置：

+ 默认为0，大小写敏感。
+ 设置1，大小写不敏感。创建的表，数据库都是以小写形式存放在磁盘上，对于sql语句都是转换为小写对表和数据库进行查找。
+ 设置2，创建的表和数据库依据语句上格式存放，凡是查找都是转换为小写进行。

> **win和linux平台SQL大小写的区别具体来说：**
> MySQL在Linux下数据库名、表名、列名、别名大小写规则是这样的：
> 1、数据库名、表名、表的别名、变量名是严格区分大小写的：
> 2、关键字、函数名称在SQL中不区分大小写；
> 3、列名（或字段名）与列的别名（或字段别名）在所有的情况下均是忽略大小写的；
>
> 
>
> MySQL在Windows的环境下全部不区分大小写

##### 6.2 linux下大小写规则设置

##### 6.3 SQL编写建议

如果你的变量名命名规范没有统一，就可能产生错误。这里有一个有关命名规范的建议：

1. **关键字和函数名**称全部大写
2. **数据库名、表名、表别名、字段名、字段别名**等全部小写：
3. SQL语句必须以分号结尾。

数据库名、表名和字段名在Linux MySQL环境下是区分大小写的，因此建议你统一这些字段的命名规则，比如全部采用小写的方式。
虽然关键字和函数名称在SQL中不区分大小写，也就是如果小写的话同样可以执行。但是同时将关键词和函数名称全部大写，以便于区分数据库名、表名、字段名。

#### 7.sql_mode的合理设置

##### 7.1宽松模式vs严格模式

1. 宽松模式

   如果设置的是宽松模式，那么我们在插入数据的时候，即便是给了一个错误的数据，也可能会被接受，并且不报错。
   举例：我在创建一个表时，该表中有一个字段为name,给name设置的字段类型时char(10),如果我在插入数据的时候，其中name这个字段对应的有一条数据的长度超过了10，例如'1234567890abc',超过了设定的字段长度10，那么不会报错，并且取前10个字符存上，也就是说你这个数据被存为了'1234567890'，而'abc'就没有了。但是，我们给的这条数据是错误的，因为超过了字段长度，但是并没有报错，并且mysql自行处理并接受了，这就
   是宽松模式的效果。
   **应用场景**：通过设置sql_mode为宽松模式，来保证大多数sql符合标准的sql语法，这样应用在**不同数据库之间进行迁移**时，则不需要对业务sq进行较大的修改。

2. 严格模式：
   出现上面宽松模式的错误，应该报错才对，所以MySQL5.7版本就将sql_mode默认值改为了严格模式。所以在生产等环境中，我们必须采用的是严格模式，进而开发、测试环境的数据库也必须要设置，这样在开发测试阶段就可以发现问题。并且我们即便是用的MySQL5.6,也应该自行将其改为严格模式。

   开发经验：MySQL等数据库总想把关于数据的所有操作都自己包揽下来，包括数据的校验，其实开发中，我们应该在自己开发的项目程序级别将这些校验给做了，虽然写项目的时候麻烦了一些步骤，但是这样做之后，我们在进行数据库迁移或者在项目的迁移时，就会方便很多。

3. 改为严格模式后可能会存在的问题：
   若设置模式中包含了NO_ZER0_DATE，那么MySQL数据库不允许插入零日期，插入零日期会抛出错误而不是警告。例如，表中含字段TIMESTAMP列（如果未声明为NULL或显示DEFAULT-子句）将自动分配DEFAULT'0000-00-00 00:00:00'(零时间戳)，这显然是不满足sql_mode中的NO_ZERO_DATE而报错。

##### 7.2 查看模式和设置

查看当前的sql_mode

```sql
select @@session.sql_mode

select @@global.sq1_mode

#或者
show variables like 'sq1_mode';
```

临时设置方式：设置当前窗口中设置sql_mode

```sql
SET GLOBAL sql_mode='modes..';#全局
SET SESSION sql_mode='modes.,.';#当前会话
```

举例：

```sql
#改为严格模式。此方法只在当前会话中生效，关闭当前会话就不生效了。
set SESSION sql_mode='STRICT_TRANS_TABLES'
#改为严格模式。此方法在当前服务中生效，重启MySQL服务后失效。
set GLOBAL sq1_mode='STRICT_TRANS_TABLES'
```

永久设置方式：在/etc/my.cnf中配置sql_mode

> 在my.cnf文件(windows系统是my.ini文件)，新增：
> [mysqld]
> sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION

然后重启MySQL。
当然生产环境上是禁止重启MySQL服务的，所以采用临时设置方式+永久设置方式来解决线上的问题，那么即便是有一天真的重启了MySQL服务，也会永久生效了。