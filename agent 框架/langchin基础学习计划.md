# LangChain 与 LLM 系统学习方案

## 1. 学习目标

通过系统学习，掌握从大模型基础调用到 LangChain 工程化开发的完整流程，形成以下能力：

1. 理解 LLM、Embedding、Token、上下文窗口、流式输出、结构化输出等基础概念。
2. 掌握 LangChain 的核心组件，包括 Model、Prompt、Chain、Output Parser、Tool、Agent。
3. 掌握 RAG 检索增强生成的基本流程。
4. 掌握 Agent 调用工具完成业务操作的开发方式。
5. 掌握 LangGraph 中状态、节点、边、条件分支等流程编排思想。
6. 能够完成一个基于 LangChain 的智能助手项目模块。

------

# 2. 学习路线总览

整体路线分为 7 个阶段：

```text
第一阶段：LLM 基础
第二阶段：LangChain 基础组件
第三阶段：Prompt 与结构化输出
第四阶段：RAG 检索增强生成
第五阶段：Agent 与 Tools
第六阶段：LangGraph 流程编排
第七阶段：项目实战整合
```

------

# 3. 第一阶段：LLM 基础

## 3.1 学习内容

1. LLM 是什么
2. Chat Model 是什么
3. Embedding Model 是什么
4. Token 是什么
5. 上下文窗口是什么
6. Temperature、top_p、max_tokens 的作用
7. System Message、User Message、Assistant Message 的作用
8. 流式输出是什么
9. 结构化输出是什么
10. Tool Calling 是什么

## 3.2 学习重点

这一阶段重点理解大模型应用的基本组成：

```text
用户输入
→ Prompt
→ 大模型
→ 模型输出
```

以及：

```text
文本
→ Embedding Model
→ 向量
→ 向量数据库
→ 相似度检索
```

## 3.3 练习内容

1. 调用一次大模型接口。
2. 实现普通问答。
3. 实现流式输出。
4. 让模型按 JSON 格式返回内容。
5. 实现一个简单的工具调用示例。

------

# 4. 第二阶段：LangChain 基础组件

## 4.1 学习内容

1. ChatModel
2. PromptTemplate
3. ChatPromptTemplate
4. Runnable
5. RunnableSequence
6. OutputParser
7. Structured Output
8. Messages
9. Chat History

## 4.2 学习重点

LangChain 的基础流程：

```text
输入变量
→ Prompt 模板
→ Model
→ Output Parser
→ 最终结果
```

核心代码形式：

```ts
const chain = prompt.pipe(model)
const result = await chain.invoke({ question: "什么是闭包？" })
```

## 4.3 练习内容

1. 创建一个 ChatModel。
2. 创建一个 ChatPromptTemplate。
3. 使用 `prompt.pipe(model)` 组成 Chain。
4. 使用 `chain.invoke()` 执行调用。
5. 使用 OutputParser 解析模型输出。
6. 使用 Structured Output 生成 JSON 数据。

------

# 5. 第三阶段：Prompt 与结构化输出

## 5.1 学习内容

1. System Prompt 设计
2. User Prompt 设计
3. Few-shot 示例
4. 输出格式约束
5. JSON Schema
6. Zod 参数校验
7. 意图识别
8. 参数提取
9. 缺失参数处理
10. 错误输出处理

## 5.2 学习重点

重点掌握自然语言到结构化参数的转换。

例如用户输入：

```text
帮我把 React 基础课程的课时改成 30
```

模型输出：

```json
{
  "intent": "update_course",
  "courseName": "React 基础",
  "field": "duration",
  "value": 30,
  "needConfirm": true
}
```

## 5.3 练习内容

1. 编写意图识别 Prompt。
2. 编写参数提取 Prompt。
3. 使用 Zod 校验输出结构。
4. 处理缺少课程名的情况。
5. 处理缺少修改字段的情况。
6. 区分查询操作、新增操作、修改操作和删除操作。

------

# 6. 第四阶段：RAG 检索增强生成

## 6.1 学习内容

1. Document Loader
2. Text Splitter
3. Chunk
4. Embedding
5. Vector Store
6. Retriever
7. topK 检索
8. Metadata
9. 上下文拼接
10. 引用来源返回

## 6.2 学习重点

RAG 的基本流程：

```text
文档上传
→ 文本解析
→ 文本切分
→ 生成向量
→ 存入向量数据库
→ 用户提问
→ 检索相关片段
→ 拼接上下文
→ 模型生成答案
→ 返回引用来源
```

## 6.3 练习内容

1. 读取 Markdown 文档。
2. 读取 PDF 文档。
3. 将文档切分为多个 chunk。
4. 生成 embedding。
5. 存入向量数据库。
6. 根据问题检索 topK 文档片段。
7. 将检索结果拼接到 Prompt 中。
8. 生成带引用来源的回答。

------

# 7. 第五阶段：Agent 与 Tools

## 7.1 学习内容

1. Tool 定义
2. Tool 参数 schema
3. Tool Calling
4. Agent Loop
5. 工具选择
6. 工具执行
7. 工具结果返回
8. 多工具调用
9. 工具权限控制
10. 二次确认机制

