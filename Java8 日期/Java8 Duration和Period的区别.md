Duraction 表示：时间的区间，用来度量秒和纳秒之间的时间值
Period 表示：一段时间的区间，用来度量年月日和几天之间的时间值
# Duraction

```java
LocalTime localTime = LocalTime.now();
LocalTime localTime2 = LocalTime.of(19, 19, 19);
Duration duration = Duration.between(localTime, localTime2);
System.out.println(duration);
System.out.println(duration.isZero());            //Duration区间是否为0
System.out.println(duration.isNegative());        //Duration区间是否为负, 如果为 true 说明 localTime < localTime2

System.out.println(duration.getSeconds());        //Duration区间值的秒数
System.out.println(duration.getNano());           //Duration区间值的纳秒数
System.out.println(duration.getUnits());          //Duration的度量单位

System.out.println(duration.toDays());            //Duration区间相差几天
System.out.println(duration.toHours());           //Duration区间差几小时
System.out.println(duration.toMinutes());         //Duration区间相差几分钟
System.out.println(duration.toMillis());          //Duration区间相差几毫秒
```

# Period

```java
LocalDate localDate = LocalDate.now();
LocalDate localDate2 = LocalDate.of(2020, 12, 12);
Period period = Period.between(localDate, localDate2);

System.out.println(period);

System.out.println(period.isZero());            //区间是否为0        
System.out.println(period.isNegative());        //区间是否为为负, 为负数说明 localDate < localDate2

System.out.println(period.getYears());          //区间的相差几年
System.out.println(period.getMonths());         //区间的相差几月
System.out.println(period.getDays());           //区间的相差几日

System.out.println(period.toTotalMonths());     //区间相差多少个月
```
