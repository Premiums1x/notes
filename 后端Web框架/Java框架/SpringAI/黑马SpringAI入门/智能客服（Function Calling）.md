需求：
![[Pasted image 20260606155202.png]]

分清大模型和传统程序的任务职责。


## 原生Function Calling(Tool)实现:
1.定义Tool，
2.chatClient配置Tool，
3.对话

大模型分析是否要调用函数，将函数名和参数传给ai应用来调用，然后将结果返回给大模型，最后分析出结果
![[Pasted image 20260606155346.png]]

SpringAI:
![[Pasted image 20260606155707.png]]
只需定义函数内容，其自动将其拼接入字符串发给大模型，需要调用时拦截识别到然后之间调用，最后将结果转成String返回。

关键：配置Tool：
![[Pasted image 20260606160812.png]]
留意注解的使用

使用MybatisPlus插件，先配置好数据库连接，然后直接生成代码：
![[Pasted image 20260606163018.png|313]]

插件的数据库连接配置：
![[Pasted image 20260606163436.png]]
主要写好数据库后面要加serverTimezone参数，不然jdbc驱动无法识别MySQL服务器时区。


直接生成数据库相关代码：
![[Pasted image 20260606163735.png]]
**排错**：实际项目这里有多个模块，要指定module，package没错。

成功生成代码：
![[Pasted image 20260606165003.png]]


### 定义查询条件实体：
字段：
- edu：例如学生学历是高中，则查询时要满足 edu <= 2
    
- type：学生的学习兴趣，要跟类型精确匹配，type = '自媒体'
    
- price：学生对价格敏感，则查询时需要按照价格升序排列：order by price asc
    
- duration: 学生对学习时长敏感，则查询时要按照时长升序：order by duration asc

在entity的query包下，创建课程查询参数实体类：
```java
package com.lancer.ai.entity.query;  
  
  
import lombok.Data;  
import org.springframework.ai.tool.annotation.ToolParam;  
  
import java.util.List;  
  
@Data  
public class CourseQuery {  
    @ToolParam(required = false, description = "课程类型：编程、设计、自媒体、其它")  
    private String type;  
    @ToolParam(required = false, description = "学历要求：0-无、1-初中、2-高中、3-大专、4-本科及本科以上")  
    private Integer edu;  
    @ToolParam(required = false, description = "排序方式")  
    private List<Sort> sorts;  
  
    @Data  
    public static class Sort {  
        @ToolParam(required = false, description = "排序字段: price或duration")  
        private String field;  
        @ToolParam(required = false, description = "是否是升序: true/false")  
        private Boolean asc;  
    }  
}
```

### 写Tool： 
tool包，写一个类，其中用@Tool注解声明方法，@ToolParam声明方法参数
Tool和Param都需要有描述信息，需要被携带给大模型，进行分析，在合适时调用。
![[Pasted image 20260606175721.png]]


### 配置Tool：
在commonconfig中再配置个新的chat客户端
![[Pasted image 20260606175822.png]]
注：这里用new是为了方便理解，实际已经通过bean注入，不要写new。

### 写Contorller处理前端请求：
![[Pasted image 20260606181917.png]]
复用部分chatrobot代码，需要改变chatClient等等。


### 给启动类加上MapperScan


### 完善前端后开始测试：
问题：
流式传输模式下，springAI调用tool时的传参我们配置的是兼容openAI平台的格式，但实际使用的百炼平台的模型时，通过流模式处理function calling会有不兼容问题

即：服务端需要从大模型的response里拿到需要去调用的Tool，即对应定义好的function和参数，然后进行函数调用，出错就在于百炼的模型对Tools函数解析返回时，返回六条结果，其实是把参数分段返回了...（解析出错的后果，OpenAI规范是仅一条）。
![[Pasted image 20260606201556.png]]

把参数拆开了一样：
![[Pasted image 20260606201733.png]]

解决方法：
- 改用阻塞传输：call().

