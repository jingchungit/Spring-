# SpringMVC基础

## 配置web.xml

注册SpringMVC的前端控制器DispatcherServlet

### a>默认配置方式

此配置作用下，SpringMVC的配置文件默认位于WEB-INF下，默认名称为<servlet-name>-servlet.xml，例如，以下配置所对应SpringMVC的配置文件位于WEB-INF下，文件名为springMVC-servlet.xml



```xml
<!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
<servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!--
        设置springMVC的核心控制器所能处理的请求的请求路径
        /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
        但是/不能匹配.jsp请求路径的请求
    -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

### b>扩展配置方式

可通过init-param标签设置SpringMVC配置文件的位置和名称，通过load-on-startup标签设置SpringMVC前端控制器DispatcherServlet的初始化时间



```xml
<!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
<servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
    <init-param>
        <!-- contextConfigLocation为固定值 -->
        <param-name>contextConfigLocation</param-name>
        <!-- 使用classpath:表示从类路径查找配置文件，例如maven工程中的src/main/resources -->
        <param-value>classpath:springMVC.xml</param-value>
    </init-param>
    <!-- 
 		作为框架的核心组件，在启动过程中有大量的初始化操作要做
		而这些操作放在第一次请求时才执行会严重影响访问速度
		因此需要通过此标签将启动控制DispatcherServlet的初始化时间提前到服务器启动时
	-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!--
        设置springMVC的核心控制器所能处理的请求的请求路径
        /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
        但是/不能匹配.jsp请求路径的请求
    -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

提示

<url-pattern>标签中使用/和/*的区别：

/所匹配的请求可以是/login或.html或.js或.css方式的请求路径，但是/不能匹配.jsp请求路径的请求

因此就可以避免在访问jsp页面时，该请求被DispatcherServlet处理，从而找不到相应的页面

/*则能够匹配所有请求，例如在使用过滤器时，若需要对所有请求进行过滤，就需要使用/*的写法

## 创建请求控制器

由于**前端控制器对浏览器发送的请求进行了统一的处理**，但是**具体的请求有不同的处理过程**，因此需要创建处理具体请求的类，即**请求控制器**

请求控制器中每一个处理请求的方法成为控制器方法

因为SpringMVC的控制器由一个POJO（普通的Java类）担任，因此需要通过@Controller注解将其标识为一个控制层组件，交给Spring的IoC容器管理，此时SpringMVC才能够识别控制器的存在

```java
@Controller
public class HelloController {
    
}
```

## 创建springMVC的配置文件

```xml
<!-- 自动扫描包 -->
<context:component-scan base-package="com.atguigu.mvc.controller"/>

<!-- 配置Thymeleaf视图解析器 -->
<bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
    <property name="order" value="1"/>
    <property name="characterEncoding" value="UTF-8"/>
    <property name="templateEngine">
        <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
            <property name="templateResolver">
                <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
    
                    <!-- 视图前缀 -->
                    <property name="prefix" value="/WEB-INF/templates/"/>
    
                    <!-- 视图后缀 -->
                    <property name="suffix" value=".html"/>
                    <property name="templateMode" value="HTML5"/>
                    <property name="characterEncoding" value="UTF-8" />
                </bean>
            </property>
        </bean>
    </property>
</bean>

<!-- 
   处理静态资源，例如html、js、css、jpg
  若只设置该标签，则只能访问静态资源，其他请求则无法访问
  此时必须设置<mvc:annotation-driven/>解决问题
 -->
<mvc:default-servlet-handler/>

<!-- 开启mvc注解驱动 -->
<mvc:annotation-driven>
    <mvc:message-converters>
        <!-- 处理响应中文内容乱码 -->
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="defaultCharset" value="UTF-8" />
            <property name="supportedMediaTypes">
                <list>
                    <value>text/html</value>
                    <value>application/json</value>
                </list>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

# RequestMapping注解

## @RequestMapping注解的功能

从注解名称上我们可以看到，@RequestMapping注解的作用就是将请求和处理请求的控制器方法关联起来，建立映射关系。

SpringMVC 接收到指定的请求，就会来找到在映射关系中对应的控制器方法来处理这个请求。

## @RequestMapping注解的位置

@RequestMapping**标识一个类**：**设置映射请求的请求路径的初始信息**

@RequestMapping**标识一个方法**：**设置映射请求请求路径的具体信息**

```java
@Controller
@RequestMapping("/test")
public class RequestMappingController {

	//此时请求映射所映射的请求的请求路径为：/test/testRequestMapping
    @RequestMapping("/testRequestMapping")
    public String testRequestMapping(){
        return "success";
    }

}
```

## @RequestMapping注解的value属性

@RequestMapping注解的**value属性通过请求的请求地址匹配**请求映射

@RequestMapping注解的value属性是一个字符串类型的数组，表示该请求映射能够匹配多个请求地址所对应的请求

@RequestMapping注解的value属性必须设置，至少通过请求地址匹配请求映射

```html
<a th:href="@{/testRequestMapping}">测试@RequestMapping的value属性-->/testRequestMapping</a><br>
<a th:href="@{/test}">测试@RequestMapping的value属性-->/test</a><br>
```

```java
@RequestMapping(
        value = {"/testRequestMapping", "/test"}
)
public String testRequestMapping(){
    return "success";
}
```

## @RequestMapping注解的method属性

@RequestMapping注解的**method属性通过请求的请求方式（get或post）匹配**请求映射

@RequestMapping注解的method属性是一个RequestMethod类型的数组，表示该请求映射能够匹配多种请求方式的请求

若当前请求的请求地址满足请求映射的value属性，但是请求方式不满足method属性，则浏览器报错405：Request method 'POST' not supported:即请求方式不被支持

```html
<a th:href="@{/test}">测试@RequestMapping的value属性-->/test</a><br>
<form th:action="@{/test}" method="post">
    <input type="submit">
</form>
```

```java
@RequestMapping(
        value = {"/testRequestMapping", "/test"},
        method = {RequestMethod.GET, RequestMethod.POST}
)
public String testRequestMapping(){
    return "success";
}
```

## @RequestMapping注解的params属性（了解）

@RequestMapping注解的params属性通过请求的请求参数匹配请求映射

@RequestMapping注解的params属性是一个字符串类型的数组，可以通过四种表达式设置请求参数和请求映射的匹配关系

"param"：要求请求映射所匹配的请求必须携带param请求参数

"!param"：要求请求映射所匹配的请求必须不能携带param请求参数

"param=value"：要求请求映射所匹配的请求必须携带param请求参数且param=value

"param!=value"：要求请求映射所匹配的请求必须携带param请求参数但是param!=value

```html
<a th:href="@{/test(username='admin',password=123456)">测试@RequestMapping的params属性-->/test</a><br>
```

```java
@RequestMapping(
        value = {"/testRequestMapping", "/test"}
        ,method = {RequestMethod.GET, RequestMethod.POST}
        ,params = {"username","password!=123456"}
)
public String testRequestMapping(){
    return "success";
}
```

## @RequestMapping注解的headers属性（了解）

@RequestMapping注解的headers属性通过请求的请求头信息匹配请求映射

@RequestMapping注解的headers属性是一个字符串类型的数组，可以通过四种表达式设置请求头信息和请求映射的匹配关系

"header"：要求请求映射所匹配的请求必须携带header请求头信息

"!header"：要求请求映射所匹配的请求必须不能携带header请求头信息

"header=value"：要求请求映射所匹配的请求必须携带header请求头信息且header=value

"header!=value"：要求请求映射所匹配的请求必须携带header请求头信息且header!=value

若当前请求满足@RequestMapping注解的value和method属性，但是不满足headers属性，此时页面显示404错误，即资源未找到

## SpringMVC支持ant风格的路径

？：表示任意的单个字符

*：表示任意的0个或多个字符

**：表示任意的一层或多层目录

注意：在使用**时，只能使用/**/xxx的方式

## SpringMVC支持路径中的占位符（重点）

原始方式：/deleteUser?id=1

rest方式：/deleteUser/1

SpringMVC路径中的占位符常用于RESTful风格中，当请求路径中将某些数据通过路径的方式传输到服务器中，就可以在相应的**@RequestMapping注解的value属性中通过占位符{xxx}表示传输的数据**，在通过@**PathVariable注解，将占位符所表示的数据赋值给控制器方法的形参**

```html
<a th:href="@{/testRest/1/admin}">测试路径中的占位符-->/testRest</a><br>
```

```java
@RequestMapping("/testRest/{id}/{username}")
public String testRest(@PathVariable("id") String id, @PathVariable("username") String username){
    System.out.println("id:"+id+",username:"+username);
    return "success";
}
//最终输出的内容为-->id:1,username:admin
```

# SpringMVC 获取请求参数

将HttpServletRequest作为控制器方法的形参，此时HttpServletRequest类型的参数表示封装了当前请求的请求报文的对象

```java
@RequestMapping("/testParam")
public String testParam(HttpServletRequest request){
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```

## 通过控制器方法的形参获取请求参数

在控制器方法的形参位置，**设置和请求参数同名的形参**，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给相应的形参

```html
<a th:href="@{/testParam(username='admin',password=123456)}">测试获取请求参数-->/testParam</a><br>
```

