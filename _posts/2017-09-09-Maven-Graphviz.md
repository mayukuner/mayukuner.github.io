---
layout: post
title: 在Eclipse项目中通过Maven使用其他包 (Package)的方法 (软工课程相关)
---

第一眼看实验1的时候，发现是真的难。在不用外部库且图美观可读的前提下画出一个带权有向图，想一想就恶心。然而，一天之后，要求就改了。画图部分，可以使用外部库了。助教学长还推荐了Graphviz这个东西。于是就搜了一下“Graphviz Java”，还真是搜出了点东西：[graphviz-java](https://github.com/nidi3/graphviz-java)——提供了能在Java中使用的Graphviz接口。但这个repo的作者在README里一点都没写怎么在项目里引入这个东西，本人也是在解奥然学长帮助下才（算是有点）搞明白，在这里粘一下我在项目里引入graphviz-java的方法吧。

### 捷径——使用Maven

> Apache Maven，是一个软件（特别是Java软件）项目管理及自动构建工具。[^1]

就通过Maven导入graphviz-java的过程而言，个人觉得Maven有点像是make，程序员设置依赖关系，然后Maven就去满足依赖。而我们就只需要在项目的依赖中加入graphviz-java，然后使用Maven构建程序，Maven就会自动下载部署所需要的依赖，graphviz-java就可以用了。

### 在Eclipse工程中添加依赖[^2]

如果想要使用Maven，首先需要新建Maven工程。菜单栏里"File" - "New" - "Other"，弹出的"New"对话框中，选择"Maven" - "Maven Project"，然后点击"Next"。

![New](/public/images/new_maven_project.png)

接下来，一路"Next"，随便填好"Group ID"和"Artifact ID"，点击"Finish"，就可以新建一个Maven工程。

![New2](/public/images/new_maven_project2.png)

在"Package Explorer"中，选择刚才新建的项目，右键 - "Maven" - "Add Dependency"，搜索"graphviz-java"，然后就能找到对应的库，选中确认即可。

![add_dependency](/public/images/add_dependency.png)

另外，graphviz-java还需要日志记录的依赖，按刚才添加graphviz-java的方法添加SLF4J或者Log4j均可 (graphviz-java的Github repo中有提示)。

之后，在项目上右键 - "Run as" - "Maven Install"，之后Maven就会自动下载所需依赖，然后构建项目。看到Console里出现"Build Success"就说明构建成功了。

```java
package com.untitled.tests233;

import static guru.nidi.graphviz.model.Factory.*;

import java.io.File;
import java.io.IOException;

import guru.nidi.graphviz.engine.*;
import guru.nidi.graphviz.model.Graph;

public class App
{
    public static void main( String[] args ) throws IOException
    {
    	Graph g = graph("example1").directed().with(node("a").link(node("b")));
    	Graphviz.fromGraph(g).width(200).render(Format.PNG).toFile(new File("example/ex1.png"));
    }
}

```

输入上面的代码 (最前面的包名可能需要修改)，执行一下，就能看到项目文件夹下"example"文件夹中多了"ex1.png"文件，说明示例也运行成功了。


[^1]: [https://zh.wikipedia.org/wiki/Apache_Maven](https://zh.wikipedia.org/wiki/Apache_Maven)
[^2]: 本人使用的Eclipse版本为Eclipse IDE for Java Developers 4.7.0

