---
layout: post
title: 在Eclipse上通过Maven配置Tomcat 9.0 + Struts 2.5 (软工课程相关)
---

实验二需要用Eclipse配置Tomcat和Struts环境，看到友班的同学用Jetbrains IntelliJ IDEA不费吹灰之力就配好了环境，心里有点羡慕。不过现在终于用Eclipse配置好了，把主要的步骤在这贴一下，最重要的还是自己踩过的坑，希望别人不用再踩了。

### 把Eclipse for Java Developers强化成Eclipse IDE for Java EE Developers

如果你像我一样，用现在的Eclipse版本用出了感情，不想再下一个Eclipse IDE for Java EE Developers，那你可能要花好几个小时装很多Eclipse的插件。因为一开始不理解，无病乱投医，装了许多插件，具体有：

通过"Help"-"Install New Software"安装的：

    1. Data Tools Platform Extender SDK, Data Tools Platform Enablement Extender SDK (数据库可视化管理相关)
    2. Eclipse Java EE Developer Tools
    3. Eclipse Java Web Developer Tools
    4. Eclipse Web Developer Tools
    5. Eclipse XML Editors and Tools
    6. JST Server Adapters, JST Server Adapters Extensions

通过"Help"-"Eclipse Marketplace"安装的：

    1. Tomcat Plugin (后来发现只支持到7.0，于是卸载了，实际上这东西也没什么大用)
    2. Emacs+ (这玩意太好用了，完美还原Emacs的快捷键)

### 安装、配置Tomcat

接下来可以[下载](http://tomcat.apache.org)Tomcat，如果下载的是绿色版，把Tomcat解压到某个目录里即可。

然后在"Window"-"Preferences"-"Server"-"Runtime Environments"里配置好Tomcat，使Eclipse能找到、正确调用Tomcat。

![]({{site.url}}/public/images/Configuring-Maven-Tomcat-Struts-on-Eclipse/configuring_tomcat.png){:center-image}

### 新建项目

这里我们新建一个Dynamic Web Project项目: "File"-"New"-"Other.."-"Web"-"Dynamic Web Project"。

![]({{site.url}}/public/images/Configuring-Maven-Tomcat-Struts-on-Eclipse/new_project_1.png){:center-image}

两次"Next"即可，在最后一步可以让向导帮我们生成web.xml。

新建完项目之后，目录结构如下：

![]({{site.url}}/public/images/Configuring-Maven-Tomcat-Struts-on-Eclipse/new_project_2.png){:center-image}

如果libraries里没有Tomcat的库，可以在项目的属性设置里的"Java Build Path"中，"Add Library"-"Server Runtime"-"Apache Tomcat V9.0"，把Tomcat的库加进Build Path。

接下来，把项目转化为Maven项目，以方便管理依赖。右键项目名-"Configure"-"Conver to Maven Project"。

### 在项目里配置Tomcat与Struts

首先把Struts添加到项目的依赖中，做法可以参考[前一篇博文](/2017/09/09/Maven-Graphviz/)。

在web-app标签中，加入这段代码，让Tomcat知道，我们使用的是Struts框架。

```XML
	<filter>
		<filter-name>struts2</filter-name>
		<filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>struts2</filter-name>
		<url-pattern>*.do</url-pattern>
	</filter-mapping>
	<filter-mapping>
		<filter-name>struts2</filter-name>
		<url-pattern>*.action</url-pattern>
	</filter-mapping>
```

接下来在"Java Resources/src"中新建struts.xml，填入以下代码，将"HelloWorld.action"映射到"HelloWorld.jsp"，中间需HelloWorld类处理：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN" "http://struts.apache.org/dtds/struts-2.3.dtd">
<struts>
	<include file="struts-default.xml" />
	<package name="test2333" extends="struts-default">
		<action name="HelloWorld" class="test2333.HelloWorld">
			<result>/HelloWorld.jsp</result>
		</action>
	</package>
</struts>
```

然后在"WEB-INF"中新建"HelloWorld.jsp"，填入以下代码：

```HTML
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    pageEncoding="ISO-8859-1"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
<title>Insert title here</title>
</head>
<body>
	<h2><s:property value="message" /></h2>
</body>
</html>
```

在"Java Resources/src"中新建"HelloWorld.java"，填入

```java
package test2333;

import com.opensymphony.xwork2.ActionSupport;

public class HelloWorld extends ActionSupport {
    public static final String MESSAGE = "Struts is up and running ...";

    public String execute() throws Exception {
        setMessage(MESSAGE);
        return SUCCESS;
    }

    private String message = "test";

    public void setMessage(String message){
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}

```

### 运行网站

这样就基本配置好了一个基于Struts的Java Web项目。想要启动它的话，只需要右击项目名-"Run As"-"Run On Server"，


![]({{site.url}}/public/images/Configuring-Maven-Tomcat-Struts-on-Eclipse/run_on_server.png){:center-image}

一路Next即可。

打开浏览器，地址栏输入"[localhost:8080/test2333/HelloWorld.action](localhost:8080/test2333/HelloWorld.action)"，就能看到效果了。

![]({{site.url}}/public/images/Configuring-Maven-Tomcat-Struts-on-Eclipse/page_on_chrome.png){:center-image}
