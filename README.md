# SSM
记录学习ssm时遇到的问题

---------
# TOC
[执行过程图](#执行过程图)<br>
[SSM搭建](#ssm搭建)<br>
[spring的配置](#spring的配置)<br>
[绝对路径](#绝对路径)<br>
[Maven 添加JSON 显示错误](#maven-添加json-显示错误)<br>
[Ajax的执行顺序](#ajax的执行顺序)<br>
[ajax动态实现下拉框从-数据库获取](#ajax动态实现下拉框-从数据库获取)<br>
[ajax动态实现下拉框-从数据库获取方法2](#ajax动态实现下拉框-从数据库获取方法2)<br>
[在SSM的controller中获表单中文乱码](#在ssm的controller中获表单中文乱码)<br>
[SpringMVC注解](#springmvc注解)<br>


----------



## 执行过程图

![过程](https://github.com/Zhangchao999/SSM/raw/master/pictures/SSM01.jpg)

## SSM搭建

使用maven开发：<br>
使用maven后会在 `Java Resources` 中生成 `main/java包` `main/resources包` `test/java包`<br>
main/java 为以后开发中写Java的目录<br>
main/resources 写除了 `.java` 文件外的其他文件<br>
test/java 写测试类<br>
<br><br>
下面开始正式搭建框架<br>
先看看完整的目录<br>

![完整的目录](https://github.com/Zhangchao999/SSM/raw/master/pictures/SSM02.png)

<br>
解释：<br>
1、dao包: 用于操作数据库的，例如

```java
//该方法用于查找id对应的User
User queryById(int id);
```


```java
//当方法的参数是两个或两个以上时，要加@Param
int insertUser(@Param("userName") String userName,@Param("password") String password);
```
在dao包中写的是interface,具体的实现用到了Mybstis，即后面的mapping目录下的.xml文件<br><br>
2、entity包: 用于编写实体类。<br><br>
3、service包: 用于编写业务，例如
```java
public User login(String userNo,String password); 
```

该方法用于查询用户名和密码是否正确，service写的也是interface,具体的实现在service.impl中<br><br>

4、service.impl包: 用于实现接口，也就是interface的具体的实现，一般会用到相应的dao。<br>在类前面写@Service，标注业务层组件，为了自动把类注册到spring下。 <br>在 dao 前面写@Autowired，为了装配bean。 <br><br>

5、web包:  用于实现控制及跳转。<br>在类前面写@Controller，用于定义控制器类，及MVC的DispatcherServlet的具体实现。在想要实现功能的方法前面写@RequestMapping(value="###"),### 为相应的url地址。<br><br>

6、mapping文件夹： 是前面dao包的实现，这是Mybatis知识。<br>该文件夹下都是.xml文件，文件中主体写在<mapper namespace="###"></mapper>中，### 为dao包的完整路径。<br><br>

7、spring文件夹： 是SSM的脊梁，SSM能完整的运行主要spring下的文件配置。<br>
spring-dao.xml
> 是连接数据库的配置文件，前提是先写了jdbc.properties文件，在该配置文件中可以读取.properties的内容。

spring-service.xml
> 主要是自动扫描注解及注册相应的类，

spring-web.xml
> 主要是MVC的注解，以及视图的显示包括前置（.jsp 的文件在那个文件夹下面）和后置（以什么结尾的 .jsp）

<br>
8、jdbc.properties 是数据库的配置文件包括 driver, url, username, password.<br><br>

9、logback.xml 是日志文件。<br><br>

10、mybatis-config.xml 文件为mybatis的相关配置。<br><br>

11、src文件夹：全部的文件都在这，包括webapp（WEB-INF，web.xml等文件）<br><br>

12、  pom.xml 是该项目全部的.jar包

### spring的配置
1、jdbc.properties
```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/db_SSM
jdbc.username=root
jdbc.password=
```

2、spring-dao.xml<br>
	数据库的连接参数（jdbc.properties);<br>
	数据库连接池<br>
	SqlSessionFactory对象<br>
	自动扫描dao接口
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd">
	
	<!-- 配置数据库的相关参数properties的属性 -->	
	<context:property-placeholder location="classpath:jdbc.properties"/>
	
	<!-- 数据库连接池 c3p0 -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<!-- 连接属性 -->
		<property name="driverClass" value="${jdbc.driver}"/>
		<property name="jdbcUrl" value="${jdbc.url}"/>
		<property name="user" value="${jdbc.username}"/>
		<property name="password" value="${jdbc.password}"/>
		<!-- c3p0连结池的属性 -->
		<property name="maxPoolSize" value="30"/>
		<property name="minPoolSize" value="10"/>
		<property name="autoCommitOnClose" value="false"/>
		<property name="checkoutTimeout" value="10000"/>
		<property name="acquireRetryAttempts" value="2"/>
	</bean>
	
	<!-- 配置SqlSessionFactroy对象 -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="configLocation" value="classpath:mybatis-config.xml"/>
		<property name="typeAliasesPackage" value="com.zc.entity"/>
		<property name="mapperLocations" value="classpath:mapping/*.xml"/>
	
	</bean>
	
	<!-- 配置扫描dao接口包，动态实现dao接口，注入到spring容器 -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
		<property name="basePackage" value="com.zc.dao"></property>
	</bean>
	
		
</beans>

```

3、spring-service.xml<br>
	扫描service包下的注解@service<br>
	配置事务管理器<br>
	配置基于注解的声明事务

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
						http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/context
						http://www.springframework.org/schema/context/spring-context.xsd
						http://www.springframework.org/schema/tx
						http://www.springframework.org/schema/tx/spring-tx.xsd">
	
	<!-- 扫描service包下所有使用注解的类型 -->
	<context:component-scan base-package="com.zc.service"/>
	
	<!-- 配置事物管理器 -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<!-- 注入数据库连接池 -->
		<property name="dataSource" ref="dataSource"/>
	</bean>

	<!-- 配置基于注解的声明事务-->
	<tx:annotation-driven transaction-manager="transactionManager"/>	
	
</beans>
```

4、spring-web.xml<br>
	开启springMVC注解<br>
	静态资源默认servlet配置<br>
	配置jsp 显示viewResolver<br>
	扫描web相关的bean @Controller
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
						http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/context
						http://www.springframework.org/schema/context/spring-context.xsd
						http://www.springframework.org/schema/mvc
						http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd">

	<!-- 开启springMVC注解模式 -->
	<mvc:annotation-driven/>
	
	<!-- 静态资源默认servlet配置 -->
	<mvc:default-servlet-handler/>
	
	<!-- 配置jsp 显示viewResolver -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
		<property name="prefix" value="/WEB-INF/jsp/"/>
	</bean>
	
	<!-- 扫描web相关的bean -->
	<context:component-scan base-package="com.zc.web"/>
</beans>
```

5、logback.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<!-- encoders are by default assigned the type ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
		<encoder>
			<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
		</encoder>
	</appender>

	<root level="debug">
		<appender-ref ref="STDOUT" />
	</root>
</configuration>
```

6、mybatis-config.xml<br>
	自动增加主键<br>
	使用列别名替换列名<br>
	开启驼峰命名转换
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!-- 配置全局属性 -->
	<settings>
		<!-- 使用jdbc的getGeneratedKeys获取数据库自增主键值 -->
		<setting name="useGeneratedKeys" value="true" />

		<!-- 使用列别名替换列名 默认:true -->
		<setting name="useColumnLabel" value="true" />

		<!-- 开启驼峰命名转换:Table{create_time} -> Entity{createTime} -->
		<setting name="mapUnderscoreToCamelCase" value="true" />
	</settings>
</configuration>
```

7、web.xml<br>
	系统的web.xml文件，把前面的`spring-dao.xml` `spring-service.xml` `spring-web.xml` 导入到web.xml,实现在服务器启动时加载配置文件。
```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	version="3.1" metadata-complete="true">
	
	<!-- 配置DispatcherServlet -->
	<servlet>
		<servlet-name>mvc-dispatcher</servlet-name>	
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring/spring-*.xml</param-value>
		</init-param>
	</servlet>
	<servlet-mapping>
		<servlet-name>mvc-dispatcher</servlet-name>
		<!-- 默认匹配所有的请求 -->
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	
</web-app>

```

## 绝对路径
```java
${pageContext.request.contextPath}
<%=request.getContextPath()%>
```

## Maven 添加JSON 显示错误
```xml
<dependency>
    <groupId>net.sf.json-lib</groupId>
    <artifactId>json-lib</artifactId>
    <version>2.4</version>
    <!-- 必须添加下面的jdk15 -->
    <classifier>jdk15</classifier>
</dependency>

```

## Ajax的执行顺序
1、 创建xmlHttpRequest对象；<br>
2、 使用xmlHttpRequest对象的open()和send()方法发送资源请求给服务器；<br>
3、 使用xmlHttpRequest对象的responseText或responseXML属性获得服务器的响应；<br>
4、 执行onreadystatechange函数；<br>



## ajax动态实现下拉框-从数据库获取

例如：
``` jsp
所在院系：
<select id="department" name="department" onclick="addDepartment()">
			
	<option value="">请选择...</option>

</select>
```


```java
public String showAllDep(HttpServletResponse response,HttpServletRequest request) throws Exception {
		// departmentService 为service类
		// allDepartment 方法获得所有的学院 返回List类
		List<Department> departments = departmentService.allDepartment();
		
		// 把List 转换成JSON的形式
		JSONArray json =  JSONArray.fromObject(departments);
		// 设置编码格式
		response.setContentType("text/html;charset=UTF-8");
		// 发送数据 使ajax接受数据
		PrintWriter write = response.getWriter();
		write.write(json.toString());
		write.close();
		return "teacher/addTeacher.jsp";
	}


```


```javascript
 var xmlHttpRequest;

 // clickNum 设置为点击下拉框的次数
 var clickNum  = 0;
 function addDepartment(){
 	clickNum++;
 	if (window.XMLHttpRequest) {
		xmlHttpRequest = new XMLHttpRequest();
	} else {
		xmlHttpRequest = new ActiveXObject("Microsoft.XMLHTTP");
	}
	// onreadystatechange触发的条件是 readyState发生改变时
	xmlHttpRequest.onreadystatechange = callBack;

	url="/BSManager/getAllPartment";
	xmlHttpRequest.open("POST", url, true);
	xmlHttpRequest.send();
 }
 
 function callBack(){
 	if (xmlHttpRequest.readyState == 4 && xmlHttpRequest.status == 200){

 		//获得后台取得的数据
 		var departments = xmlHttpRequest.responseText;
 		// eval 把JSONArray 拆成字符数组
 		var dep = eval("("+departments+")");

 		// 获得select对象
 		var dd = document.getElementById("department");
 		
 		/*var id = deps[0].id;
 		alert(id);
 		var name = deps[0].departmentName;
 		alert(name);*/
 		
 		// if-else 是为了 在点击下拉框时避免重复
 		// clickNum 为 奇数 时，把下拉框设置为空，在添加options
 		// clickNum 为 偶数 时，是为了点击要选择的 学院 
 		if(clickNum%2==0){
 			for(var i=0;i<dep.length;i++){
 				// 获得 dep 的id 和 departmentName
 				var id = dep[i].id;
 				var name = dep[i].departmentName;
 				// 添加新的option 
	 		 	dd.options.add(new Option(name,id));
	 		}
 		}else if(clickNum%2==1){
 			// 把下拉框设置为空
 			dd.options.length = 0;
 			// 添加下拉框
 			for(var i=0;i<dep.length;i++){
 				var id = dep[i].id;
 				var name = dep[i].departmentName;
	 		 	dd.options.add(new Option(name,id));
	 		}
 		}			
 	}
 }
```

## ajax动态实现下拉框-从数据库获取方法2

```jsp

所有的书籍：
<select id="bookTable" name="bookTable">
	<option value="">请选择...</option>		
</select>
```

```java

java 方法把数据库获得的书籍信息由list转换为JSONArray，并返回。

```

```javascript
// 页面加载完后，执行showInfo()方法
window.onload = function(){
	showInfo();
}
 
 var xmlHttpRequest;
 function showInfo(){
 	if (window.XMLHttpRequest) {
		xmlHttpRequest = new XMLHttpRequest();
	} else {
		xmlHttpRequest = new ActiveXObject("Microsoft.XMLHTTP");
	}
	xmlHttpRequest.onreadystatechange = callBack;
	url="/dropdownTest/getInfo";
	xmlHttpRequest.open("POST", url, true);
	xmlHttpRequest.send();
 }
 
 function callBack(){
 	if (xmlHttpRequest.readyState == 4 && xmlHttpRequest.status == 200){
 		var books = xmlHttpRequest.responseText;
 		var book = eval("("+books+")");
 		var bt = document.getElementById("bookTable");
 			for(var i=0;i<book.length;i++){
 				var id = book[i].id;
 				var name = book[i].name;
	 		 	bt.options.add(new Option(name,id));
 			}
 		
 	}
 }
```

两种方法的比较：<br>
	相同点：都可以实现下拉框数据从数据库中动态获得。<br>
	不同点：使用onclick，根据点击的次数开获得数据，可能出现点击后还是显示两倍的数据。而使用window.onload，只是在页面加载完之后执行Ajax，不用判断点击的次数。<br>

## 在SSM的controller中获表单中文乱码

``` java

@RequestMapping(value="/add",method=RequestMethod.POST)
	
	// 从表单获得的数据为teacherNo，teacherName，sex ，inputMan ，phone ，department

	public void addTeacher(HttpServletRequest request,String teacherNo, String teacherName,String sex,String inputMan,String phone,String department,Model model) throws Exception {

		// 乱码的原因是 获得的字符的编码为 iso-8859-1
		teacherNo = new String(teacherNo.getBytes("iso-8859-1"),"utf-8");
		teacherName = new String(teacherName.getBytes("iso-8859-1"),"utf-8");
		sex = new String(sex.getBytes("iso-8859-1"),"utf-8");
		inputMan = new String(inputMan.getBytes("iso-8859-1"),"utf-8");
		phone = new String(phone.getBytes("iso-8859-1"),"utf-8");
		department = new String(department.getBytes("iso-8859-1"),"utf-8");
		
		// 测试获得的数据
		System.out.println(teacherNo+" "+teacherName+" "+sex+" "+inputMan+" "+phone+" "+department);
		
	}
```


## SpringMVC注解

1. @Controller <br>

> 在SpringMVC中，控制器Controller负责处理DispatcherServlet分发的请求，他把用户请求的数据经过处理层处理后分装成一个Model，然后把该Model返回给View进行展示。

>  你无需继承特定的类或者特定的接口，只要用@Controller标记一个类是Controller，然后用@RequestMapping和@RequestOaram等注解定义URL请求和Controller之间的映射，这样Controller就能被外界访问了。

> @Controller只是标记一个类。分发处理器会扫描使用该注解的类的方法上，并检测是否使用了@RequestMapping注解。@Controller 只是定义了控制器，真正处理请求的处理器是 @RequestMapping注解。

> < context:component-scan base-package = "com.zc.web" />包名为要扫描的包

2. @RequestMapping <br>

> RequestMapping用来处理请求地址映射的注解，可用在类或方法上，在类上表示父路径。

> value,method : value指定请求的实际地址。method指定请求的method类型。

> consumes,produces : consumes指定请求提交内容类型(Content-Type),例如 application/json, text/html。priduces指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回。

3. @Resource和@Autowired <br>

> @Resource和@Autowired 都是bean注入时使用的。

> 不同点：@Autowired 按类型装配(这个注解属于spring)，可以结合@Qualifier注解一起使用。@Resource 按name装配(有J2EE提供)，可以使用name属性。

4. @RequestParam <br>

> @RequestParam 将指定的请求参数复制给方法中的形参，与@RequestMapping搭配使用。

``` java
// 该controller 用于注册。
@RequestMapping(value="/register",method=RequestMethod.POST)
	public String register(
		   @RequestParam("userId") Integer userId,
		   @RequestParam("password")  String password,
		   @RequestParam("userName") String userName) {
		User user = new User();
		user.setUserId(userId);
		user.setUserName(userName);
		user.setPassword(password);

		userList.add(user);	

		// 返回到loginForm.jsp页面
		return "loginForm";
	}
```

5. @PathVariable <br>

> @PathVariable 可以方便的获得请求的动态参数，只支持value属性，类型为String 表示绑定的名称，如果省略则默认绑定同名参数。

6. @RequestHead <br>

> @RequestHead用于将请求的头信息区数据映射到功能方法的参数上。

7. @CookieValue <br>

> @CookieValue 用于将请求的Cookie数据映射到功能处理的参数上。

8. @SessionAttributes <br>

> @SessionAttributes 注解有选择的指定Model中的哪些属性需要转存到HttpSession对象中。

9. @ModelAttribute <br>

> @ModelAttribute将请求参数绑定到Model对象。