```java
@RequestMapping("/testParam")
public String testParam(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```

## @RequestParam

@RequestParam是将请求参数和控制器方法的形参创建映射关系

@RequestParam注解一共有三个属性：

value：指定为形参赋值的请求参数的参数名

required：设置是否必须传输此请求参数，默认值为true

若设置为**true**时，则当前请求**必须传输value所指定的请求参数**，若**没有传输该请求参数**，且**没有设置defaultValue属性**，则页面**报错400**：Required String parameter 'xxx' is not present；若**设置为false**，则当前请求不是必须传输value所指定的请求参数，若**没有传输**，则注解所标识的形参的**值为null**

defaultValue：不管required属性值为true或false，**当value所指定的请求参数没有传输或传输的值为""**时，则使用默认值为形参赋值

## @RequestHeader

@RequestHeader是将请求头信息和控制器方法的形参创建映射关系

@RequestHeader注解一共有三个属性：value、required、defaultValue，用法同@RequestParam

## @CookieValue

@CookieValue是将cookie数据和控制器方法的形参创建映射关系

@CookieValue注解一共有三个属性：value、required、defaultValue，用法同@RequestParam

## 通过POJO获取请求参数

可以在控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的请求参数的参数名和实体类中的属性名一致，那么请求参数就会为此属性赋值

```html
<form th:action="@{/testpojo}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    性别：<input type="radio" name="sex" value="男">男<input type="radio" name="sex" value="女">女<br>
    年龄：<input type="text" name="age"><br>
    邮箱：<input type="text" name="email"><br>
    <input type="submit">
</form>
```

```java
@RequestMapping("/testpojo")
public String testPOJO(User user){
    System.out.println(user);
    return "success";
}
//最终结果-->User{id=null, username='张三', password='123', age=23, sex='男', email='123@qq.com
```

## 解决获取请求参数的乱码问题

解决获取请求参数的乱码问题，可以使用SpringMVC提供的编码过滤器CharacterEncodingFilter，但是必须在web.xml中进行注册

- 源码

```java
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String encoding = this.getEncoding();
        if (encoding != null) {
            if (this.isForceRequestEncoding() || request.getCharacterEncoding() == null) {
                request.setCharacterEncoding(encoding); //设置请求的编码
            }

            if (this.isForceResponseEncoding()) {
                response.setCharacterEncoding(encoding);//设置相应的编码
            }
        }

        filterChain.doFilter(request, response);
    }
```

- 所以我们需要在处理编码过滤器中，使这两个if条件都成立

```xml
<!--配置springMVC的编码过滤器-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

SpringMVC中处理编码的过滤器一定要配置到其他过滤器之前，否则无效

# 域对象共享数据

## 使用ServletAPI向request域对象共享数据

```java
@RequestMapping("/testServletAPI")
public String testServletAPI(HttpServletRequest request){
    request.setAttribute("testScope", "hello,servletAPI");
    return "success";
}
```

## 使用ModelAndView向request域对象共享数据

```java
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
    /**
     * ModelAndView有Model和View的功能
     * Model主要用于向请求域共享数据
     * View主要用于设置视图，实现页面跳转
     */
    ModelAndView mav = new ModelAndView();
    //向请求域共享数据
    mav.addObject("testScope", "hello,ModelAndView");
    //设置视图，实现页面跳转
    mav.setViewName("success");
    return mav;
}
```

## 使用Model向request域对象共享数据

```java
@RequestMapping("/testModel")
public String testModel(Model model){
    model.addAttribute("testScope", "hello,Model");
    return "success";
}
```

## 使用map向request域对象共享数据

```java
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
    map.put("testScope", "hello,Map");
    return "success";
}
```

## 使用ModelMap向request域对象共享数据

```java
@RequestMapping("/testModelMap")
public String testModelMap(ModelMap modelMap){
    modelMap.addAttribute("testScope", "hello,ModelMap");
    return "success";
}
```

## Model、ModelMap、Map的关系

**Model、ModelMap、Map**类型的参数其实**本质上都是 BindingAwareModelMap 类型***的

```java
public interface Model{} //接口
public class ModelMap extends LinkedHashMap<String, Object> {}
public class ExtendedModelMap extends ModelMap implements Model {}
public class BindingAwareModelMap extends ExtendedModelMap {}//本质
```

## 向session域共享数据

```java
@RequestMapping("/testApplication")
public String testApplication(HttpSession session){
	ServletContext application = session.getServletContext();
    application.setAttribute("testApplicationScope", "hello,application");
    return "success";
}
```

## 向application域共享数据

```java
@RequestMapping("/testApplication")
public String testApplication(HttpSession session){
	ServletContext application = session.getServletContext();
    application.setAttribute("testApplicationScope", "hello,application");
    return "success";
}
```

# SpringMVC的视图

SpringMVC中的视图是View接口，视图的作用渲染数据，将模型Model中的数据展示给用户

SpringMVC视图的种类很多，默认有转发视图和重定向视图

当工程引入jstl的依赖，转发视图会自动转换为JstlView

若使用的视图技术为Thymeleaf，在SpringMVC的配置文件中配置了Thymeleaf的视图解析器，由此视图解析器解析之后所得到的是ThymeleafView

## ThymeleafView

当**控制器方法中所设置的视图名称没有任何前缀**时，此时的视图名称会被SpringMVC配置文件中所配置的视图解析器解析，视图名称拼接视图前缀和视图后缀所得到的最终路径，会通过转发的方式实现跳转

```java
@RequestMapping("/testHello")
public String testHello(){
    return "hello";
}
```

## 转发视图

**SpringMVC中默认的转发视图是InternalResourceView**

SpringMVC中创建转发视图的情况：

当控制器方法中所设置的**视图名称以"forward:"为前缀**时，创建**InternalResourceView视图**，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"forward:"去掉，剩余部分作为最终路径通过转发的方式实现跳转

例如"forward:/"，"forward:/employee"

```java
@RequestMapping("/testForward")
public String testForward(){
    return "forward:/testHello";
}
```

## 重定向视图

**SpringMVC中默认的重定向视图是RedirectView**

当控制器方法中所设置的**视图名称以"redirect:"为前缀**时，创建RedirectView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"redirect:"去掉，剩余部分作为最终路径通过重定向的方式实现跳转

例如"redirect:/"，"redirect:/employee"

```java
@RequestMapping("/testRedirect")
public String testRedirect(){
    return "redirect:/testHello";
}
```

## 视图控制器view-controller

当控制器方法中，仅仅用来实现页面跳转，即只需要设置视图名称时，可以将处理器方法使用view-controller标签进行表示

```xml
<!--
	path：设置处理的请求地址
	view-name：设置请求地址所对应的视图名称
-->
<mvc:view-controller path="/testView" view-name="success"></mvc:view-controller>
```

> 提示
>
> 当SpringMVC中设置任何一个view-controller时，其他控制器中的请求映射将全部失效，此时需要在SpringMVC的核心配置文件中设置开启mvc注解驱动的标签：
>
> <mvc:annotation-driven />

# RESTFul

## RESTFul简介

REST：**Re**presentational **S**tate **T**ransfer，表现层资源状态转移。

## RESTful的实现

具体说，就是 HTTP 协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。

它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE 用来删除资源。

REST 风格提倡 URL 地址使用统一的风格设计，从前到后各个单词使用斜杠分开，不使用问号键值对方式携带请求参数，而是将要发送给服务器的数据作为 URL 地址的一部分，以保证整体风格的一致性。

| 操作     | 传统方式         | REST风格                |
| -------- | ---------------- | ----------------------- |
| 查询操作 | getUserById?id=1 | user/1-->get请求方式    |
| 保存操作 | saveUser         | user-->post请求方式     |
| 删除操作 | deleteUser?id=1  | user/1-->delete请求方式 |
| 更新操作 | updateUser       | user-->put请求方式      |

## HiddenHttpMethodFilter

由于浏览器只支持发送get和post方式的请求，那么该如何发送put和delete请求呢？

SpringMVC 提供了 **HiddenHttpMethodFilter** 帮助我们**将 POST 请求转换为 DELETE 或 PUT 请求**

**HiddenHttpMethodFilter** 处理put和delete请求的条件：

a>当前请求的请求方式必须为post

b>当前请求必须传输请求参数_method

满足以上条件，**HiddenHttpMethodFilter** 过滤器就会将当前请求的请求方式转换为请求参数_method的值，因此请求参数_method的值才是最终的请求方式

在web.xml中注册**HiddenHttpMethodFilter**

```xml
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 提示
>
> 目前为止，SpringMVC中提供了两个过滤器：CharacterEncodingFilter和HiddenHttpMethodFilter
>
> 在web.xml中注册时，必须先注册CharacterEncodingFilter，再注册HiddenHttpMethodFilter
>
> 原因：
>
> - 在 CharacterEncodingFilter 中通过 request.setCharacterEncoding(encoding) 方法设置字符集的
> - request.setCharacterEncoding(encoding) 方法要求前面不能有任何获取请求参数的操作
> - 而 HiddenHttpMethodFilter 恰恰有一个获取请求方式的操作：
>
> 
>
> ```java
> String paramValue = request.getParameter(this.methodParam);
> ```

# HttpMessageConverter

