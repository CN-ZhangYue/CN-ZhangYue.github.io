---
title: javaEE-framework——Struts2
date: 2019-05-06 10:40:48
description: 习惯了学习时记笔记，虽然不多，但是包含了Strust2的大部分内容，如果内容有错误的地方，欢迎大家批评指正
tags:
 - Struts2
 - javaEE framework
categories: javaEE framework
---
[TOC]

### 一、知识详解

#### 1、简介

##### 1）概念

运行web层，处理访问服务器的请求，代理Servlet

struct2与struct1区别：struct2前身是webwork框架

##### 2）优势

自动封装参数、参数校验、结果的处理(转发(重定向))、国际化、显式等待页面、表单防止重复提交

##### 3）搭建

- 导包
- 书写action类
- 书写src/structs.xml核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
	"-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
	"http://struts.apache.org/dtds/struts-2.3.dtd">
<struts>
	<package name="hello" namespace="/hello" extends="structs-default">
		<action name="HelloAction" class="cn_struct2.HelloAction" method="hello">
			<result name="success" >/hello.jsp</result>
		</action>
	</package>		
</struts>
```

- 将structs2核心过滤器配置到web.xml

```xml
<filter>
  	<filter-name>structs2</filter-name>
  	<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
  </filter>
  <filter-mapping>
  	<filter-name>structs2</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
```

##### 4）Struts构造原理图

<img src="D:\CN-ZhangYue.github.io\source\img\javaWeb\struts原理.JPG" height=550px>

##### 5）配置详解

###### struts.xml配置

```xml
<struts>
<!--package:将Action配置封装，可在package里配置多个action
name：包的名字（任意），起到标识作用，不能与其他包重复
namespace：给action的访问路径中定义一个命名空间
extends：必选属性，继承一个指定包，默认struts-default
adstract:包是否是抽象的，标识性属性，该包不能独立运行，只能被继承
 -->
	<package name="hello" namespace="/hello" extends="struts-default">
		<!-- action类：配置action类
		name：决定了Action访问资源 
		calss:action的完整类名
		method：指定调用Action中的哪个方法进行处理请求
		 -->
		<action name="HelloAction" class="cn_struct2.HelloAction" method="hello">
			<!-- result：结果配置
			name：标识结果处理的名称：与action方法的返回值对应
			type：指定定义哪个Result类来处理结果，默认使用转发
			标签体：填写页面的相对路径
			 -->
			<result name="success" >/hello.jsp</result>
		</action>
	</package>		
</struts>
```

###### struts2常量配置

**default.properties文件所在位置：**

​	Libraries

​		Web App Libraries

​			struts2-core-2.5.20.jar

​				static

​					default.properties

**常量的三种修改方式：**

方式先后也是文件加载顺序

- struts.xml中,在\<struts>根下，配置(常用)：

  ```
  <struts>
  	<constant name="" value="">
  	...
  </struts>	
  ```

- src下新建struts.properties，在文件里填写修改的键值对

- web.xml中，filter前进行配置

  ```
  <context-param>
  	<param-name> </param-name>
  	<param-value> </param-value>
  </context-param>	
  <filter>......	
  ```

**常量**

```xml
<!-- i18n：国际化，使用配置文件配置多个语言，可解决post提交乱码问题-->
<constant name="struts.i18n.encoding=UTF-8:" value="UTF-8">

<!--struts.action.extension:指定访问action的后缀名-->
<constant name="struts.action.extension" value="action,,">
<!--逗号标识隔开两个值-->

<!--指定struts是否以开放模式运行
	1、可热加载主配置（不需重启即可生效）
	2、提供更多错误信息输出，方便调试-->
<constant name="struts.devMode" value="true">
```

###### struts2配置进阶

**动态方法调用**

方式一：（了解）

- 调用方法时，方法前加 ‘ ！’

- 主配置文件，struts下配置：

```
<!-- 配置动态方法调用是否开启的常量，默认是false -->
<constant name="struts.enable.DynamicMethodInvocation" value="true"/>
```

- 次配置

```xml
<struts>	
	<package name="dynamic" namespace="/dynamic" extends="struts-default">
		<global-allowed-methods>add,update</global-allowed-methods>
		<action name="Demo1Action" class="cn_struct2.Demo1Action">
			<result name="success">/hello.jsp</result>
		</action>
	</package>	
</struts>
```

方式二：

次配置

```
<struts>	
	<package name="dynamic" namespace="/dynamic" extends="struts-default">
		<global-allowed-methods>add,update</global-allowed-methods>
		<action name="Demo1Action_*" class="cn_struct2.Demo1Action" method="{1}">
			<result name="success">/hello.jsp</result>
		</action>
	</package>	
