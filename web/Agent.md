在 Agent 开发里，可以这样理解：

> **Tool 解决“能不能做动作”的问题。**
> **MCP 解决“不同工具怎么统一接入”的问题。**
> **Skill 解决“复杂任务怎么稳定按流程做”的问题。**

------

## 1. Tool：给 Agent 一个“可调用能力”

**Tool** 就是 Agent 可以调用的一个外部能力，例如：

```text
查天气
查数据库
读文件
写文件
调用搜索接口
发送邮件
执行 Python 代码
调用你后端的某个 API
```

大模型本身只会生成文本，它不能真的访问数据库、不能真的发请求、不能真的改文件。所以需要把这些能力封装成 tool。

OpenAI 文档里也把 tool/function 理解为“提供给模型的功能”，模型判断需要时会发起 tool call，然后由应用侧执行并把结果返回给模型。([OpenAI 开发者](https://developers.openai.com/api/docs/guides/function-calling))

典型流程是：

```text
用户提问
↓
模型判断需要调用工具
↓
模型输出 tool_call，例如 get_weather({ city: "北京" })
↓
你的程序真正执行 get_weather
↓
把结果返回给模型
↓
模型组织最终回答
```

### Tool 解决的问题

以前只有 Prompt：

```text
用户：帮我查今天北京天气
模型：我无法实时查询……
```

有了 Tool：

```text
模型调用 weather_api
拿到真实天气
再回答用户
```

所以 tool 的核心作用是：

```text
让大模型从“只会说”变成“能调用外部能力做事”
```

在工程里，tool 一般是一个具体函数或 API：

```ts
const tools = [
  {
    name: "search_docs",
    description: "搜索知识库文档",
    parameters: {
      type: "object",
      properties: {
        query: { type: "string" }
      },
      required: ["query"]
    }
  }
]
```

------

## 2. MCP：统一工具接入协议

**MCP，全称 Model Context Protocol**，可以理解为：

```text
给 Agent 接入外部工具、数据、提示词的一套标准协议
```

MCP 不是某一个具体工具，而是一个“连接标准”。

官方 MCP Python SDK 文档里说，MCP 允许应用以标准化方式向 LLM 提供上下文，并且 MCP server 可以暴露 Resources、Tools 和 Prompts。([GitHub](https://github.com/modelcontextprotocol/python-sdk))

MCP 里常见三类能力：

| 类型      | 作用               | 类比                               |
| --------- | ------------------ | ---------------------------------- |
| Resources | 提供数据上下文     | GET 接口，读文件、读数据库、读文档 |
| Tools     | 执行动作           | POST 接口，发请求、改数据、跑任务  |
| Prompts   | 提供可复用提示模板 | 固定工作流模板                     |

例如你有这些系统：

```text
GitHub
本地文件系统
PostgreSQL
Google Drive
企业知识库
飞书 / Slack
```

如果不用 MCP，每接入一个 Agent 框架都要单独适配：

```text
LangChain 接一遍
OpenAI Agent 接一遍
Claude 接一遍
Cursor 接一遍
自研 Agent 再接一遍
```

这会变成：

```text
N 个工具 × M 个 Agent 平台 = 很多重复集成
```

MCP 的目标是把它变成：

```text
工具实现一次 MCP Server
不同 Agent 作为 MCP Client 直接连接
```

### MCP 解决的问题

MCP 主要解决的是：

```text
工具接入标准不统一
上下文读取方式不统一
权限认证方式不统一
不同 Agent 框架重复开发适配层
```

比如你做一个 `github-mcp-server`，里面暴露：

```text
list_issues
read_file
create_pr
search_repo
```

那么支持 MCP 的 Agent 都可以连接它，而不是每个 Agent 都重新写一套 GitHub 集成。

所以 MCP 的重点不是“让模型更聪明”，而是：

```text
让 Agent 更容易、安全、标准化地连接外部系统
```

------

## 3. Skill：封装“怎么做一类任务”的经验

**Skill** 更像是一个“任务能力包”。

OpenAI 文档里把 Skill 定义为：带有 `SKILL.md` 清单文件的一组版本化文件包，可以用来编码流程、规范、约定，例如公司风格指南、多步骤工作流等。([OpenAI 开发者](https://developers.openai.com/api/docs/guides/tools-skills))

ChatGPT Skills 文档也强调，Skill 是可复用、可分享的工作流，可以包含说明、示例，甚至代码，用来让 ChatGPT 更稳定地完成某类任务。([OpenAI Help Center](https://help.openai.com/en/articles/20001066-skills-in-chatgpt?utm_source=chatgpt.com))

它不一定是一个 API，而是告诉 Agent：

```text
遇到这种任务时，应该按什么流程做
用什么规范
注意哪些限制
使用哪些脚本
输出什么格式
```

例如一个“生成 PDF 的 Skill”可能包含：

```text
1. 必须先读取 PDF 生成规范
2. 使用指定脚本生成 PDF
3. 检查字体、分页、图片清晰度
4. 最后返回下载链接
```

一个“简历优化 Skill”可能包含：

```text
1. 先提取岗位要求
2. 再提取用户经历
3. 按 STAR 法重写项目
4. 控制在一页内
5. 输出中文正式简历风格
```

### Skill 解决的问题

Tool 只能解决“调用一个动作”。

但很多任务不是一个动作，而是一套流程。

例如：

```text
生成一份开题答辩 PPT
```

这不是简单调用一个 `create_ppt()` 就结束了，它里面包含：

```text
理解主题
组织目录
设计页面结构
生成图示
统一风格
检查页数
导出 pptx
```

如果每次都靠 Prompt 临时写，容易不稳定：

```text
这次忘了检查格式
下次忘了统一标题
再下次生成风格不一致
```

Skill 的作用就是把这些经验沉淀下来：

```text
让 Agent 对某一类任务形成稳定、可复用、可版本管理的操作流程
```

------

## 4. 三者的核心区别

可以用一句话区分：

| 概念  | 本质                   | 解决什么问题                     | 例子                                           |
| ----- | ---------------------- | -------------------------------- | ---------------------------------------------- |
| Tool  | 一个可调用函数/能力    | 让模型能执行外部动作             | 查天气、查数据库、发邮件                       |
| MCP   | 工具和上下文的接入协议 | 让不同工具能标准化接入不同 Agent | GitHub MCP、数据库 MCP、文件系统 MCP           |
| Skill | 一套任务流程和规范     | 让复杂任务稳定按经验执行         | PDF 生成 Skill、论文润色 Skill、PPT 制作 Skill |

更直观地说：

```text
Tool = 一个按钮
MCP = 统一插座
Skill = 操作手册 + 配套脚本
```

------

## 5. 它们为什么会依次出现？

### 第一阶段：只有 Prompt

最早的大模型主要靠 Prompt：

```text
你是一个助手，请帮我……
```

问题是：

```text
不能访问实时数据
不能操作外部系统
不能保证复杂流程稳定执行
```

------

### 第二阶段：出现 Tool / Function Calling

为了解决“模型不能做事”的问题，出现了 tool/function calling。

它让模型可以调用外部函数：

```text
search()
query_database()
send_email()
generate_image()
run_code()
```

解决了：

```text
模型不能访问外部世界
模型不能执行真实操作
模型输出无法结构化落地
```

------

### 第三阶段：工具越来越多，出现 MCP

Tool 多了之后，又出现新问题：

```text
每个平台的工具定义方式不同
每个 Agent 框架都要重复接入外部系统
工具发现、权限、上下文读取没有统一标准
```

所以 MCP 出现了。

它的目标是：

```text
让外部系统通过统一协议暴露给 Agent
让 Agent 通过统一方式发现、调用、读取这些能力
```

这类似你之前遇到 OSS 私有读图片的问题：固定 URL、签名 URL、权限策略本质上都是“外部资源不能随便直接访问，必须有标准化访问方式和权限边界”。 MCP 在 Agent 场景中也强调这种“访问方式、能力边界、权限控制”的标准化。

------

### 第四阶段：复杂任务不只是调用工具，出现 Skill

即使有了 tool 和 MCP，Agent 仍然可能做不好复杂任务。

比如你让 Agent：

```text
帮我根据模板生成一份正式投标文档
```

它可能需要：

```text
读取模板
理解章节结构
检索证据
组织内容
控制术语
生成正文
生成引用映射
检查格式
导出文件
```

这不是一个简单 tool 能解决的。

所以 Skill 出现了，用来沉淀：

```text
任务流程
领域规范
格式要求
示例文件
辅助脚本
检查规则
```

它解决的是：

```text
复杂任务执行不稳定
每次都要重复写 Prompt
团队规范难以复用
Agent 行为难以标准化
```

------

## 6. 开发时怎么选？

### 只需要执行一个明确动作：用 Tool

例如：

```text
查询用户信息
搜索知识库
调用支付接口
读取订单状态
```

适合封装成 tool。

------

### 要把一批外部系统标准化接入 Agent：用 MCP

例如：

```text
让 Agent 访问 GitHub
让 Agent 读取数据库
让 Agent 操作文件系统
让 Agent 连接企业知识库
让多个 Agent 共用同一批工具
```

适合做 MCP Server。

------

### 要让 Agent 稳定完成一类复杂任务：用 Skill

例如：

```text
生成 PPT
生成 PDF
写投标文档
做代码审查
整理会议纪要
根据公司规范写周报
```

适合做 Skill。

------

## 7. 面试里可以这样回答

可以这样说：

> 在 Agent 开发中，Tool、MCP 和 Skill 解决的是不同层面的问题。Tool 是最基础的外部能力封装，用来让模型调用函数或 API，解决大模型只能生成文本、不能真实执行动作的问题。MCP 是工具和上下文的标准接入协议，解决不同 Agent 框架和外部系统之间集成方式不统一、重复适配的问题。Skill 则更偏向任务流程封装，它把某一类复杂任务的步骤、规范、示例和脚本沉淀下来，解决 Agent 在复杂任务中执行不稳定、每次都依赖临时 Prompt 的问题。简单说，Tool 是能力，MCP 是连接标准，Skill 是可复用的任务经验。