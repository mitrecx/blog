---
title: java运算符instanceof
date: 2018-11-02 10:16:24
tags: java
categories: java
---
关键字 **instanceof** 用于 判断一个对象是否是 **<font color=red>某一指定类型</font>**  
```java
if (objectReference instanceof type)
```

```java
  public static void main(String[] a) {
    String s = "Hello";
    if (s instanceof java.lang.String) {
      System.out.println("is a String");
    }
  }
```
结果：
```
is a String
```

下面看一个常用的 **instanceof** 示例：  

```java
public class Person {
  //属性略 
	public void printInterest() {
		System.out.println(interest);
	}
}


public class Student extends Person {
  //属性略
	public void printScore() {
		System.out.println(score);
	}
}

public class Teacher extends Person{
  //属性略
	public void printTeache() {
		System.out.println(teache);
	}
}
```

```java
public static void main(String[] args) {
		Person person=new Student("cx",24,new BigDecimal("172"),"show","12345",100);
		//Person person=new Teacher("lyy",34,new BigDecimal("165"),"teache","00001","math");
		if(person instanceof Student) {
			System.out.print("分数：");
			((Student) person).printScore();
		}else {
			System.out.println("the person is not a student ");
		}
		if(person instanceof Teacher) {
			System.out.println("教学：");
			((Teacher) person).printTeache();
		}else {
			System.out.println("this person is not a teacher");
		}
	}
```
结果：

```
分数：100
this person is not a teacher
```
分析：  
在此例中，我们知道，**<font color='#00cc00'>person</font>** 只能调用 **<font color='#8f00b3'>Person</font>** 类的方法，不能调用 **<font color='cc7a00'>Student</font>** 类的方法。    

**<font color='#00cc00'>person</font>** 要调用其子类 **<font color='cc7a00'>Student</font>** 的方法的话，必须强转为 **<font color='cc7a00'>Student</font>** 类型。  

如果 **<font color='#00cc00'>person</font>** 是 **<font color='cc7a00'>Student</font>** 实例化来的，那么强转为 **<font color='cc7a00'>Student</font>** 没问题。  
如果 **<font color='#00cc00'>person</font>** 是 **<font color='#cc0066'>Teacher</font>** 实例化来的，那么强转为 **<font color='cc7a00'>Student</font>** 会报错。  

所以，**父类对象/引用 强转为 子类类型 时，应当用instanceof 判断是否可以强制转换**。  
