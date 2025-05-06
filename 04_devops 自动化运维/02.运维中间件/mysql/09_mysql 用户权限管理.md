## mysql 用户权限管理

查询指定用户权限：

```sql
$ show grants for '用户名'@'访问地址' # 访问地址 可以%，也可以IP段
```

### 权限管理

#### 权限管理简介

##### 1、访问权限

主要是通过方式限制数据库的访问权限，先通过以下三个字段判断连接的资源是否存在和验证正确；

- 用户
- 密码
- 访问IP

##### 2、权限级别

1、全局性权限管理： 作用于整个mysql 实例

2、数据库级别的权限管理： 作用于某一个数据库上或者所有数据库上

3、数据库对象级别权限管理： 作用于指定的数据库对象（视图、表等）或者所有数据库对象上

权限验证顺序如下:

- 优先检查全局权限表user 表，如果user 表中的权限为`Y`  则此用户对于所有的数据库的权限都为`Y` ,将不会再继续下一步的检查，如果为N，则在DB表中检查此用户对应的具体数据库的权限是否为Y， 依次类推；

```mermaid
flowchart LR;
	start --> user --> db --> tables_priv --> columns_priv;
		style user fill:#AFEEEE,stroke:#333,stroke-width:1px;
		style db fill:#00FFFF,stroke:#333,stroke-width:1px;
		style tables_priv fill:#00CED1,stroke:#333,stroke-width:1px;
		style columns_priv fill:#008B8B,stroke:#333,stroke-width:1px;
		
	
```

##### 3、权限查询

1、查询存在哪些mysql 用户

```sql
> select * from mysql.user\G;
```

2、查看root 用户在权限表中有哪些权限；

```sql
> use mysql;
#所有权限都是Y ，就是什么权限都有
> select * from user where user='root' and host='localhost'\G; 
> select * from db where user='root' and host='localhost'\G;
> select * from columns_priv where user='root' and host='localhost'\G;
> select * from procs_priv where user='root' and host='localhost'\G;

```

##### 4、权限详解

1. **All/All Privileges 代表全局或者全数据库的所有权限**
2. Alter权限代表允许修改表结构的权限，但必须要求有`create`和`insert`权限配合。如果是`rename`表名，则要求有`alter`和`drop`原表， `create`和`insert`新表的权限
3. `Alter routine`权限代表允许修改或者删除存储过程、函数的权限
4. `Create`权限代表允许创建新的数据库和表的权限
5. `Create routine`权限代表允许创建存储过程、函数的权限
6. `Create tablespace`权限代表允许创建、修改、删除表空间和日志组的权限
7. `Create temporary tables`权限代表允许创建临时表的权限
8. `Create user`权限代表允许创建、修改、删除、重命名`user`的权限
9. `Create view`权限代表允许创建视图的权限
10. `Delete`权限代表允许删除行数据的权限
11. `Drop`权限代表允许删除数据库、表、视图的权限，包括`truncate table`命令
12. `Event`权限代表允许查询，创建，修改，删除`MySQL`事件
13. `Execute`权限代表允许执行存储过程和函数的权限
14. `File`权限代表允许在`MySQL`可以访问的目录进行读写磁盘文件操作，可使用的命令包括`load data infile,select … into outfile,load file()`函数
15. `Grant option`权限代表是否允许此用户授权或者收回给其他用户你给予的权限,重新付给管理员的时候需要加上这个权限
16. `Index`权限代表是否允许创建和删除索引
17. `Insert`权限代表是否允许在表里插入数据，同时在执行`analyze table,optimize table,repair table`语句的时候也需要`insert`权限
18. `Lock`权限代表允许对拥有`select`权限的表进行锁定，以防止其他链接对此表的读或写
19. `Process`权限代表允许查看`MySQL`中的进程信息，比如执行`show processlist, mysqladmin processlist, show engine`等命令
20. `Reference`权限是在`5.7.6`版本之后引入，代表是否允许创建外键
21. `Reload`权限代表允许执行`flush`命令，指明重新加载权限表到系统内存中，`refresh`命令代表关闭和重新开启日志文件并刷新所有的表
22. `Replication client`权限代表允许执行`show master status,show slave status,show binary logs`命令
23. `Replication slave`权限代表允许`slave`主机通过此用户连接`master`以便建立主从复制关系
24. `Select`权限代表允许从表中查看数据，某些不查询表数据的`select`执行则不需要此权限，如`Select 1+1， Select PI()+2；`而且`select`权限在执行`update/delete`语句中含有`where`条件的情况下也是需要的
25. `Show databases`权限代表通过执行`show databases`命令查看所有的数据库名
26. `Show view`权限代表通过执行`show create view`命令查看视图创建的语句
27. `Shutdown`权限代表允许关闭数据库实例，执行语句包括`mysqladmin shutdown`
28. `Super`权限代表允许执行一系列数据库管理命令，包括kill强制关闭某个连接命令， `change master to`创建复制关系命令，以及`create/alter/drop server`等命令
29. `Trigger`权限代表允许创建，删除，执行，显示触发器的权限
30. `Update`权限代表允许修改表中的数据的权限
31. `Usage`权限是创建一个用户之后的默认权限，其本身代表连接登录权限