HttpMessageConverter，报文信息转换器，**将请求报文转换为Java对象**，或将**Java对象转换为响应报文**

HttpMessageConverter提供了两个注解和两个类型：@RequestBody，@ResponseBody，RequestEntity，ResponseEntity

## @RequestBody

**@RequestBody可以获取请求体**，需要在控制器方法设置一个形参，使用@RequestBody进行标识，当前请求的请求体就会为当前注解所标识的形参赋值

```html
<form th:action="@{/testRequestBody}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    <input type="submit">
</form>
```

```java
@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String requestBody){
    System.out.println("requestBody:"+requestBody);
    return "success";
}
```

输出结果：

requestBody:username=admin&password=123456

## RequestEntity

RequestEntity封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息

```java
@RequestMapping("/testRequestEntity")
public String testRequestEntity(RequestEntity<String> requestEntity){
    System.out.println("requestHeader:"+requestEntity.getHeaders());
    System.out.println("requestBody:"+requestEntity.getBody());
    return "success";
}
```

输出结果： requestHeader:[host:"localhost:8080", connection:"keep-alive", content-length:"27", cache-control:"max-age=0", sec-ch-ua:"" Not A;Brand";v="99", "Chromium";v="90", "Google Chrome";v="90"", sec-ch-ua-mobile:"?0", upgrade-insecure-requests:"1", origin:"http://localhost:8080", user-agent:"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36"] requestBody:username=admin&password=123

## @ResponseBody

@ResponseBody用于标识一个控制器方法，可以将该方法的返回值直接作为响应报文的响应体响应到浏览器

```java
@RequestMapping("/testResponseBody")
@ResponseBody
public String testResponseBody(){
    return "success";
}
```

结果：浏览器页面显示success

## SpringMVC处理json

@ResponseBody处理json的步骤：

a>导入jackson的依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.1</version>
</dependency>
```

b>在SpringMVC的核心配置文件中开启mvc的注解驱动，此时在HandlerAdaptor中会自动装配一个消息转换器：MappingJackson2HttpMessageConverter，可以将响应到浏览器的Java对象转换为Json格式的字符串

```text
<mvc:annotation-driven />
```

c>在处理器方法上使用@ResponseBody注解进行标识

d>将Java对象直接作为控制器方法的返回值返回，就会自动转换为Json格式的字符串

```java
@RequestMapping("/testResponseUser")
@ResponseBody
public User testResponseUser(){
    return new User(1001,"admin","123456",23,"男");
}
```

浏览器的页面中展示的结果：

{"id":1001,"username":"admin","password":"123456","age":23,"sex":"男"}

## SpringMVC处理ajax

a>请求超链接：

```html
<div id="app">
	<a th:href="@{/testAjax}" @click="testAjax">testAjax</a><br>
</div>
```

b>通过vue和axios处理点击事件：

```html
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
<script type="text/javascript" th:src="@{/static/js/axios.min.js}"></script>
<script type="text/javascript">
    var vue = new Vue({
        el:"#app",
        methods:{
            testAjax:function (event) {
                axios({
                    method:"post",
                    url:event.target.href,
                    params:{
                        username:"admin",
                        password:"123456"
                    }
                }).then(function (response) {
                    alert(response.data);
                });
                event.preventDefault();
            }
        }
    });
</script>
```

c>控制器方法：

```java
@RequestMapping("/testAjax")
@ResponseBody
public String testAjax(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "hello,ajax";
}
```

## @RestController注解

@RestController注解是springMVC提供的一个复合注解，标识在控制器的类上，就相当于为类添加了@Controller注解，并且为其中的每个方法添加了@ResponseBody注解

## ResponseEntity

ResponseEntity用于控制器方法的返回值类型，该控制器方法的返回值就是响应到浏览器的响应报文

# 文件上传和下载

## 文件下载

使用ResponseEntity实现下载文件的功能

```java
@RequestMapping("/testDown")
public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException {
    //获取ServletContext对象
    ServletContext servletContext = session.getServletContext();
    //获取服务器中文件的真实路径
    String realPath = servletContext.getRealPath("/static/img/1.jpg");
    //创建输入流
    InputStream is = new FileInputStream(realPath);
    //创建字节数组
    byte[] bytes = new byte[is.available()];
    //将流读到字节数组中
    is.read(bytes);
    //创建HttpHeaders对象设置响应头信息
    MultiValueMap<String, String> headers = new HttpHeaders();
    //设置要下载方式以及下载文件的名字
    headers.add("Content-Disposition", "attachment;filename=1.jpg");
    //设置响应状态码
    HttpStatus statusCode = HttpStatus.OK;
    //创建ResponseEntity对象
    ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers, statusCode);
    //关闭输入流
    is.close();
    return responseEntity;
}
```

## 文件上传

文件上传要求form表单的请求方式必须为post，并且添加属性enctype="multipart/form-data"

SpringMVC中将上传的文件封装到MultipartFile对象中，通过此对象可以获取文件相关信息

上传步骤：

a>添加依赖：

```xml
<!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
```

b>在SpringMVC的配置文件中添加配置：

```xml
<!--必须通过文件解析器的解析才能将文件转换为MultipartFile对象-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
```

c>控制器方法：

```java
@RequestMapping("/testUp")
public String testUp(MultipartFile photo, HttpSession session) throws IOException {
    //获取上传的文件的文件名
    String fileName = photo.getOriginalFilename();
    //处理文件重名问题
    String hzName = fileName.substring(fileName.lastIndexOf("."));
    fileName = UUID.randomUUID().toString() + hzName;
    //获取服务器中photo目录的路径
    ServletContext servletContext = session.getServletContext();
    String photoPath = servletContext.getRealPath("photo");
    File file = new File(photoPath);
    if(!file.exists()){
        file.mkdir();
    }
    String finalPath = photoPath + File.separator + fileName;
    //实现上传功能
    photo.transferTo(new File(finalPath));
    return "success";
}
```

## 拦截器的配置

SpringMVC中的拦截器用于拦截控制器方法的执行

SpringMVC中的拦截器需要实现HandlerInterceptor

SpringMVC的拦截器必须在SpringMVC的配置文件中进行配置：

```xml
<bean class="com.atguigu.interceptor.FirstInterceptor"></bean>
<ref bean="firstInterceptor"></ref>
<!-- 以上两种配置方式都是对DispatcherServlet所处理的所有的请求进行拦截 -->
<mvc:interceptor>
    <mvc:mapping path="/**"/>
    <mvc:exclude-mapping path="/testRequestEntity"/>
    <ref bean="firstInterceptor"></ref>
</mvc:interceptor>
<!-- 
	以上配置方式可以通过ref或bean标签设置拦截器，通过mvc:mapping设置需要拦截的请求，通过mvc:exclude-mapping设置需要排除的请求，即不需要拦截的请求
