

>  JDK8中，考虑使用Instant 代替 Date， LocalDateTime 代替 Calendar，DateTimeFormatter 代替 SimpleDateFormat。

## 获取时间戳
- System.currentTimeMillis();    (推荐!)
- Calendar.getInstance().getTimeInMillis();
- new Date().getTime();

## Date类
### 常用方法
- Date(long date):  通过时间戳获取对象
- long getTime():  获取其时间戳
- boolean before(Date when):  A是否比B时间早
- boolean after(Date when):  A是否比B时间晚
- static Date from(Instant instant): Instant装换为Date类型
- Instant toInstant():  Date装换为Instant类型

## Calendar类
>  Calendar为抽象类(日历)，由于语言敏感性，Calendar类在创建对象时并非直接创建，而是通过静态方法创建
### 常用方法
-  static Calendar getInstance() : 使用默认时区和语言环境获取一个日历!
-  final void set(int year, int month, int date)//设置某个日历时期,   
-  int get(int field) :  获取时间字段值,比如当前的年份,月份值-1,日份等
-  void add(int field,int amount)   //指定字段增加某值

<details>
<summary>代码演示:</summary>

```Java
        Calendar c = Calendar.getInstance();
        c.set(2021,Calendar.JUNE,26);
        int vale = c.get(Calendar.DAY_OF_MONTH);
        int vale2 = c.get(Calendar.DAY_OF_WEEK);
        //修改当前时间为3天后
        c.add(Calendar.DATE, 3);
        //修改当前时间为5小时后
        c.add(Calendar.HOUR, 5);
        Date time = c.getTime();
        long timeInMillis = c.getTimeInMillis();
```
```
        - YEAR 年
        - MONTH 月 (从0开始算起,最大11;0代表1月,11代表12月)
        - DATE 天
        - HOUR 时
        - MINUTE 分
        - SECOND 秒
```

</details>

