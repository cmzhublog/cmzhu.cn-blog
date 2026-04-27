## mysql 常用命令

1、 为字段创建索引

```sql
# 建表后修改索引
$ ALTER TABLE `table_name` 
		ADD KEY `index_name`(`column_name`);
```



2、批量创建数据库表对某些字段创建加密视图的脚本

```sql
SELECT CONCAT(
    'CREATE OR REPLACE SQL SECURITY DEFINER VIEW  `<数据库名>`.`<需要加视图的表名>_v` AS\n',
    'SELECT\n  ',
    GROUP_CONCAT(
        CASE 
            WHEN COLUMN_NAME = 'mobile' 
            THEN CONCAT(
                "`<数据库名>`.`hex`(`<数据库名>`.`aes_encrypt`(`mobile`, '<加密key>')) AS `mobile`\n"
            )
            WHEN COLUMN_NAME = 'config' 
            THEN CONCAT(
                "`<数据库名>`.`hex`(`<数据库名>`.`aes_encrypt`(`config`, '<加密key>')) AS `config`\n"
            )
            WHEN COLUMN_NAME = 'device_id' 
            THEN CONCAT(
                "`<数据库名>`.`hex`(`<数据库名>`.`aes_encrypt`(`device_id`, '<加密ke y>')) AS `device_id`\n"
            )
            WHEN COLUMN_NAME = 'contact_phone' 
            THEN CONCAT(
                "`<数据库名>`.`hex`(`<数据库名>`.`aes_encrypt`(`contact_phone`, '<加密key>')) AS `contact_phone`\n"
            )
            ELSE CONCAT('  ', '`',COLUMN_NAME,'`')
        END
        ORDER BY ORDINAL_POSITION
        SEPARATOR ',\n  '
    ),
    '\nFROM `<数据库名>`.`<需要加视图的表名>`;\n'
) AS create_view_sql
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = '<数据库名>'
  AND TABLE_NAME = '<需要加视图的表名>';


```

3、生成删除库中所有表的语句列表

```sql
SELECT CONCAT('DROP TABLE IF EXISTS ', TABLE_NAME, ';') AS drop_statement
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = '<数据库名>';
```

4、查询库中容量（表+索引）超过10G 的表

```sql
SELECT
    table_schema                             AS db_name,
    table_name,
    engine,
    table_rows,
    ROUND((data_length + index_length) / 1024 / 1024 / 1024, 2) AS total_gb,
    ROUND(data_length / 1024 / 1024 / 1024, 2)                 AS data_gb,
    ROUND(index_length / 1024 / 1024 / 1024, 2)                AS index_gb,
    ROUND(data_free / 1024 / 1024 / 1024, 2)                   AS free_gb
FROM information_schema.tables
WHERE
    table_schema NOT IN ('information_schema','mysql','performance_schema','sys')
    AND (data_length + index_length) >= 1024 * 1024 * 1024
ORDER BY (data_length + index_length) DESC;
```

5、 清除分区表中的某一个分区

```mysql
 ALTER TABLE 表名 DROP PARTITION 分区名;
```

