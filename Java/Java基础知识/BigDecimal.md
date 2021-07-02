
https://www.cnblogs.com/ljk-shm-0208/p/13821161.html
* 保留两位小数
```java
String bachelorScience = "1023.029";
BigDecimal basicSalaryBD = new BigDecimal(bachelorScience).setScale(2, RoundingMode.HALF_UP);
System.out.println(basicSalaryBD.toString());
```
