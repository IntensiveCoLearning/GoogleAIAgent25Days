---
timezone: UTC+8
---

# Michelle

**GitHub ID:** xinyuuum2

**Telegram:** @michelleeee98

## Self-introduction

很高兴看到这个课程！目前是在职ds，对AI agent很感兴趣 正好想通过这个机会系统学习一下

## Notes

<!-- Content_START -->
# 2026-01-05
<!-- DAILY_CHECKIN_2026-01-05_START -->
## 一、为什么传统的 “Append-Everything” 会失败

### 1️⃣ 传统做法的本质

```text
Context = System Prompt
        + 全部历史对话
        + 所有工具输出
        + 当前用户输入
```

* * *

### 三个不可避免的失败

（1）Cost & Latency Spirals

-   token 越多 → 推理成本线性甚至超线性增长
    
-   time-to-first-token 明显变慢
    
-   在生产环境中不可控
    

（2）Lost in the Middle

-   LLM 对 **上下文中间的信息注意力最弱**
    
-   重要事实即使“存在”，也会被忽略或产生幻觉
    
-   大 context window 只能缓解，不能根治
    

> **关键认知**：  
> \*“上下文是否被正确组织”比“上下文是否足够长”更重要

（3）Security & Prompt Injection

-   system 指令、用户输入、工具输出混在一起
    
-   用户可通过输入“覆盖”系统行为
    
-   安全边界消失
    

* * *

## 二、ADK 的核心设计思想

> **Context = 从多个状态源中，经过规则编译后，生成的最小必要视图**

* * *

## 三、ADK 四层 Context 架构（非常重要）

### 1️⃣ Working Context（发送给 LLM）

-   **唯一真正进入模型的内容**
    
-   生命周期：**单次调用**
    
-   由系统自动“编译”生成
    

结构拆分

```
Stable Prefix（可缓存）
- system prompt
- agent identity
- schema / tools / policies

Variable Suffix（动态）
- 当前 user turn
- 本轮 tool 输出
- 检索到的 memory 片段
```

这是 **Context Caching** 能成立的根本原因。

* * *

### 2️⃣ Session（会话状态）

-   生命周期：**一次对话**
    
-   职责：
    
    -   Event log（对话、工具调用）
        
    -   Key-Value State（便签纸）
        

```python
session.state["customer_tier"] = "premium"
session.state["current_order_id"] = "ORD-12345"
```

**Session State ≠ Memory**

-   只存**当前对话有用的临时信息**
    

* * *

### 3️⃣ Memory（长期记忆）

-   生命周期：**跨会话**
    
-   存什么？
    
    -   用户偏好
        
    -   重要历史事实
        
    -   会话摘要
        

工作模式：

```text
会话结束 → 写入 Memory
新会话开始 → Search Memory → 片段注入 Context
```

**不是把 memory 全塞进 prompt，而是“检索后注入”**

* * *

### 4️⃣ Artifacts（大文件）

-   生命周期：**独立持久化**
    
-   解决的问题：
    
    -   PDF / CSV / 图片 / 长文本
        

* * *

## 四、Context = 编译流水线（Compiler Pipeline）

ADK 把 Context 构建当成一个 **编译过程**：

```
filter → compact → summarize → format
```

这意味着：

-   不是所有历史都值得进入 prompt
    
-   Context 构建是 **规则驱动的工程系统**
    
-   可以被测试、优化、替换
    

* * *

## 五、Static vs Turn Instructions（生产级关键模式）

### 1️⃣ Static Instructions（静态指令）

**放在 system role**

-   内容稳定：
    
    -   Policy
        
    -   Schema
        
    -   Tool 定义
        
    -   Few-shot 示例
        
-   **可被模型 API 缓存（prefix caching）**
    

```text
You are a customer service agent…
Policies…
Response JSON Schema…
```

📌 更新频率：低  
📌 成本：一次算，多次复用

* * *

### 2️⃣ Turn Instructions（轮次指令）

**放在 user role，但由后端生成**

-   每一轮动态变化：
    
    -   当前目标
        
    -   用户状态
        
    -   检索到的 memory
        
-   与用户输入物理隔离 → **安全**
    

