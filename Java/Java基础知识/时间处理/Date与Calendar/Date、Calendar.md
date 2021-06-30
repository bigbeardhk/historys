
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
> 常用方法
- 