-->
```

## 拦截器的三个抽象方法

SpringMVC中的拦截器有三个抽象方法：

preHandle：控制器方法执行之前执行preHandle()，其boolean类型的返回值表示是否拦截或放行，返回true为放行，即调用控制器方法；返回false表示拦截，即不调用控制器方法

postHandle：控制器方法执行之后执行postHandle()

afterCompletion：处理完视图和模型数据，渲染视图完毕之后执行afterCompletion()

## 多个拦截器的执行顺序

## 多个拦截器的执行顺序

- 若**每个拦截器的preHandle()都返回true**

此时多个拦截器的执行顺序和拦截器在SpringMVC的配置文件的配置顺序有关：

**preHandle()会按照配置的顺序执行**，而**postHandle()\**和\**afterCompletion()会按照配置的反序执行**

- 若**某个拦截器的preHandle()返回了false**

**preHandle()返回false和它之前的拦截器的preHandle()都会执行**，postHandle()都不执行，**返回false的拦截器之前的拦截器的afterCompletion()会执行**

<details class="custom-block details" open="" style="display: block; position: relative; border-radius: 2px; margin: 1em 0px; padding: 1.6em; background-color: var(--customBlockBg); border: 1px solid rgb(221, 221, 221); color: rgb(0, 50, 60); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Ubuntu, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><div class="language-java line-numbers-mode" style="overflow: hidden; transition: height 0.3s ease 0s; margin-top: 0.85rem; position: relative; background-color: var(--codeBg); border-radius: 6px; padding-bottom: 0.5rem; height: 667px;"><pre class="language-java codecopy-enabled" style="position: relative !important; user-select: text; line-height: 1.4; padding: 1.25rem 1.5rem 1.25rem 3.5rem; margin: 30px 0px 0.85rem; background: transparent; border-radius: 6px; overflow: auto; z-index: 1; color: rgb(204, 204, 204); text-shadow: none; font-family: Consolas, Monaco, &quot;Andale Mono&quot;, &quot;Ubuntu Mono&quot;, monospace; font-size: 1em; text-align: left; white-space: pre; word-spacing: normal; word-break: normal; overflow-wrap: normal; tab-size: 4; hyphens: none; vertical-align: middle;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: var(--codeColor); padding: 0px; margin: 0px; font-size: 0.9em; background-color: transparent; border-radius: 0px;"><span class="token comment" style="color: rgb(153, 153, 153);">/**
 * @author frx
 * @version 1.0
 * @date 2022/1/23  14:22
 */</span>
<span class="token annotation punctuation" style="color: rgb(204, 204, 204);">@Component</span>  <span class="token comment" style="color: rgb(153, 153, 153);">//普通组件</span>
<span class="token keyword" style="color: rgb(204, 153, 205);">public</span> <span class="token keyword" style="color: rgb(204, 153, 205);">class</span> <span class="token class-name" style="color: rgb(248, 197, 85);">FirstInterceptor</span> <span class="token keyword" style="color: rgb(204, 153, 205);">implements</span> <span class="token class-name" style="color: rgb(248, 197, 85);">HandlerInterceptor</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">{</span>
    <span class="token comment" style="color: rgb(153, 153, 153);">//在控制器方法执行之前执行</span>
    <span class="token annotation punctuation" style="color: rgb(204, 204, 204);">@Override</span>
    <span class="token keyword" style="color: rgb(204, 153, 205);">public</span> <span class="token keyword" style="color: rgb(204, 153, 205);">boolean</span> <span class="token function" style="color: rgb(240, 141, 73);">preHandle</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletRequest</span> request<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletResponse</span> response<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Object</span> handler<span class="token punctuation" style="color: rgb(204, 204, 204);">)</span> <span class="token keyword" style="color: rgb(204, 153, 205);">throws</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Exception</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">{</span>
        <span class="token class-name" style="color: rgb(248, 197, 85);">System</span><span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>out<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span><span class="token function" style="color: rgb(240, 141, 73);">println</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token string" style="color: rgb(126, 198, 153);">"FirstInterceptor--preHandle"</span><span class="token punctuation" style="color: rgb(204, 204, 204);">)</span><span class="token punctuation" style="color: rgb(204, 204, 204);">;</span>
        <span class="token keyword" style="color: rgb(204, 153, 205);">return</span> <span class="token boolean" style="color: rgb(240, 141, 73);">true</span><span class="token punctuation" style="color: rgb(204, 204, 204);">;</span>
    <span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>

    <span class="token comment" style="color: rgb(153, 153, 153);">//在控制器方法执行之后</span>
    <span class="token annotation punctuation" style="color: rgb(204, 204, 204);">@Override</span>
    <span class="token keyword" style="color: rgb(204, 153, 205);">public</span> <span class="token keyword" style="color: rgb(204, 153, 205);">void</span> <span class="token function" style="color: rgb(240, 141, 73);">postHandle</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletRequest</span> request<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletResponse</span> response<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Object</span> handler<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">ModelAndView</span> modelAndView<span class="token punctuation" style="color: rgb(204, 204, 204);">)</span> <span class="token keyword" style="color: rgb(204, 153, 205);">throws</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Exception</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">{</span>
        <span class="token class-name" style="color: rgb(248, 197, 85);">System</span><span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>out<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span><span class="token function" style="color: rgb(240, 141, 73);">println</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token string" style="color: rgb(126, 198, 153);">"FirstInterceptor--postHandle"</span><span class="token punctuation" style="color: rgb(204, 204, 204);">)</span><span class="token punctuation" style="color: rgb(204, 204, 204);">;</span>
    <span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>
    
    <span class="token comment" style="color: rgb(153, 153, 153);">//在视图渲染之后执行</span>
    <span class="token annotation punctuation" style="color: rgb(204, 204, 204);">@Override</span>
    <span class="token keyword" style="color: rgb(204, 153, 205);">public</span> <span class="token keyword" style="color: rgb(204, 153, 205);">void</span> <span class="token function" style="color: rgb(240, 141, 73);">afterCompletion</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletRequest</span> request<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletResponse</span> response<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Object</span> handler<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Exception</span> ex<span class="token punctuation" style="color: rgb(204, 204, 204);">)</span> <span class="token keyword" style="color: rgb(204, 153, 205);">throws</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Exception</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">{</span>
        <span class="token class-name" style="color: rgb(248, 197, 85);">System</span><span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>out<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span><span class="token function" style="color: rgb(240, 141, 73);">println</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token string" style="color: rgb(126, 198, 153);">"FirstInterceptor--afterCompletion"</span><span class="token punctuation" style="color: rgb(204, 204, 204);">)</span><span class="token punctuation" style="color: rgb(204, 204, 204);">;</span>
    <span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>
<span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>
</code></pre><div class="line-numbers-wrapper" style="position: absolute; top: 0px; width: 2.5rem; text-align: center; color: rgb(158, 158, 158); padding: 1.25rem 0px; line-height: 1.4; margin-top: 30px;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">1</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">2</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">3</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">4</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">5</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">6</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">7</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">8</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">9</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">10</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">11</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">12</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">13</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">14</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">15</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">16</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">17</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">18</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">19</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">20</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">21</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">22</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">23</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">24</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">25</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">26</span><br style="user-select: none;"></div><div class="expand icon-xiangxiajiantou iconfont" style="font-size: 16px; font-style: normal; -webkit-font-smoothing: antialiased; font-family: iconfont !important; width: 16px; height: 16px; cursor: pointer; position: absolute; z-index: 3; top: 0.8em; right: 0.5em; color: rgba(238, 255, 255, 0.8); font-weight: 900; transition: transform 0.3s ease 0s;"></div><div class="circle" style="position: absolute; top: 0.8em; left: 0.9rem; width: 12px; height: 12px; border-radius: 50%; background: rgb(252, 98, 93); box-shadow: rgb(253, 188, 64) 20px 0px, rgb(53, 205, 75) 40px 0px;"></div></div><div class="language-java line-numbers-mode" style="overflow: hidden; transition: height 0.3s ease 0s; margin-top: 0.85rem; position: relative; background-color: var(--codeBg); border-radius: 6px; padding-bottom: 0.5rem; height: 667px;"><i class="code-copy" title="复制失败" style="color: rgb(170, 170, 170); fill: rgba(238, 255, 255, 0.8); font-size: 14px; display: inline-block; cursor: pointer; position: absolute; top: 0.8rem; right: 2rem; opacity: 1;"><svg style="color:#aaa;font-size:14px" t="1572422231464" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3201" width="14" height="14"><path d="M866.461538 39.384615H354.461538c-43.323077 0-78.769231 35.446154-78.76923 78.769231v39.384616h472.615384c43.323077 0 78.769231 35.446154 78.769231 78.76923v551.384616h39.384615c43.323077 0 78.769231-35.446154 78.769231-78.769231V118.153846c0-43.323077-35.446154-78.769231-78.769231-78.769231z m-118.153846 275.692308c0-43.323077-35.446154-78.769231-78.76923-78.769231H157.538462c-43.323077 0-78.769231 35.446154-78.769231 78.769231v590.769231c0 43.323077 35.446154 78.769231 78.769231 78.769231h512c43.323077 0 78.769231-35.446154 78.76923-78.769231V315.076923z m-354.461538 137.846154c0 11.815385-7.876923 19.692308-19.692308 19.692308h-157.538461c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h157.538461c11.815385 0 19.692308 7.876923 19.692308 19.692308v39.384615z m157.538461 315.076923c0 11.815385-7.876923 19.692308-19.692307 19.692308H216.615385c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h315.076923c11.815385 0 19.692308 7.876923 19.692307 19.692308v39.384615z m78.769231-157.538462c0 11.815385-7.876923 19.692308-19.692308 19.692308H216.615385c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h393.846153c11.815385 0 19.692308 7.876923 19.692308 19.692308v39.384615z" p-id="3202"></path></svg></i><pre class="language-java codecopy-enabled" style="position: relative !important; user-select: text; line-height: 1.4; padding: 1.25rem 1.5rem 1.25rem 3.5rem; margin: 30px 0px 0.85rem; background: transparent; border-radius: 6px; overflow: auto; z-index: 1; color: rgb(204, 204, 204); text-shadow: none; font-family: Consolas, Monaco, &quot;Andale Mono&quot;, &quot;Ubuntu Mono&quot;, monospace; font-size: 1em; text-align: left; white-space: pre; word-spacing: normal; word-break: normal; overflow-wrap: normal; tab-size: 4; hyphens: none; vertical-align: middle;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: var(--codeColor); padding: 0px; margin: 0px; font-size: 0.9em; background-color: transparent; border-radius: 0px;"><span class="token comment" style="color: rgb(153, 153, 153);">/**
 * @author frx
 * @version 1.0
 * @date 2022/1/23  14:59
 */</span>
 <span class="token annotation punctuation" style="color: rgb(204, 204, 204);">@Component</span>
 <span class="token keyword" style="color: rgb(204, 153, 205);">public</span> <span class="token keyword" style="color: rgb(204, 153, 205);">class</span> <span class="token class-name" style="color: rgb(248, 197, 85);">SecondInterceptor</span> <span class="token keyword" style="color: rgb(204, 153, 205);">implements</span> <span class="token class-name" style="color: rgb(248, 197, 85);">HandlerInterceptor</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">{</span>
    <span class="token comment" style="color: rgb(153, 153, 153);">//在控制器方法执行之前执行</span>
    <span class="token annotation punctuation" style="color: rgb(204, 204, 204);">@Override</span>
    <span class="token keyword" style="color: rgb(204, 153, 205);">public</span> <span class="token keyword" style="color: rgb(204, 153, 205);">boolean</span> <span class="token function" style="color: rgb(240, 141, 73);">preHandle</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletRequest</span> request<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletResponse</span> response<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Object</span> handler<span class="token punctuation" style="color: rgb(204, 204, 204);">)</span> <span class="token keyword" style="color: rgb(204, 153, 205);">throws</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Exception</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">{</span>
        <span class="token class-name" style="color: rgb(248, 197, 85);">System</span><span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>out<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span><span class="token function" style="color: rgb(240, 141, 73);">println</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token string" style="color: rgb(126, 198, 153);">"SecondInterceptor--preHandle"</span><span class="token punctuation" style="color: rgb(204, 204, 204);">)</span><span class="token punctuation" style="color: rgb(204, 204, 204);">;</span>
        <span class="token keyword" style="color: rgb(204, 153, 205);">return</span> <span class="token boolean" style="color: rgb(240, 141, 73);">true</span><span class="token punctuation" style="color: rgb(204, 204, 204);">;</span>
    <span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>

    <span class="token comment" style="color: rgb(153, 153, 153);">//在控制器方法执行之后</span>
    <span class="token annotation punctuation" style="color: rgb(204, 204, 204);">@Override</span>
    <span class="token keyword" style="color: rgb(204, 153, 205);">public</span> <span class="token keyword" style="color: rgb(204, 153, 205);">void</span> <span class="token function" style="color: rgb(240, 141, 73);">postHandle</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletRequest</span> request<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletResponse</span> response<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Object</span> handler<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">ModelAndView</span> modelAndView<span class="token punctuation" style="color: rgb(204, 204, 204);">)</span> <span class="token keyword" style="color: rgb(204, 153, 205);">throws</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Exception</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">{</span>
        <span class="token class-name" style="color: rgb(248, 197, 85);">System</span><span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>out<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span><span class="token function" style="color: rgb(240, 141, 73);">println</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token string" style="color: rgb(126, 198, 153);">"SecondInterceptor--postHandle"</span><span class="token punctuation" style="color: rgb(204, 204, 204);">)</span><span class="token punctuation" style="color: rgb(204, 204, 204);">;</span>
    <span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>

    <span class="token comment" style="color: rgb(153, 153, 153);">//在视图渲染之后执行</span>
    <span class="token annotation punctuation" style="color: rgb(204, 204, 204);">@Override</span>
    <span class="token keyword" style="color: rgb(204, 153, 205);">public</span> <span class="token keyword" style="color: rgb(204, 153, 205);">void</span> <span class="token function" style="color: rgb(240, 141, 73);">afterCompletion</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletRequest</span> request<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">HttpServletResponse</span> response<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Object</span> handler<span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Exception</span> ex<span class="token punctuation" style="color: rgb(204, 204, 204);">)</span> <span class="token keyword" style="color: rgb(204, 153, 205);">throws</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Exception</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">{</span>
        <span class="token class-name" style="color: rgb(248, 197, 85);">System</span><span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>out<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span><span class="token function" style="color: rgb(240, 141, 73);">println</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token string" style="color: rgb(126, 198, 153);">"SecondInterceptor--afterCompletion"</span><span class="token punctuation" style="color: rgb(204, 204, 204);">)</span><span class="token punctuation" style="color: rgb(204, 204, 204);">;</span>
    <span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>
 <span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>
 </code></pre><div class="line-numbers-wrapper" style="position: absolute; top: 0px; width: 2.5rem; text-align: center; color: rgb(158, 158, 158); padding: 1.25rem 0px; line-height: 1.4; margin-top: 30px;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">1</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">2</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">3</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">4</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">5</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">6</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">7</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">8</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">9</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">10</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">11</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">12</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">13</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">14</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">15</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">16</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">17</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">18</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">19</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">20</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">21</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">22</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">23</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">24</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">25</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">26</span><br style="user-select: none;"></div><div class="expand icon-xiangxiajiantou iconfont" style="font-size: 16px; font-style: normal; -webkit-font-smoothing: antialiased; font-family: iconfont !important; width: 16px; height: 16px; cursor: pointer; position: absolute; z-index: 3; top: 0.8em; right: 0.5em; color: rgba(238, 255, 255, 0.8); font-weight: 900; transition: transform 0.3s ease 0s;"></div><div class="circle" style="position: absolute; top: 0.8em; left: 0.9rem; width: 12px; height: 12px; border-radius: 50%; background: rgb(252, 98, 93); box-shadow: rgb(253, 188, 64) 20px 0px, rgb(53, 205, 75) 40px 0px;"></div></div><div class="language-java line-numbers-mode" style="overflow: hidden; transition: height 0.3s ease 0s; margin-top: 0.85rem; position: relative; background-color: var(--codeBg); border-radius: 6px; padding-bottom: 0.5rem; height: 393px;"><i class="code-copy" title="复制失败" style="color: rgb(170, 170, 170); fill: rgba(238, 255, 255, 0.8); font-size: 14px; display: inline-block; cursor: pointer; position: absolute; top: 0.8rem; right: 2rem; opacity: 1;"><svg style="color:#aaa;font-size:14px" t="1572422231464" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3201" width="14" height="14"><path d="M866.461538 39.384615H354.461538c-43.323077 0-78.769231 35.446154-78.76923 78.769231v39.384616h472.615384c43.323077 0 78.769231 35.446154 78.769231 78.76923v551.384616h39.384615c43.323077 0 78.769231-35.446154 78.769231-78.769231V118.153846c0-43.323077-35.446154-78.769231-78.769231-78.769231z m-118.153846 275.692308c0-43.323077-35.446154-78.769231-78.76923-78.769231H157.538462c-43.323077 0-78.769231 35.446154-78.769231 78.769231v590.769231c0 43.323077 35.446154 78.769231 78.769231 78.769231h512c43.323077 0 78.769231-35.446154 78.76923-78.769231V315.076923z m-354.461538 137.846154c0 11.815385-7.876923 19.692308-19.692308 19.692308h-157.538461c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h157.538461c11.815385 0 19.692308 7.876923 19.692308 19.692308v39.384615z m157.538461 315.076923c0 11.815385-7.876923 19.692308-19.692307 19.692308H216.615385c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h315.076923c11.815385 0 19.692308 7.876923 19.692307 19.692308v39.384615z m78.769231-157.538462c0 11.815385-7.876923 19.692308-19.692308 19.692308H216.615385c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h393.846153c11.815385 0 19.692308 7.876923 19.692308 19.692308v39.384615z" p-id="3202"></path></svg></i><pre class="language-java codecopy-enabled" style="position: relative !important; user-select: text; line-height: 1.4; padding: 1.25rem 1.5rem 1.25rem 3.5rem; margin: 30px 0px 0.85rem; background: transparent; border-radius: 6px; overflow: auto; z-index: 1; color: rgb(204, 204, 204); text-shadow: none; font-family: Consolas, Monaco, &quot;Andale Mono&quot;, &quot;Ubuntu Mono&quot;, monospace; font-size: 1em; text-align: left; white-space: pre; word-spacing: normal; word-break: normal; overflow-wrap: normal; tab-size: 4; hyphens: none; vertical-align: middle;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: var(--codeColor); padding: 0px; margin: 0px; font-size: 0.9em; background-color: transparent; border-radius: 0px;"><span class="token comment" style="color: rgb(153, 153, 153);">/**
 * @author frx
 * @version 1.0
 * @date 2022/1/23  14:13
 */</span>
  <span class="token annotation punctuation" style="color: rgb(204, 204, 204);">@Controller</span>
  <span class="token keyword" style="color: rgb(204, 153, 205);">public</span> <span class="token keyword" style="color: rgb(204, 153, 205);">class</span> <span class="token class-name" style="color: rgb(248, 197, 85);">TestController</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">{</span>

    <span class="token annotation punctuation" style="color: rgb(204, 204, 204);">@RequestMapping</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token string" style="color: rgb(126, 198, 153);">"/testInterceptor"</span><span class="token punctuation" style="color: rgb(204, 204, 204);">)</span>
    <span class="token keyword" style="color: rgb(204, 153, 205);">public</span> <span class="token class-name" style="color: rgb(248, 197, 85);">String</span> <span class="token function" style="color: rgb(240, 141, 73);">testInterceptor</span><span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token punctuation" style="color: rgb(204, 204, 204);">)</span><span class="token punctuation" style="color: rgb(204, 204, 204);">{</span>
        <span class="token keyword" style="color: rgb(204, 153, 205);">return</span> <span class="token string" style="color: rgb(126, 198, 153);">"success"</span><span class="token punctuation" style="color: rgb(204, 204, 204);">;</span>
    <span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>
    <span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>

</code></pre><div class="line-numbers-wrapper" style="position: absolute; top: 0px; width: 2.5rem; text-align: center; color: rgb(158, 158, 158); padding: 1.25rem 0px; line-height: 1.4; margin-top: 30px;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">1</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">2</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">3</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">4</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">5</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">6</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">7</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">8</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">9</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">10</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">11</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">12</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">13</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">14</span><br style="user-select: none;"></div><div class="expand icon-xiangxiajiantou iconfont" style="font-size: 16px; font-style: normal; -webkit-font-smoothing: antialiased; font-family: iconfont !important; width: 16px; height: 16px; cursor: pointer; position: absolute; z-index: 3; top: 0.8em; right: 0.5em; color: rgba(238, 255, 255, 0.8); font-weight: 900; transition: transform 0.3s ease 0s;"></div><div class="circle" style="position: absolute; top: 0.8em; left: 0.9rem; width: 12px; height: 12px; border-radius: 50%; background: rgb(252, 98, 93); box-shadow: rgb(253, 188, 64) 20px 0px, rgb(53, 205, 75) 40px 0px;"></div></div><div class="language-html line-numbers-mode" style="overflow: hidden; transition: height 0.3s ease 0s; margin-top: 0.85rem; position: relative; background-color: var(--codeBg); border-radius: 6px; padding-bottom: 0.5rem; height: 326px;"><i class="code-copy" title="复制失败" style="color: rgb(170, 170, 170); fill: rgba(238, 255, 255, 0.8); font-size: 14px; display: inline-block; cursor: pointer; position: absolute; top: 0.8rem; right: 2rem; opacity: 1;"><svg style="color:#aaa;font-size:14px" t="1572422231464" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3201" width="14" height="14"><path d="M866.461538 39.384615H354.461538c-43.323077 0-78.769231 35.446154-78.76923 78.769231v39.384616h472.615384c43.323077 0 78.769231 35.446154 78.769231 78.76923v551.384616h39.384615c43.323077 0 78.769231-35.446154 78.769231-78.769231V118.153846c0-43.323077-35.446154-78.769231-78.769231-78.769231z m-118.153846 275.692308c0-43.323077-35.446154-78.769231-78.76923-78.769231H157.538462c-43.323077 0-78.769231 35.446154-78.769231 78.769231v590.769231c0 43.323077 35.446154 78.769231 78.769231 78.769231h512c43.323077 0 78.769231-35.446154 78.76923-78.769231V315.076923z m-354.461538 137.846154c0 11.815385-7.876923 19.692308-19.692308 19.692308h-157.538461c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h157.538461c11.815385 0 19.692308 7.876923 19.692308 19.692308v39.384615z m157.538461 315.076923c0 11.815385-7.876923 19.692308-19.692307 19.692308H216.615385c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h315.076923c11.815385 0 19.692308 7.876923 19.692307 19.692308v39.384615z m78.769231-157.538462c0 11.815385-7.876923 19.692308-19.692308 19.692308H216.615385c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h393.846153c11.815385 0 19.692308 7.876923 19.692308 19.692308v39.384615z" p-id="3202"></path></svg></i><pre class="language-html codecopy-enabled" style="position: relative !important; user-select: text; line-height: 1.4; padding: 1.25rem 1.5rem 1.25rem 3.5rem; margin: 30px 0px 0.85rem; background: transparent; border-radius: 6px; overflow: auto; z-index: 1; color: rgb(204, 204, 204); text-shadow: none; font-family: Consolas, Monaco, &quot;Andale Mono&quot;, &quot;Ubuntu Mono&quot;, monospace; font-size: 1em; text-align: left; white-space: pre; word-spacing: normal; word-break: normal; overflow-wrap: normal; tab-size: 4; hyphens: none; vertical-align: middle;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: var(--codeColor); padding: 0px; margin: 0px; font-size: 0.9em; background-color: transparent; border-radius: 0px;"><span class="token doctype" style="color: rgb(153, 153, 153);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;!</span><span class="token doctype-tag">DOCTYPE</span> <span class="token name">html</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>html</span> <span class="token attr-name" style="color: rgb(226, 119, 122);">lang</span><span class="token attr-value" style="color: rgb(126, 198, 153);"><span class="token punctuation attr-equals" style="color: rgb(204, 204, 204);">=</span><span class="token punctuation" style="color: rgb(204, 204, 204);">"</span>en<span class="token punctuation" style="color: rgb(204, 204, 204);">"</span></span> <span class="token attr-name" style="color: rgb(226, 119, 122);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">xmlns:</span>th</span><span class="token attr-value" style="color: rgb(126, 198, 153);"><span class="token punctuation attr-equals" style="color: rgb(204, 204, 204);">=</span><span class="token punctuation" style="color: rgb(204, 204, 204);">"</span>http://www.thymeleaf.org<span class="token punctuation" style="color: rgb(204, 204, 204);">"</span></span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>head</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
    <span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>meta</span> <span class="token attr-name" style="color: rgb(226, 119, 122);">charset</span><span class="token attr-value" style="color: rgb(126, 198, 153);"><span class="token punctuation attr-equals" style="color: rgb(204, 204, 204);">=</span><span class="token punctuation" style="color: rgb(204, 204, 204);">"</span>UTF-8<span class="token punctuation" style="color: rgb(204, 204, 204);">"</span></span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
    <span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>title</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>Title<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;/</span>title</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;/</span>head</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>body</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>h1</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>首页<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;/</span>h1</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>a</span> <span class="token attr-name" style="color: rgb(226, 119, 122);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">th:</span>href</span><span class="token attr-value" style="color: rgb(126, 198, 153);"><span class="token punctuation attr-equals" style="color: rgb(204, 204, 204);">=</span><span class="token punctuation" style="color: rgb(204, 204, 204);">"</span>@{/testInterceptor}<span class="token punctuation" style="color: rgb(204, 204, 204);">"</span></span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>测试拦截器<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;/</span>a</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>br</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;/</span>body</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;/</span>html</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>`
