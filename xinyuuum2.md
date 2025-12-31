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
