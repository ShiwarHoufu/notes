
把 Tool 注册到 chatClient：
```java
ChatClient chatClient = chatClientBuilder  
       .defaultTools()  
       .defaultAdvisors() 
       .build();
```
`defaultTools` 可以接收：
- **ToolCallback 实例**
- **ToolCallbackProvider**
- 带 **@Tool** 注解的 POJO 对象


# ToolCallBack

`ToolCallback` 是 Spring AI 中**工具调用机制的核心抽象接口**。
每个工具最终都会被包装成一个 `ToolCallback` 实例。

```java
public interface ToolCallback {
    // 返回工具定义（名称、描述、输入参数的 JSON Schema），供模型决策
    ToolDefinition getToolDefinition();
    // 接收模型生成的 JSON 参数，执行工具逻辑，返回字符串结果
    String call(String toolInput);
    
    // 返回工具的元数据（如 returnDirect 等执行策略）
    default ToolMetadata getToolMetadata() {
        return ToolMetadata.builder().build();
}
```
**两个核心实现类:**
- **`MethodToolCallback`**：用于将**一个 Java 方法**包装为工具（`@Tool`注解的底层实现）。
- **`FunctionToolCallback`**：用于将**一个函数式接口**包装为工具。

## 如何获得 ToolCallBack

1. **@Tool注解（最常用）**：在方法上标记 `@Tool`，Spring AI 会自动将其适配为 **MethodToolCallback**
```java
static class MyTools {  
    @Tool(description = "Get the current weather for a given location")  
    public String weather(String location) {  
       return "The current weather in " + location + " is sunny with a temperature of 25°C.";  
    }  
}
```
1. **函数式**：直接通过 `FunctionToolCallback.builder(...)` 编程式构建，将 Lambda 或方法引用快速转为工具。
```java
FunctionToolCallback weatherTool = FunctionToolCallback
        .builder("weather", (String location) -> {
            return "The current weather in " + location + " is sunny with a        temperature of 25°C.";})
        .description("Get the current weather for a given location")
        .inputType(String.class)
        .build();
```
1. **显式定义**：直接声明 `ToolCallback` 类型的 **Bean**。
```java
    @Bean
    public ToolCallback weatherToolCallback() {
        return FunctionToolCallback
                .builder("weather", (String location) -> {
                    return "The current weather in " + location + " is sunny with  a temperature of 25°C.";})
                .description("Get the current weather for a given location")
                .inputType(String.class)
                .build();
    }
```

# ToolCallBackProvider

代表**一组工具的集合**，它的核心职责是向框架**批量**提供 `ToolCallback` 实例。

`ToolCallbackProvider` 是一个**函数式接口**:
```java
public interface ToolCallbackProvider {  
  
    // 具体实现类负责实现这个方法，从而批量提供ToolCallback
    ToolCallback[] getToolCallbacks();  
  
	// 静态工厂：从现成的 List 或数组快速创建 Provider
    static ToolCallbackProvider from(List<? extends ToolCallback> toolCallbacks) { 
       return new StaticToolCallbackProvider(toolCallbacks);  
    }
    static ToolCallbackProvider from(ToolCallback... toolCallbacks) {  
       return new StaticToolCallbackProvider(toolCallbacks);  
    }
}
```
所以可以在`defaultTools()`中传入 provider；框架会在需要时调用其 **getToolCallbacks()** 方法来获取工具集合。
