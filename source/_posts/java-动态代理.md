---
title: Java 动态代理
date: 2019-04-09 15:10:32
tags: java
categories: java
---
最近看 《HeadFirst 设计模式》 看到 **代理模式** ，包括 *远程代理*，*保护代理*，*虚拟代理* 等。  
其中 *保护代理* 是使用 Java API的 **动态代理** 实现的。  

此外， Spring AOP 底层也是 **动态代理** 实现的。  
于是，借此机会学习一下 Java动态代理。  

# 1 动态代理
Java 在 java.lang.reflect 包中有自己的代理支持，利用这个包 可以实现在运行时 动态地创建 一个代理类，实现一个或多个接口，并将方法转发(dispatch)到指定的类。  
因为实际的代理类是 **运行时创建** 的， 于是称此技术为：动态代理。  

Java 动态代理 类图：  
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog2019/pic04/2019-04-09_221058.png)

# 2 Java 动态代理 API
Java 动态代理 API 提供了一个 **接口 InvocationHandler** 和 一个 **类 Proxy** 。  
正是 InvocationHandler 和 Proxy 让我们可以用简单的代码实现 动态代理 。  
## 2.1 InvocationHandler 接口
InvocationHandler is the interface implemented by the invocation handler of a proxy instance.  
Each proxy instance has an associated invocation handler.   
When a method is invoked on a proxy instance, the method invocation is encoded and dispatched to the invoke method of its invocation handler.  
InvocationHandler 是由 代理对象 的 **调用处理器** 实现的。  
每一个 代理对象 都有一个对应的 **调用处理器**。  
当一个 代理实例 的方法被调用时， 这个方法调用 会委派调用  **调用处理器** 的方法。  

InvocationHandler 接口 只有一个方法：  
**<font color=red>Object</font> <font color='#8a00e6'>invoke</font>(<font color=red>Object</font> proxy, <font color=red>Method</font> method, <font color=red>Object[]</font> args)**  
Processes a method invocation on a proxy instance and returns the result.  
动态代理 最重要的就是 要实现 **<font color='#8a00e6'>invoke</font>** 方法，**代理对象** 利用 **<font color='#8a00e6'>invoke</font>** 完成对 **被代理对象** 中的方法的调用。    

## 2.2 Proxy 类
Proxy provides static methods for creating dynamic proxy classes and instances, and it is also the superclass of all dynamic proxy classes created by those methods.  
**Proxy** 提供了 **创建动态代理类** 和 实例 的 静态方法。Proxy 同时是 所有 动态代理类(由其静态方法创建) 的超类。  

Proxy 有一个成员变量，一个构造方法，和四个 静态成员方法：  
```java
// 成员变量
protected InvocationHandler h;
// 构造方法
protected Proxy(InvocationHandler h);
// 静态成员方法
public static InvocationHandler getInvocationHandler(Object proxy);
public static Class<?> getProxyClass(ClassLoader loader, Class<?>... interfaces);
public static boolean isProxyClass(Class<?> cl);
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h);
```
创建 动态代理对象时 用 **<font color='#8a00e6'>newProxyInstance</font>** 方法，Java API 对此方法介绍如下：  
Returns an instance of a proxy class for the specified interfaces that dispatches method invocations to the specified invocation handler.  
返回指定接口的代理对象，此代理对象 能够把 对代理对象的方法调用 委派让 调用处理器 执行。  
**简而言之，客户调用 代理对象的方法， 代理对象会调用 InvocationHandler 的 invoke 方法， 然后 invoke 方法 调用 被代理对象的对应方法。**  
**这样在客户看来，调用代理对象 就像 调用原对象(被代理对象) 一样。**  

**<font color='#8a00e6'>newProxyInstance</font>** 方法的三个参数：  
**<font color='#ff9900'>loader </font>**: 一个ClassLoader对象，由此ClassLoader对象来对生成的代理对象进行加载。  
**<font color='#ff9900'>interfaces </font>**: 一个Interface对象的数组，值是被代理对象实现的接口。代理对象会宣称实现了这些接口，这样就能 用代理对象 调用这组接口中的方法了。   
**<font color='#ff9900'>h </font>**: 一个InvocationHandler对象，当动态代理对象在调用方法的时候，会调用此 InvocationHandler 对象 的 invoke 方法。  

# 3 动态代理示例
再看一遍 Java动态代理 的类图：  
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog2019/pic04/2019-04-09_221058.png)  
Java API 已经提供了 **Proxy 类** 和 **InvocationHandler 接口** ， 因此，只要写一个 **<font color='#2eb82e'>Subject 接口</font>**， 一个 **<font color='#2eb82e'>Subject 的实现类</font>**，和 一个 **<font color='#2eb82e'>InvocationHandler 的实现类</font>** 就行了。  

下面是示例的具体代码。  

**<font color='#2eb82e'>Subject 接口</font>** :  
```java
public interface Subject {
	String location();
	void printLocation(String s);
}
```

**<font color='#2eb82e'>Subject 的实现类</font>** :   
```java
public class RealSubject implements Subject{

	@Override
	public String location() {
		return "江苏省常州市金坛区江南农商银行";
	}

	@Override
	public void printLocation(String s) {
		System.out.println(s+"江南");
	}
}
```

**<font color='#2eb82e'>InvocationHandler 的实现类</font>** :  
```java
public class ConcreteInvocationHandler implements InvocationHandler {
	private Subject subject;
	public ConcreteInvocationHandler(Subject subject) {
		this.subject=subject;
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("invoke 方法被调用");
		Object o=method.invoke(subject, args);
		return o;
	}

}
```
注意对比看类图，**InvocationHandler 是实现动态代理的关键！** InvocationHandler 的实现类 **有一个 (has-A)** RealSubject 被代理对象，invoke 方法 正是调用 此对象的 方法 实现动态代理的。  


测试代码：  
```java
public class DynamicProxyDrive {
	public static void main(String[] args) throws IllegalArgumentException, ClassNotFoundException {
		// 原对象(被代理对象)
		Subject subject = new RealSubject();

		InvocationHandler ih = new ConcreteInvocationHandler(subject);
		// 代理对象
		Subject proxySubject = (Subject) Proxy.newProxyInstance(Class.forName("cn.mitre.subject.RealSubject").getClassLoader(),  Class.forName("cn.mitre.subject.RealSubject").getInterfaces(), ih);
		// 代理对象 代理了 原对象，于是可以把 代理对象 当成原对象使用
		proxySubject.printLocation("四月");
		System.out.println(proxySubject.location());
	}
}
```
输出结果：  
```
invoke 方法被调用
四月江南
invoke 方法被调用
江苏省常州市金坛区江南农商银行
```

# 参考
1、 HeadFirst 设计模式  
2、 [java的动态代理机制详解](https://www.cnblogs.com/xiaoluo501395377/p/3383130.html)  
