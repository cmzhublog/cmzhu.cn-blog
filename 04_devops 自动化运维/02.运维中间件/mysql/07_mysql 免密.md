## mysql免密使用

  1. 在shell脚本中，不要输入数据库密码，即可获取数据库相关信息

### 实现方案

> 只列出一种自己认为最合适的方法使用MySQL自带工具mysql_config_editor
> 1. 使用MySQL自带工具mysql_config_editor
>     ```
>     ]# mysql_config_editor set --login-path=client --host=localhost --user=root --password
>     Enter password:
>     ]# mysql_config_editor print --all #密码加密，且~/.mylogin.conf也是二进制文件，不会被查看到密码；不过此时也要注意到的是root用户的安全性
>     [client]
>     user = root
>     password = *****
>     host = localhost
>     ]# mysql_config_editor reset #重置
>     ```