```text
## Current Context
Customer tier: premium
Retrieved memory: previous delay resolved

## This Turn’s Goal
Answer order status
```

* * *

### 3️⃣ 最终结构

```
[system]  Static Instructions   ← cacheable
[user]    Turn Instructions     ← backend-controlled
[user]    User Message          ← raw input
```

👉 这是 **高性能 + 高安全 Agent 的标准结构**

* * *

## 六、为什么 Context Caching 能显著降本

| 请求 | Static | Dynamic | 实际计算 |
| --- | --- | --- | --- |
| #1 | 2000 | 500 | 2500 |
| #2 | cached | 600 | ~600 |
| #3 | cached | 450 | ~450 |

📌 现实影响：

-   延迟显著下降
    
-   成本可预测
    
-   高并发系统可行
    

* * *

## 七、InvocationContext vs ToolContext

### 核心思想

> **工具 ≠ agent，不应该看到全部状态**

-   InvocationContext：
    
    -   agent 主逻辑
        
    -   全访问
        
-   ToolContext：
    
    -   精准授权
        
    -   防止数据泄露
        

```text
Cart Tool: 只能访问 cart
Payment Tool: 才能看到 payment_token
```

* * *

## 八、完整 Agent 示例的真实含义

你最后看到的 demo 并不是重点在“客服机器人”，而是在展示：

-   Session / Artifact / Tool 是如何解耦的
    
-   Static instruction 如何长期稳定
    
-   Context 如何被 runner 自动编译
    
-   Agent 不直接关心“上下文拼接”
<!-- DAILY_CHECKIN_2026-01-05_END -->

# 2026-01-04
<!-- DAILY_CHECKIN_2026-01-04_START -->

今天有点忙…学习了一下课程里的Retail site agent

明天再补笔记
<!-- DAILY_CHECKIN_2026-01-04_END -->

# 2026-01-03
<!-- DAILY_CHECKIN_2026-01-03_START -->


| 概念 | 学到了什么 |
| --- | --- |
| Code Execution | 让 LLM 在沙箱中执行代码，获得精确结果 |
| GenAI SDK | 使用 types.Tool(code_execution=...) |
| ADK Agent | 使用 code_executor=BuiltInCodeExecutor() |
| 沙箱安全 | 代码在隔离环境运行，有执行限制 |

### **代码模板**

**GenAI SDK 直接调用：**

```
from google.genai import types

tool = types.Tool(code_execution=types.ToolCodeExecution())
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=question,
    config=types.GenerateContentConfig(tools=[tool])
)
```

**ADK Agent 配置：**

```
from google.adk.agents import Agent
from google.adk.code_executors import BuiltInCodeExecutor

agent = Agent(
    name="code_agent",
    model="gemini-2.0-flash",
    instruction="...",
    code_executor=BuiltInCodeExecutor()
)
```
<!-- DAILY_CHECKIN_2026-01-03_END -->

# 2026-01-02
<!-- DAILY_CHECKIN_2026-01-02_START -->



昨天忘记了 先补一下昨天的…

A. 第一层：系统行为追踪 (Agent Telemetry)

-   **工具：** Cloud Trace
    
-   **作用：** 解决“慢”和“错”的问题。
    
-   **核心概念 - Span（跨度）：** 它把一个完整的用户请求拆解成了详细的链条： `收到请求` → `解析输入(50ms)` → `调用LLM(800ms)` → `搜索工具(600ms)` → `格式化输出` _通过这个链条，你能一眼看出瓶颈在哪里。_
    

B. 第二层：交互与消耗记录 (Prompt-Response Logging)

-   **工具：** GCS (存储) + BigQuery (查询)
    
-   **作用：** 解决“成本”和“质量”的问题。
    
-   **核心模式 - 隐私保护模式 (**`NO_CONTENT`**)：**
    
    -   默认情况下，系统**只记录元数据**（用了什么模型、消耗了多少 Token、耗时多久、时间戳）。
        
    -   **不记录**用户的原始 Prompt 和 LLM 的具体回答（除非你显式开启 `ALL` 模式）。
        
