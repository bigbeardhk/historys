
>  JDK8中，考虑使用Instant 代替 Date， LocalDateTime 代替 Calendar，DateTimeFormatter 代替 SimpleDateFormat。

https://www.cnblogs.com/lxx2014/p/9394841.html

https://blog.csdn.net/pxy_1996/article/details/103049362

## Date类
> 常用方法
- Date(long date):  通过时间戳获取对象
- long getTime():  获取其时间戳
- boolean before(Date when):  A是否比B时间早
- boolean after(Date when):  A是否比B时间晚
- static Date from(Instant instant): Instant装换为Date类型
- Instant toInstant():  Date装换为Instant类型


## Calendar类
>  Calendar为抽象类，由于语言敏感性，Calendar类在创建对象时并非直接创建，而是通过静态方法创建
>  常用方法
-  static getInstance() : 使用默认时区和语言环境获取一个日历!
> Calendar c = Calendar.getInstance();  //返回当前时间
-  int get(int field) :  获取时间字段值
>  YEAR 年    MONTH 月,从0开始算起,最大11;0代表1月,11代表12月。  DATE 天   HOUR 时   MINUTE分   SECOND秒
