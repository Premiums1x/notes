RAG原理：类似于请求时携带额外数据（知识）
RAG文档检索：把数据根据语义，通过算法将其转成一组数字，再将这组数字映射成空间中的坐标，确保内容相似的，向量相似度越高。
![[Pasted image 20260607103523.png]]


SpringAI配置向量模型：
![[Pasted image 20260607103848.png]]


### 单元测试Embedding模型：
注：单元测试要配置API-KEY：
![[Pasted image 20260607105057.png]]

后续是写个工具类去得到欧式相似度和余弦相似度，接着写一个测试方法：写入一条文本数据模拟提示词，以及一个数组模拟向量库，比较他们之间的相似度。



### 向量知识库：用于管理转为向量后的文本片段，并检索出与用户提示词有关的部分，给到提示词一起发给大模型。
![[Pasted image 20260607112407.png]]


配置Redis stack
![[Pasted image 20260607112734.png]]


### 所有向量数据库都实现自VectorStore接口：
逻辑即为增删查，仅每种数据库实现不一样：
```java
public interface VectorStore extends DocumentWriter {

    default String getName() {
		return this.getClass().getSimpleName();
	}

    void add(List<Document> documents);

    void delete(List<String> idList);

    void delete(Filter.Expression filterExpression);

    default void delete(String filterExpression) { ... };

    List<Document> similaritySearch(String query);

    List<Document> similaritySearch(SearchRequest request);

    default <T> Optional<T> getNativeClient() {
		return Optional.empty();
	}
}
```


#### 将文件向量化，存入向量数据库的过程：
![[Pasted image 20260607161735.png]]


#### 读取、拆分文档，转变为`List<Document>`：
![[Pasted image 20260607162419.png]]
引依赖、用实体类。


### 测试向量库实现：
演示用SimpleVectorStore实现（内存实现），并未使用真实向量库：
`CommonConfiguration`，添加一个`VectorStore`的Bean：`

引入`PagePdfDocumentReader` PDF文件读取依赖

创建单元测试方法：`testVectorStore`
1. PDF读取转换为`List<Document>`
2. 写入向量库`vectorStore.add(docs);`
3. 查询，遍历打印结果（SearchRequest是对query进行一层包装，携带更多参数以限制返回）
4. 还有个filterExpression方法，用来避免读取文件冲突，可以配置的值为metadata里的键值对。


运行单元测试报错：`No Such Method`：
解决：
```md
（1）根因： spring-boot-starter-test 从 Spring Boot 3.4+ 开始将 junit-platform-launcher 标记为 optional，不再自动传递。IDE 使用自带的 Launcher（编译自 JUnit Platform 1.12.x，该版本使用
  getMethodParameterTypes() 方法），但项目中实际运行的 junit-platform-engine:6.0.3 已将此方法重命名为 getParameterTypeNames()，导致 NoSuchMethodError。

  修复： 在 pom.xml 中显式添加了匹配版本的 junit-platform-launcher 依赖：

  <dependency>
      <groupId>org.junit.platform</groupId>
      <artifactId>junit-platform-launcher</artifactId>
      <scope>test</scope>
  </dependency>

  额外修复： spring-beans 依赖之前被错误地限制为 <scope>test</scope>，导致主代码编译失败，已修正为默认的 compile 作用域。

（2）Maven 依赖树显示所有 JUnit jar 版本都是 5.9.1 / 1.9.1（内部一致），但
  IntelliJ IDEA 内置了较新的 JUnit Platform Launcher（1.10+），它期望
  MethodSelector 有 getMethodParameterTypes() 方法，而你项目的
  junit-platform-engine:1.9.1 没有这个方法。版本不匹配导致报错。

  修复办法：升级 junit-jupiter 到 5.10.x 以匹配 IntelliJ 内置的 JUnit Platform。
```



## 项目实现：
![[Pasted image 20260612124437.png]]
需求：
- 会话ID除了与用户的对话记录关联外还需与上传文件关联：上传文件时建立会话ID与文件的映射关系
- 跟据文档内容进行对话即RAG技术


### SpringAI对RAG技术的简化：
![[Pasted image 20260612144714.png]]
无需手动将用户query向量化然后去向量库检索，再手动拼接提示词。

为chatPDF配置Client：
1. RAG 环绕advisor
![[Pasted image 20260612145004.png]]

2. 对话检索
![[Pasted image 20260612145316.png]]
