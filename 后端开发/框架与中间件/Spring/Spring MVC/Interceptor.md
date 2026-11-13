拦截器(Interceptor) 是 **Spring MVC 自身提供**的一种请求拦截机制，用于对请求进行统一的**预处理**和**后处理**。

处于 DispatcherServlet 内部；在 Handler 已经确定之后、调用 Controller 方法之前介入。

常见应用场景：
- 登录校验、权限判断、日志记录、耗时统计、防重复提交等。

核心接口是 `org.springframework.web.servlet.HandlerInterceptor`,包含三个方法:
```java
public interface HandlerInterceptor {

    // 目标方法执行前调用。返回 false 则中断请求,不再往下走
    default boolean preHandle(HttpServletRequest request,
                              HttpServletResponse response,
                              Object handler) throws Exception {
        return true;
    }

    // 目标方法执行完、视图渲染前调用(此时 ModelAndView 还没渲染)
    default void postHandle(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler,
                            @Nullable ModelAndView modelAndView) throws Exception {
    }

    // 整个请求完成(视图渲染完毕)后调用,类似 finally,可拿到异常
    default void afterCompletion(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Object handler,
                                 @Nullable Exception ex) throws Exception {
    }
}
```

多个拦截器组成一个**拦截器链**，**栈式结构**: