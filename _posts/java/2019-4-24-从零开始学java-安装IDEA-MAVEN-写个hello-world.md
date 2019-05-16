---
layout: post
title: 从零开始学java-安装IDEA-MAVEN-写个hello-world
categories: java
tag: 环境安装
---

# 说明
从零开始学java 如题
- IntelliJ IDEA  Java集成开发环境
- Maven 
	- 基于项目对象模型(POM project object model)，可以通过一小段描述信息（配置）来管理项目的构建，报告和文档的软件项目管理工具
	- 粗浅的理解就是个依赖包管理器，中央仓库下载jar包
	- 方便构建

# 安装
### 安装jdk

https://www.oracle.com/technetwork/java/javase/downloads/jdk12-downloads-5295953.html

- 选择系统对应类型 下载安装即可（本人win10）

- 设置环境变量

	系统变量 `JAVA_HOME` 设置jdk 安装路径 `C:\Program Files\Java\jdk1.8.0_201 `
	
	`Path` 变量 添加 `%JAVA_HOME%\bin` 和 `%JAVA_HOME%\jre\bin`

### 安装IntelliJ IDEA
https://www.jetbrains.com/idea/download/#section=windows
下载安装

### Maven
http://maven.apache.org/download.cgi

- 选对应文件下载 ，解压到一个目录（自选）
- 设置环境变量 
	
	系统变量 `MAVEN_HOME` 添加 `D:\javaStudy\apache-maven-3.6.1`

	`Path` 变量添加 `%MAVEN_HOME%\bin`

### IDEA创建个hellow world项目 

![](/styles/images/2019-4/newMavenProject.png)
![](/styles/images/2019-4/newMavenProject1.png)
![](/styles/images/2019-4/newMavenProject2.png)
![](/styles/images/2019-4/newMavenProject5.png)
```
import java.lang.System;
public class hello {
    public static void main(String[] args){
        System.out.println("hello world");
    }
}
```
![](/styles/images/2019-4/newMavenProject3.png)
![](/styles/images/2019-4/newMavenProject4.png)

