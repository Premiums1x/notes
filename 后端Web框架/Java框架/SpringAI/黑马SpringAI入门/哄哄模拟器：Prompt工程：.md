Prompt工程：通过优化提示词，使大模型输出的内容尽可能理想。
**几个要点**：
![[Pasted image 20260526190742.png]]


使用qwen大模型（原因是DeepSeek模型的思考太多，而这个模块无需思考过程），因springAI不支持，需使用OpenAI规范API：
- 引依赖：引OpenAI的而非Ollama
- 配模型：用openai选项而非ollama
```java
<groupId>org.springframework.ai</groupId>  
<artifactId>spring-ai-starter-model-openai</artifactId>
```

KEY环境变量配置：
![[Pasted image 20260526192916.png]]




- 配客户端：（CommonConfig）![[Pasted image 20260526191254.png]]
用OpenAichatModel:
```java
@Bean  
public ChatClient gameChatClient(OpenAiChatModel model, ChatMemory chatMemory){  
    return ChatClient  
            .builder(model)//创建工厂，传入模型  
            .defaultSystem(SystemConstants.GAME_PROMPT)//默认系统提示词  
            .defaultAdvisors(  
                    new SimpleLoggerAdvisor(),  
                    MessageChatMemoryAdvisor.builder(chatMemory).build() )//日志环绕增强  
            .build();//创建客户端  
}
```
就换了OpenAiChatModel，以及将提示词封在一个类的静态final String中。

新建GameController