-   **数据流向：** 日志会自动存入 Google Cloud Storage，并同步到 BigQuery。可以直接写 SQL 语句来查询：“过去24小时，Gemini-2.0 模型一共消耗了多少 Token？”
<!-- DAILY_CHECKIN_2026-01-02_END -->

# 2025-12-31
<!-- DAILY_CHECKIN_2025-12-31_START -->




今天的内容有点点多，用gemini老师辅助了一下，记录一下大概内容吧。

### 第一阶段：准备工作 (环境搭建)

在开始“建工厂”之前，我们需要先准备好工具。

Step 1: 安装必要工具

这相当于工地的“入场准备”。你需要两个核心工具：

1.  `uv`: 一个极速的 Python 包管理器（比 pip 快很多）。
    
2.  `gcloud CLI`: Google Cloud 的命令行工具，用来在你的电脑上控制云端资源。
    

**操作指令：** 打开你的终端（Terminal），分别运行：

Bash

```
# 1. 安装 UV (如果你的电脑是 Mac/Linux)
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. 确认 gcloud 是否安装 (如果没有，需要去 Google Cloud 官网下载安装包)
gcloud --version
```

* * *

### 第二阶段：创建项目 (生成工厂蓝图)

这一步就是 Day 4 提到的“快速开始”。我们不需要从零开始写几十个配置文件，直接用 Google 提供的“启动包” (`agent-starter-pack`) 生成一套标准的目录结构。

Step 2: 生成代码框架

我们要创建一个名为 `my-agent` 的新项目。这个命令会自动帮你生成 `Dockerfile`、`cloudbuild.yaml`（流水线图纸）等复杂文件。

**操作指令：**

Bash

```
# 这行命令的意思是：用 agent-starter-pack 工具，创建一个叫 my-agent 的项目
# -a adk_base: 使用 ADK 基础模板
# -d agent_engine: 目标是部署到 Agent Engine
uvx agent-starter-pack create my-agent -a adk_base -d agent_engine

# 进入这个新创建的文件夹
cd my-agent
```

**此时你的文件夹里会有这些东西：**

-   `agent/`: 放你写真正 Agent 逻辑（Prompt, Tools）的地方。
    
-   `deployment/`: 放流水线配置的地方（不用动）。
    
-   `Makefile`: 一个快捷指令文件（稍后我们要用它来一键部署）。
    

* * *

### 第三阶段：连接云端 (配置身份)

代码在本地生成好了，现在要告诉它：“我们要把这个 Agent 送到 Google Cloud 的哪个房间（Project）去”。

Step 3: 设置环境变量

你需要告诉代码三个信息：

1.  **Project ID**: 你的 Google Cloud 项目 ID。
    
2.  **Region**: 服务器在哪（通常选 `us-central1`）。
    
3.  **Staging Bucket**: 一个云存储桶（相当于中转站，代码先上传到这，再部署）。
    

**操作指令：**

Bash

```
# 替换下面的 "your-project-id" 为你真实的 ID
export GOOGLE_CLOUD_PROJECT="your-project-id"
export GOOGLE_CLOUD_LOCATION="us-central1"

# 替换下面的 bucket 名字 (你需要先在 Google Cloud Console 里创建一个 Bucket)
export STAGING_BUCKET="gs://your-staging-bucket-name"

# 登录你的账号，确让终端有权限操作
gcloud auth login
gcloud config set project $GOOGLE_CLOUD_PROJECT
```

* * *

### 第四阶段：正式部署 (启动流水线)

这是最关键的一步。我们要把“说明书”（源代码）发送给 Google Cloud，让它根据说明书把 Agent 运行起来。

Step 4: 一键部署

这里用到了 `Make` 命令，它把一长串复杂的上传、构建命令封装成了一个简单的单词 `deploy`。

**操作指令：**

Bash

```
# 在 my-agent 目录下运行
make deploy
```

**这一步背后发生了什么？（流水线在工作）**

1.  电脑把你的 `agent/` 文件夹里的代码打包。
    
2.  上传到你刚才设置的 `STAGING_BUCKET`（中转站）。
    
3.  通知 Vertex AI Agent Engine：“嘿，代码到了，请按照 `pyproject.toml` 里的依赖安装环境，并启动服务。”
    
