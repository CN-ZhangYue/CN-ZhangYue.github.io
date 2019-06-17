---
title: maven
date: 2019-06-17 20:05:48
description: 软件构造课上学习了maven，也是JavaEE框架中的一部分，使用起来确实非常爽~
mathjax: true
tags:
 - 软件构造
 - javaEE框架
categories: 软件构造

---

#### 一、Maven简介

apache下的开源项目，纯java开发，只用来管理java项目

- 本地仓库索引

- 坐标：(struts2-core-2.3.24.jar)哪个公司或组织哪个项目哪个版本：

- 项目一键构建：使用命令tomcat:run运行项目（编译、测试、运行、打包、部署）

- 好处：

  - 依赖管理：对jar包的统一管理，可节省空间

  - 一键构建
  - 跨平台
  - 主要应用于大型项目，可以提高开发效率

  - 分块开发：互联网项目：按业务分，传统项目：按三层分

#### 二、安装配置

环境变量配置：Maven3.3版本及以上，都需要1.7以上的版本

本地仓库、远程仓库、中央仓库

##### 目录结构：

src

​	main

​		java：java代码

​		resources：配置文件

​	test

​		java：java代码

​		resources：Junit测试需要的配置文件，如果没有配置文件，默认从main里找

pom.xml

##### 常用命令

Clean 清理编译好的target文件

Compile 编译，只编译主目录文件

Test 编译Test文件

Package 打包

insatll 将项目发布到本地仓库

Tomcat:run  一键启动

deploy 发布到私服

Site 生成项目的站点文件

#####  Maven的生命周期

- 三种生命周期：

  - Clean生命周期：

    pre-clean：执行一些需要在clean之前完成的工作

    clean：移除所有上一次构建生成的文件

    post-clean：执行一些需要在clean之后立刻完成的工作

  - Default生命周期：Compile test package install deploy

  - Site生命周期：Site

- 命令和生命周期的阶段的关系：

  不同的生命周期的命令可以同时执行：mvn clean package

##### 添加依赖

**网络搜索**：

​	http://mvnrespository.com/

​	http://search.maven.org/

**在本地重建索引，以索引的方式搜索**

#### 三、项目构建

- 在webapp文件夹下创建WEB-INF文件夹，并在该文件夹下创建web.xml

- 处理编译版本

```xml
<build>
		<!-- 配置了很多插件 -->
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.5.1</version>  
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<!-- <plugin> -->
			<!-- <groupId>org.codehaus.mojo</groupId> -->
			<!--<artifactId>tomcat-maven-plugin</artifactId> -->
			<!-- <version>1.1</version> -->
			<!-- <configuration> -->
			<!-- <port>8888</port> -->
			<!-- <path>/first</path> -->
			<!-- </configuration> -->
			<!-- </plugin> -->
			<plugin>
				<groupId>org.apache.tomcat.maven</groupId>
				<artifactId>tomcat7-maven-plugin</artifactId>
				<version>2.2</version>
				<configuration>
				 <path>/f</path>
				</configuration>
			</plugin>
		</plugins>
	</build>
```

- 添加jar包：

```xml
<dependencies>
  	<dependency>
  		<groupId>javax.servlet</groupId>
  		<artifactId>jsp-api</artifactId>
  		<version>2.0</version>
  		<scope>provided</scope>
  	</dependency>
  	<dependency>
  		<groupId>javax.servlet</groupId>
  		<artifactId>servlet-api</artifactId>
  		<version>2.5</version>
  		<scope>provided</scope>
  	</dependency>
  	<dependency>
  		<groupId>junit</groupId>
  		<artifactId>junit</artifactId>
  		<version>4.9</version>
  		<scope>test</scope>
  	</dependency>
  	<dependency>
  		<groupId>org.apache.struts</groupId>
  		<artifactId>struts2-core</artifactId>
  		<version>2.3.24</version>
  	</dependency>
  </dependencies>
```



#### 四、依赖管理

##### 依赖范围：

Compile(默认)：编译时需要，测试时需要，运行时需要，打包需要(eg:struts2-core)

Provided：编译时需要，测试时也需要，运行时不需要，打包时不需要(eg:Servlet,jsp)

Runtime：编译时不需要，测试时需要，运行时需要，打包需要(eg:数据库驱动包)

Test:编译时不需要，测试时需要运行时不需要(eg:Junit.jar)

#### 五、整合ssh框架

##### 依赖版本冲突的解决

- 调节原则

  - 路径近者优先

  - 第一声明者优先原则

- 排除原则

- 版本锁定

```xml
<!--全局版本锁定-->
<properties>
  	<!-- 自定义标签 -->
  	<spring.version>4.2.4RELEASE</spring.version>
  </properties>
  
  <dependencyManagament>
  	<dependencies>
  		<denpendy>
  			<groupId>org.springframework</groupId>
  			<artifactId>spring-beans</artifactId>
  			<version>${speing.version}</version>
  		</denpendy>
  	</dependencies>
  </dependencyManagament>
```



```xml
<!--Servlet中排除spring-beans包-->
<dependency>
  	<groupId>javax.servlet</groupId>
  	<artifactId>jsp-api</artifactId>
  	<version>2.0</version>
  	<scope>provided</scope>
  	<exclusions>
  		<exclusion>
  			<groupId>org.springframework</groupId>
  			<artifactId>spring-beans</artifactId>
  		</exclusion>
  	</exclusions>
</dependency>
```



#### 六、分模块开发

##### 分层

entity、Web、Dao、Service

##### 依赖范围对传递造成的影响

---了解----

依赖会有依赖范围，依赖范围对传递依赖也有影响

| 直接\传递依赖 | compile  | provided | runtime  | test |
| ------------- | -------- | -------- | -------- | ---- |
| compile       | compile  | -        | runtime  | -    |
| provided      | provided | provided | provided | -    |
| runtime       | runtime  | -        | runtime  | -    |
| test          | test     | -        | test     | -    |



#### 七、私服

nexus安装

- 本地访问路径：localhost：8081/nexus

- 启动失败：打开nexus的bin/jvm/conf文件，将wrapper.java,command修改为java.exe所在的全路径(javaJDK)

- 登录：用户名：admin          密码：admin123

- 仓库类型：
  - virtual:虚拟仓库，不起作用
  - Proxy：代理Apache Snapshots和Central,即非正式jar包和中央仓库
  - Hosted:宿主仓库/本地仓库：3rd party、Releases、Snapshots
  - group:连接不知道类型的文件

项目的pom.xml配置私服

```xml
<distributionManagement>
  	<repository>
  		<id>releases</id>
  		<url>http://localhost:8081/nexus/content/repositories/releases</url>
  	</repository>
  	<snapshotRepository>
  		<id>snapshots</id>
  		<url>http://localhost:8081/nexus/content/repositories/snapshots</url>
  	</snapshotRepository>
  </distributionManagement>
```

settings.xml配置：

配置用户名和密码

```xml
<server>
      <id>releases</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    <server>
      <id>snapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
```

配置私服的访问方式及路径

```xml
<profile>
      <id>env-dev</id>

      <activation>
        <property>
          <name>target-env</name>
          <value>dev</value>
        </property>
      </activation>

      <properties>
        <tomcatPath>/path/to/tomcat/instance</tomcatPath>
      </properties>
    </profile>
    -->
  </profiles>
```

活化profile


```xml
<activeProfiles>
    <activeProfile>alwaysActiveProfile</activeProfile>
    <activeProfile>anotherAlwaysActiveProfile</activeProfile>
  </activeProfiles>
```

