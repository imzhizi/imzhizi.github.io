---
title: "个人对于 Maven 的理解"
date: 2020-03-07T01:10:48+08:00
author: "质子"
description: ""
tags:
- tool
categories: 
- tycho
image: 
---


Maven 一直都在使用, 但如果说是不是真的懂 Maven, 很难谈得上. 或许什么时候系统地学习一下, 但在那之前, 打算先记下自己目前对于 Maven 的理解, 之后再进行对比, 看有哪些疏漏和错误.

### Maven 基础

最直观的, Maven 使用 pom.xml 文件来管理项目中所使用的依赖, 这跟 Python 的 requirement.txt、JS 的 `package.json` 或者是 Ruby 的 `Gemfile` 都类似.

不过, 其他语言的依赖文件规定的往往是某个包的最低版本, 在实际安装的时候, 很可能会安装最新版, 这个时候实际版本会形成另外一个 lock 文件, 像 `package.json.lock`. 但是 Maven 不会, 因为在 Maven 的一个依赖节点中, 会要求声明具体的版本.

其次, 因为和构建相关, 所以 pom.xml 中需要声明项目所属的组织、项目的名称、版本, 同时还要在 `properties` 节点中声明使用的 Java 版本、编码等信息, `build` 也是经常见到的一个节点, 但我不是很理解它的作用.

```
<!--项目信息-->  
<groupId\>域名/项目名</groupId\>  
<artifactId\>项目名/组件</artifactId\>  
<version\>版本</version\>  
​  
<properties\>  
 <!-- 可以声明编码 -->   
 <!-- 声明 jdk 版本-->  
 <maven.compiler.source\>1.8</maven.compiler.source\>  
 <maven.compiler.target\>1.8</maven.compiler.target\>  
 <!-- 可以自定义一些版本值 -->  
</properties\>  
​  
<!-- 一些构建相关的信息 -->  
<build\>  
 ……  
</build\>  
​  
<!-- 最重要的部分, 列出所依赖的包 -->  
<dependencies\>  
 <dependency\>  
 <groupId\>域名/项目名</groupId\>  
 <artifactId\>项目名/组件</artifactId\>  
 <version\>版本</version\>  
 </dependency\>  
 ……  
</dependencies\>
```


同时, 和其他包管理工具相比, Maven 不像 pip、npm、gem 一样可以使用 `install xxxx` 命令安装依赖, 而是需要自己向 `pom.xml` 中添加 `<dependency>……</dependency>` 节点来添加依赖, 然后执行 `mvn install` 命令才能安装.

项目名, 包名, 版本号往往琐碎而难以记忆, 所以常常借用现有的 `pom.xml` 文件; 或者像 `Spring initializer` 一样生成 `pom.xml` 文件; 同时也可以依赖搜索引擎, 比如说 [Maven Repository Search](https://mvnrepository.com/) 就是一个很好用的 Maven 搜索工具.

但在包管理之外, Maven 还可以进编译和打包, Maven 的使用大大简化了项目编译, 打包, 部署的工作量, 因为操作非常简单:

-   运行是 `mvn run`, 或者有一些特殊的项目可以定义特殊的启动方式, 如 `mvn spring-boot:run` 命令;
    
-   打包是 `mvn package` , 打包之后产生的 `fat jar package` 就可以通过 `java -jar xx.jar` 来运行;
    
-   `mvn clean` 命令可以删除掉项目中的 `target` 文件夹, 也就是清除掉所有的编译后的内容.
    

### JSP 项目

而对于传统的项目, 我以前并不太会使用 Maven 处理, 最近有需求之后有这些经验.

一个JSP 项目的 Maven 的典型目录是 `java`、`resources`、`webapp`. 在编译时, webapp 文件夹会成为根文件夹, `java` 文件夹中代码会被编译成 `.class` 文件放到 `webapp/WEB-INF/lib` 中, `resources` 文件夹和 `java` 文件夹去的地方一样, 但因为是配置文件, 所以不会被编译.

最最重要的一点, 就是 Maven 如何把这个项目识别成一个传统项目呢, 就是在项目信息下方添加 packaging 信息, 也就是 `<packaging>war</packaging>`, 这样, Maven 就会识别 `java`、`resources`、`webapp` 的结构了. 在拿到这样一个 xx.war 包之后, 把它拷贝到服务器 Tomcat 的 `webapps` 文件夹下面, 重启 Tomcat 就可以成功部署.

### Spring Boot 项目

我 Spring Boot 相关的项目经验并不多, 仅有的经验是 Spring Boot 项目会在根目录产生 `mvnw` 脚本, 通过该脚本, 可以使用 `mvn spring-boot:run` 命令运行项目.

因为 Spring Boot 项目往往具有共享的版本号, 所以可以添加 `<parent>...</parent>` 节点来一次性指定 Spring Boot 的版本, 之后可以省

```
<parent\>  
 <groupId\>org.springframework.boot</groupId\>  
 <artifactId\>spring-boot-starter-parent</artifactId\>  
 <version\>2.0.7.RELEASE</version\>  
</parent\>
````

略.

PS: 在使用 `mvn package` 时会要求数据库必须有连通性, 所以应当是本地打包完成后再在服务器部署使用.

PS: `mvn spring-boot:run` 命令为何可用, 是因为 添加以下插件带来的效果.

```
<build\>  
 <plugins\>  
 <plugin\>  
 <groupId\>org.springframework.boot</groupId\>  
 <artifactId\>spring-boot-maven-plugin</artifactId\>  
 </plugin\>  
 </plugins\>  
</build\>

```