###### 4.1 系统权限表

1. `User`表：存放用户账户信息以及全局级别（所有数据库）权限，决定了来自哪些主机的哪些用户可以访问数据库实例，如果`有全局权限则意味着对所有数据库都有此权限` 
2. Db表：存放`数据库级别`的权限，决定了来自哪些主机的哪些用户可以访问此数据库 
3. Tables_priv表：`存放表级别的权限`，决定了来自哪些主机的哪些用户可以访问数据库的这个表 
4. Columns_priv表：`存放列级别的权限`，决定了来自哪些主机的哪些用户可以访问数据库表的这个字段 
5. Procs_priv表：`存放存储过程和函数`级别的权限

##### 5、表结构详解

###### 5.1 user 表和db 表结构

| 表名           | `user`                   | `db`                    |
| -------------- | ------------------------ | ----------------------- |
| **范围列**     | `Host`                   | `Host`                  |
|                | `User`                   | `Db`                    |
|                |                          | `User`                  |
| **权限列**     | `Select_priv`            | `Select_priv`           |
|                | `Insert_priv`            | `Insert_priv`           |
|                | `Update_priv`            | `Update_priv`           |
|                | `Delete_priv`            | `Delete_priv`           |
|                | `Index_priv`             | `Index_priv`            |
|                | `Alter_priv`             | `Alter_priv`            |
|                | `Create_priv`            | `Create_priv`           |
|                | `Drop_priv`              | `Drop_priv`             |
|                | `Grant_priv`             | `Grant_priv`            |
|                | `Create_view_priv`       | `Create_view_priv`      |
|                | `Show_view_priv`         | `Show_view_priv`        |
|                | `Create_routine_priv`    | `Create_routine_priv`   |
|                | `Alter_routine_priv`     | `Alter_routine_priv`    |
|                | `Execute_priv`           | `Execute_priv`          |
|                | `Trigger_priv`           | `Trigger_priv`          |
|                | `Event_priv`             | `Event_priv`            |
|                | `Create_tmp_table_priv`  | `Create_tmp_table_priv` |
|                | `Lock_tables_priv`       | `Lock_tables_priv`      |
|                | `References_priv`        | `References_priv`       |
|                | `Reload_priv`            |                         |
|                | `Shutdown_priv`          |                         |
|                | `Process_priv`           |                         |
|                | `File_priv`              |                         |
|                | `Show_db_priv`           |                         |
|                | `Super_priv`             |                         |
|                | `Repl_slave_priv`        |                         |
|                | `Repl_client_priv`       |                         |
|                | `Create_user_priv`       |                         |
|                | `Create_tablespace_priv` |                         |
| **安全专栏**   | `ssl_type`               |                         |
|                | `ssl_cipher`             |                         |
|                | `x509_issuer`            |                         |
|                | `x509_subject`           |                         |
|                | `plugin`                 |                         |
|                | `authentication_string`  |                         |
|                | `password_expired`       |                         |
|                | `password_last_changed`  |                         |
|                | `password_lifetime`      |                         |
|                | `account_locked`         |                         |
| **资源控制列** | `max_questions`          |                         |
|                | `max_updates`            |                         |
|                | `max_connections`        |                         |
|                | `max_user_connections`   |                         |

