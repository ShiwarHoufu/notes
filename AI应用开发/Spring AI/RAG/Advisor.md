
# RetrievalAugmentationAdvisor

**RetrievalAugmentationAdvisor** 基于模块化架构，把 RAG 流水线拆成独立可替换**模块**：查询改写、多查询扩展、文档召回、文档过滤、上下文组装等。
```xml
<dependency>
   <groupId>org.springframework.ai</groupId>
   <artifactId>spring-ai-rag</artifactId>
</dependency>
```

RetrievalAugmentationAdvisor 实现了 `baseAdvisor`，为最常用的 RAG 流程提供 *开箱即用* 的实现。以下内容就根据源码展开，分析其逻辑。

## “组件”一览
```java
private final List<QueryTransformer> queryTransformers;  // 查询转换器列表
private final @Nullable QueryExpander queryExpander;     // 查询扩展器
private final DocumentRetriever documentRetriever;       // 文档检索器
private final DocumentJoiner documentJoiner;             // 文档融合器
private final List<DocumentPostProcessor> documentPostProcessors; // 文档后处理器列表
private final QueryAugmenter queryAugmenter;             // 查询增强器
```
## before()

8 个标准步骤：原始查询构建 → 查询转换 → 查询扩展 → 并行检索 → 结果融合 → 文档后处理 → 提示词增强 → 请求回写

### 1. 查询转换（Query Transformation）
```java
Query transformedQuery = originalQuery;
for (var queryTransformer : this.queryTransformers) {
    transformedQuery = queryTransformer.apply(transformedQuery);
}
```
- 责任链模式：**按顺序执行多个查询转换器**，对原始用户问题做优化，提升检索准确率。
- 常见实现：
	- 查询改写（把口语化问题改写成更适合检索的句式）；
	- 关键词提取；
	- query 纠错、 压缩；
	- 多语言翻译；

### 2. 查询扩展（Query Expansion）
```java
List<Query> expandedQueries = this.queryExpander != null 
    ? this.queryExpander.expand(transformedQuery) 
    : List.of(transformedQuery);
```
- 将单个用户查询扩展为多个查询，从单路检索升级为**多路召回，提升召回率**。
- 常见实现：
    - 多查询生成（用 LLM 基于原问题生成 3-5 个同义的查询）
    - 混合查询扩展（同时生成向量查询 + 关键词查询）
    - 子问题拆分（把复杂问题拆成多个子问题分别检索）

### 3. 并行多路检索
```java
Map<Query, List<List<Document>>> documentsForQuery = expandedQueries.stream()
    .map(query -> CompletableFuture.supplyAsync(() -> getDocumentsForQuery(query), this.taskExecutor))
    .toList()
    .stream()
    .map(CompletableFuture::join)
    .collect(Collectors.toMap(Map.Entry::getKey, entry -> List.of(entry.getValue())));
```

- **并行优化**：通过 `CompletableFuture` + 自定义线程池 `taskExecutor`，对多个查询同时发起检索，降低耗时。
- `List<List<Document>>`：支持多数据源混合检索。内层 List 对应**单个数据源**返回的文档列表，外层代表多个数据源。

### 4. 多源结果融合（Document Joiner）
```java
List<Document> documents = this.documentJoiner.join(documentsForQuery);
```

- 把“多个查询 × 多个数据源”的零散文档结果合并、排序、去重，输出单一的文档列表。
- 常见融合算法：
    - RRF（倒数排名融合）：常用的无监督融合算法，适配多路检索结果打分口径不一致的场景
    - 加权融合：给不同数据源 / 查询配置权重
    - 简单拼接：基础场景直接合并去重

### 5. 文档后处理
```java
for (var documentPostProcessor : this.documentPostProcessors) {
    documents = documentPostProcessor.process(originalQuery, documents);
}
context.put(DOCUMENT_CONTEXT, documents);
```
- 责任链模式，对初筛文档做处理，提升 RAG 准确率。
- 常见处理器：
    - 重排序（Rerank）：用跨编码器模型对文档做精准语义排序，提升召回精度
    - Token 截断：按大模型上下文窗口限制，裁剪文档长度
    - 元数据过滤：按权限、时间、业务标签过滤无效文档
    - 冗余去重：去掉内容高度重复的片段
- 处理完成后，将最终文档存入请求上下文 `context`，供后续链路复用。
> 这里传入的是 `originalQuery` 而非转换后的查询，保证后处理对齐用户真实意图。

### 6. 查询增强（Prompt Augmentation）
```java
Query augmentedQuery = this.queryAugmenter.augment(originalQuery, documents);
```
- 将检索到的文档内容格式化，拼接到用户问题中，构造最终给大模型的提示词。比如：
```plaintext
请基于以下资料回答用户问题，不要编造信息：
---
【文档1】xxx
【文档2】xxx
 ---
用户问题：{originalQuery.text()}
```

### 7. 请求回写
```java
return chatClientRequest.mutate()
    .prompt(chatClientRequest.prompt().augmentUserMessage(augmentedQuery.text()))
    .context(context)
    .build();
```
- 把增强后的用户消息和携带文档的上下文写回请求，后续 `ChatClient` 会带着这个增强后的 prompt 调用大模型。

## after()


## 总结

 整体来看，这套设计是典型的**管道 - 过滤器架构**，面向接口的可插拔组件化，业务方可以按需替换任意环节的实现（比如更换向量库、替换重排序模型、自定义查询改写），不需要修改主流程代码，具备很好的扩展性和可维护性。