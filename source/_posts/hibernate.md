---
title: javaEE-framework开篇——hibernate
date: 2019-04-18 14:05:48
description: hibernate是我学习JavaEE framework的开始，分享学习的笔记给大家~
tags:
 - hibernate
 - javaEE framework
categories: javaEE framework
---

​	hibernate是我学习JavaEE framework的开始，分享学习的笔记给大家~

#### 搭建

1）导包

2）创建数据库

3）书写Orm元数据(对象与表的映射配置文件)

导入约束、实体、orm元数据

4）书写主配置文件

hibernate.cfg.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
	"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
	
<hibernate-configuration>
	<session-factory>
		<property name="hibernate.connection.dirver_class">com.mysql.cj.jdbc.Driver</property>
		<property name="hibernate.connection.url">jdbc:mysql:///hibernate?useSSL=false&amp;serverTimezone=GMT%2b8&amp;allowPublicKeyRetrieval=true</property>
		<property name="hibernate.connection.username">root</property>
		<property name="hibernate.connection.password">yue18800422369</property>
		<property name="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</property>
		
		<property name="hibernate.show_sql">true</property>
		<property name="hibernate.format_sql">true</property>
		
		<property name="hibernate.hbm2ddl.auto">update</property>
		
		<mapping resource="domain/Customer.hbm.xml"/>
	</session-factory>
</hibernate-configuration>
```

5）书写代码测试

映射文件:Customer.hbm.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<!-- 配置表与实体对象的关系 -->
<hibernate-mapping>
	<class name="domain.Customer" table="cst_customer">
		<id name="cust_id" column="cust_id">
			<generator class="native"></generator>
		</id>
		<property name="cust_name" column="cust_name"></property>
		<property name="cust_source" column="cust_source"></property>
		<property name="cust_industry" column="cust_industry"></property>
		<property name="cust_level" column="cust_level"></property>
		<property name="cust_linkman" column="cust_linkman"></property>
		<property name="cust_phone" column="cust_phone"></property>
		<property name="cust_mobile" column="cust_mobile"></property>
	
	</class>
</hibernate-mapping>
```



#### hibernateAPI详解

##### configuration

读取配置文件：主配置和orm元数据

##### SessionFactory

用于创建操作数据库核心对象session对象的工厂
注：1、sessionFactory负责保存和使用所有配置信息，消耗内存资源非常大
​	2、sessionFactory属于线程安全的对象设计
结论：保证在web项目中，只创建一个sessionFactory

##### Session

表达hibernate框架与数据库之间的连接（会话），类似于JDBC的connection对象，可实现数据库的增删改查

##### Transaction

封装了事务的操作：打开事务、提交事务、回滚事务

```java
//1、创建
		Configuration conf = new Configuration().configure();
		//2、读取主配置文件,无参读取：hibernate.cfg.xml
		conf.configure();
		
//		//读取指定orm元数据(old)
//		conf.addResource(resourceName);
//		conf.addClass(persistentClass);
		//4、根据配置信息，创建sessionFactory对象
		SessionFactory sessionFactory = conf.buildSessionFactory();
	
		//5、获得session
		Session session = sessionFactory.openSession();
		//打开一个新的session对象
		//sessionFactory.openSession();
		//获得一个与线程绑定的session对象
		//sessionFactory.getCurrentSession();
		
		//6、session获得操作事务的Transaction对象
		//获得操作事务的tx对象
		//Transaction tx = session.getTransaction();
		//tx.begin();
		//开启事务并获得操作事务的tx对象（建议使用）
		Transaction tx2 = session.beginTransaction();
		
		//...................
		//增
		Customer customer = new Customer();
		customer.setCust_name("best");
		session.save(customer);	
		//查询
		Customer customer2 = session.get(Customer.class, 1l);	
		//修改
		Customer customer3  = session.get(Customer.class,1L);
		customer3.setCust_name("angle");
		session.update(customer3);
		//删除
		Customer customer4 = session.get(Customer.class, 1l);
		session.delete(customer4);
		//...............
		
		tx2.commit();
		session.close();
		sessionFactory.close();
		
```

