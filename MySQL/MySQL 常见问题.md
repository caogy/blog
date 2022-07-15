
# 常见问题 FAQ

## mysqldump 

### 使用 PowerShell 导出中文乱码问题

改为使用 cmd 命令行导出。

### 异常 Unknown table 'column_statistics' in information_schema

这是因为 `column-statistics` 在 mysqldump 8 是默认启用的。使用高版本的 mysqldump 连接低版本的 mysql 时报异常。

解决方法1：

``` cmd
mysqldump --column-statistics=0 -h192.168.10.12 -uroot -P3306 -p databaseName > test.sql
```

解决方法2：

在 mysql 配置文件，如 `my.ini`或`my.cnf`增加以下配置。

``` cmd
[mysqldump]
column-statistics = 0
```