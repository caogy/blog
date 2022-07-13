
# MySQL 必知必会

## 第3章 使用 MySQL

### MySQL 常用命令

- 启动服务：sudo service mysql start
- 关闭服务：sudo service mysql stop
- 重启服务：sudo service mysql restart
- 连接：mysql -u root -P 3306 -h localhost -p
- 连接字符串：jdbc:msql://localhost:3306/databaseName?serverTimeZone=UTC
- 列出数据库：show databases;
- 选择数据库：use databaseName;
- 列出数据表：show tables;
- 显示数据表列的属性：show columns from tableName; 或者 describe tableName;
- 显示创建数据库的脚本：show create database databaseName;
- 显示创建数据表的脚本：show create table tableName;
- 修改列名称：alter tableName rename column oldName to newName;
- 增加一列：alter table tableName add column fieldName dataType;
- 检索数据：select * from tableName limit 4 offset 5;

## 第6章 过滤数据

MySQL 在执行匹配时默认不分区大小写。

``` sql
select id,name from user where name = 'jone';
--结果可以匹配：name = Jone
```

使用 BETWEEN 时，使用 AND 关键字分隔，匹配范围中的说有值，包括指定的开始值和结束值（闭区间）。

``` sql
select * from from user where id between 1 and 5;
--检索用户编号：[1,5]
```

## 第11章 使用数据处理函数

### 常用日期和时间处理函数

函数|说明
---|---
AddDate()|增加一个日期（天、周等）
AddTime()|增加一个时间（时、分等）
CurDate()|返回当前日期
CurTime()|返回当前时间
Date()|返回日期时间的日期部分
DateDiff()|计算两个日期差
Date_Add()|高度灵活的日期运算函数
Date_Format()|返回一个格式化的日期或时间串
Day()|返回一个日期的天数部分
DayOfWeek()|对于一个日期，返回对应的星期几
Hour()|返回一个时间的小时部分
Minute()|返回一个时间的分部分
Month()|返回一个日期的月部分
Now()|返回当前日期和时间
Second()|返回一个时间的秒部分
Time()|返回一个日期时间的时间部分
Year()|返回一个日期的年部分

## 第12章 汇总数据

函数|说明
---|---
Avg()| Avg() 函数忽略列值为 null 的值。
Count()|Count(*) 对表中行的数目进行计数，不管表列中包含的是空值(null)还是非空值；Count(columnName) 对特定列具有值的进行计数，忽略 null 值。