#### 实体规则

##### 创建规则

- 持久化类提供无参构造(通过反射创建对象)

- 成员变量私有，需提供get/set（属性）

- 持久化类中的属性，应尽量使用包装类型

- 持久化类需要oid与数据库中的主键列对应
- 不要用final修饰class（hibernate使用cglib代理生成代理对象，代理对象继承被代理对象，如果被final修饰，将无法代理）

##### 主键类型

###### 自然主键

表的业务列中，其中必须有且不可重复的列可作为主键使用

###### 代理主键

表的业务列中，没有可作为主键的列，则创建一个没有业务的列作为主键

##### 主键生成策略

每条记录生成时，主键的生成规则（7个）

- identity：主键自增，由数据库来维护主键值，录入时不需指定主键
- sequence：Oracle中的主键生成策略
- increment（了解）：主键自增，由hibernate来维护，每次插入前，会先查询表中id最大值，然后加1作为主键值（线程不安全）

- hilo（了解）：高低位算法，由hibernate维护
- native：hilo+sequence+identity，自动三选一
- uuid： 产生随机字符串作为主串
- assigned：自然主键生成，由用户录入生成

#### hibernate的对象状态

对象分为三种状态：

- 瞬时状态：没有id，没有与session关联

- 持久化状态：有id，与session关联

  持久化状态的对象的任何变化都会自动同步到数据库中，不需要执行update方法

- 游离/托管状态：有id，没有与session关联

注：有id指的是有和数据库中对应的id

状态转化：

起点$\rightarrow$瞬时（new）

起点$\rightarrow$持久化（get）

瞬时 $\rightarrow $持久化(save) $\rightarrow $游离/托管状态(close)

瞬时 $\leftarrow $持久化(delete) $\leftarrow $游离/托管状态(update)

saveOrUpdate:将所有对象转化为持久化状态

（持久化对象就是放入session缓存的对象）

#### hibernate的进阶--缓存

提高效率的方式：

- 提高查询效率（缓冲区）
- 减少不必要的修改语句发送（利用快照）

#### 事务

- 性质：

  - 原子性

  - 一致性：数据在提交前和提交后总量保持一致

  - 隔离性

  - 持久性：数据提交之后必须要被持久化保存

- 并发问题：

  - 脏读：读取到了正在操作并未保存的数据；B事务读取了A事务没有提交的数据

  - 不可重复读：在一个事务中，两次读取的事务不一致

  - 幻/虚读：一般针对整表的操作；一个事务中，两次读取的数据的数量不一致

- 事务的隔离级别：

  - 读未提交 1*：脏读、不可重复读、幻读（可产生问题）

  - 读已提交 2*：不可重复读、幻读

  - 可重复读 4*（MySQL默认级别）：幻读

  - 串行化 8*

- 在hibernate指定隔离级别：1 2 4 8

- 在项目中管理项目

  dao层数据操作和service层控制事务都需要session对象，要确保使用的是同一个session对象：

  1）在hibernate中调用getCurrentSession方法获得与线程绑定的session对象，调用该方法时，必须在主配置中主配置中说明：

  ```xml
  <property name="hibernate.current_session_context_class">thread</property>
  ```

  2）通过getCurrentSession方法获得session对象，当事务提交时session会自动关闭，不许手动。

#### 批量查询

##### HQL查询

多表查询，但不复杂时使用

hibernate独家查询语言，面向对象

```javascript
//1、书写HQL语句
		//String hql = "from 对象的完整类名";
		//String hql = "from Customer where cust_id = 1";//cust_id是属性
		//String hql = "from Customer";//查询所有Customer对象
		//String hql = "from Customer where cust_id = ?";//问号占位符
		String hql = "from Customer where cust_id ：id";
		
		//2、根据HQL语句创建查询对象
		Query<Customer> query = session.createQuery(hql);
		
		//分页查询
		//设置分页信息
		query.setFirstResult(0);//第0条开始查询
		query.setMaxResults(2);//最多查询两条
		
		//设置参数(占位符）
		query.setLong(0, 1);//问号占位符
		query.setParameter(0, 1);//类型通用
		query.setParameter("id", 1);//命名占位符
		//3、根据查询对象获得查询结果
		List list = (List) query.list();
		//query.uniqueResult();//接收唯一的查询对象
```