</struts>
```

**默认配置**（了解）

```xml
<package name="dynamic" namespace="/dynamic" extends="struts-default">
		<!-- 找不到包下的action，会使用DemoAction作为默认action请求 -->
		<default-action-ref name="DemoAction"></default-action-ref>
		<!-- method:execute -->
		<!-- result的name属性：success -->
		<!-- result的type属性：dispatcher转发 -->	
		<!-- class属性：com.opensymphony.xwork2.ActionSupport -->
		<action name="Demo1Action">
			<result name="success">/hello.jsp</result>
		</action>	
</package>
```

##### 6）action类创建方式

action类：public,返回String，可抛异常

1、创建一个类，可以是POJO(普通类，不需要继承任何父类或实现接口)，使struts2代码侵入性更低

2、实现Action接口，

​	1）里面有execute方法，提供Action方法的规范

​	2）Action接口预置了一些字符串，可在返回时使用

3、继承一个类，ActionSupport（重点）

​	帮助实现了Action、Validateable、ValidationAwre、TextProvider、LocaleProvider

#### 2、结果跳转方式

##### 转发（默认 ）

##### 重定向

##### 转发到Action

```xml
<result name="success" type="chain">
	<param name="actionName" >DemoAction</param>
	<param name="namespace" >/</param>
</result>
```

##### 重定向到Action

```xml
<result name="success" type="rediretAction">
	<param name="actionName" >DemoAction</param>
	<param name="namespace" >/</param>
</result>
```

#### 3、访问ServletAPI方式

**ActionContext**:数据中心，可获得原生request(HttpServletRequest)、原生response(HttpServletResponse)、原生ServletContext、request域(Map)、session域(Map)、application域(Map)、params(Map)、attr域(Map,三个域合一)、ValueStack、、、

生命周期：每次请求都会创建一个与请求对应的ActionContext对象，请求处理完域销毁，与当前线程绑定

三种方式本质都是从ActionContext获得

##### 1）通过actionContext

```java
public String hello() {
		//request域(struts2不推荐用request域）
		//request域和ActionContext生命周期一致，可用ActionContext代替request域
		Map<String, Object> object = (Map<String, Object>)ActionContext.getContext().get("request");
		ActionContext.getContext().put("name", "value");
		
		//session域
		Map<String, Object> session = ActionContext.getContext().getSession();
		
		//application域
		Map<String, Object> application = ActionContext.getContext().getApplication();
		
		System.out.println("hello world");
		return "success";
	}
```

##### 2）通过ServletActionContext

不推荐

```java
	//获得actionAPI方式2
		public String hello2() {
			//原生request域
			HttpServletRequest request = ServletActionContext.getRequest();
			
			//原生response域
			HttpServletResponse response = ServletActionContext.getResponse();
			
			//原生session域
			HttpSession session =request.getSession();
			
			//原生servletContext域
			ServletContext servletContext = ServletActionContext.getServletContext();
			
			System.out.println("hello world");
			return "success";
		}
```

##### 3）实现接口Aware

```java 
Public class Demo entends ActioSUpport implments ServletRequestAware{
    private HttpServletRequest request;
    
    public String execute() throws Exception{
        System.out.println("原生Request");
        return SUCCESS;
    }
    
    @Override
    ppublic void setServletRequest(HttpServletRequest request){
        this.request = request;
    }
}
```

#### 4、获得参数

##### 1)扩展

- **Struts MVC**

Filter：Ctroller

Action:Module

Result:View

- Action生命周期：每次请求到来时，都会创建一个新的Action实例，不会产生并发现象，线程安全，可使用成员变量来接受参数

##### 2）属性驱动获得参数

Action准备与参数键同名属性

可自动转换八大基本类型及其包装类和Date(支持特定字符串类型,例：yyyy-MM-dd)

```java 
//获得参数
public class Demo8Action extends ActionSupport{
	//每次请求都会创建新的Action对象
	//准备与参数键名称相同的属性
	private String cust_name;
	
	public String execute() {
		System.out.println("name的参数值："+cust_name);
		return "success";
	}

	public String getCust_name() {
		return cust_name;
	}

	public void setCust_name(String cust_name) {
		this.cust_name = cust_name;
	}	
}
```



```xml
<form action="${pageContext.request.contextPath }/Demo8Action">
		用户名：<input type="text" name="Customer.cust_name"/><br>
		<input type="submit" value="提交"/> 
