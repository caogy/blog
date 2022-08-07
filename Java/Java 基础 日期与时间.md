# 日期与时间

现在是 `2022-8-3 11:40:34`，我们说时间的时候，说的是本地时间，在国内就是北京时间，全球现在一共分为 24 个时区，伦敦所在的时区称为标准时区，其他时区按东/西偏移的小时来区分，北京的时区为东八区。

2022-8-3 这是日期。时间分为两种：`11:40:34` 与 `2022-8-3 11:40:34`。只有带日期格式的时间再加上时区才能确定某个时刻。不过时区一般省略，默认为东八区，北京时间。`2022-8-3 11:40:34` 这个时刻在计算机存储为一个整数，我们称为`Epoch Time`。

`Epoch Time`是计算从 1970年1月1日零点（格林威治时区/GMT+00:00）到现在所经历的秒数。

## 旧标准库API

旧标准库API定义在包`java.util`里边，主要包括`Date`、`Calendar`、`TimeZone`。

### Date

Date 类的基本用法。

``` java
@Test
public void test() {
    Date date = new Date();
    //年必须要加上 1900
    System.out.println(date.getYear() + 1900);
    //月必须加上1
    System.out.println(date.getMonth() + 1);
    //1-31 不用加1
    System.out.println(date.getDate());
    //转为时间字符串
    System.out.println(date);
    //转为本地时间字符串
    System.out.println(date.toLocaleString());
}

//输出
//2022
//8
//3
//Wed Aug 03 17:23:19 CST 2022
//2022-8-3 17:23:19
```

打印本地时区表示的日期和时间时，不同的计算机同一时刻可能显示的格式不同。如果我们需要控制日期和时间的格式，可以使用`SimpleDateFormat`对一个`Date`进行转换。

``` java
@Test
public void test1(){
    Date date = new Date();
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd h:mm:ss:SSS");
    System.out.println(sdf.format(date));
}

// 2022-08-04 5:18:46:667
```

预定义的格式化格式：
格式|说明
---|---
yyyy|四位数年份，例如:2022
yy|两位数年份，例如：22
MM|两位数月份，不满两位数，前边补0，例如8月显示 08
M|月份，不满两位数，不会补0，例如8月显示 8
dd|天，不满两位数，前补0
d|天，不满两位数，前不补0
HH| 24小时制小时数，不满两位前边补0
H| 24小时制小时数，不满两位前边不补0
hh|12小时制小时数，不满两位前边补0
h| 12小时制小时数，不满两位前边不补0
ss|秒数，补0
s|秒数，不补0
SSS|毫秒数

### Calendar

``` java
@Test
public void testCalendar(){
    //获取当前时间，本地时间
    Calendar calendar = Calendar.getInstance();
    int y = calendar.get(Calendar.YEAR);
    //月份必须加 1
    int m = calendar.get(Calendar.MONTH)+1;
    int d = calendar.get(Calendar.DAY_OF_MONTH);
    int hh = calendar.get(Calendar.HOUR_OF_DAY);
    int mm = calendar.get(Calendar.MINUTE);
    int ss = calendar.get(Calendar.SECOND);
    int ms = calendar.get(Calendar.MILLISECOND);
    System.out.println(String.format("%s-%s-%s %s:%s:%s:%s",y,m,d,hh,mm,ss,ms));
    System.out.println("hour:"+calendar.get(Calendar.HOUR));
    //本周的第几天，1~7分别表示周日，周一,...,周六
    System.out.println("day of week:"+calendar.get(Calendar.DAY_OF_WEEK));
    //本周是当月的第几周
    System.out.println("day of week in month:"+calendar.get(Calendar.DAY_OF_WEEK_IN_MONTH));
    System.out.println("day of year:"+calendar.get(Calendar.DAY_OF_YEAR));
}

/*
2022-8-5 11:0:33:339
hour:11
day of week:6
day of week in month:1
day of year:217
*/
```

