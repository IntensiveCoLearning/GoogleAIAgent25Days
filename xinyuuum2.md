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