##### Criteria查询

单表查询

hibernate自创的无语句面向对象查询

```
	//Criteria查询
	//查询所有Customer对象
	Criteria criteria = session.createCriteria(Customer.class);
	//普通查询
	List<Customer> list2  = criteria.list();
	//条件查询
	criteria.add(Restrictions.eq("cust_id",1));
	//criteria.add(Restrictions.IdEq(1));
	Customer c = (Customer) criteria.uniqueResult();
	//分页
	criteria.setFirstResult(1);
	criteria.setMaxResults(1);
	//查询总记录
	//设置查询的聚合函数=》总行数
	criteria.setProjection(Projections.rowCount());
	Long count = (Long) criteria.uniqueResult();
```

常见查询条件:

| >    | gt   | between and | between   |
| ---- | ---- | ----------- | --------- |
| >=   | lt   | like        | like      |
| <    | lt   | is not null | isNotNull |
| <=   | le   | is null     | isNull    |
| ==   | eq   | or          | or        |
| !=   | in   | and         | and       |

##### 原生SQL查询

复杂的业务查询

```javascript
	//原生SQL查询
		//String sql = "select * from cst_customer";
		//条件查询与分页
		String sqlString  = "select * from csy_customer limit ?,?";
		
		SQLQuery<Object[]> query2 = session.createSQLQuery(sql);
		
		query2.setParameter(0, 0);
		query2.setParameter(1, 1);
		
		//查询到的每行结果放到一个object数组
		//List<Object[]> list3 = query2.list();
		
		//封装结果
		query2.addEntity(Customer.class);
		List<Customer> list3 = query2.list();
```

#### 多对多关系

##### 一对多、多对一

###### 关系的表达

```
<!-- 集合，一对多关系的配置 -->
		<!--
		 name:集合属性名
		 column:外键列名
		 class：关联的对象的完整类名
		 -->
		<set name="LinkMen" cascade：save-update>
			<key column="lkm_cust_id"></key>
			<one-to-many class="LinkMan"/>
		</set>
		
		<!-- 多对一的配置 -->
		<many-to-one name="customer" column="lkm_cust_id" class="Customer"></many-to-one>
		
```

###### 操作

```
	//保存客户以及客户下的联系人
		Customer customer = new Customer();
		customer.setCust_name("blog");
		
		LinkMan lMan1 = new LinkMan();
		lMan1.setLkm_name("man1");
		LinkMan lMan2 = new LinkMan();
		lMan2.setLkm_name("man2");
		LinkMan lMan3 = new LinkMan();
		lMan3.setLkm_name("man3");
		//表达一对多，一个客户有多个联系人
		customer.getLinkmen().add(lMan1);
		customer.getLinkmen().add(lMan2);
		//表达多对一，联系人属于哪个客户
		lMan1.setCustomer(customer);
		lMan2.setCustomer(customer);
		
		session.save(customer);
		session.save(lMan1);
		session.save(lMan2);
```

###### 进阶

**联操作**

cascade（配置文件配置）

​	save-update:级联保存更新

​	delete：级联删除

​	all：save-update+delete

**关系维护**

在保存时，两方都会维护关系，冗余

inverse属性：配置关系是否维护，默认是false(维护），但是多的一方不能放弃维护

##### 多对多

配置

```
<set name="set集合名" table="中间表名">

	<key column="对方引用的自己的外键名>"

	<many-to-many class="与哪个别的类多对多" column="引用的别人的外键">

</set>
```

注：开发中利用inverse属性，根据业务方向选择一方放弃维护关系

**小结**

**一对多/多对一**

- O：对象  

    一的一方使用集合，多的一方直接引用一的一方

- R：关系型数据库      

  多的一方使用外键引用一的一方主键

- M：映射文件  

  ```
  一：<set name="">
  
  		<key column = ""/>
  
  		<one-to-many name="" column="" class=""/>
  
  	</set>
  
  多：<many-to-one name="" column="" class=""/>
  ```

  关系操作：级别属性

  进阶：

  ​	inverse：反转关系维护,性能优化（true：放弃，默认false维护），一对多中，一的一方可放弃维护关系

  ​	casecade：级联操作，减少代码（nono(默认)：不级联save-update、delete、all)

