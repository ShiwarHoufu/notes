JUnit 是 **Java 生态最主流、开源的单元测试框架**（xUnit 家族），由 Kent Beck、Erich Gamma 开发，用来写**自动化单元测试**，对单个方法 / 类做白盒测试，是 Java 开发事实标准。

**使用场景：**
- 配合 IDEA  直接右键运行测试
- 集成 Maven / Gradle，CI 流水线自动执行，生成测试报告
- 配合 [[Mockito]] 做 mock 测试
# 一、Junit5 核心

## 注解

| 注解            | 作用                  |
| ------------- | ------------------- |
| `@Test`       | 标记这是一条测试方法          |
| `@BeforeEach` | **每个测试方法执行前**执行     |
| `@AfterEach`  | **每个测试方法执行后**执行     |
| `@BeforeAll`  | 所有测试方法**只执行一次（静态）** |
| `@AfterAll`   | 所有测试方法**只执行一次（静态）** |
| `@Disabled`   | 跳过该测试               |
## 断言（Assertain）

用来判断实际结果是否等于预期，失败直接抛出测试失败：

- `assertEquals(预期值,实际值)`：相等
- `assertTrue(布尔表达式)`：条件为真
- `assertNull(obj)` / `assertNotNull(obj)`：判空
- `assertThrows(异常.class, ()->{...})`：断言抛出指定异常（JUnit5 新增

## Maven项目指令
```bash
# 执行所有测试
mvn test

# 只执行指定测试类
mvn test -Dtest=DemoTest

# 只执行某个测试方法
mvn test -Dtest=DemoTest#testAdd

# 执行多个测试类，逗号分隔
mvn test -Dtest=DemoTest,UserTest
```