- 自行重写openaiChatModel：
`Generation buildGeneration`中如果choice.message为null就返回空list，不为空就把choice.message做stream流遍历，通过map的方式把每条消息映射成成一个个ToolCall对象，**现在就是用reduce去做前后合并**：注意只是参数被拆成六个ToolCall对象，所以一开始的id和type都可以一直保留，只对支离破碎的arguments参数做拼接，

首先，我们自己写一个遵循阿里巴巴百炼平台接口规范的`ChatModel`，其中大部分代码来自SpringAI的`OpenAiChatModel`，只需要重写接口协议不匹配的地方即可，重写部分会以黄色高亮显示。

新建一个`AlibabaOpenAiChatModel`类：
```java
.reduce((tc1, tc2) -> new AssistantMessage.ToolCall(tc1.id(), "function", tc1.name(), tc1.arguments() + tc2.arguments()))
 //改写部分：仅对拆散的参数作拼接
```

还需要：把`AliababaOpenAiChatModel`配置到Spring容器。
修改`CommonConfiguration`，添加配置：xxx，但是原OpenAIChatModel的构造方法基本都已弃用，所以这个类需使用工厂builder方法：拷贝springAI的官方代码并进行改造，使得自行改写的AlibabaChatModel具备builder工厂函数功能。
```java
```java
@Bean
public AlibabaOpenAiChatModel alibabaOpenAiChatModel(OpenAiConnectionProperties commonProperties, OpenAiChatProperties chatProperties, ObjectProvider<RestClient.Builder> restClientBuilderProvider, ObjectProvider<WebClient.Builder> webClientBuilderProvider, ToolCallingManager toolCallingManager, RetryTemplate retryTemplate, ResponseErrorHandler responseErrorHandler, ObjectProvider<ObservationRegistry> observationRegistry, ObjectProvider<ChatModelObservationConvention> observationConvention) {
    String baseUrl = StringUtils.hasText(chatProperties.getBaseUrl()) ? chatProperties.getBaseUrl() : commonProperties.getBaseUrl();
    String apiKey = StringUtils.hasText(chatProperties.getApiKey()) ? chatProperties.getApiKey() : commonProperties.getApiKey();
    String projectId = StringUtils.hasText(chatProperties.getProjectId()) ? chatProperties.getProjectId() : commonProperties.getProjectId();
    String organizationId = StringUtils.hasText(chatProperties.getOrganizationId()) ? chatProperties.getOrganizationId() : commonProperties.getOrganizationId();
    Map<String, List<String>> connectionHeaders = new HashMap<>();
    if (StringUtils.hasText(projectId)) {
        connectionHeaders.put("OpenAI-Project", List.of(projectId));
    }

    if (StringUtils.hasText(organizationId)) {
        connectionHeaders.put("OpenAI-Organization", List.of(organizationId));
    }
    RestClient.Builder restClientBuilder = restClientBuilderProvider.getIfAvailable(RestClient::builder);
    WebClient.Builder webClientBuilder = webClientBuilderProvider.getIfAvailable(WebClient::builder);
    OpenAiApi openAiApi = OpenAiApi.builder().baseUrl(baseUrl).apiKey(new SimpleApiKey(apiKey)).headers(CollectionUtils.toMultiValueMap(connectionHeaders)).completionsPath(chatProperties.getCompletionsPath()).embeddingsPath("/v1/embeddings").restClientBuilder(restClientBuilder).webClientBuilder(webClientBuilder).responseErrorHandler(responseErrorHandler).build();
    AlibabaOpenAiChatModel chatModel = AlibabaOpenAiChatModel.builder().openAiApi(openAiApi).defaultOptions(chatProperties.getOptions()).toolCallingManager(toolCallingManager).retryTemplate(retryTemplate).observationRegistry((ObservationRegistry)observationRegistry.getIfUnique(() -> ObservationRegistry.NOOP)).build();
    Objects.requireNonNull(chatModel);
    observationConvention.ifAvailable(chatModel::setObservationConvention);
    return chatModel;
}

```

最后，让之前的`ChatClient`都使用自定义的`AlibabaOpenAiChatModel`，修改`CommonConfiguration`中的ChatClient配置。

### 测试结果：
![[Pasted image 20260606190000.png]]

表中成功写入数据
![[Pasted image 20260606185936.png]]