</code></pre><div class="line-numbers-wrapper" style="position: absolute; top: 0px; width: 2.5rem; text-align: center; color: rgb(158, 158, 158); padding: 1.25rem 0px; line-height: 1.4; margin-top: 30px;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">1</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">2</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">3</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">4</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">5</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">6</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">7</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">8</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">9</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">10</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">11</span><br style="user-select: none;"></div><div class="expand icon-xiangxiajiantou iconfont" style="font-size: 16px; font-style: normal; -webkit-font-smoothing: antialiased; font-family: iconfont !important; width: 16px; height: 16px; cursor: pointer; position: absolute; z-index: 3; top: 0.8em; right: 0.5em; color: rgba(238, 255, 255, 0.8); font-weight: 900; transition: transform 0.3s ease 0s;"></div><div class="circle" style="position: absolute; top: 0.8em; left: 0.9rem; width: 12px; height: 12px; border-radius: 50%; background: rgb(252, 98, 93); box-shadow: rgb(253, 188, 64) 20px 0px, rgb(53, 205, 75) 40px 0px;"></div></div><div class="language-html line-numbers-mode" style="overflow: hidden; transition: height 0.3s ease 0s; margin-top: 0.85rem; position: relative; background-color: var(--codeBg); border-radius: 6px; padding-bottom: 0.5rem; height: 304px;"><i class="code-copy" title="复制失败" style="color: rgb(170, 170, 170); fill: rgba(238, 255, 255, 0.8); font-size: 14px; display: inline-block; cursor: pointer; position: absolute; top: 0.8rem; right: 2rem; opacity: 1;"><svg style="color:#aaa;font-size:14px" t="1572422231464" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3201" width="14" height="14"><path d="M866.461538 39.384615H354.461538c-43.323077 0-78.769231 35.446154-78.76923 78.769231v39.384616h472.615384c43.323077 0 78.769231 35.446154 78.769231 78.76923v551.384616h39.384615c43.323077 0 78.769231-35.446154 78.769231-78.769231V118.153846c0-43.323077-35.446154-78.769231-78.769231-78.769231z m-118.153846 275.692308c0-43.323077-35.446154-78.769231-78.76923-78.769231H157.538462c-43.323077 0-78.769231 35.446154-78.769231 78.769231v590.769231c0 43.323077 35.446154 78.769231 78.769231 78.769231h512c43.323077 0 78.769231-35.446154 78.76923-78.769231V315.076923z m-354.461538 137.846154c0 11.815385-7.876923 19.692308-19.692308 19.692308h-157.538461c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h157.538461c11.815385 0 19.692308 7.876923 19.692308 19.692308v39.384615z m157.538461 315.076923c0 11.815385-7.876923 19.692308-19.692307 19.692308H216.615385c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h315.076923c11.815385 0 19.692308 7.876923 19.692307 19.692308v39.384615z m78.769231-157.538462c0 11.815385-7.876923 19.692308-19.692308 19.692308H216.615385c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h393.846153c11.815385 0 19.692308 7.876923 19.692308 19.692308v39.384615z" p-id="3202"></path></svg></i><pre class="language-html codecopy-enabled" style="position: relative !important; user-select: text; line-height: 1.4; padding: 1.25rem 1.5rem 1.25rem 3.5rem; margin: 30px 0px 0.85rem; background: transparent; border-radius: 6px; overflow: auto; z-index: 1; color: rgb(204, 204, 204); text-shadow: none; font-family: Consolas, Monaco, &quot;Andale Mono&quot;, &quot;Ubuntu Mono&quot;, monospace; font-size: 1em; text-align: left; white-space: pre; word-spacing: normal; word-break: normal; overflow-wrap: normal; tab-size: 4; hyphens: none; vertical-align: middle;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: var(--codeColor); padding: 0px; margin: 0px; font-size: 0.9em; background-color: transparent; border-radius: 0px;"><span class="token doctype" style="color: rgb(153, 153, 153);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;!</span><span class="token doctype-tag">DOCTYPE</span> <span class="token name">html</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>html</span> <span class="token attr-name" style="color: rgb(226, 119, 122);">lang</span><span class="token attr-value" style="color: rgb(126, 198, 153);"><span class="token punctuation attr-equals" style="color: rgb(204, 204, 204);">=</span><span class="token punctuation" style="color: rgb(204, 204, 204);">"</span>en<span class="token punctuation" style="color: rgb(204, 204, 204);">"</span></span> <span class="token attr-name" style="color: rgb(226, 119, 122);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">xmlns:</span>th</span><span class="token attr-value" style="color: rgb(126, 198, 153);"><span class="token punctuation attr-equals" style="color: rgb(204, 204, 204);">=</span><span class="token punctuation" style="color: rgb(204, 204, 204);">"</span>http://www.thymeleaf.org<span class="token punctuation" style="color: rgb(204, 204, 204);">"</span></span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>head</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
    <span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>meta</span> <span class="token attr-name" style="color: rgb(226, 119, 122);">charset</span><span class="token attr-value" style="color: rgb(126, 198, 153);"><span class="token punctuation attr-equals" style="color: rgb(204, 204, 204);">=</span><span class="token punctuation" style="color: rgb(204, 204, 204);">"</span>UTF-8<span class="token punctuation" style="color: rgb(204, 204, 204);">"</span></span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
    <span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>title</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>Title<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;/</span>title</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;/</span>head</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;</span>body</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