</form>
```

##### 3）对象驱动

Action准备与参数键同名属性    xxx.yyy

```java
public class Demo8Action extends ActionSupport{
	//准备customer属性
	Customer customer;
	
	public String execute() {
		System.out.println(customer.toString);
		return "success";
	}

	public Customer getCustomer() {
		return customer;
	}

	public void setCustomer(Customer customer) {
		this.customer = customer;
	}	
}
```



```xml
<form action="${pageContext.request.contextPath }/Demo8Action">
		用户名：<input type="text" name="customer.name"/><br>
		年龄：<input type="text" name="customer.age"/><br>
		<input type="submit" value="提交"/> 
</form>
```

##### 4）模型驱动

实现ModelDrivern接口，实现getModel方法，返回需要封装参数的对象

```java 
public class Demo8Action extends ActionSupport implements ModelDriven<Customer>{
	//准备customer对象
	private Customer customer = new Customer();
	
	public String execute() {
		System.out.println(customer.toString);
		return "success";
	}
	
	@Override
	public Customer getModel(){
        return customer
	}
}
```

```xml
<form action="${pageContext.request.contextPath }/Demo8Action">
		用户名：<input type="text" name="name"/><br>
		年龄：<input type="text" name="age"/><br>
		<input type="submit" value="提交"/> 
</form>
```

##### 5）集合类型封装获得参数：

```xml
<form action="${pageContext.request.contextPath }/Demo8Action">
		list：<input type="text" name="list"/><br>
		list：<input type="text" name="list[3]"/><br>
		map：<input type="text" name="map['haha']"/><br>
		<input type="submit" value="提交"/> 
</form>	
```

注：封装集合类型参数在前端可直接使用，使用时，map需先给出key

### 二、表达式

#### 1、OGNL表达式

OGNL:对象视图导航语言，例：$(user.addr.name)这种写法就成为对象视图导航

OGNL不仅仅可视图导航，还支持比EL表达式更加丰富的功能

##### 1）准备工作

OGNL从OGNLContext对象取值（复习：EL取值：11大内置对象）

OGNLContext对象分为ROOT和Context两部分：

​	ROOT：可放置任何对象

​	Context:	只存放Map

##### 2）基本语法

###### 取值

ROOT取值：expression直接写要取值的属性名

Context取值：expression写:#key.属性

```java 
public class demo1 {
	public void fun1() throws OgnlException {
		//1、准备OGNLContext
		//准备ROOT
		User rootUser = new User("Tom", 5);
		//准备Context
		Map<String ,User> context = new HashMap();
		context.put("user1",new User("lucy", 7));
		context.put("user2",new User("rose", 8));
		
		OgnlContext oc = new OgnlContext();
		oc.setRoot(rootUser);
		oc.setValues(context);
		
		//2、书写OGNL表达式
		String name = Ognl.getValue("name", context, oc.getRoot());
		String name = Ognl.getValue("#user1.name", context, oc.getRoot());
	}
}
```

###### 赋值

ROOT：属性=值

Context:#key.属性=值

```java 
//赋值
	String name = Ognl.getValue("name='jerry'", context, oc.getRoot());
	String name = Ognl.getValue("#user1.name='jerry'", context, oc.getRoot());
	//表达式可串联，但是如果表达式都有返回值，则取最后一个表达式的值
```

###### 调用方法

```java
Ognl.getValue("setName('jerry')", context, oc.getRoot());
String name = Ognl.getValue("getName()", context, oc.getRoot());

String name = Ognl.getValue("#user1.setName('jerry'),#user2.getName()", context, oc.getRoot());
String name = Ognl.getValue("#user1.getName()", context, oc.getRoot());

//静态方法调用
String name = （String）Ognl.getValue("@完整包名@方法名", context, oc.getRoot());
Double pi = (Double)Ognl.getValue("@java.lang.Math@PI", context, oc.getRoot());
//Math是OGNL对象的内置对象
(Double)Ognl.getValue("@@PI", context, oc.getRoot());
```

###### 创建List和Map对象

```java
//创建list对象
Ognl.getValue("{'hello','tom','penny'}", context, oc.getRoot());
String name1 = Ognl.getValue("{'hello','tom','penny'}[0]", context, oc.getRoot());
String name2 = Ognl.getValue("{'hello','tom','penny'}.get(1)", context, oc.getRoot());

