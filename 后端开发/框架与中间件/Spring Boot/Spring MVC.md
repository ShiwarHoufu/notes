Model（模型）+ View（视图）+ Controller（控制器）

Spring MVC 是基于 **Servlet API** 的 MVC Web 框架，用来开发 Java Web 后端接口 / 网页，SpringBoot Web 底层默认就是它。

> **Servlet** 是 Java EE 规范里的一套接口，用来在服务端**接收处理 HTTP请求，生成 HTTP响应**。
> Tomcat（Web容器）实现了这套规范，负责：创建Servlet实例、调用方法、管理生命周期。

# 核心组件

1. **DispatcherServlet 前端控制器**
	所有请求先进这里。负责调度所有组件。
2. **HandlerMapping 处理器映射器** 
	在项目启动的时候，扫描所有 `@RequestMapping`，记录`url`与`Handler`的映射关系
3. **HandlerAdapter 处理器适配器** 
	适配不同写法的 Handler，调用 Handler 
4. **Handler 控制器**
	写业务、接收参数、封装数据；
5. **Model 模型** 
	存放要传给页面的数据（POJO 对象）。
6. **ViewResolver 视图解析器** 
	把返回的视图名字（如`"index"`）拼成真实页面路径（`/WEB-INF/jsp/index.jsp`）
7. **View 视图** 
	渲染页面（现在前后端分离一般不用视图，直接返回 JSON）
8. **HandlerExceptionResolver 异常解析器** 
	统一捕获请求流程里的异常。
9. **MultipartResolver 文件上传解析器** 
	解析 multipart 表单，支持上传文件。

# 核心注解

**@RestController**= @Controller + @ResponseBody

1. **@Controller**：把当前类标记成一个 Spring MVC 的控制器，表示这个类处理 Web 请求。这个类里面带`@PostMapping/@GetMapping`这类注解的方法，都是 MVC 中的**Handler**

2. **@ResponseBody** ：类中所有方法返回值**不交给视图解析器**，而是通过消息转换器序列化成 **JSON字符串**，写入 HttpServletResponse 响应体返回给前端。
