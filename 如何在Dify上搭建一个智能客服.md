

在[[Agent和Workflow的关系是什么]]」中我们描述了Agent和Workflow，现在AI工具迭代非常快，Dify在后续的产品中对Agent、Workflow做出了进一步的定义，然后还引入了`Chatflow`，一个`Agent`是一个基本的大语言模型对话对象，而`Chatflow`在`Agent`的基础上多了其他例如问题判断、代码执行、流程控制等多种功能，在本文将阐述如何在Dify平台中搭建一个基本的，具备知识库检索功能的对话流（Chatflow）。

## 搭建RAG知识库

```ad-note
title: RAG（检索增强生成Retrieval-Augmented Generation）

检索增强生成（Retrieval-Augmented Generation，简称RAG），就是日常销售中所称的**知识库**，它实际上是一种结合信息检索与生成式人工智能模型的技术，旨在提升大型语言模型（LLM）的响应准确性和可靠性，使得生成回答时可以参考本地知识库中的文件内容。

**RAG的工作流程：**

1. **数据索引**：也就是Embedding，是将外部知识库中的数据转换为向量表示，并存储在向量数据库中。这些数据可以是非结构化、半结构化或结构化的文本、图像等。

2. **检索**：当用户提出查询时，系统根据查询向量从向量数据库中检索出最相关的文档或信息片段。

3. **增强**：将检索到的信息与用户的原始查询相结合，形成增强后的输入。

4. **生成**：利用语言模型基于增强后的输入生成回答，确保回答包含最新或特定领域的信息。


**RAG知识库的组成：**

- **数据源**：包含领域相关的文档、知识图谱等信息，用于支持模型生成准确的回答，这里主要是指本地上传的文件内容。

- **向量数据库**：存储数据源的向量表示，支持高效的相似度检索。

通过这种方式，RAG知识库使生成式模型能够动态地访问和利用用户自己文件中的内容进行答案生成。

以下是一个典型的RAG索引生成回答的过程的示意图：

![[Pasted image 20250310150523.png]]
```

### 在知识库中添加文档
在Dify中构建一个知识库的方法非常简单，点击用户界面上方的`知识库`，选择数据源，上传我们准备好的文件，然后点击下一步进入知识库的配置界面。


![[iShot_2025-05-21_08.56.34.png]]

```ad-caution
注意，在此处我们选择的是文本的方式，Dify也提供了其他的方式导入数据，特别是第三个`同步自Web站点`，可以直接通过网址的方式，将网址中的内容作为知识库，天翼云提供了比较详细的产品文档，其网址在`https://ctyun.cn/document`，但是天翼云的网站有严格的反爬虫机制，所以这种通过爬虫的方式不能很好的获取到有效信息，故本次实验用的是提前准备好的材料。
```


### 知识库配置

知识库的配置选择默认的配置即可满足绝大部分需求，需要手动设置的是`Embdding模型`，关于如何添加`Embedding模型`，请参考[[在Dify中如何添加语言模型]]，在这里选择索引方式为`高质量`，然后选择我们添加好的Embedding模型，此处为`BGE-m3`。

最后点击完成，就可以在知识库的界面中看到我们添加的文档了。

![[iShot_2025-05-21_08.57.13.png]]


## 配置Chatflow

```ad-caution
Chatflow是Dify在后续产品更新后添加的模式，与Workflow不同，Chatflow是一种对话框架，特别显著的区别在于，Chatflow的开始节点中有`sys.query`变量，该变量也就是对话中输入的问题，会提示用户输入问题，而Workflow中没有这个参数，Workflow更偏向于完成一项自动化任务。
```

整个流程的概览如下图所示，本文后续内容将介绍如何创建一个这一个的Chatflow。
![[iShot_2025-05-21_13.44.18 1.png]]


### 从一个空白应用开始创建

Step 1. 在用户界面上方点击`工作室`，点击`创建空白应用`

![[iShot_2025-05-21_11.37.41.png]]

Step 2. 选择类型为`Chatflow`，并填写有关基本信息

![[iShot_2025-05-21_09.55.25.png]]

然后就会看到一个最基本的流程界面：
![[iShot_2025-05-21_09.57.48.png]]

接下来就可以开始配置该客服机器人的聊天流程。

###  配置天翼云智能客服Chatflow

##### 配置问题分类器
添加问题分类器的作用是排除不需要回答的问题，减少调用LLM的次数。如果不相关的问题可以直接回复，而不用调用LLM。

Step 1. 添加一个问题分类器节点

![[iShot_2025-05-21_11.44.49.png]]

Step 2. 设置分类器的类别

![[iShot_2025-05-21_09.59.29.png]]

Step 3. 配置分类器的提示词

![[iShot_2025-05-21_10.02.13.png]]

分类器其实也是一个大语言模型，在第一个截图中展示了这个分类器使用的大语言模型是`Qwen3-32B`，所以对于这个大语言模型就可以输入提示词指示如何进行分类，以下是我填写的提示词：

```
现在你作为一个问题分类专家，需要对客户提出的问题进行分类，如果用户的提问和弹性云主机有关，那么请将问题标记为“与弹性云主机有关”，否则请标记为“与弹性云主机无关”
```

还需要注意的是，在提示词的设置框中需要填写上下文，这里传入的上下文参数是最刚开始的问题，也就是`sys.query`。

```ad-info
title: 什么是上下文（Context）
在计算机领域很多地方都有上下文（Context）这个概念，实际上这个名词的含义就是指在一个任务中所涉及的信息。在大模型中，上下文是指模型在当前对话或任务中“记住”的信息，用于理解你的输入和生成响应。

