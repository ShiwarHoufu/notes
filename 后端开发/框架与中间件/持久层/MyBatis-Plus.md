# 核心机制
## 一、通用单表 CRUD 不需要写 SQL

```java
public interface BaseMapper<T> {
    int insert(T entity);
    int deleteById(Serializable id);
    T selectById(Serializable id);
    List<T> selectList(Wrapper<T> queryWrapper);
    int updateById(T entity);
    // ... 还有十几个
}
```
1. 扫描到`@Mapper` 接口，发现它继承了`BaseMapper<User>`
2. 读`User` 实体上的`@TableName` （表名 user）、`@TableId` （主键）等注解，建立“实体字段 ↔ 表列”的映射
3. 用内置的 SQL 注入器， 在运行时自动生成并注册`INSERT INTO user (id, username, ...) VALUES (?, ?, ...)` 这样预生成的 SQL