success
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;/</span>body</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
<span class="token tag" style="color: rgb(226, 119, 122);"><span class="token tag" style="color: rgb(226, 119, 122);"><span class="token punctuation" style="color: rgb(204, 204, 204);">&lt;/</span>html</span><span class="token punctuation" style="color: rgb(204, 204, 204);">&gt;</span></span>
</code></pre><div class="line-numbers-wrapper" style="position: absolute; top: 0px; width: 2.5rem; text-align: center; color: rgb(158, 158, 158); padding: 1.25rem 0px; line-height: 1.4; margin-top: 30px;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">1</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">2</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">3</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">4</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">5</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">6</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">7</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">8</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">9</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">10</span><br style="user-select: none;"></div><div class="expand icon-xiangxiajiantou iconfont" style="font-size: 16px; font-style: normal; -webkit-font-smoothing: antialiased; font-family: iconfont !important; width: 16px; height: 16px; cursor: pointer; position: absolute; z-index: 3; top: 0.8em; right: 0.5em; color: rgba(238, 255, 255, 0.8); font-weight: 900; transition: transform 0.3s ease 0s;"></div><div class="circle" style="position: absolute; top: 0.8em; left: 0.9rem; width: 12px; height: 12px; border-radius: 50%; background: rgb(252, 98, 93); box-shadow: rgb(253, 188, 64) 20px 0px, rgb(53, 205, 75) 40px 0px;"></div></div></details>

