[<img src="../../index.jpg" width = "80" height = "80"  />](../../index.md#index)

<h1 id="jy">SpringMVC文件校验</h1>

### **1.** **添加所需的jar包**

```xml
	<dependency>
		<groupId>org.hibernate</groupId>
		<artifactId>hibernate-validator</artifactId>
		<version>5.4.1.Final</version>
	</dependency>
```

### **2.** **配置springmvc.xml**

```xml
<!-- validator校验器注入到适配器 -->
	<mvc:annotation-driven validator="validator" />
	<!-- 校验器 -->
	<bean id="validator"
		class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
		<!-- hibernate校验器 -->
		<property name="providerClass"
			value="org.hibernate.validator.HibernateValidator" />
		<!-- 指定存放校验信息的文件 -->
		<property name="validationMessageSource"
			ref="validatorMsgSource" />
	</bean>
	<!-- 校验信息文件 -->
	<bean id="validatorMsgSource"
		class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
		<!-- 文件路径 -->
		<property name="basenames">
			<list>
				<value>classpath:validationMsg</value>
			</list>
		</property>

		<!-- 资源文件编码格式 -->
		<property name="defaultEncoding" value="UTF-8"></property>
		<!-- 资源文件内容缓存时间 -->
		<property name="cacheSeconds" value="120"></property>
	</bean>
```

### **3.编写资源文件validationMsg.properties**

```properties
Size.student.sName=\u540D\u5B57\u5FC5\u987B\u57286\u523016\u4F4D\u4E4B\u95F4\uFF01
DecimalMax.student.age=\u5E74\u9F84\u5FC5\u987B\u5904\u4E8E20~30\u4E4B\u95F4\uFF01
Length.student.sPhone=\u7535\u8BDD\u5FC5\u987B\u57288\u523011\u4F4D\uFF01
Email.student.sEmail=\u90AE\u7BB1\u683C\u5F0F\u4E0D\u6B63\u786E\uFF01
DateTimeFormat.student.sBirthday=\u65E5\u671F\u683C\u5F0F\u4E0D\u6B63\u786E\uFF01
```

**4.写一个实体类Student.java**

```java
public class Student {

	@Size(min=2,max=16)
	private String sName;
	private String sSex;
	@DecimalMax(value = "30",inclusive = true)
    	@DecimalMin(value = "19",inclusive = false)
	private int sAge;//年龄范围[20-30]
	@Length(min=8,max=11)
	private String sPhone;
	@Email
	private String sEmail;
	@Past
   	@DateTimeFormat(pattern="yyyy-MM-dd")
	private Date sBirthday;
......
省略了get/set方法
}
```

### **5.** **写一个controller处理**

```java
@Controller
@RequestMapping(value="/common")
public class StudentValidatorController {

	@RequestMapping(value="/indexPage",method= {RequestMethod.GET})
	public String getPage(@ModelAttribute(value="stu")Student student) {
		return "register";
	}
	@RequestMapping(value="/indexPage",method= {RequestMethod.POST})
	public ModelAndView validator(@Validated @ModelAttribute(value="stu")Student student,
			BindingResult br) {
		ModelAndView mv = new ModelAndView();
		if(br.getErrorCount()>0) {
			List<ObjectError> allErrors = br.getAllErrors();
			for (ObjectError objectError : allErrors) {
				System.out.println(objectError.getDefaultMessage());
			}
			mv.setViewName("register");
		}else {
			System.out.println(student);
			mv.setViewName("registerSuccess");
		}
		return mv;
	}
}
```

### **6.** **页面**

#### **(1):register.jsp**

```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<form:form method="post" modelAttribute="stu">
		<table>
			<tr>
				<td><label>姓名</label></td>
				<td>
					<form:input path="sName"/><form:errors path="sName"/>
				</td>
			</tr>
			<tr>
				<td><label>年龄</label></td>
				<td>
					<form:input path="sAge"/><form:errors path="sAge"/>
				</td>
			</tr>
			<tr>
				<td><label>性别</label></td>
				<td>
					<form:radiobutton path="sSex" label="男" value="男"/>
					<form:radiobutton path="sSex" label="女" value="女"/>
					<form:errors path="sSex"/>
				</td>
			</tr>
			<tr>
				<td><label>联系方式</label></td>
				<td>
					<form:input path="sPhone"/><form:errors path="sPhone"/>
				</td>
			</tr>
			<tr>
				<td><label>出生日期</label></td>
				<td>
					<form:input path="sBirthday"/><form:errors path="sBirthday"/>
				</td>
			</tr>
			<tr>
				<td><label>Email</label></td>
				<td>
					<form:input path="sEmail"/><form:errors path="sEmail"/>
				</td>
			</tr>
			<tr>
				<td colspan="2">
					<input type="submit" value="注册">
				</td>
			</tr>
		</table>
</form:form>
```

#### **(2)** **:registerSuccess.jsp**

```jsp
<table>
			<tr>
				<td><label>姓名</label></td>
				<td>
					${stu.sName }
				</td>
			</tr>
			<tr>
				<td><label>年龄</label></td>
				<td>
					${stu.sAge }
				</td>
			</tr>
			<tr>
				<td><label>性别</label></td>
				<td>
					${stu.sSex }
				</td>
			</tr>
			<tr>
				<td><label>联系方式</label></td>
				<td>
					${stu.sPhone }
				</td>
			</tr>
			<tr>
				<td><label>出生日期</label></td>
				<td>
					${stu.sBirthday}
				</td>
			</tr>
			<tr>
				<td><label>Email</label></td>
				<td>
					${stu.sEmail}
				</td>
			</tr>
		</table>
```

### **7.** **运行**

<http://localhost:8888/SpringMVC2/common/indexPage>

![](img/jy001.png)