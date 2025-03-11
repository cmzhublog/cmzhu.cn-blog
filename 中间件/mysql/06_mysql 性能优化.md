## 检查空闲连接
空闲连接是指"sleep"很长时间的连接，消耗服务器资源，建议根据空闲连接数进行优化
1. 查看空闲连接
    ```shell
    ]# mysqladmi processlist -uroot -p |grep "Sleep"
    Enter password: 
    | 1  | adminzbx | localhost | zabbix | Sleep   | 2    |       |                  | 0.000    |
    | 4  | adminzbx | localhost | zabbix | Sleep   | 3    |       |                  | 0.000    |
    | 5  | adminzbx | localhost | zabbix | Sleep   | 5    |       |                  | 0.000    |
    | 6  | adminzbx | localhost | zabbix | Sleep   | 4    |       |                  | 0.000    |
    | 7  | adminzbx | localhost | zabbix | Sleep   | 1    |       |                  | 0.000    |
    | 8  | adminzbx | localhost | zabbix | Sleep   | 0    |       |                  | 0.000    |
    | 9  | adminzbx | localhost | zabbix | Sleep   | 2    |       |                  | 0.000    |
    | 21 | adminzbx | localhost | zabbix | Sleep   | 1    |       |                  | 0.000    |
    | 45 | adminzbx | localhost | zabbix | Sleep   | 55   |       |                  | 0.000    |
    | 49 | adminzbx | localhost | zabbix | Sleep   | 33   |       |                  | 0.000    |
    | 50 | adminzbx | localhost | zabbix | Sleep   | 28   |       |                  | 0.000    |
    ```
2. 查看数据库配置
   关键词：wait_timeout interactive_timeout，同时减小值为60，单位秒
    ```mysql
    MariaDB [(none)]> show variables like '%timeout%';
    +----------------------------+----------+
    | Variable_name              | Value    |
    +----------------------------+----------+
    | connect_timeout            | 10       |
    | deadlock_timeout_long      | 50000000 |
    | deadlock_timeout_short     | 10000    |
    | delayed_insert_timeout     | 300      |
    | innodb_lock_wait_timeout   | 50       |
    | innodb_rollback_on_timeout | OFF      |
    | interactive_timeout        | 28800    |
    | lock_wait_timeout          | 31536000 |
    | net_read_timeout           | 30       |
    | net_write_timeout          | 60       |
    | slave_net_timeout          | 3600     |
    | thread_pool_idle_timeout   | 60       |
    | wait_timeout               | 28800    |
    +----------------------------+----------+
    13 rows in set (0.00 sec)
    
    ]# vim /etc/my.cnf
    [mysqld]
    ...
    interactive_timeout = 60
    wait_timeout = 60
    ```
3. 重启MySQL服务检查是否生效

## 启用MySQL慢查询日志
记录慢查询可以帮助你定位数据库中的问题并帮助你调试
    
    ```shell
    ]# vim /etc/my.cnf
    ...
    slow-query-log = 1 //启用慢查询
    slow-query-log-file = /var/log/mariadb/mysql-slow.log //日志保存位置
    long-query-time = 1 //查询超过多少秒才记录，默认10秒，修改为1秒
    ```
## 设置 MySQL 的最大连接数
max_connections 指令告诉你当前你的服务器允许多少并发连接

    ```mysql
    MariaDB [(none)]> show variables like '%thread_cache_size%'; 
    +-------------------+-------+
    | Variable_name     | Value |
    +-------------------+-------+
    | thread_cache_size | 0     |
    +-------------------+-------+
    1 row in set (0.00 sec)
    MariaDB [(none)]> set global thread_cache_size = 16;
    Query OK, 0 rows affected (0.00 sec)
    
    MariaDB [(none)]> show variables like '%thread_cache_size%';
    +-------------------+-------+
    | Variable_name     | Value |
    +-------------------+-------+
    | thread_cache_size | 16    |
    +-------------------+-------+
    1 row in set (0.00 sec)
    ```

## 配置 MySQL 的线程缓存数量

- thread_cache_size 指令用来设置你服务器缓存的线程数量
- 当客户端断开连接时，如果当前线程数小于 thread_cache_size，它的线程将被放入缓存中
- 下一个请求通过使用缓存池中的线程来完成
- 要提高服务器的性能，可以设置 thread_cache_size 的值相对高一些
    ```mysql
    MariaDB [(none)]> show status like 'Threads_created';
    MariaDB [(none)]> show status like 'Connections';
    ```
    1. 计算线程池的命中率：100 - ((Threads_created / Connections) * 100)
    2. 如果你得到一个较低的数字，这意味着大多数 mysql 连接使用新的线程，而不是从缓存加载。在这种情况下，你需要增加 thread_cache_size
        ```mysql
        MariaDB [(none)]> set global thread_cache_size = 16;
        ```