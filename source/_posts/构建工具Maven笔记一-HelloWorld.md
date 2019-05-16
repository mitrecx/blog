---
title: '构建工具Maven笔记一: HelloWorld'
date: 2019-03-20 22:09:13
tags: Maven
categories: Maven
---
# 1 构建工具
构建 ( build ) 是把项目源代码变成项目产品的过程。其中包含编译，测试，打包，部署等工作。  

常见的构建工具：  
**<font color=green>Make</font>** 是最早的构建工具，Stuart Feldman 于 1977 年在 Bell实验室 创建。 Make 由 **Makefile** 脚本文件驱动。常用于C++ 项目。  

**<font color=green>Apache Ant</font>** (Another Neat Tool 另一个简洁的工具), Ant 像是Java版本的 Make。 Ant 由 **build.xml** 构建配置文件驱动。它最早用来构建 Tomcat， 现在已经逐渐被 **Maven** 和 **Gradle** 取代了。  

**<font color=green>Apache Maven</font>** 是在 Ant 之后流行的构建工具。Maven 项目的核心是 **pom.xml** 文件。POM (Project Object Model 项目对象模型) 定义了项目的基本信息。  

**<font color=green>Gradle</font>** 是一个基于Apache Ant和Apache Maven概念的项目自动化构建工具。Gradle 使用基于 **Groovy** 的特定领域语言(DSL)来声明项目设置，抛弃了基于XML的各种繁琐配置。    

# 2 Window10 安装 Maven
[Apache Maven 下载地址](http://maven.apache.org/download.cgi), 点击下载 apache-maven-3.6.0-bin.tar.gz 即可。  
下载完成后：   
1、 解压  
2、 配置环境变量 M2_HOME 指向 解压的 Maven 目录  
3、 配置环境变量 Path， 添加一项 **%M2_HOME%\bin**  
4、 win+R , cmd 打开终端 输入 <code>echo %M2_HOME%</code> , 检查环境变量是否配置好  
5、 终端输入， <code>mvn -v</code> 查看 Maven 版本。显示版本即可。  

# 3 Maven项目入门程序

## 3.1 编写 pom.xml 文件
创建一个文件夹/项目目录 hello-world , 在此文件夹下 创建 pom.xml 文件。  
pom.xml 文件内容如下：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <groupId>cn.mitre</groupId>
  <artifactId>hello-world</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>Maven Hello World Project</name>

</project>
```
**&lt;project&gt;** 是 所有 pom.xml 的根元素。  
modelVersion 指定了当前 POM模型 的版本，对于 Maven3 来说，只能是4.0.0。  
**groupId** 定义了项目属于哪个组。 groupId一般是 公司域名倒写.(点)项目名，例如，com.bilibili.myapp。  
**artifactId** 定义了当前 Maven项目在 组中唯一的构件id。  
**version**  指定了Maven项目当前的版本。  
name 元素 声明了项目名称，name元素不是必须的，为了方便查看常声明name元素。  

**<font color=orange>groupId</font>**, **<font color=orange>artifactId</font>**, **<font color=orange>version</font>**, 这 3 个元素定义了一个项目的基本坐标。在Maven项目中，任何jar、 pom、 war都是基于这 3 个元素区分的。    

## 3.2 编写主代码
在 hello-world 文件夹下(与pom.xml同级目录)，创建文件   
<code>src/main/java/cn/mitre/helloworld/HelloWorld.java</code> 。  
HelloWorld.java 代码如下：  
```java
package cn.mitre.helloworld;

public class HelloWorld{
	public String sayHello(){
		return "Hello Maven";
	}

	public static void main(String[] args){
		System.out.println(new HelloWorld().sayHello());
	}
}
```
**<font color=red>注意</font>**：java 代码放在 src/main/java/ 目录下 是为了遵循 Maven 的约定，这样无需额外的配置。  

打开终端，进入到 项目目录(hello-world)下，执行以下命令进行 **编译**：  
```sh
mvn clean compile
```
clean 会清除输出目录 target。  
compile 会编译项目主代码。  

## 3.3 编写测试代码
Maven 项目默认的测试代码 目录是 src/test/java 。  
和编写主代码一个道理，创建文件   
src/test/java/cn/mitre/helloworld/HelloWorldTest.java  
编写测试代码之前，需要在 pom.xml 中配置 单元测试 JUnit 依赖：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <groupId>cn.mitre</groupId>
  <artifactId>hello-world</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>Maven Hello World Project</name>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.7</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

</project>
```
Maven 会根据 **<font color=orange>groupId</font>**, **<font color=orange>artifactId</font>**, **<font color=orange>version</font>** 定位并下载 junit-4.7.jar 。  
scope 指定了依赖范围，test 表示只对测试代码有效， compile 表示对主代码和测试代码都有效。  

有了测试依赖，就可以编写测试代码了：  
```java
package cn.mitre.helloworld;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class HelloWorldTest{
	@Test
	public void testSayHello(){
		HelloWorld helloWorld = new HelloWorld();
		String result=helloWorld.sayHello();
		assertEquals("Hello Maven", result); //测试 result 是否等于 "Hello Maven"
	}
}
```
JUnit4 要求所有的测试方法以 test 开头， 同时加上 @Test 注解。  

测试代码写完后， 打开终端，执行以下命令完成 **测试**：  
```sh
mvn clean test
```
Maven 在执行 **测试** 之前，会自动 **编译** 主代码和测试代码。  


## 3.4 打包和运行
终端执行以下命令 进行 **打包**：  
```
mvn clean package
```
Maven 在执行 **打包** 之前， 会自动 **编译** 和 **测试** 。  
**打包** 命令执行成功后，会在输出目录 target 下 生成一个 名称为 artifact-version.jar 的包。  

如果想让其他 Maven 项目直接引用这个 jar 包，还需要执行 **安装** 命令：  
```
mvn clean install
```
Maven 在执行 **安装** 之前， 会自动 **打包**。  
**install** 命令执行成功后，可以在本地仓库对应的目录(C:\Users\cx141\.m2\repository\cn\mitre\hello-world\1.0-SNAPSHOT)看到 pom 和 jar 文件。  

## 3.5 生成可执行的 jar 文件(可选)
在上面的 HelloWorld 类里 有一个 main 方法，默认 **打包** 生成的 jar 是不可以执行的。  
为了生成可执行的 jar 包， 需要借助 maven-shade-plugin 插件， 在 pom.xml 文件中 配置该插件：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>cn.mitre</groupId>
  <artifactId>hello-world</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>Maven Hello World Project</name>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.7</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.1.1</version>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
            <configuration>
              <transformers>
                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                  <mainClass>cn.mitre.helloworld.HelloWorld</mainClass>
                </transformer>
              </transformers>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>

</project>
```
配置 mainClass 元素为 cn.mitre.helloworld.HelloWorld ，项目在 **打包** 时，会把该信息放在 MANIFEST 中。  
有了 main 方法信息，再执行 <code>mvn clean install</code> 就会在 target 目录下生成两个 jar ， 分别是 hello-world-1.0-SNAPSHOT.jar 和 original-hello-world-1.0-SNAPSHOT.jar 。前者是带 mainClass 信息可运行的 jar， 后者是原生的 jar。  
打开终端，进入到项目目录，执行 hello-world-1.0-SNAPSHOT.jar ：  
```
java -jar target\hello-world-1.0-SNAPSHOT.jar
```
执行结果如下：  
```
Hello Maven
```

# 参考
1、 Maven 实战【许晓斌】
