## BigDecimal
> 资料引用: https://www.cnblogs.com/ljk-shm-0208/p/13821161.html
---
## 概述
> - 加减乘除、舍入等操作其实最终都返回的是一个新的BigDecimal对象，因为BigInteger与BigDecimal都是不可变的（immutable）的，在进行每一步运算时，都会产生一个新的对象

### 舍入模式  (RoundingMode)
> 若指定舍入的小数位 >= 实际可操作的最大位数 ,则BigDecimal的值不会变化
- **ROUND_UP**           //截取掉后面的小数位,如存在,则保留的最后位加一;不存在,则BigDecimal不会变化
- **ROUND_DOWN**         //与ROUND_UP相比,截取小数时,不会进一
- **ROUND_CEILING**      //向正无穷方向舍入；如果 BigDecimal 为正，则舍入行为与 ROUND_UP 相同; 如果为负，则舍入行为与 ROUND_DOWN 相同。
- **ROUND_FLOOR**        //向负无穷方向舍入；如果 BigDecimal 为正，则舍入行为与 ROUND_DOWN 相同;如果为负，则舍入行为与 ROUND_UP 相同。
- **ROUND_HALF_UP**      //四舍五入
- **ROUND_HALF_DOWN**    //五舍六入
- **ROUND_HALF_EVEN**    //如果保留位数是奇数，使用ROUND_HALF_UP，如果是偶数，使用ROUND_HALF_DOWN
- **ROUND_UNNECESSARY**  //计算结果是精确的，不需要舍入模式

<details>
<summary>代码演示:</summary>

```java

BigDecimal temp1 = new BigDecimal("10.085");
BigDecimal temp2 = temp1.setScale(2, RoundingMode.HALF_UP);
System.out.println(temp1.doubleValue()); //10.085
System.out.println(temp2.doubleValue()); //10.09

BigDecimal temp3 = new BigDecimal("5");
BigDecimal temp4 = new BigDecimal("4");
BigDecimal temp5 = temp3.divide(temp4, 1, RoundingMode.HALF_UP);
System.out.println(temp5.doubleValue());//1.3

```

</details>


## 格式化

### 百分比
```java
BigDecimal b1 = new BigDecimal("0.0825");
NumberFormat percentInstance = NumberFormat.getPercentInstance();
percentInstance.setMaximumFractionDigits(2); //百分比的小数点最多2位
String format = percentInstance.format(b1);
System.out.println(format); //8.25%
```