需要注意的是`Calendar.getInstance()`获取的是本地当前时间，获取月份`calendar.get(Calendar.MONTH)`必须要加`1`。1~7分别表示周日,周一,...,周六。如果需要设置时间，需要先调用`calendar.clear()`。

``` java
@Test
public void testCalendar(){
    //获取当前时间，本地时间
    Calendar calendar = Calendar.getInstance();
    //清除当前时间
    calendar.clear();
    calendar.set(Calendar.YEAR,2022);
    calendar.set(Calendar.MONTH,7);
    calendar.set(Calendar.DAY_OF_MONTH,5);
    //Calendar to Date
    Date date = calendar.getTime();
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    System.out.println(sdf.format(date));
    System.out.println("月份："+date.getMonth());
}

/*
2022-08-05
月份：7
*/
```

### TimeZone

获取默认的时区和可用的时区编号。

``` java
@Test
public void testTimeZone() {
    TimeZone defaultZone = TimeZone.getDefault();
    String[] availableZones = TimeZone.getAvailableIDs();
}
```

## 新标准库API

Java 8 新增了 java.time 包，提供了日期与时间新的实现类：

![日期与时间](/image/20220805-01.png)

每个类对应的说明如下：
类名|说明
---|---
LocalDate|获取当前的日期，不包含具体时间，时区信息。
LocalTime|获取当前的时间信息，不包含具体日期，时区信息。
LocalDateTime|获取当前日期与时间信息，为LocalDate与LocalTime组合。
OffsetDateTime|在LocalDateTime的基础上增加偏移量信息。
ZonedDateTime|在OffsetDateTime基础上增加时区信息。
ZoneOffset|时区偏移量信息，比如+8:00或者-5:00等。
ZoneId|具体时区信息。比如Asia/ShangHai或者America/Chicago。

### LocalDate、LocalTime、LocalDateTime

``` java
@Test
public void testLocalDateTime() {
    LocalDate localDate = LocalDate.now();
    LocalTime localTime = LocalTime.now();
    LocalDateTime localDateTime = LocalDateTime.now();

    System.out.println(localDate);//严格按照 ISO 8601 格式打印
    System.out.println(localTime);//严格按照 ISO 8601 格式打印
    System.out.println(localDateTime);//严格按照 ISO 8601 格式打印

    //从 LocalDateTime 得到 LocalDate、LocalTime
    System.out.println(localDateTime.toLocalDate());
    System.out.println(localDateTime.toLocalTime());

    //通过 of 创建
    localDate = LocalDate.of(2022,3,4);//2022:03:04
    localTime = LocalTime.of(20,12,2);//20:12:02
    localDateTime = LocalDateTime.of(2022,8,5,15,54,32);//2022:08:05T15:54:32
    
    //通过 parse 将字符串创建，字符串必须严格遵守 ISO 8601 格式，否则异常。
    localDate = LocalDate.parse("2022-09-09");
    localTime = LocalTime.parse("20:03:22");
    localDateTime = LocalDateTime.parse("2022-09-09T23:22:02.321");
    System.out.println(localDate);
    System.out.println(localTime);
    System.out.println(localDateTime);

    //异常，正确应该为;2022-09-09
    localDate = LocalDate.parse("2022-09-9");
}
```

ISO 8601 格式为：`yyyy-MM-ddTHH:mm:ss.SSS`，日期和时间的分隔符`T`。在使用 `parse` 时字符串必须符合 ISO 8601 格式，否则运行异常。

如果要自定义输出或自定义解析格式，需要使用 `DateTimeFormatter`。

``` java
@Test
public void testDateTimeFormatter() {
    // 自定义格式化:
    DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
    System.out.println(dtf.format(LocalDateTime.now()));
    // 用自定义格式解析:
    LocalDateTime dt2 = LocalDateTime.parse("2022/08/30 15:16:17", dtf);
    System.out.println(dt2);
}
```

`LocalDateTime`提供了加减的链式调用，操作不会影响当前对象，例如调用方法：`withMonth`、`plusMonths`时不影响 `ldt` 对象。另外将月份调整为9月时，因为9月份没有31号，将变成9月30号。