- 结果输出顺序

```java
FirstInterceptor--preHandle
SecondInterceptor--preHandle
SecondInterceptor--postHandle
FirstInterceptor--postHandle
SecondInterceptor--afterCompletion
FirstInterceptor--afterCompletion
```

- 若FirstInterceptor-preHandle返回True,SecondInterceptor--preHandle返回False,结果为

```java
FirstInterceptor--preHandle
SecondInterceptor--preHandle
FirstInterceptor--afterCompletion
```

## 自定义拦截器

1. 定义一个自定义拦截实现 HandlerInterceptor 接口，重写三个方法。

```java
/**
 * @author frx
 * @version 1.0
 * @date 2022/4/25  23:05
 */
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("1-----preHandle----->");
        return true;
    }
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("3-----postHandle----->");
    }
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("4-----afterCompletion----->");
    }
}
```

2. 定义一个配置类实现 WebMvcConfigurer 接口，重写 addInterceptors 方法，注册拦截器并定义规则。

```java
/**
 * @author frx
 * @version 1.0
 * @date 2022/4/25  23:12
 */
@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {
    @Bean
    public LoginInterceptor getLoginInterceptor() {
        return new LoginInterceptor();
    }
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(getLoginInterceptor()).addPathPatterns("/api/**");
    }
}
```

3. 定义测试接口。

```java
/**
 * @author frx
 * @version 1.0
 * @date 2022/4/25  23:13
 */
@RestController
@Slf4j
public class HelloController {
    @GetMapping("/api/hello")
    public String hello() {
        log.info("2-----hello----->");
        return "hello";
    }
}
```

4. 结果

```java
2022-04-25 23:21:05.142  INFO 19276 --- [nio-8888-exec-5] c.f.i.config.LoginInterceptor            : 1-----preHandle----->
2022-04-25 23:21:05.143  INFO 19276 --- [nio-8888-exec-5] c.f.i.controller.HelloController         : 2-----hello----->
2022-04-25 23:21:05.148  INFO 19276 --- [nio-8888-exec-5] c.f.i.config.LoginInterceptor            : 3-----postHandle----->
2022-04-25 23:21:05.148  INFO 19276 --- [nio-8888-exec-5] c.f.i.config.LoginInterceptor    
```

## 单个拦截器执行流程

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/20220423/image.2yt4trspxew0.webp)

## 多个拦截器的执行流程

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/20220423/image.3ukw3z3zmk60.webp)

## 基于配置的异常处理

SpringMVC提供了一个处理控制器方法执行过程中所出现的异常的接口：HandlerExceptionResolver

HandlerExceptionResolver接口的实现类有：DefaultHandlerExceptionResolver和SimpleMappingExceptionResolver

SpringMVC提供了自定义的异常处理器SimpleMappingExceptionResolver，使用方式：

```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
        <props>
        	<!--
        		properties的键表示处理器方法执行过程中出现的异常
        		properties的值表示若出现指定异常时，设置一个新的视图名称，跳转到指定页面
        	-->
            <prop key="java.lang.ArithmeticException">error</prop>
        </props>
    </property>
    <!--
    	exceptionAttribute属性设置一个属性名，将出现的异常信息在请求域中进行共享
    -->
    <property name="exceptionAttribute" value="ex"></property>
</bean>
```

## 基于注解的异常处理

```java
//@ControllerAdvice将当前类标识为异常处理的组件
@ControllerAdvice
public class ExceptionController {

    //@ExceptionHandler用于设置所标识方法处理的异常
    @ExceptionHandler(ArithmeticException.class)
    //ex表示当前请求处理中出现的异常对象
    public String handleArithmeticException(Exception ex, Model model){
        model.addAttribute("ex", ex);
        return "error";
    }

}
```

# 注解配置SpringMVC

## 创建初始化类，代替web.xml

在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContainerInitializer接口的类，如果找到的话就用它来配置Servlet容器。 Spring提供了这个接口的实现，名为SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer的类并将配置的任务交给它们来完成。Spring3.2引入了一个便利的WebApplicationInitializer基础实现，名为AbstractAnnotationConfigDispatcherServletInitializer，当我们的类扩展了AbstractAnnotationConfigDispatcherServletInitializer并将其部署到Servlet3.0容器的时候，容器会自动发现它，并用它来配置Servlet上下文。

```java
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {

    /**
     * 指定spring的配置类
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    /**
     * 指定SpringMVC的配置类
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    /**
     * 指定DispatcherServlet的映射规则，即url-pattern
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    /**
     * 添加过滤器
     * @return
     */
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
        encodingFilter.setEncoding("UTF-8");
        encodingFilter.setForceRequestEncoding(true);
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
        return new Filter[]{encodingFilter, hiddenHttpMethodFilter};
    }
}
```

## 创建SpringConfig配置类，代替spring的配置文件

```java
@Configuration
public class SpringConfig {
	//ssm整合之后，spring的配置信息写在此类中
}
```

## 创建WebConfig配置类，代替SpringMVC的配置文件

```java
@Configuration
//扫描组件
@ComponentScan("com.atguigu.mvc.controller")
//开启MVC注解驱动
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    //使用默认的servlet处理静态资源
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //配置文件上传解析器
    @Bean
    public CommonsMultipartResolver multipartResolver(){
        return new CommonsMultipartResolver();
    }

    //配置拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        FirstInterceptor firstInterceptor = new FirstInterceptor();
        registry.addInterceptor(firstInterceptor).addPathPatterns("/**");
    }
    
    //配置视图控制
    
    /*@Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
    }*/
    
    //配置异常映射
    /*@Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
        Properties prop = new Properties();
        prop.setProperty("java.lang.ArithmeticException", "error");
        //设置异常映射
        exceptionResolver.setExceptionMappings(prop);
        //设置共享异常信息的键
        exceptionResolver.setExceptionAttribute("ex");
        resolvers.add(exceptionResolver);
    }*/

    //配置生成模板解析器
    @Bean
    public ITemplateResolver templateResolver() {
        WebApplicationContext webApplicationContext = ContextLoader.getCurrentWebApplicationContext();
        // ServletContextTemplateResolver需要一个ServletContext作为构造参数，可通过WebApplicationContext 的方法获得
        ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(
                webApplicationContext.getServletContext());
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setCharacterEncoding("UTF-8");
        templateResolver.setTemplateMode(TemplateMode.HTML);
        return templateResolver;
    }

    //生成模板引擎并为模板引擎注入模板解析器
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        return templateEngine;
    }

    //生成视图解析器并未解析器注入模板引擎
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");
        viewResolver.setTemplateEngine(templateEngine);
        return viewResolver;
    }


}
```

