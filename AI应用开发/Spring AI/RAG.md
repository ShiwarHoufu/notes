# ETL Class Diagram

![[etl-class-diagram.jpg|700]]
ETL 流水线包含三大核心组件：
- **DocumentReader（文档读取器）**：实现 `Supplier<List<Document>>`
- **DocumentTransformer（文档转换器）**：实现 `Function<List<Document>, List<Document>>`
- **DocumentWriter（文档写入器）**：实现 `Consumer<List<Document>>`

## Document

Document 是 Spring AI 里所有 RAG 流程的统一数据单元。
文档读取、切分、向量化、入库、检索、拼 prompt，流转的都是 Document。
```java
public class Document {  
    public static final ContentFormatter DEFAULT_CONTENT_FORMATTER = DefaultContentFormatter.defaultConfig();  
  
	private final String id;  // 默认随机UUID
	
	// text和media 二选一，不能都为空，也不能都有
	private final @Nullable String text;  // 文本内容
	private final @Nullable Media media;  // 图片、音频等媒体内容
	
	// 元数据，参与过滤和展。(写入数据库时变为 payload)
    private final Map<String, Object> metadata;  
    // 创建和入库时通常为null，检索时由向量库填上
	private final @Nullable Double score; 
```
![[Pasted image 20261025171702.png]]

## DocumentTransformer

### TextSplitter（文档分片器）

`TextSplitter` 是 RAG 文档ETL阶段的一个抽象类，负责把一篇长文档切分成多个小块（chunk）
**TokenTextSplitter** 是它的一个子类：**按 Token 数量切割文本**（不是按字符）
```java
TextSplitter splitter = TokenTextSplitter.builder()
        .withChunkSize(500)                // 目标块大小：500 token
        .withMinChunkSizeChars(100)        // 单个分片最少 100 字符，不够就不切
        .withMinChunkLengthToEmbed(20)     // 分片字符小于20，直接丢弃，不入库
        .withMaxNumChunks(1000)             // 一份文档最多切出1000个分片
        .withKeepSeparator(true)            // 保留分割符（换行/句号等），语义更连贯
        .build();
```
- **目标块大小（chunkSize）**：是切割时的参考，告诉分割器**我希望每一块多大**。
- **MinChunkSizeChars**：防止切出很短、语义残缺的小碎片；尽量保证分片具备完整语义。
> 它只是合并，不会丢弃，短的片段会并入前一块。
- **withMinChunkLengthToEmbed**：所有分片切完之后，再做一次兜底过滤。短文本向量噪声极大，很容易造成误召回、脏数据。