4.  等待几分钟，直到终端显示 `Deploy Success`。
    

* * *

### 第五阶段：测试 (验收成果)

现在 Agent 应该已经在云端运行了。我们要在本地写几行 Python 代码来远程调用它，看看它能不能回答问题。

Step 5: 写一个测试脚本

在 `my-agent` 目录下新建一个文件叫 `test_agent.py`，粘贴以下代码：

Python

```
from google.cloud import aiplatform

# 1. 初始化连接
aiplatform.init(
    project="your-project-id", # 记得替换 ID
    location="us-central1"
)

# 2. 找到刚才部署的 Agent
# 注意：你需要去 Google Cloud Console 的 Vertex AI -> Agent Builder 页面
# 找到生成的 Agent ID，替换下面这一串
agent_id = "projects/your-project-id/locations/us-central1/agents/YOUR_AGENT_ID"
agent = aiplatform.Agent(agent_id)

# 3. 创建一个聊天会话
session = agent.create_session()

# 4. 发送测试消息
print("正在询问 Agent...")
response = session.chat("你好，请做一个自我介绍。")
print("Agent 回复:", response.text)
```

**操作指令：**

Bash

```
python test_agent.py
```

  
记录一下成功了！！

![截屏2025-12-31 14.01.07.png](https://raw.githubusercontent.com/IntensiveCoLearning/GoogleAIAgent25Days/main/assets/xinyuuum2/images/2025-12-31-1767160916971-__2025-12-31_14.01.07.png)
<!-- DAILY_CHECKIN_2025-12-31_END -->

# 2025-12-30
<!-- DAILY_CHECKIN_2025-12-30_START -->






-   One liner with Agent Starter Pack
    

```bash
uvx agent-starter-pack create -y --api-key YOUR_GEMINI_API_KEY
```

-   Using ADK CLI
    

```bash
uv init
uv add google-adk
uv add google-genai
export GOOGLE_API_KEY="YOUR_API_KEY"
source .venv/bin/activate
adk create my_agent
```

-   Download sample and run locally
    

```bash
curl '<https://raw.githubusercontent.com/GoogleCloudPlatform/devrel-demos/refs/heads/main/ai-ml/agent-labs/gemini-3-pro-agent-demo/my_agent/agent.py>' > my_agent/agent.py
adk web
```

# Google Search Tool

**Python Configuration**

```
from google.adk.agents import Agent
from google.adk.tools import google_search

agent = Agent(
    name="search_assistant",
    model="gemini-2.5-flash",
    instruction="You are a helpful assistant. Use Google Search when needed.",
    tools=[google_search]
)
```

### **YAML Configuration**

```
name: search_assistant
model: gemini-2.5-flash
description: An assistant that can search the web.
instruction: |
  You are a helpful assistant.
  Use Google Search for current events and factual information.
tools:
  - name: google_search
```
<!-- DAILY_CHECKIN_2025-12-30_END -->

# 2025-12-29
<!-- DAILY_CHECKIN_2025-12-29_START -->








ADK 智能体配置功能让你无需编写代码即可构建 ADK 工作流。智能体配置使用 YAML 格式的文本文件，包含智能体的简要描述，允许几乎任何人组装和运行 ADK 智能体。以下是一个基本智能体配置定义的简单示例：

```
name: assistant_agent
model: gemini-2.5-flash
description: A helper agent that can answer users' questions.
instruction: You are an agent to help answer users' various questions.
```
<!-- DAILY_CHECKIN_2025-12-29_END -->

# 2025-12-28
<!-- DAILY_CHECKIN_2025-12-28_START -->









ADK 智能体配置功能让你无需编写代码即可构建 ADK 工作流。智能体配置使用 YAML 格式的文本文件，包含智能体的简要描述，允许几乎任何人组装和运行 ADK 智能体。以下是一个基本智能体配置定义的简单示例：

```
name: assistant_agent
model: gemini-2.5-flash
description: A helper agent that can answer users' questions.
instruction: You are an agent to help answer users' various questions.
```
<!-- DAILY_CHECKIN_2025-12-28_END -->
<!-- Content_END -->
