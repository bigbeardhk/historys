# Java 8——日期时间工具库（java.time）

> 资料来源: https://www.cnblogs.com/lxyit/p/9442135.html

## 概述
> java.time包括**年、月、星期、日期时间、持续时间段、瞬时、时钟、时区**的抽象及处理。且api的设计上使用易读易于理解的名称和设计模式，让使用者欣然接受。而且提供旧版和新版api之间的互通以处理兼容性问题。

下面看张概览图，从宏观角度了解下java.time:
![image](https://user-images.githubusercontent.com/60782137/124485143-dae74580-ddde-11eb-8dd8-0c374131d06c.png)

- 第一层是对年、月、月中日、星期的抽象；
- 第二层是对日期、日期时间、时区的抽象，其中时区分为时区Id（Europe/Paris）和时区偏移量(Z/+hh:mm/-hh:mm)；
- 第三层是对区域时间和便宜时间的抽象；
- 第四层是对瞬时和时钟的抽象；
- 第五层是对时序时段和持续周期的抽象
- 右侧层是辅助工具类，如：日期时间格式、日期时间调整器、其他的日历系统；

## 常见api
### Period类
> 持续周期的描述类,一般表示两个LocalDate之间相差多少天,多少周等,可以表示的周期单位为**年,月,周,天**(不能表示小时及以下时间单位!)
- static Period between(LocalDate startDateInclusive, LocalDate endDateExclusive) //获取两段日期的日期段
- int getDays() //获取天数

### Duration类
> 时序时段的描述类,与Period类不同,它是对时间差的叙述,提供单位为天,时,分,最小为纳秒
- static Duration between(Temporal startInclusive, Temporal endExclusive) //Temporal类不能为LocalDate
- Duration abs() //取绝对值
- long getSeconds() //秒
- long toDays() //天
- long toMinutes()  //分钟
- long toMillis() //毫秒
<details>
<summary>代码演示:</summary>

```
LocalDateTime createDate = LocalDateTime.of(2021, 07, 01, 16, 01);
LocalDateTime today = LocalDateTime.of(2021, 07, 01, 16, 01,26);
Duration p = Duration.between(today,createDate).abs();
long num = p.getSeconds();
System.out.println(num);
```
</details>

### Instant与ZonedDateTime类
> Instant瞬时类
- ZonedDateTime atZone(ZoneId zone)
- boolean isAfter(Instant otherInstant)
- boolean isBefore(Instant otherInstant)
- boolean equals(Object otherInstant)
> ZonedDateTime区域时间
- LocalDateTime toLocalDateTime()
- int getMonthValue()
- Month getMonth()
- DayOfWeek getDayOfWeek()
<details>
<summary>代码演示:</summary>

```
Date date = new Date();
Instant createInstant = date.toInstant();
LocalDate createDate = createInstant.atZone(ZoneId.systemDefault()).toLocalDate();
```

</details>


