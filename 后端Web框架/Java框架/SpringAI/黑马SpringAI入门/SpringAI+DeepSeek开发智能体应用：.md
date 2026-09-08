### AI机制：
![[Pasted image 20260523151953.png]]
关键：Transformer：用于大模型中，可以根据上文内容进行推理预测，从而得出下文。


#### 模型部署：
- 本地
- 云服务器
- API调用

#### System Prompt/ User Prompt:
![[Pasted image 20260523153911.png]]
携带系统提示词去给大模型身份、约束大模型的回答


#### 调用大模型：（底层就是通过http发送请求）
![[Pasted image 20260523154207.png]]

Ollma启动后监听端口11434，我们可利用该端口暴露的api去调用服务。

 注意请求方法是POST ：
![[Pasted image 20260523155339.png]]

#### 理解AI大模型应用与传统应用：
![[Pasted image 20260523160038.png]]

#### 大模型应用领域：
![[Pasted image 20260523160308.png]]


#### AI应用开发技术架构：
整体四种方向：
![[Pasted image 20260523161722.png]]


1. Prompt问答
![[Pasted image 20260523161654.png]]
2. Function Calling
![[Pasted image 20260523162022.png]]
注：函数调用时应用本身执行，只是告诉大模型调用函数所需的一些参数等等，大模型返回给应用去调用。

3. RAG，外挂知识库：
![[Pasted image 20260523162336.png]]

4. Fine-tuning：模型微调，用特定参数等去训练调整大模型，让其满足业务需求。

#### JAVA开发框架
SpringAI与Langchain4j（基于Python的Langchain框架）
对比如下：
![[Pasted image 20260523162857.png]]



