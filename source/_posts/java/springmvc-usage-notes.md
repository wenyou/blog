---
title: SpringMVC使用笔记
author: Zeeny
comments: false
date: 2015-02-12
updated: 2015-02-12
tags: [Java,Spring,SpringMvc]
categories: [Java,Spring]
summary: SpringMVC使用笔记
---


# SpringMVC使用笔记

## 常见问题

* 使用spring mvc上传文件的常见错误解决办法

1. __异常__：org.apache.catalina.connector.RequestFacade cannot be cast to org.springframework.web.multipart.MultipartHttpServletRequest

	解决办法：在spring里面配置
		
		`<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver" />`
		
	重点：id一定要是multipartResolver，否则也会出同样的错误。


2. __异常__：nested exception is java.lang.NoClassDefFoundError: org/apache/commons/fileupload/FileItemFactory

	解决办法：需要common-fileupload.jar包

	
3. __异常__：java.lang.ClassNotFoundException: org.apache.commons.io.output.DeferredFileOutputStream
	
		解决办法：需要common-io.jar包
	
	
* NetworkError: 415 Unsupported Media Type,Http请求415 Unsupported Media Type的问题
	
		在SprinvMVC的Web程序中，我在页面发送Ajax 的POST请求，然后在服务器端利用@requestBody接收请求body中的参数，当时运行过程中，我想服务器发送Ajax请求，浏览器一直反馈415 Unsupported Media Type或者400的状态码，以为是Ajax写的有问题。便查找了半天资料，才发现spring-mvc.config文件的配置中少了东西，当然也有可能是你真的在Ajax中缺少了对Content-Type参数的设置。分析后应该是我springMVC-config.xml文件配置有问题。
		（注）：400：（错误请求） 服务器不理解请求的语法。 415：（不支持的媒体类型） 请求的格式不受请求页面的支持。
		
		解决方法：
			在springMVC-config.xml文件中，增加了一个StringHttpMessageConverter请求信息转换器，配置片段如下：
```
<!--- StringHttpMessageConverter bean -->
<bean id = "stringHttpMessageConverter" class = "org.springframework.http.converter.StringHttpMessageConverter"/>

<!-- 启动Spring MVC的注解功能，完成请求和注解POJO的映射 -->
<bean class ="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter" >
	<property name= "messageConverters" >
		<list>
			<ref bean= "mappingJacksonHttpMessageConverter" />
			<!-- 新增的StringMessageConverter bean-->
			<ref bean= "stringHttpMessageConverter" />
			<ref bean= "jsonHttpMessageConverter" />           
			<ref bean= "formHttpMessageConverter" />
		</list>
	</property>
</bean>
```

## 相关知识点

### 1. SpringMVC常用注解

		@Controller 
		  	负责注册一个bean 到spring 上下文中。
		  
		@RequestMapping
			注解为控制器指定可以处理哪些 URL 请求。
			
		@RequestBody
			该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上，再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。
			
		@ResponseBody
			该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。
			
		@ModelAttribute
			在方法定义上使用 @ModelAttribute 注解：Spring MVC 在调用目标处理方法前，会先逐个调用在方法级上标注了@ModelAttribute 的方法。
			在方法的入参前使用 @ModelAttribute 注解：可以从隐含对象中获取隐含的模型数据中获取对象，再将请求参数 –绑定到对象中，再传入入参将方法入参对象添加到模型中。
		
		@RequestParam　
			在处理方法入参处使用 @RequestParam 可以把请求参数传递给请求方法。
			
		@PathVariable
			绑定 URL 占位符到入参。
		
		@ExceptionHandler
			注解到方法上，出现异常时会执行该方法。
			
		@ControllerAdvice
			使一个Contoller成为全局的异常处理类，类中用@ExceptionHandler方法注解的方法可以处理所有Controller发生的异常。
			
			
### 2. 重要的接口和类的简单说明

		DispatcherServlet：   前端控制器，用于接收请求。
		
		
		HandlerMapping接口：   用于处理请求的映射。
		DefaultAnnotationHandlerMapping：   HandlerMapping接口的实现，用于把一个URL映射到具体的Controller类上。
		
		
		HandlerAdapter接口： 用于处理请求的映射。
		AnnotationMethodHandlerAdapter： HandlerAdapter接口的实现，用于把一个URL映射到对应Controller类的某个方法上。
		
		ViewResolver接口：用于解析View。
		InternalResourceViewResolver： ViewResolver接口的实现，用于把ModelAndView的逻辑视图名解析为具体的View。
		
		
		

### 3. URL通配符映射

		通配符有“？”和“*”两个字符。其中“？”表示1个字符，“*”表示匹配多个字符，“**”表示匹配0个或多个路径。
		
		例如：
		“/cp/test?”可以匹配“/cp/testA”、“/cp/testB”，但不能匹配“/cp/test”也不能匹配“/cp/testAA”；

		“/cp/test*”可以匹配“/cp/test”、“/cp/testA”、“/cp/testAA”但不能匹配“/cp/test/A”；

		“/cp/test/*”可以匹配“/cp/test/”、“/cp/test/A”、“/cp/test/AA”、“/cp/test/AB” 但不能匹配“/cp/test”、“/cp/test/A/B”;

		“/cp/test/**”可以匹配“/cp/test/”下的多个子路径，比如：“/cp/test/A/B/C/D”;

		如果现在有“/cp/test”和“/cp/*”，如果请求地址为“/cp/test”那么将如何匹配？Spring MVC会按照最长匹配优先原则（即和映射配置中哪个匹配的最多）来匹配，所以会匹配“/cp/test”。
		

### 4. URL正则表达式映射

		eg:  @RequestMapping(value="/reg/{name:\\w+}-{age:\\d+}", method = {RequestMethod.GET})
		
		
### 5. 绑定数据的注解

		@RequestParam , 绑定单个请求数据，可以是URL中的数据，表单提交的数据或上传的文件； 
		@PathVariable ,绑定URL模板变量值；
		@CookieValue , 绑定Cookie数据；
		@RequestHeader , 绑定请求头数据；
		@ModelAttribute , 绑定数据到Model；
		@SessionAttributes , 绑定数据到Session；
		@RequestBody , 用来处理Content-Type不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等； 
		@RequestPart , 绑定“multipart/data”数据，并可以根据数据类型进行对象转换；
 