//创建Map对象
Ognl.getValue("#{'name':'Tome','age':18}", context, oc.getRoot());
String name3 = Ognl.getValue("{'hello','tom','penny'}['name']", context, oc.getRoot());
int age = = Ognl.getValue("{'hello','tom','penny'}.get('age')", context, oc.getRoot());
```



#### 2、OGNL与struts2结合

OGNlContext-->ValueStack值栈

ROOT：栈，栈中放置当前访问的Action对象

Context：ActionContext（数据中心）

##### 结合体现：

###### 1）参数接收

struts2的参数交给OGNL引擎处理

<img src="D:\CN-ZhangYue.github.io\source\img\javaWeb\OGNL_Struts原理.JPG" height=200px>

<img src="D:\CN-ZhangYue.github.io\source\img\javaWeb\OGNL_Struts原理2.JPG" height=250px>

```java
//将参数压入栈
//方法一
//public class DemoAction extends ActionSupport implements Preparable{
//	private Customer u = new Customer();
//	public String execute() {

//		return SUCCESS;
//	}
//	@Override
//	public void prepare() throws Exception {
//		// TODO Auto-generated method stub
//		//压入栈顶
//		//1、获得栈值
//		ValueStack vs = ActionContext.getContext().getValueStack();
//				
//		//2、将u压人栈顶
//		vs.push(u);
//	}
//}

//方法二
public class DemoAction extends ActionSupport implements ModelDriven<Customer>{
	private Customer u = new Customer();
	public String execute() {
		// TODO Auto-generated method stub
		//压入栈顶
		//1、获得栈值
		ValueStack vs = ActionContext.getContext().getValueStack();				
		//2、将u压人栈顶
		vs.push(u);
		return SUCCESS;
	}
	@Override
	public Customer getModel() {
		// TODO Auto-generated method stub
		return u;
	}
}
```

###### 2）配置文件

${ognl表达式}

```xml
<result name="success" type="redirectAction">
	<param name="actionName" >DemoAction</param>
	<param name="namespace" >/</param>
	<!--如果提交的参数struts看不懂，就会作为参数附加在重定向的路径之后，如果参数是动态的，可以OGNL		表达式：
	  age=xx，${OGNL表达式}-->
	<param name="age" >${age}</param>
</result>
```

###### 3）struts2标签



#### 3、扩展:源码流程

- request.getAttribute()查找顺序：
  - 原生request域
  - ValueStack的栈（Root）
  - ValueStack的Context部分(ActionContext)

- 拦截器的调用：递归调用

  defaultActionInvocation调用interceptor.intercept()，interceptor调用defaultActionInvocation.invoke()

### 三、拦截器

准备工作：用户登录

```java
public class UserAction extends ActionSupport implements ModelDriven<Customer>{
	private Customer customer = new Customer();
	private UserService us = new UserService();
	
	public String login() {
		//调用service，执行登录操作
		Customer c = us.login(customer);
		//2、将返回的Customer对象放入session域作为登录标识
		ActionContext.getContext().getSession().put("customer",c);
		//3、重定向到项目的首页
		return "toHome";//重定向页面的name属性
	
	}
		
	@Override
	public Customer getModel() {
		// TODO Auto-generated method stub
		return null;
	}	
}
```



```java
public class UserService {
	public Customer login(Customer customer) {
		//Hibernate打开事务
	
		CustomerDao cd = new CustomerDao();
		//1、调用Dao根据登录名称查询的Customer对象
		Customer existCustomer = cd.getByCode(customer.getCust_name().hashCode());
		
		//提交事务
		
		//抛出异常，用户不存在
		if(existCustomer==null) {
			throw new RuntimeException("同户名不存在");
		}
		//2、比对密码是否一致（Id)
		if(!existCustomer.getCust_id().equals(customer.getCust_id())) {
			//不一致，抛出异常提示
			throw new RuntimeException("密码错误");
		}
		
		//将数据库查询到的Customer返回
		return existCustomer;
	}	
}
```

```xml
<package name="crm" namespace="/" extends="struts-default">
	<global-exception-mappings>
		<!--若出现该异常，跳转到结果为error的页面-->
		<exception-mapping result="error" exception"exception完整类名">
		</exception-mapping>
	</global-exception-mappings>
	
	<action name="Demo1Action_*" class="cn_struct2.Demo1Action" method="{1}">
			<result name="success">/hello.jsp</result>
			<result name="error">/login.jsp</result>
	</action>
</package>
```

```
<!--密码错误在新页面提示-->
<!--跳转到的页面使用debug标签-->
<%@ taglib prefix="s" uri="/struts-tags" %>

