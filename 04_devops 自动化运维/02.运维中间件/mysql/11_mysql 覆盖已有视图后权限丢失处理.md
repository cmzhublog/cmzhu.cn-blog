## mysql 修改视图后权限丢失处理

### 背景

1、 业务因为某些字段属于加密字段，不能暴露给某些人员使用，并且开发是使用明文存储

2、 使用视图对特殊字段进行加密，并为使用人员开通视图权限，不开通源表权限

3、 有表字段的DDL ，然后视图未更新后发现业务查询权限丢失了



### 处理过程

1、 针对特殊情况创建视图，这里为cmzhu库中的 源表`cmzhu`创建一个`cmzhu_v`视图，包含`id` 字段和secret字段，并对secret` 字段进行加密，实现如下：

```sql
# 新建视图
create  view `call`.`cmzhu_v` AS SELECT `id`, 
 `cmzhu`.`hex`(`cmzhu`.`aes_encrypt`(`secret`, '<加密密钥>')) AS `secret`
FROM `cmzhu`
```

2、如果已有视图情况下，需要为视图添加新字段`new_column`，请使用`SQL SECURITY DEFINER` 覆盖( 用于解决背景3 中问题)：

```sql
# 修改视图
CREATE OR REPLACE SQL SECURITY DEFINER VIEW `call`.`cmzhu_v` AS SELECT `id`, 
 `cmzhu`.`hex`(`cmzhu`.`aes_encrypt`(`secret`, '<加密密钥>')) AS `secret`,
 `new_column`
FROM `cmzhu`
```

3、 为指定用户`cmzhu_read` 授权该视图`SELECT `权限

```sql
GRANT SELECT `cmzhu`.`cmzhu_v` TO 'cmzhu_read'@'%'
```