一个语言模型的上下文通常包含以下几类信息：

1. **当前对话内容（提示词，也就是Prompt）**
    
    - 包括你输入的文本 + 模型之前生成的内容（多轮对话）。
        
    - 模型会基于这些内容来理解你现在的输入是什么意思。
        
2. **系统设定信息（System Prompt）**
    
    - 例如 ChatGPT 的角色设定、行为风格、指令约束等。
        
    - 这些信息也属于上下文的一部分。
        
3. **外部知识/记忆（也就是Rag知识库，以后还会涉及到MCP的信息，但这是后话目前不涉及）**
    
    - 像 ChatGPT 中的“记忆”功能，是一种长期上下文。
        
    - 用户手动提供的文档、数据库、API返回的数据也可以作为上下文嵌入进去。


每个模型处理上下文是有限的，比如：

- GPT-4-32k：其含义是模型`GPT-4`最多可处理约 32,000 个 token（相当于约 24,000 个英文词）。超出这个范围的内容将被模型遗忘。
```

#### 对分类后的不同分支添加处理节点

##### Flow one
Step 1. 首先对第二个类别，也就是`与弹性云主机无关`添加一个直接回复的节点，这条流程比较简单我们先设置这条。
![[iShot_2025-05-21_10.05.06.png]]

Step 2. 在直接回复的节点中，添加上需要直接回复的内容：

![[iShot_2025-05-21_10.05.39.png]]

##### Flow two
Step 1. 对第一个类别，也就是`与弹性云主机有关`的分支，添加一个`知识检索`节点。

![[iShot_2025-05-21_10.08.13.png]]

`知识检索`节点的作用，就是在我们所制作的知识库中检索需要的信息。

![[iShot_2025-05-21_10.09.30.png]]

在`知识检索`节点的设置界面中，将`查询变量`设置为`sys.query`，`知识库`设置为我们创建的知识库。

Step 2. 在`知识检索`节点之后，添加一个`LLM`节点
![[iShot_2025-05-21_10.10.10.png]]

`LLM`就是Large Language Model，其作用就是调用大语言模型对问题进行回答。
![[iShot_2025-05-21_10.14.26.png]]

在`LLM`节点的配置界面，将`模型`选定为我们添加的模型（此处为`Qwen3-32B`），在System中输入有关的上下文和提示词。这里选择的上下文是`sys.query`，以下是我所使用的提示词：

```ad-note
title: LLM提示词
- Role: 天翼云专家客服

- Background: 客户提出了关于天翼云的问题，需要以天翼云专家客服的身份，根据知识库中的内容回答这些问题。

- Profile: 你是一位专业的天翼云专家客服，对天翼云的产品和服务有着深入的了解，能够根据知识库中的内容为客户提供准确、专业的解答。

- Skills: 你具备丰富的天翼云产品知识、技术背景和客户服务经验，能够快速理解客户问题并提供针对性的答案。

- Goals: 根据知识库中的内容，为客户提供准确、专业的解答，确保客户的问题得到满意的解决。

- Constrains: 回答应基于知识库中的内容，确保信息的准确性和专业性。

- OutputFormat: 输出应为简洁明了的答案，直接针对客户的问题进行回复。

- Workflow:

1. 仔细阅读客户的问题。

2. 根据知识库中的内容，查找与问题相关的信息。

3. 提供准确、专业的答案。
```

Step 3. 在`LLM`节点之后添加一个`直接回复`节点

![[iShot_2025-05-21_13.42.33.png]]

回复的内容中，引用`LLM/text`变量，这表示直接将上一个`LLM`节点的输出内容回复给用户。

### 整体框架

最后我们得到的整个流程如下图所示：

![[iShot_2025-05-21_13.44.18.png]]


### 简单的测试

测试一：我们输入了一个“你好”，被问题分类器判定为`与弹性云主机无关`，所以直接给出了回复，这符合预期。

![[iShot_2025-05-21_10.18.16.png]]

测试二：我们询问了一下“弹性云主机是什么”，然后可以看到该问题被判定为`与弹性云主机有关`，所以走了第一个流程，先是在知识库中进行了检索，然后调用了LLM回答有关问题，这也符合预期。

![[iShot_2025-05-21_11.14.46.png]]