<s:debug></s:debug>
```

##### 自定义拦截器

拦截器生命周期：随项目的启动创建，随项目的关闭而销毁

###### 1）拦截器的创建

方式一：实现接口Interceptor

方式二：继承AbstractInterceptor(该继承抽象类空实现了Interceptor)

方式三：继承MethodFilterInterceptor，功能：定制拦截器拦截的方法,可定制需要拦截或不需要拦截的方法

```java
//拦截器：创建方式一
public class MyInterceptor implements Interceptor{}
//方法二：
public class MyInterceptor_2 extends AbstractInterceptor{}
//方式二：
//功能：定制拦截器拦截的方法
//实现方法doIntercept();
public class MyInterceptor_2 extends MethodFilterInterceptor{
    @Override
	protected String doIntercept(ActionInvocation arg0) throws Exception {
		//前处理
		
		//放行
		invocation.invoke();
		
		//后处理
		return null;
	}
}
```

###### 2）拦截器配置

###### 3)拦截方法指定

不拦截和需要拦截的方法不能同时指定

```xml
<package>
	<interceptors>
		<!--注册拦截器-->
		<interceptor name="myInter" class="完整类名"></interceptor>
		<!--注册拦截器栈-->
		<interceptor-stack name="myStack">
			<!--自定义拦截器默认 放在20个拦截器之前-->
			<interceptor-ref name="myInter">
            	<param name="excludeMythods">add,delete</param>                               </interceptor-ref>
            	<param name="includeMythods">add,delete</param>                               </interceptor-ref>
			<interceptor-ref name="defaultStack"></interceptor-ref>
		</interceptor-stack>
	</interceptors>
	<!--指定默认拦截器栈,作用范围是整个包-->
	<default-interceptor-ref name="myStack"></default-interceptor-ref>
	<action name="myInter" class="">
		<!--或为action单独指定哪个拦截器-->
		<default-interceptor-ref name="myStack"></default-interceptor-ref>
		<result name="error">/index.jsp</result>
	</action>
</package>
```



**定义全局结果集**

```
<global-results></global-results>
```



### 四、标签

（了解）

struts标签分类：

- 普通标签：
  - 控制标签(iterater、if、elseif、else)
  - 数据标签(property)

- ul标签：
  - 表单标签(form、textfield、password、file、checkboxlist、redio...)
  - 非表单标签(Actionerror)

##### 普通标签

```jsp
<!--普通标签-->
<body>
	<!-- 遍历标签 iterator -->
	<s:interator value="#list">
		<s:property/>
	</s:interator>
	<!-- 遍历方式二 -->
	<s:interator value="#list" var="name">
		<s:property value="#name"/>
	</s:interator>
	
	<!-- if标签 -->
	<s:if test="#list.size()==4">
		list长度为4
	</s:if>
	<s:esle>
		list长度不为4
	</s:esle>
	
	<!-- property标签 ,配合ognl表达式页面取值使用-->
	<s:property value="#value.size()"/>
	<s:property value="#session.use.name"/>		
</body>
```

##### 表单标签

```jsp
<!-- struts表单标签 -->
	<!-- 好处：内置了一套样式；自动回显，根据栈中的属性 -->
	<!-- theme属性，指定表单主题，默认xml主题 -->
	<s:form action="DemoAction" namespace="/" theme="simple">
		<!-- 表单提交，提示：用户名 -->
		<s:textfield name="name"lable="用户名"></s:textfield>
		<s:password name="password" lable="密码"></s:password>
		<!-- 单选 -->
		<s:radio list="{'男',''女}"  name="gender" value="性别"></s:radio>
		<s:radio list="#{1:'男',0:'女'}"  name="gender" value="性别"></s:radio>
		<!-- 多选 -->
		<s:checkboxlist list="#{1:'游泳',0:'打球',2:'跑步'}" name="hobby" lable="爱好"></s:checkboxlist>
		<!-- 下拉选 -->
		<s:select list="#{2:'大专',1:'本科',0:'硕士' }" headerKey="" headerValue="--请选择--" name="edu" lable="学历"></s:select>
		<!-- 文件上传 -->
		<s:file name="photo" lable="近照"></s:file>
		<!-- 文本域 -->
		<s:textarea name="desc" lable="个人简介"></s:textarea>
		<s:submit value="提交"></s:submit>
	</s:form>
```

##### 非表单标签

```jsp
//action中的方法加入错误提示信息
this.addActionError("你错了")!

<!--使错误信息在页面显示-->
<s:actionerror/>
```

















































































