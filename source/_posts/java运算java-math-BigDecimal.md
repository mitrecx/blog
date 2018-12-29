---
title: java运算java.math.BigDecimal
date: 2018-10-31 10:38:51
tags: java
categories: java
---
**空值可以强转为 BigDecimal 类型** ：   
```java
Object a=null;
BigDecimal b=(BigDecimal)a;
//b=null
```
**<font color=red>调用b时，注意非空判断。</font>**   

-----

BigDecimal常用的构造方法：  
```java
BigDecimal(double val)  
BigDecimal(int val)
BigDecimal(long val)
BigDecimal(char[] in)
BigDecimal(String val)  // 建议使用
```
参考[java API](https://docs.oracle.com/javase/7/docs/api/) .  
**参数类型为double的构造方法的结果有一定的不可预知性。**  
**new BigDecimal(0.01)** 结果是：0.010000000000000000208...  
而 **new BigDecimal("0.01")** 就是 0.01  。   


# 1. 加减乘除
```java
//加
add(BigDecimal)
//减  
subtract(BigDecimal)   
//乘
multiply(BigDecimal)  
```
加减乘除 其实最终都返回的是一个新的BigDecimal对象，  
因为BigInteger与BigDecimal都是不可变的（immutable）的，  
在进行每一步运算时，都会产生一个新的对象。  
所以a.add(b);虽然做了加法操作，但是a并没有保存加操作后的值，  
**正确的用法应该是a=a.add(b);**    

加减乘一般不用考虑精度，而 **除法需要考虑精度 (scale) 问题**，BigDecimal 的除法重载了6次，一般用下面的方法就够了：  
```java
//除
BigDecimal divide(BigDecimal divisor, int scale, int roundingMode)
```
例子：  
```java
BigDecimal b1=new BigDecimal("100");
BigDecimal b3=new BigDecimal("3");
System.out.println(b1.divide(b3, 10, BigDecimal.ROUND_HALF_UP));
```
结果：  
```
33.3333333334
```
**分析**:  
1. 精度scale=10，保留10位小数。  
2. **BigDecimal.ROUND_HALF_UP** 四舍五入。  

BigDecimal 提供了 8 中 舍入模式(**<font color='#cc0066'>Rounding mode</font>**)：  
```java
  public final static int ROUND_UP =           0; //
  public final static int ROUND_DOWN =         1; //
  public final static int ROUND_CEILING =      2;
  public final static int ROUND_FLOOR =        3;
  public final static int ROUND_HALF_UP =      4; //
  public final static int ROUND_HALF_DOWN =    5;
  public final static int ROUND_HALF_EVEN =    6;
  public final static int ROUND_UNNECESSARY =  7;
```
常用的有：  
* ROUND_UP 进一法(不论保留位是多少，都进一)
* ROUND_DOWN 截断法(不论保留位是多少，都舍弃)
* ROUND_HALF_UP 四舍五入法(保留位大于等于5进一，否则就舍弃)
* ROUND_HALF_DOWN 五舍五入(**保留位及以后位大于5进一**，否则舍弃)  

ROUND_HALF_UP 和 ROUND_HALF_DOWN 很像，只是在对5 的进位尺度有一点点不同。  
比如，  
**3.0500** 保留两位小数：  
**ROUND_HALF_UP**：3.1  
**ROUND_HALF_DOWN**：3.0    

**3.0501** 保留两位小数：  
**ROUND_HALF_UP**：3.1  
**ROUND_HALF_DOWN**：3.1  

# 2. 设置精度
```java
BigDecimal setScale(int newScale, int roundingMode)
```
newScale : 精度，保留小数的位数。  
**<font color='#cc0066'>RoundingMode</font>** : 进位模式，上面 8 种模式之一。


# 3. 比较大小

对比：Returns:-1, 0, or 1 as this BigDecimal is numerically less than, equal to, or greater than val.  
```java
int compareTo(BigDecimal val)
```
说明：**a.compareTo(b)**  
 * **若a>b : 返回1**   
 * 若a=b : 返回0  
 * 若a<b : 返回-1

# 4. 其他
**乘方**：Returns a BigDecimal whose value is (this^n), The **power** is computed exactly, tounlimited precision.   
```java
BigDecimal pow(int n)
```

**取绝对值**：Returns a BigDecimal whose value is the **absolute value** of this BigDecimal, and whose scale is this.scale().  
```java
BigDecimal abs()
```

**取反**：Returns a BigDecimal whose value is (-this),and whose scale is this.scale().    
```java
BigDecimal negate()
```