###### 5.2 Tables_priv和columns_priv权限表结构

| 表名       | `tables_priv` | `columns_priv` |
| ---------- | ------------- | -------------- |
| **范围列** | `Host`        | `Host`         |
|            | `Db`          | `Db`           |
|            | `User`        | `User`         |
|            | `Table_name`  | `Table_name`   |
|            |               | `Column_name`  |
| **权限列** | `Table_priv`  | `Column_priv`  |
|            | `Column_priv` |                |
| **其他列** | `Timestamp`   | `Timestamp`    |
|            | `Grantor`     |                |

**Tables_priv和columns_priv权限值**

| Table Name     | Column Name   | Possible Set Elements                                        |
| -------------- | ------------- | ------------------------------------------------------------ |
| `tables_priv`  | `Table_priv`  | `'Select', 'Insert', 'Update', 'Delete', 'Create', 'Drop', 'Grant', 'References', 'Index', 'Alter', 'Create View', 'Show view', 'Trigger'` |
| `tables_priv`  | `Column_priv` | `'Select', 'Insert', 'Update', 'References'`                 |
| `columns_priv` | `Column_priv` | `'Select', 'Insert', 'Update', 'References'`                 |
| `procs_priv`   | `Proc_priv`   | `'Execute', 'Alter Routine', 'Grant'`                        |

###### 5.3 procs_priv权限表结构

| Table Name            | `procs_priv`   |
| --------------------- | -------------- |
| **Scope columns**     | `Host`         |
|                       | `Db`           |
|                       | `User`         |
|                       | `Routine_name` |
|                       | `Routine_type` |
| **Privilege columns** | `Proc_priv`    |
| **Other columns**     | `Timestamp`    |
|                       | `Grantor`      |

- Routine_type是枚举类型，代表是存储过程还是函数
- Timestamp和grantor两个字段暂时没用

**系统权限表字段长度限制表**

| Column Name            | Maximum Permitted Characters |
| ---------------------- | ---------------------------- |
| `Host`, `Proxied_host` | 60                           |
| `User`, `Proxied_user` | 32                           |
| `Password`             | 41                           |
| `Db`                   | 64                           |
| `Table_name`           | 64                           |
| `Column_name`          | 64                           |
| `Routine_name`         | 64                           |

**权限认证中的大小写敏感问题**

- 字段user,password,authencation_string,db,table_name大小写敏感
- 字段host,column_name,routine_name大小写不敏感

##### 6、权限查询方案

1、 查询存在哪些用户

```sql
> select user,host from mysql.user;
```

2、查询为用户授予了哪些权限

```sql
> show grants for root@'localhost';
```

3、 查询用户的其他非授权信息

```sql
> show create user root@'localhost';
```



### 权限修改

#### 1. 创建mysql 用户

1、 执行 `create user /grant` 命令来创建（推荐）

2、通过mysql 语句直接操作mysql 系统权限表

示例：

```sql
# 创建finley 这只是创建用户并没有权限
> CREATE USER 'finley'@'localhost' IDENTIFIED BY 'password';
## 把finley 变成管理员用户
> GRANT ALL PRIVILEGES ON *.* TO 'finley'@'localhost' WITH GRANT OPTION;

#创建用户并赋予RELOAD,PROCESS权限 ，在所有的库和表上
> GRANT RELOAD,PROCESS ON *.* TO 'admin'@'localhost' identified by '123456';
# 创建keme用户，在test库，temp表， 上的id列只有select 权限
> grant select(id) on test.temp to keme@'localhost' identified by '123456';

```





### 权限查询