``` java
    @Test
    public void test2() {

        LocalDateTime ldt = LocalDateTime.of(2022, 8, 31, 1, 1, 1);
        LocalDateTime ldt2 = ldt.withMonth(9);
        LocalDateTime ldt3 = ldt.plusMonths(1);

        System.out.println("原始："+ldt);
        System.out.println(ldt2);
        System.out.println("再次输出："+ldt);
        System.out.println(ldt3);
        System.out.println("再次输出："+ldt);
    }

/*
原始：2022-08-31T01:01:01
2022-09-30T01:01:01
再次输出：2022-08-31T01:01:01
2022-09-30T01:01:01
再次输出：2022-08-31T01:01:01
*/
```

比较两个`LocalDateTime`的大小。

``` java
    @Test
    public void test3(){
        LocalDateTime ldt = LocalDateTime.now();
        LocalDateTime ldt2 = LocalDateTime.parse("2022-05-04T23:34:59");

        System.out.println(ldt.isAfter(ldt2));//true
    }
```

注意到`LocalDateTime`无法与时间戳进行转换，因为`LocalDateTime`没有时区，无法确定某一时刻。后边介绍的`ZonedDateTime`具有时区，可以与`long`表示的时间戳进行转换。

### Duration 和 Period

- Duration 表示两个时刻之间的时间间隔。
- Period 表示两个日期之间的天数。

待完善。

### ZonedDateTime

`ZonedDateTime`可以理解为`LocalDateTime`+`ZoneId`。

``` java
    @Test
    public void testZonedDateTime(){
        ZonedDateTime zonedDateTime = ZonedDateTime.now();
        ZonedDateTime zonedDateTime1 = ZonedDateTime.now(ZoneId.systemDefault());
        ZonedDateTime zonedDateTime2 = LocalDateTime.now().atZone(ZoneId.systemDefault());

        LocalDateTime localDateTime = zonedDateTime2.toLocalDateTime();

        System.out.println(zonedDateTime);
        System.out.println(zonedDateTime1);
        System.out.println(zonedDateTime2);
        System.out.println(localDateTime);
    }
/*
2022-08-07T14:46:29.386+08:00[Asia/Shanghai]
2022-08-07T14:46:29.387+08:00[Asia/Shanghai]
2022-08-07T14:46:29.387+08:00[Asia/Shanghai]
2022-08-07T14:46:29.387
*/
```

### Instant

``` java
    @Test
    public void testInstant() {
        Instant instant = Instant.now();
        System.out.println(instant.getEpochSecond());//秒
        System.out.println(instant.toEpochMilli());//毫秒
        System.out.println(System.currentTimeMillis());//毫秒

        //根据时间戳获取时刻
        Instant instant1 = Instant.ofEpochMilli(1659855709655L);
        //时刻+时区 可以得到 ZonedDateTime
        ZonedDateTime zonedDateTime = instant1.atZone(ZoneId.systemDefault());

        System.out.println(instant1);
        System.out.println(zonedDateTime);
    }
/*
1659855954
1659855954445
1659855954445
2022-08-07T07:01:49.655Z
2022-08-07T15:01:49.655+08:00[Asia/Shanghai]
*/
```

## 新旧API转换

时间在计算机表示为一个正整数（时间戳），新旧API转换之间的转换都需要通过时间戳。

### 旧API转新

将`Date`或`Calendar`通过`ToInstant`方法转换成`Instant`对象，继而转换成`ZonedDateTime`、转换成`LocalDateTime`。

``` java
    @Test
    public void testApiChange(){
        Date date = new Date();
        LocalDateTime localDateTime = date.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(sdf.format(date));

        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        System.out.println(dtf.format(localDateTime));
    }
```

### 新API转旧

``` java
    @Test
    public void testApiChange(){

        Long timespan = LocalDateTime.now().atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();
        Date date = new Date(timespan);

        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(simpleDateFormat.format(date));
    }
```