# SpringMVC执行流程

## pringMVC常用组件

- DispatcherServlet：**前端控制器**，不需要工程师开发，由框架提供

作用：统一处理请求和响应，整个流程控制的中心，由它调用其它组件处理用户的请求

- HandlerMapping：**处理器映射器**，不需要工程师开发，由框架提供

作用：根据请求的url、method等信息查找Handler，即控制器方法

- Handler：**处理器**，需要工程师开发

作用：在DispatcherServlet的控制下Handler对具体的用户请求进行处理

- HandlerAdapter：**处理器适配器**，不需要工程师开发，由框架提供

作用：通过HandlerAdapter对处理器（控制器方法）进行执行

- ViewResolver：**视图解析器**，不需要工程师开发，由框架提供

作用：进行视图解析，得到相应的视图，例如：ThymeleafView、InternalResourceView、RedirectView

- View：**视图**

作用：将模型数据通过页面展示给用户

## DispatcherServlet初始化过程

DispatcherServlet 本质上是一个 Servlet，所以天然的遵循 Servlet 的生命周期。所以宏观上是 Servlet 生命周期来进行调度。

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/SpringMVC/images/11/01.png)

### 初始化WebApplicationContext

所在类：org.springframework.web.servlet.FrameworkServlet



```java
protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext =
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;

    if (this.webApplicationContext != null) {
        // A context instance was injected at construction time -> use it
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {
                // The context has not yet been refreshed -> provide services such as
                // setting the parent context, setting the application context id, etc
                if (cwac.getParent() == null) {
                    // The context instance was injected without an explicit parent -> set
                    // the root application context (if any; may be null) as the parent
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        // No context instance was injected at construction time -> see if one
        // has been registered in the servlet context. If one exists, it is assumed
        // that the parent context (if any) has already been set and that the
        // user has performed any initialization such as setting the context id
        wac = findWebApplicationContext();
    }
    if (wac == null) {
        // No context instance is defined for this servlet -> create a local one
        // 创建WebApplicationContext
        wac = createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        // Either the context is not a ConfigurableApplicationContext with refresh
        // support or the context injected at construction time had already been
        // refreshed -> trigger initial onRefresh manually here.
        synchronized (this.onRefreshMonitor) {
            // 刷新WebApplicationContext
            onRefresh(wac);
        }
    }

    if (this.publishContext) {
        // Publish the context as a servlet context attribute.
        // 将IOC容器在应用域共享
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
    }

    return wac;
}
```

### 创建WebApplicationContext

所在类：org.springframework.web.servlet.FrameworkServlet



```java
protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
    Class<?> contextClass = getContextClass();
    if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
        throw new ApplicationContextException(
            "Fatal initialization error in servlet with name '" + getServletName() +
            "': custom WebApplicationContext class [" + contextClass.getName() +
            "] is not of type ConfigurableWebApplicationContext");
    }
    // 通过反射创建 IOC 容器对象
    ConfigurableWebApplicationContext wac =
        (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);

    wac.setEnvironment(getEnvironment());
    // 设置父容器
    wac.setParent(parent);
    String configLocation = getContextConfigLocation();
    if (configLocation != null) {
        wac.setConfigLocation(configLocation);
    }
    configureAndRefreshWebApplicationContext(wac);

    return wac;
}
```

### DispatcherServlet初始化策略

FrameworkServlet创建WebApplicationContext后，刷新容器，调用onRefresh(wac)，此方法在DispatcherServlet中进行了重写，调用了initStrategies(context)方法，初始化策略，即初始化DispatcherServlet的各个组件

所在类：org.springframework.web.servlet.DispatcherServlet

```java
protected void initStrategies(ApplicationContext context) {
   initMultipartResolver(context);
   initLocaleResolver(context);
   initThemeResolver(context);
   initHandlerMappings(context);
   initHandlerAdapters(context);
   initHandlerExceptionResolvers(context);
   initRequestToViewNameTranslator(context);
   initViewResolvers(context);
   initFlashMapManager(context);
}
```

## DispatcherServlet调用组件处理请求

### processRequest()

FrameworkServlet重写HttpServlet中的service()和doXxx()，这些方法中调用了processRequest(request, response)

所在类：org.springframework.web.servlet.FrameworkServlet



```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    initContextHolders(request, localeContext, requestAttributes);

    try {
		// 执行服务，doService()是一个抽象方法，在DispatcherServlet中进行了重写
        doService(request, response);
    }
    catch (ServletException | IOException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (Throwable ex) {
        failureCause = ex;
        throw new NestedServletException("Request processing failed", ex);
    }

    finally {
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }
        logResult(request, response, failureCause, asyncManager);
        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```

### doService()

所在类：org.springframework.web.servlet.DispatcherServlet



```java
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    logRequest(request);

    // Keep a snapshot of the request attributes in case of an include,
    // to be able to restore the original attributes after the include.
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    // Make framework objects available to handlers and view objects.
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    if (this.flashMapManager != null) {
        FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
        if (inputFlashMap != null) {
            request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
        }
        request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
        request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
    }

    RequestPath requestPath = null;
    if (this.parseRequestPath && !ServletRequestPathUtils.hasParsedRequestPath(request)) {
        requestPath = ServletRequestPathUtils.parseAndCache(request);
    }

    try {
        // 处理请求和响应
        doDispatch(request, response);
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Restore the original attribute snapshot, in case of an include.
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
        if (requestPath != null) {
            ServletRequestPathUtils.clearParsedRequestPath(request);
        }
    }
}
```

### doDispatch()

所在类：org.springframework.web.servlet.DispatcherServlet



```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // Determine handler for the current request.
            /*
            	mappedHandler：调用链
                包含handler、interceptorList、interceptorIndex
            	handler：浏览器发送的请求所匹配的控制器方法
            	interceptorList：处理控制器方法的所有拦截器集合
            	interceptorIndex：拦截器索引，控制拦截器afterCompletion()的执行
            */
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // Determine handler adapter for the current request.
           	// 通过控制器方法创建相应的处理器适配器，调用所对应的控制器方法
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            // Process last-modified header, if supported by the handler.
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }
			
            // 调用拦截器的preHandle()
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // Actually invoke the handler.
            // 由处理器适配器调用具体的控制器方法，最终获得ModelAndView对象
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(processedRequest, mv);
            // 调用拦截器的postHandle()
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            // As of 4.3, we're processing Errors thrown from handler methods as well,
            // making them available for @ExceptionHandler methods and other scenarios.
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        // 后续处理：处理模型数据和渲染视图
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                               new NestedServletException("Handler processing failed", err));
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // Instead of postHandle and afterCompletion
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```

### processDispatchResult()



```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
                                   @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
                                   @Nullable Exception exception) throws Exception {

    boolean errorView = false;

    if (exception != null) {
        if (exception instanceof ModelAndViewDefiningException) {
            logger.debug("ModelAndViewDefiningException encountered", exception);
            mv = ((ModelAndViewDefiningException) exception).getModelAndView();
        }
        else {
            Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
            mv = processHandlerException(request, response, handler, exception);
            errorView = (mv != null);
        }
    }

    // Did the handler return a view to render?
    if (mv != null && !mv.wasCleared()) {
        // 处理模型数据和渲染视图
        render(mv, request, response);
        if (errorView) {
            WebUtils.clearErrorRequestAttributes(request);
        }
    }
    else {
        if (logger.isTraceEnabled()) {
            logger.trace("No view rendering, null ModelAndView returned.");
        }
    }

    if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
        // Concurrent handling started during a forward
        return;
    }

    if (mappedHandler != null) {
        // Exception (if any) is already handled..
        // 调用拦截器的afterCompletion()
        mappedHandler.triggerAfterCompletion(request, response, null);
    }
}
```



##  SpringMVC的执行流程

1. 用户向服务器发送请求，请求被SpringMVC 前端控制器 DispatcherServlet捕获。
2. DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI），判断请求URI对应的映射：

- 不存在
  1. 再判断是否配置了mvc:default-servlet-handler
  2. 如果没配置，则控制台报映射查找不到，客户端展示404错误

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/SpringMVC/images/11/02.png)

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/SpringMVC/images/11/03.png)

 如果有配置，则访问目标资源（一般为静态资源，如：JS,CSS,HTML），找不到客户端也会展示404错误

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/SpringMVC/images/11/04.png)

![01](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting@master/studynotes/SpringMVC/images/11/05.png)

- 存在则执行下面的流程

1. 根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain执行链对象的形式返回。
2. DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。
3. 如果成功获得HandlerAdapter，此时将开始执行拦截器的preHandler(…)方法【正向】
4. 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)方法，处理请求。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：
   - HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息
   - 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等
   - 数据格式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等
   - 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中
5. Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象。
6. 此时将开始执行拦截器的postHandle(...)方法【逆向】。
7. 根据返回的ModelAndView（此时会判断是否存在异常：如果存在异常，则执行HandlerExceptionResolver进行异常处理）选择一个适合的ViewResolver进行视图解析，根据Model和View，来渲染视图。
8. 渲染视图完毕执行拦截器的afterCompletion(…)方法【逆向】。
9. 将渲染结果返回给客户端。