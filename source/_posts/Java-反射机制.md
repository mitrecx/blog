---
title: Java 反射机制
date: 2019-02-13 09:57:17
tags: java
categories: java
---
# 1 反射概述
在程序运行时，**对于任意一个类，都能获得这个类的所有的属性和方法**；  
对于任意一个对象，都能够调用它的任意一个方法和属性(包括私有的方法和属性)。  
这种 动态获取的类信息 以及 动态调用对象的方法 的功能 就称为java语言的反射机制。  

要想对一个类进行反射，必须先获取该类的 字节码文件(.class)。反射是通过 **<font color=red>Class类</font>** 中的方法实现的。  
每一个 **类** 对应着一个 **字节码文件** 也就对应着一个 **Class类型对象**，也就是字节码文件对象。  


获取Class对象的三种方式:  
1、通过类名获取 类名.class  
2、通过对象获取 对象名.getClass()  
3、**<font color='#1a75ff'>通过全类名获取 Class.forName(全类名)</font>**  

```java
public static void main(String[] args) {

  // 1-类名.class
  Class s1Class=Student.class;
  System.out.println(s1Class.getName());

  // 2-对象名.getClass
  Student s2=new Student();
  Class s2Class=s2.getClass();
  System.out.println(s2Class.getName());
  System.out.println(s1Class==s2Class);

  // 3-Class.forName
  try {
    Class s3Class = Class.forName("domain.Student");
    System.out.println(s3Class.getName());
    System.out.println(s2Class==s3Class);
  } catch (ClassNotFoundException e) {
    e.printStackTrace();
  }
}
```
运行结果：  
```
domain.Student
domain.Student
true
domain.Student
true
```
从上面程序结果可以看出，一个类Student 只有一个 类类型Class 对象，不论Class对象是如何创建的，不论Class对象创建多少次。  

事实上, 通过 <code>ClassLoader</code> 也能 获取 Class对象:  
```Java
Class<?> clazz = MitreTest.class.getClassLoader().loadClass(("cn.mitrecx.Hello"));
```

# 2 获取构造方法 并调用--newInstance
**Class对象** 获取 **<font color=red>所有 public 构造函数</font>**： <font color='#8a00e6'>getConstructors()</font>  

获取 **<font color=red>所有构造函数(声明的构造函数)</font>**： <font color='#8a00e6'>getDeclaredConstructors()</font>  

获取 **<font color=red>指定的 public 构造函数</font>**：   
```java
public Constructor<T> getConstructor(Class<?>... parameterTypes)  
//getConstructor 的参数 要和 某个构造函数的参数一致，才能获取到对应的构造函数
```

获取 **<font color=red>指定的 非public 构造函数</font>**：   
```java
public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)
// 一样的道理，getDeclaredConstructor 的参数 要和 某个构造函数的参数一致
```

Constructor对象 通过调用 **<font color='#8a00e6'>newInstance方法</font>** 创建对象。  

示例：  
```java
public class ReflectDemo2 {
  public static void main(String[] args) throws Exception {
    Class<?> clazz = Class.forName("domain.Student");
    System.out.println("===获取所有public构造函数");
    Constructor[] constructorArrayPub = clazz.getConstructors();
    for (Constructor c : constructorArrayPub) {
      System.out.println(c);
    }

    System.out.println("\n===获取所有的构造方法(声明的构造函数)");
    Constructor[] constructorArrayAll = clazz.getDeclaredConstructors();
    for (Constructor c : constructorArrayAll) {
      System.out.println(c);
    }

    System.out.println("\n===获取指定的 public构造方法");
    Constructor constructor1 = clazz.getConstructor(java.lang.String.class, int.class, char.class);
    Object obj = constructor1.newInstance("嘉宣", 24, 'M');
    System.out.println(obj);

    System.out.println("\n===获取指定的 private构造方法");
    Constructor constructor2 = clazz.getDeclaredConstructor(java.lang.String.class, int.class);
    constructor2.setAccessible(true);// private方法，调用之前必须设置为可见
    Object obj2 = constructor2.newInstance("嘉文", 26);
    System.out.println(obj2);
  }
}
```

执行结果：  
```
===获取所有public构造函数
public domain.Student(java.lang.String,int,char)
public domain.Student()

===获取所有的构造方法(声明的构造函数)
public domain.Student(java.lang.String,int,char)
private domain.Student(java.lang.String,int)
protected domain.Student(java.lang.String)
public domain.Student()

===获取指定的 public构造方法
姓名:嘉宣 年龄:24 性别:M

===获取指定的 private构造方法
姓名:嘉文 年龄:26 性别:
```

# 3 获取成员变量 并调用--set
**Class对象** 获取 **<font color=red>所有 public 成员变量</font>**： <font color='#8a00e6'>getFields()</font>  

获取 **<font color=red>所有 成员变量</font>**： <font color='#8a00e6'>getDeclaredFields()</font>  

获取 **<font color=red>指定变量名的 成员变量</font>**：   
```java
public Field getDeclaredField(String name)
```

例如：  
```java
Constructor c=clazz.getConstructor(String.class,int.class,char.class);
Student student=(Student) c.newInstance("cherry",24,'F');
System.out.println("\n===指定字段名的 private 字段");
Field fieldName=clazz.getDeclaredField("name");
System.out.println(fieldName);

// 为字段赋值，因为是 private字段，所以要先设置为可访问的
fieldName.setAccessible(true);
// 直接给对象的 成员变量赋值
fieldName.set(student,"xun");
System.out.println(student);
```
结果：  
```
===指定字段名的 private 字段
private java.lang.String domain.Student.name
姓名:xun 年龄:24 性别:F
```
获取 **<font color=red>指定变量名的 public 成员变量</font>**：  
```java
public Field getField(String name)
```

# 4 获取成员方法并调用--invoke

**Class对象** 获取 **<font color=red>所有 public 成员方法(包括继承来的方法)</font>**： <font color='#8a00e6'>getMethods(String name)</font>   

获取 **<font color=red>所有 成员方法(没有来继承的方法)</font>**： <font color='#8a00e6'>getDeclaredMethods(String name)</font>    

获取 **<font color=red>指定的方法名和参数类型的 public 成员方法</font>**：   
```java
// name 是方法名，其他参数 指定 方法的变量类型
public Method getMethod(String name, Class<?>... parameterTypes)
```
例子：  
```java
Constructor constructor=clazz.getConstructor(String.class,int.class,char.class);
Student student=(Student)constructor.newInstance("轩辕荣耀",21,'M');
Method methodSetName=clazz.getMethod("setName", String.class);
/* invoke 调用方法 */
methodSetName.invoke(student, "永强");
System.out.println(student);
```
结果：  
```
姓名:永强 年龄:21 性别:M
```

获取 **<font color=red>指定的方法名和参数类型的 成员方法</font>**：  
```java
public Method getDeclaredMethod(String name, Class<?>... parameterTypes)
```
**<font color='#8a00e6'>invoke</font>** private方法之前，必须 **<font color='#8a00e6'>setAccessible(true)</font>**。  


# 参考
1、 [Java中反射机制详解](https://www.cnblogs.com/whgk/p/6122036.html)  
2、 [Java基础之—反射](https://blog.csdn.net/sinat_38259539/article/details/71799078)  
