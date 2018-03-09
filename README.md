# SSM
记录学习ssm的知识

### 执行过程图

![过程](https://github.com/Zhangchao999/SSM/raw/master/pictures/SSM01.jpg)

### 绝对路径
``` java
${pageContext.request.contextPath}
```

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
		
		// 把数据以JSON的形式发送
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
 		// clickNum 为 1 时，把下拉框设置为空，在添加options
 		// clickNum 为 2 时，是为了点击要选择的 学院 
 		if(clickNum%2==0){
 			for(var i=0;i<dep.length;i++){
 				// 获得 dep 的id 和 departmentName
 				var id = dep[i].id;
 				var name = dep[i].departmentName;
 				// 添加新的option 
	 		 	dd.options.add(new Option(name,id));
	 		}
 		}else if(clickNum%2==1){
 			dd.options.length = 0;
 			for(var i=0;i<dep.length;i++){
 				var id = dep[i].id;
 				var name = dep[i].departmentName;
	 		 	dd.options.add(new Option(name,id));
	 		}
 		}			
 	}
 }