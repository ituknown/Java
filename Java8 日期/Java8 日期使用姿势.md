
JDK8 下常用的日期 API 包括 LocalDate、LocalTime、LocalDateTime 以及 Instant。

LocalDate 表示当前（或指定）日期，默认格式为：yyyy-MM-dd
LocalTime 表示当前（或指定）时间，默认格式为：HH:mm:ss SSS
LocalDateTime 表示当前（或指定）日期时间，默认格式为：yyyy-MM-ddTHH:mm:ss SSS
Instant 表示当前（或指定）时间瞬时点，或者瞬间点。

这几个 API 的用法基本一致，看下下面的几个示例：

# LocalDate

创建实例：

```java
// 默认以当前日期创建实例
LocalDate localDate = LocalDate.now();

// 指定日期(年月日)创建实例
LocalDate localDate = LocalDate.of(2020, 1, 1);
```

## 获取当前时间属性

```java
System.out.println(localDate.getYear());               //获取当前年份: 2021
System.out.println(localDate.getMonth());              //获取当前月份, 英文: DECEMBER
System.out.println(localDate.getMonthValue());         //获取当前月份, 数字: 12

System.out.println(localDate.getDayOfMonth());         //获取当前日期是所在月的第几天
System.out.println(localDate.getDayOfWeek());          //获取当前日期是星期几(星期的英文全称): SATURDAY
System.out.println(localDate.getDayOfYear());          //获取当前日期是所在年的第几天: 341

System.out.println(localDate.lengthOfYear());          //获取当前日期所在年有多少天
System.out.println(localDate.lengthOfMonth());         //获取当前日期所在月份有多少天
System.out.println(localDate.isLeapYear());            //获取当前年份是否是闰年
```

## 日期加减

```java
System.out.println(localDate.minusYears(1));           //将当前日期减1年
System.out.println(localDate.minusMonths(1));          //将当前日期减1月
System.out.println(localDate.minusDays(1));            //将当前日期减1天

System.out.println(localDate.plusYears(1));            //将当前日期加1年
System.out.println(localDate.plusMonths(1));           //将当前日期加1月
System.out.println(localDate.plusDays(1));             //将当前日期加1天
```

## 日期比较

```java
System.out.println(localDate.isAfter(localDate2));                    //localDate在localDate2日期之后, 即小于
System.out.println(localDate.isBefore(localDate2));                   //localDate在localDate2日期之前, 即大于
System.out.println(localDate.isEqual(localDate2));                    //localDate和localDate2日期是否相等
```

## 指定日期操作

```java
System.out.println(localDate.with(TemporalAdjusters.firstDayOfMonth()));            //本月的第1天
System.out.println(localDate.with(TemporalAdjusters.firstDayOfNextMonth()));        //下月的第1天
System.out.println(localDate.with(TemporalAdjusters.firstDayOfNextYear()));         //下年的第1天
```

```java
System.out.println(localDate.withDayOfMonth(2));                                    //本月的第2天
System.out.println(localDate.with(TemporalAdjusters.lastDayOfMonth()));             //本月的最后一天
System.out.println(localDate.with(TemporalAdjusters.previous(DayOfWeek.SUNDAY)));   //上一周星期天是几号
System.out.println(localDate.with(TemporalAdjusters.next(DayOfWeek.MONDAY)));       //下一周星期一是几号
```

## 验证日期支持的类型

```java
System.out.println(localDate.isSupported(ChronoField.DAY_OF_YEAR));    //当前时间支持的时间类型是：一年中的某一天，这个不知道应用场景
System.out.println(localDate.isSupported(ChronoUnit.DAYS));            //当前日期支持的单元：天(说明当前时间是按天来算的)
```


# LocalTime

创建实例：

```java
// 默认以当前时间创建实例
LocalTime localTime = LocalTime.now(); 

// 指定小时、分钟创建实例
LocalTime localTime = LocalTime.of(18, 18);

// 指定时分秒创建实例
LocalTime localTime = LocalTime.of(18, 18, 18);
```

## 获取当前时间属性

```java
System.out.println(localTime);                   // 当前时间: 12:59:31.816
System.out.println(localTime.getHour());         // 获取当前小时: 12
System.out.println(localTime.getMinute());       // 获取当前分钟: 59
System.out.println(localTime.getSecond());       // 获取当前秒数: 31
```

## 日期加减

```java
System.out.println(localTime.plusHours(1));      //将当前时间加1小时
System.out.println(localTime.plusMinutes(1));    //将当前时间加1分钟
System.out.println(localTime.plusSeconds(1));    //将当前时间加1秒

System.out.println(localTime.minusHours(1));     //将当前时间减1小时
System.out.println(localTime.minusMinutes(1));   //将当前时间减1分钟
System.out.println(localTime.minusSeconds(1));   //将当前时间减1秒
```

# LocalDateTime

创建实例：

```java
// 默认以当前日期时间创建实例
LocalDateTime localDateTime = LocalDateTime.now();

// 指定日期时间创建实例
LocalDate localDate = LocalDate.now();
LocalTime localTime = LocalTime.now();
LocalDateTime localDateTime = LocalDateTime.of(localDate, localTime);
```

# Instant

创建实例：

```java
Instant instant = Instant.now();  // 2021-08-15T05:39:16.401Z
```

## 获取当前时间属性

```java
System.out.println(instant.getNano());             //纳秒数
System.out.println(instant.getEpochSecond());      //1970年到现在的秒数
System.out.println(instant.toEpochMilli());       //1970年到现在的毫秒数(和new Date().getTime() System.currentTimeMillis 一样)
```
