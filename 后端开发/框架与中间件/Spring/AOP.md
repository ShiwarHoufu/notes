# 概念
（面向切面编程，Aspect-Oriented Programming）

AOP 是一种编程思想，用来把和业务逻辑无关的通用代码抽离出来，统一织入到目标方法执行的各个节点，不修改原有业务代码。

> OOP（面向对象）是按**对象 / 模块**划分；AOP 是按**横切关注点**划分。

**好处：**
- 减少重复代码
- 不侵入原有代码
- 维护方便

| 术语                | 含义                                                            |
| ----------------- | ------------------------------------------------------------- |
| **切面 Aspect**     | 横切逻辑所在的**类**，里面封装通知 + 切入点                                     |
| **通知 Advice**     | 切面里要执行的动作，最终体现为一个方法。分 5 种：                                    |
| **切入点 Pointcut**  | 定义哪些类 / 哪些方法需要被切面拦截（表达式，如 `execution(* com.service.*.*(..))`） |
| **连接点 JoinPoint** | 切入点匹配成功之后，**被拦截到的每一个方法**本身                                    |
| **织入 Weaving**    | 把切面代码插入到目标方法的过程；Spring AOP 是运行期**动态代理**织入                     |
| **目标对象 Target**   | 被切面增强的原始对象                                                    |
| **代理对象 Proxy**    | 织入切面逻辑之后生成的代理对象，对外暴露                                          |
# 通知
## 种类

- Before：目标方法**执行前**执行
- After：目标方法**结束后**（无论成功异常）执行
- AfterReturning：目标方法**正常返回后**执行
- AfterThrowing：目标方法**抛出异常时**执行
- **Around**：**环绕通知**，包裹目标方法。可以控制要不要执行目标方法。最常用

代码示例：
```java
@Aspect
@Component
public class LogAspect {

    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}

    @Around("serviceMethods()")
    public Object log(ProceedingJoinPoint pjp) throws Throwable {
        // pjp 就是本次命中的连接点
        System.out.println("调用: " + pjp.getSignature().getName()
                + " 参数: " + Arrays.toString(pjp.getArgs()));
        return pjp.proceed();
    }
}
```
> 1. 单独加 @Aspect 不会生效，必须配合 `@Component` 交给 Spring 管理；同时需要开启 AOP
> 2. 环绕通知返回值必须要用 **Object** ，用来接收原始方法的返回值
> 3. 不调用 `ProceedingJoinPoint.proceed()` → 原方法直接不执行（权限拦截就可以这样做）

## **通知顺序**

- **同一个切面中**，不同类型的通知有固定的优先级排序：
	`@Around` > `@Before` > `@After` > `@AfterReturning` > `@AfterThrowing`

- **不同切面**的通知匹配同一个连接点时，如需控制顺序，可以指定优先级：
	在切面类上添加 `@Order` 注解，指定数值

Spring 遵循进入时优先级高的先执行，退出时优先级高的后执行的**栈式规则**


# Thought

1. 为什么**校验token**不用 AOP？
	   Interceptor 比 切 Controller 的 AOP **更早执行**。token 校验放在越早越好，提前拦截无效请求，不用走到 AOP 代理那一步，节省资源。