**多对多**

- O：对象

  两方都使用集合

- R：使用中间表，至少两列，作为外键引用两张表的主键

- M：映射文件

```
多：
<set name="" table="中间表名">
	<key column="别人引用的自己的外键">
	<many-to-many class="" column="引用别人的外键名"/>
</set>	
```

操作：操作管理级别属性

- casede:级联属性，减少代码书写

​	nono(默认)：不级联

​	save-update、delete、all

- inverse:反转关系维护，属于性能优化，必须根据业务关系选择一方放弃维护主键关系

##### 检索

###### 查询小结

oid查询-get

对象属性导航查询

HQL

Criteria

原生SQL

###### HQL查询

**基本语法**

```
String hql = “from java.lang.Object"//查询所有Object及其子类
```

**排序语法**

```
String hql = "from Customer order by id asc"//asc：升序

String hql = "from Customer order by id desc，xxxx  desc/asc

"//desc：降序,如果相同，可根据别的列按照desc/asc再排序
```

**条件查询**

```
String hql = "from Customer where id =？"

String hql = "from Customer where id: id"
```

**分页查询**

```
String hql = "from con.itcast.domain.Customer";

query query = session.creatQuery(hql);

//limit ?， ?

query.setFirstResult(0);//从第0条开始查询

query.setFirstResult(2);//每页查询两条
```

**统计查询**

聚合函数：count（计数）、sum(求和）、avg（平均数）、max、min

```
String hql = "select count(*) from Customer"//总记录数 
//String hq2 = "select sum(id) from Customer"
//String hql = "select avg(id) from Customer"
//String hql = "select max(id) from Customer"
//String hql = "select min(id) from Customer"
Query query = session.creatQuery(hql);

Number number = (Number) query.uniqueResult();
```

**投影查询**

```
String hql = "select cust_name from Customer"//查询name
String hql = "select cust_name,cust_id from Customer";//查询name和id两列，返回数组
String hql = "select newCustomer(cust_name,cust_id) from Customer";//将查询的每个name和id封装到对应Customer对象中
//注：必须在Customerclass类中添加参数为name和id的构造方法以及无参构造方法
```

**多表查询**

- 原生SQL：

  交叉连接：笛卡尔积selecte * from A,B

  内连接：

  ​	隐式内连接：selecte * from A,B where 条件

  ​	显式外连接：selecte * from A inner join B on  条件

  外连接：

  ​	左外连接：selecte * from A left [outer] join B on 条件

  ​	右外连接:selecte * from A right [outer] join B on 条件

- HQL查询：

  - 内连接：

  ​	String hql = "from Customer c inner join c.LinkMen";//将对应的两个对象分别封装到数组

  ​	迫切：String hql = "from Customer c inner join fetch c.LinkMen";//把第二个表的东西封装到第一个对象中

  - 左外连接：String hql = "from Customer c left join c.LinkMen"

  - 右外连接：String hql = "from Customer c right join c.LinkMen"

###### Criterla查询（QBC）

**基本**

```
//Criteria查询
	//查询所有Customer对象
	Criteria criteria = session.createCriteria(Customer.class);
	
	//普通查询
	List<Customer> list2  = criteria.list();
	
	//条件查询
	criteria.add(Restrictions.eq("cust_id",1));
	//criteria.add(Restrictions.IdEq(1));
	Customer c = (Customer) criteria.uniqueResult();
	
	//分页
	criteria.setFirstResult(1);
	criteria.setMaxResults(1);
	
	//查询总记录
	//设置查询的聚合函数=》总行数
	criteria.setProjection(Projections.rowCount());
	Long count = (Long) criteria.uniqueResult();
```

**条件**

```
Criteria c = session.creatCriteria(Customer.class);
criteria.add(Restrictions.eq("cust_id",1));
//criteria.add(Restrictions.IdEq(1));
Customer c = (Customer) criteria.uniqueResult();
```

**分页**

```
Criteria c = session.creatCriteria(Customer.class);
c.setFirstResult(0);
c.setMaxResult(2);
List<Customer> list = c.list;
```

**排序**

```
Criteria c = session.creatCriteria(Customer.class);
c.addOrder(Order.asc("cust_id"));
List<Customer> list = c.list;
```

**统计**

```
Criteria c = session.creatCriteria(Customer.class);
c.setProject(Projections.rowCount());
List<Long> list = c.list;
```

**离线查询**

传统的criteria依赖于session创建（dao层）

离线的criteria，可脱离session，在任意层均可凭空创建（扩大session的作用范围）

```
//web或service层
DetachedCriteria dc = DetachedCriteria.forClass(Customer.class);
dc.add(Restrictions.idEq(61));//拼装条件与普通一致


//dao层
Criteria c = dc.getExecutableCriteria(session)
List list = c.list();
```

##### 查询优化

###### 类级别查询

懒加载|延迟加载

get:立即加载，执行方法时立即发送SQL语句查询结果

load(默认)：结果和get一样，执行时，不发送任何SQL语句，返回一个对象，使用该对象时，才执行查询

延时加载：仅仅获得不使用，不会查询，使用时才进行查询，（针对load方法），原理是动态代理
是否对类进行延迟加载，可通过在class元素上配置lazy属性控制：
​	lazy=true：加载时，不查询，使用时才查询
​	lazy=false：加载时立刻查询

###### 关联级别查询

**集合级别的关联**

lazy属性:决定是否延迟加载：

​	true(默认):延迟加载，懒加载；

​	false：立即加载

​	extra：及其懒惰（集合级别）

fetch属性：决定加载策略，使用什么类型的SQL语句加载集合数据

​	select(默认)：单表查询加载

​	join：使用多表查询加载集合

​	subselect:使用子查询加载集合

```
<set name="" lazy="" fetch="">
	<key volumn="lkm_cust_id></key>
	<one_to_many class=""/>
</set>
```

- fetch:select（单表查询） , lazy=true(使用时才加载集合数据)，该组合为默认
- fetch:select（/单表查询） , lazy=false(立即加载集合数据）

- fetch:select（/单表查询） , lazy=extra(与懒加载效果基本一致，但如果只获得集合的size，extra只获得集合的size：‘count语句’）

- fetch:join , lazy=true/false/extra：多表查询立即加载集合，lazy属性失效

- fetch:subselect，lazy=true:如果用不到子查询，效果相当于select，先单表查询，集合使用时才立即查询加载
- fetch:subselect，lazy=false:立即查询加载相关集合数据

- fetch:subselect，lazy=extra:先单表查询，集合使用时才立即查询加载，如果只获得集合的size，extra只获得集合的size：‘count语句’

**属性级别的关联**

根据被关联的对象获得主对象

```
Linkman lm = session.get(Linkman.class,2);
Customer c = lm.getCustomer();
System.out.println(customer);
```

配置中属性：

```
<many-to-one name="Customer" column="" class="" fetch="" lazy=">
</many-to-one>
```

fetch属性：决定加载策略，使用什么类型的SQL语句加载数据

​	select(默认)：单表查询加载

​	join：使用多表查询加载集合

lazy属性:决定是否延迟加载：

​	false：立即加载

​	proxy：由主类的类级别策略决定

- fetch:select,lazy=proxy(true),单表查询，懒加载
- fetch:select,lazy=proxy(false，extra),多表立即加载
- fetch:join,lazy属性失效,

**结论**：为了提高效率，fetch的选择应选择select，lazy应选择true（默认）

##### 批量抓取

每次抓取几个集合：

```
<--每次抓取3个客户的集合-->
<set name="" batch-size="3">
	<key volumn="lkm_cust_id></key>
	<one_to_many class=""/>
</set>
```

































































