## 7.2 学习重点

Agent 的基本流程：

```text
用户输入
→ Agent 判断任务
→ 选择 Tool
→ 生成 Tool 参数
→ 执行 Tool
→ 获取 Tool 返回结果
→ 继续判断或生成最终回答
```

课程管理工具示例：

```text
queryCourses：查询课程
getCourseDetail：查询课程详情
createCourse：新增课程
updateCourse：修改课程
deleteCourse：删除课程
queryLearningRecords：查询学习记录
generateLearningSummary：生成学习总结
```

## 7.3 练习内容

1. 定义课程查询 Tool。
2. 定义课程新增 Tool。
3. 定义课程修改 Tool。
4. 定义课程删除 Tool。
5. 定义学习记录查询 Tool。
6. 为每个 Tool 编写 Zod 参数结构。
7. 实现查询类操作直接执行。
8. 实现新增、修改、删除类操作二次确认。
9. 记录工具调用日志。

------

# 8. 第六阶段：LangGraph 流程编排

## 8.1 学习内容

1. State
2. Node
3. Edge
4. Conditional Edge
5. Graph
6. Checkpoint
7. Interrupt
8. Resume
9. Human-in-the-loop
10. Memory

## 8.2 学习重点

LangGraph 的核心结构：

```text
State：保存当前任务状态
Node：执行具体步骤
Edge：连接不同节点
Conditional Edge：根据条件选择下一步
Checkpoint：保存执行进度
Interrupt：暂停流程
Resume：恢复流程
```

课程管理 Agent 图流程：

```text
用户输入
→ 意图识别节点
→ 参数提取节点
→ 权限判断节点
→ 是否写操作
→ 确认节点
→ 工具执行节点
→ 结果总结节点
```

## 8.3 练习内容

1. 定义 AgentState。
2. 创建意图识别节点。
3. 创建参数提取节点。
4. 创建权限判断节点。
5. 创建确认节点。
6. 创建工具执行节点。
7. 创建结果总结节点。
8. 使用条件边控制流程走向。
9. 实现中断和恢复。
10. 保存 pendingAction。

------

# 9. 第七阶段：项目实战整合

## 9.1 项目名称

在线学习管理平台智能助手

## 9.2 核心功能

1. 自然语言查询课程。
2. 自然语言新增课程。
3. 自然语言修改课程信息。
4. 自然语言删除课程。
5. 查询学习记录。
6. 生成学习总结。
7. 写操作二次确认。
8. 工具调用日志记录。
9. 流式返回回答。
10. 前端展示确认卡片。

## 9.3 技术栈

```text
前端：React + TypeScript + Vite + Ant Design
后端：Koa + TypeScript
AI 编排：LangChain.js + LangGraph.js
参数校验：Zod
数据库：PostgreSQL / MongoDB
流式输出：Fetch + ReadableStream + TextDecoder
日志记录：agent_logs 表
```

## 9.4 后端目录结构

```text
src/
  agent/
    tools/
      course.tools.ts
      learning.tools.ts
    prompts/
      intent.prompt.ts
      summary.prompt.ts
    schemas/
      course.schema.ts
      learning.schema.ts
    graph/
      courseAgent.graph.ts
    services/
      agent.service.ts
  routes/
    agent.route.ts
  controllers/
    agent.controller.ts
  models/
    agentLog.model.ts
```

## 9.5 核心执行流程

```text
用户输入自然语言指令
→ 前端发送到 /api/agent/chat
→ 后端接收用户输入
→ LangChain / LangGraph 处理任务
→ 判断用户意图
→ 提取工具参数
→ 使用 Zod 校验参数
→ 判断是否属于写操作
→ 查询类操作直接执行
→ 写操作返回确认信息
→ 用户确认后继续执行
→ 调用业务接口或数据库
→ 记录工具调用日志
→ 返回执行结果
```

------

# 10. 每日学习安排

## 10.1 每日结构

```text
第一部分：学习一个核心概念
第二部分：阅读对应代码示例
第三部分：手写一个最小案例
第四部分：整理一段面试表达
```

## 10.2 单日模板

```text
1. 今天学习的概念是什么
2. 它解决什么问题
3. 它在项目中对应哪个功能
4. 最小代码如何实现
5. 面试中如何表达
```

------

# 11. 阶段成果

## 11.1 第一阶段成果

能够解释 LLM、Token、Embedding、上下文窗口和流式输出。

## 11.2 第二阶段成果

能够使用 LangChain.js 完成：

```text
Prompt → Model → Output
```

的基础调用。

## 11.3 第三阶段成果

能够让模型输出结构化 JSON，并使用 Zod 校验。

## 11.4 第四阶段成果

能够完成一个简单文档问答 RAG。

## 11.5 第五阶段成果

能够定义 Tools，并让 Agent 根据用户输入调用工具。

## 11.6 第六阶段成果

能够使用 LangGraph 设计带状态管理的多步骤 Agent 流程。

## 11.7 第七阶段成果

能够完成一个课程管理智能助手模块，并用于简历项目描述。