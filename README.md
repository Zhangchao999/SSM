# SSM
记录学习ssm时遇到的问题

### 执行过程图

![过程](https://github.com/Zhangchao999/SSM/raw/master/pictures/SSM01.jpg)

*******
### SSM的搭建

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
解释：
`dao包` 用于操作数据库的，例如：User queryById(int id); 查找id 对应的User,<font color="red">在dao包中写的是interface,具体的实现用到了Mybstis，即后面的mapping目录下的.xml文件</font><br>











### 绝对路径
``` java
${pageContext.request.contextPath}
```

### Maven 添加JSON 显示错误
```xml
<dependency>
    <groupId>net.sf.json-lib</groupId>
    <artifactId>json-lib</artifactId>
    <version>2.4</version>
    <!-- 必须添加下面的jdk15 -->
    <classifier>jdk15</classifier>
</dependency>

```

### Ajax的执行顺序
1、 创建xmlHttpRequest对象；<br>
2、 使用xmlHttpRequest对象的open（）和send（）方法发送资源请求给服务器；<br>
3、 使用xmlHttpRequest对象的responseText或responseXML属性获得服务器的响应；<br>
4、 执行onreadystatechange函数；<br>



### ajax动态实现下拉框（从数据库获取）

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


### 在SSM的controller中获表单中文乱码

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

