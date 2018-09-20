---
title: tomcat启动错误
date: 2018-09-20 09:20:54
tags: tomcat
categories: servlet
---
九月 02, 2018 4:30:51 上午 org.apache.tomcat.util.digester.SetPropertiesRule begin
信息: Starting Servlet Engine: Apache Tomcat/8.0.36
九月 02, 2018 4:30:54 上午 org.apache.jasper.servlet.TldScanner scanJars
信息: At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
九月 02, 2018 4:30:54 上午 org.apache.catalina.core.ContainerBase startInternal
**<font color=red>严重: A child container failed during start</font>**
java.util.concurrent.ExecutionException: org.apache.catalina.LifecycleException:
**<font color=red>Failed to start component [StandardEngine[Catalina].StandardHost[localhost].StandardContext[/web01copy]]</font>**

```xml
<servlet>
  <servlet-name>action</servlet-name>
  <servlet-class>emp.ActionServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>action</servlet-name>
  <url-pattern>*.do</url-pattern>
</servlet-mapping>
```
<url-pattern\>**\*.do**</url-pattern\>被我写成了  
 <url-pattern\>**/\*.do**</url-pattern\>  而导致了错误。  
 仅仅一个 "/" 导致了起不了Tomcat。。平时一定要细心呀。  


我在网上搜了半天都没找到解决方法，  
而这个错误就在 console 输出的错误提示中。  

总结：
我们把应用部署到 Tomcat 中，当Tomcat 启动不了的时候，  
无非有两种可能(**不考虑Tomcat不兼容**)：    
1、 缺少jar包（或版本不兼容）  
2、 **应用配置有问题**  
而这两种可能都是应用的问题。  
