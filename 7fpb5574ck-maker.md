---
timezone: UTC+8
---

# Leo

**GitHub ID:** 7fpb5574ck-maker

**Telegram:** @runes_leo

## Self-introduction

AI × Crypto 实践者，关注 AI Agent、自动化与工具构建，正在用 AI 提升个人效率与真实执行能力，偏好从真实问题出发做产品。

## Notes

<!-- Content_START -->
# 2026-01-01
<!-- DAILY_CHECKIN_2026-01-01_START -->
# 📅 2026-01-01 Day 05 学习日记

## 📝 学习主题：Production Observability (Agent Starter Pack)

新年第一天，学习了 Google Cloud 的 **Agent Starter Pack**，重心放在了 Agent 的**生产级可观测性（Observability）**。

### 💡 核心收货

1.  **Agent Starter Pack 的价值**：
    
    -   它不仅仅是一个模板，而是一套**工程化标准**。通过 `agent-starter-pack create` 可以快速生成包含后端、部署配置和观测性代码的项目，极大提升了从 Demo 到生产的效率。
        
2.  **可观测性的两层深度理解**：
    
    -   **Agent Telemetry (Cloud Trace)**：这是"骨架"。记录了请求怎么走、每个工具跑了多久。
        
    -   **Prompt-Response Logging (GCS/BigQuery)**：这是"细节"。记录了 AI 到底说了什么，用了多少 Token。
        
    -   **隐私权衡**：学到了 `NO_CONTENT` 模式，在记录元数据进行成本/性能监控的同时，保护用户隐私。
        
3.  **ReAct 模式的实战感悟**：
    
    -   通过本地 Playground 的交互，清晰地看到了模型如何自主决定调用 `get_weather`。这种"思考 -> 行动 -> 观察 -> 总结"的闭环是当前最实用的 Agent 范式。
        

### 🚀 对 Prediction Copilot 的启发

-   **决策溯源**：在预测市场中，AI 为什么判断为 YES 必须是可追溯的。利用 Cloud Trace 这种工具，可以把"新闻检索 -> 鲸鱼分析 -> 逻辑推理"这个过程可视化，增加产品的透明度和信任度。
    
-   **成本控制**：通过观测性面板看到的 Token 统计，可以帮助精准核算每次分析的成本，从而制定合理的定价或使用限制策略。
    

### 🔧 技术细节记录

-   **环境配置**：遇到 SSL 问题，通过 `venv` 环境和 `uv` 工具成功解决。
    
-   **配置切换**：学会了如何在 `agent.py` 中通过环境变量 `GOOGLE_GENAI_USE_VERTEXAI` 在 GCP Vertex AI 和 Google AI Studio 之间快速切换认证模式。
    

### 🌟 总结

如果说前几天是在学"怎么做 Agent"，今天是学"怎么管 Agent"。在一个真实的生产项目中，能看清 Agent 的运行状态比做对一个功能更重要。

* * *

**下一站**：Day 06 - ADK Ready (Antigravity, Gemini CLI, Cursor, Firebase)
<!-- DAILY_CHECKIN_2026-01-01_END -->

# 2025-12-31
<!-- DAILY_CHECKIN_2025-12-31_START -->

# **Day 04 学习笔记: Source-Based Deployment (基于源的部署)**

## **1\. 核心问题：为什么需要 Source-Based Deployment？**

### **传统部署方式的痛点**

传统 Agent 部署通常使用 **序列化 (Serialization)**，如 Python 的 `pickle` 或 `cloudpickle`：

```
# 传统方式：把整个 Agent 对象"腌制"成二进制文件
```

import cloudpickle

with open("agent.pkl", "wb") as f:

cloudpickle.dump(my\_agent, f)

**问题**：

1.  **Python 版本敏感**：本地 3.11，云端 3.10？💥 报错
    
2.  **依赖地狱**：本地 numpy 1.24，云端 1.21？💥 不兼容
    
3.  **调试困难**：出错后看到的是一堆二进制乱码
    
4.  **安全隐患**：`pickle` 可被利用执行恶意代码
    

### **Source-Based 的解法**

**直接上传源码文件**，让云端从头重建 Agent：

```
本地代码 → 打包 .tar.gz → 上传 → 云端 pip install → 云端 import agent
```

* * *

## **2\. 核心 API 解析**

### `AdkApp`**: Agent 的"容器"**

```
from vertexai.preview.reasoning_engines import AdkApp
```

adk\_app = AdkApp(

agent=retail\_agent, # 你的 ADK Agent

enable\_tracing=True, # 启用 Cloud Trace 追踪

)

-   **作用**：把 ADK Agent 包装成一个符合 Vertex AI 规范的"应用"
    
-   **类比**：就像把你的代码放进一个标准化的 Docker 容器
    

### `agent_engines.create()`**: 部署命令**

```
from vertexai import agent_engines
```

remote\_agent = agent\_engines.create(

adk\_app, # AdkApp 实例

display\_name="my\_agent", # 显示名称

requirements=\["google-adk", ...\], # pip 依赖

extra\_packages=\["./day03"\], # 本地源码目录

)

**关键参数**：

| 参数 | 作用 |
| --- | --- |
| requirements | Python 依赖列表，等同于 requirements.txt |
| extra_packages | 本地源码目录，会被打包上传 |
| env_vars | 环境变量（如 API Key） |

* * *

## **3\. 部署流程图解**

```
┌──────────────────┐
```

│ 本地开发环境 │

│ day03/[agent.py](http://agent.py) │

└────────┬─────────┘

│ AdkApp 包装

▼

┌──────────────────┐

│ agent\_engines │

│ .create() │

└────────┬─────────┘

│ 打包上传

▼

┌──────────────────┐

│ Cloud Storage │

│ (Staging Bucket)│

└────────┬─────────┘

│ Agent Engine 拉取

▼

┌──────────────────┐

│ Agent Engine │

│ pip install → │

│ import agent → │

│ 运行服务 │

└──────────────────┘

* * *

## **4\. 为什么这种方式更好？**

| 特性 | Pickle 序列化 | Source-Based |
| --- | --- | --- |
| 版本兼容性 | ❌ 易出错 | ✅ 云端重建 |
| 调试体验 | ❌ 二进制难读 | ✅ 看源码 |
| CI/CD 友好 | ❌ 难以集成 | ✅ 像部署普通应用 |
| 安全性 | ❌ pickle 风险 | ✅ 标准代码 |

* * *

## **5\. 代码已就绪 (无需运行)**

我们为 Day 04 创建了完整的部署脚本：

-   [**deploy.py**](http://deploy.py): 完整的 create/list/delete 逻辑
    
-   **.env.example**: 环境变量模板
    

**未来如需实际部署**，只需配置 GCP 项目和 Bucket，运行：

```
python day04/deploy.py --create
```

* * *

## **6\. 关键 Takeaway**

1.  **AdkApp 是桥梁**：连接 ADK Agent 和 Vertex AI Agent Engine
    
2.  **extra\_packages 是核心**：你的源码通过它被打包上传
    
3.  **requirements 定义环境**：确保云端有正确的依赖
    
4.  **Source > Pickle**：现代部署的最佳实践
<!-- DAILY_CHECKIN_2025-12-31_END -->

# 2025-12-30
<!-- DAILY_CHECKIN_2025-12-30_START -->


# **Day 03 学习笔记: Gemini 3 与 神经符号智能体 (Neuro-Symbolic Agents)**

## **1\. 核心理念: 神经符号 AI (Neuro-Symbolic AI)**

Day 03 标志着我们从“聊天机器人”迈向了“解决问题的智能体”。

-   **Neuro (神经)**: **Gemini 3 Pro**。大模型作为“大脑”，负责理解非结构化的用户意图（“我要开店”）、制定高层战略（“需要做差异化竞争”）。
    
-   **Symbolic (符号)**: **代码执行 (Code Execution)**。Python 解释器作为“计算器”，负责处理结构化数据、精确计算（平均分 4.46）、逻辑推导。
    

**价值**: 这种结合解决了 LLM 的两大死穴：**幻觉 (Hallucination)** 和 **计算能力弱 (Poor Math)**。

## **2\. 架构设计 (Architecture)**

我们的 Retail Agent 采用了经典的 **ReAct (Reason + Act)** 变体模式：

1.  **感知 (Perception)**: 用户输入 "分析 KR Puram 的健身房竞争"。
    
2.  **行动 1 (Tool 1)**: 调用
    
    **search\_places** 获取外部数据（Competitor Data）。
    
3.  **推理 (Reasoning)**: 拿到数据后，发现需要量化分析。
    
4.  **行动 2 (Tool 2)**: 编写并调用
    
    **execute\_code**，计算市场饱和度、加权评分。
    
5.  **决策 (Decision)**: 综合数据与常识，输出最终战略报告。
    

## **3\. 工程实践 (Engineering Highlights)**

### **A. 模拟数据 (Mocking Strategy)**

由于没有 Google Maps Billing 权限，我们采用了 **Mock（模拟）** 策略。

-   **实现**: 编写
    
    **mock\_**[**data.py**](http://data.py) 和假的
    
    **search\_places** 工具。
    
-   **启发**: 在 Agent 开发初期，不要被外部 API 卡住。先用 Mock 数据跑通逻辑，架构稳定后再接入真实 API。
    

### **B. 自定义工具 (Custom Tools)**

官方 ADK 的 `BuiltInCodeExecutor` 与部分模型存在兼容性问题（400 Error）。

-   **解决方案**: 我们手动实现了一个
    
    **execute\_code(code: str)** 工具，底层使用 Python 的
    
    **exec()**。
    
-   **启发**: Agent 的本质只是 LLM + Functions。不必迷信官方封装，只要能把函数描述清楚（Docstring）并正确返回结果，任何代码都可以成为 Agent 的手。
    

### **C. 模型探索 (Model Discovery)**

我们发现即便 UI 显示 "Gemini 3 Pro"，API 标识符可能不同。

-   **Gemini 3 Flash**: `models/gemini-3-flash-preview`
    
-   **Gemini 3 Pro**: `models/gemini-3-pro-preview`
    
-   **经验**: 永远使用 `client.models.list()` 来确认当前可用的真实 Model ID，而不是盲猜。
    

## **4\. 关键成果 (Key Outcomes)**

-   **Data-Driven Insight**: Agent 不是瞎编建议，而是基于算出来的 "4.46 平均分" 警告我们 "市场竞争极度激烈"。
    
-   **Autonomous Chaining**: Agent 自主完成了 `Search` -> `Code` 的数据流转，全程无需人类干预。
    

* * *

## **5\. 代码片段 (Golden Snippets)**

**Agent 如何自我思考并写代码：**

```
# Agent 生成的用来分析市场的代码
```

import pandas as pd

data = \[...\] # 自动注入了上一步搜索到的数据

\# 自动计算加权平均分

df\['weighted\_score'\] = df\['rating'\] \* df\['reviews'\]

overall\_weighted\_avg = df\['weighted\_score'\].sum() / total\_reviews

\# 自动进行细分市场切片

market\_leaders = df\[(df\['rating'\] >= 4.5) & (df\['reviews'\] >= 100)\]

niche\_players = df\[(df\['rating'\] >= 4.5) & (df\['reviews'\] < 100)\]

这段逻辑完全由 Gemini 3 Pro 自主生成，证明了其强大的逻辑推理能力。
<!-- DAILY_CHECKIN_2025-12-30_END -->

# 2025-12-29
<!-- DAILY_CHECKIN_2025-12-29_START -->



````markdown
# Day 02: Introduction to Declarative Agents (2025-12-29)

## 🎯 今日目标 (Goal)
- 学习 **Declarative Agents (声明式智能体)**：无需编写 Python 代码，通过 YAML 配置文件即可定义 Agent。
- 掌握 **Google Agent Development Kit (ADK)** 的基础命令：CLI 与 Web UI。
- 解决模型配额 (Quota) 与工具调用 (Tool Use) 的兼容性挑战。

## 📝 核心知识点 (Key Concepts)

### 1. YAML Configuration
Agent 的“灵魂”都在 `root_agent.yaml` 中定义：
```yaml
name: search_agent
model: gemini-2.5-flash  # 选择适合自己配额的模型
description: A helpful assistant that can search the web.
instruction: |
  You are a helpful assistant.
  Use Google Search for current events and factual information.
tools:
  - name: google_search  # 内置工具，声明即可使用
```

### 2. 运行模式
- **命令行 (CLI)**: 快速测试
  ```bash
  venv/bin/adk run day02
  ```
- **网页界面 (Web UI)**: 图形化交互与调试
  **关键点**: 必须在**项目根目录**运行，否则找不到 Agent！
  ```bash
  venv/bin/adk web .
  ```
  *(注: 在 Web UI 左上角下拉菜单选择 `day02` 即可开始对话)*

## 🛠️ 踩坑与解决方案 (Troubleshooting)

### 1. 模型选择与配额 (429/404 Errors)
- **现象**: `gemini-1.5-flash` 报 404 (Not Found)，`gemini-2.0-flash` 报 429 (Resource Exhausted).
- **原因**: 账号处于 Early Access 阶段，旧模型未授权，新模型无免费配额。
- **解决**: 使用 `list_models` 脚本扫描可用模型，最终锁定 **`gemini-2.5-flash`** 或 **`gemini-3-flash-preview`** (取决于账号权限)。

### 2. 工具兼容性 (400 Invalid Argument)
- **现象**: `Tool use with function calling is unsupported`.
- **原因**: ADK 当前版本在封装 `gemini-2.5` 系列请求时，如果混合使用 Python 工具与 Google Search，可能触发协议不兼容。
- **解决**: 暂时移除自定义 Python 工具，仅保留 `google_search`，回归课程最基础的“纯 YAML + 搜索”配置，成功跑通。

### 3. Web UI 找不到 Agent
- **现象**: `Warning: No agents found in current folder`.
- **原因**: 在 `day02` 目录下运行 `adk web .`，ADK 会去搜子目录而忽略当前目录。
- **解决**: 回到项目根目录 (`cd ..`)，运行 `adk web .`，ADK 就能正确扫描到 `day02` 作为一个 App。

## ✅ 成果 (Outcome)
成功创建了一个只需几行 YAML 配置就能通过 Google Search 回答实时问题（如“谁是 Leo”）的智能体，跑通了全流程。

---
*Next: Day 03 - Gemini Search Agent (Deep Dive into Grounding)*
````
<!-- DAILY_CHECKIN_2025-12-29_END -->

# 2025-12-28
<!-- DAILY_CHECKIN_2025-12-28_START -->




**\[Day 01\] Google AI Agent 开发环境搭建与初探**

**📅 日期**：2025-12-28 **🎯 目标**：跑通 Google ADK 基础流程，解决网络与鉴权问题。

**1\. 技术栈概览**

-   **Framework**: Google ADK (Agent Development Kit)
    
-   **Model**: Gemini 1.5 Flash (平衡速度与成本)
    
-   **Runtime**: Python 3.11
    

**2\. 核心步骤记录**

-   **依赖安装**：如果不使用 uv/poetry，直接
    
    ```
    pip install
    ```
    
    核心包，注意
    
    ```
    pydantic
    ```
    
    的版本兼容性。
    
-   **代理配置 (Crucial)**：
    
    -   国内开发需注意 SDK 的连接问题。
        
    -   解决方案：在代码入口处显式注入
        
        ```
        http_proxy
        ```
        
        /
        
        ```
        https_proxy
        ```
        
        环境变量（指向本地代理端口）。
        

**3\. 遇到的坑与解决**

-   **Runner 接口**：
    
    ```
    Runner.run()
    ```
    
    在新版 ADK 中推荐使用
    
    ```
    run_debug()
    ```
    
    进行快速的单轮/多轮对话测试，更加直观。
    
-   **鉴权**：
    
    **.env** 文件必须配置
    
    ```
    GOOGLE_API_KEY
    ```
    
    ，且需要安装
    
    ```
    python-dotenv
    ```
    
    自动加载。
    

**4\. 下一步计划**

-   深入研究
    
    ```
    SequentialAgent
    ```
    
    (顺序执行 Agent)。
    
-   探索 Tool Use (工具调用)，让 Agent 具备联网或执行代码的能力。
<!-- DAILY_CHECKIN_2025-12-28_END -->
<!-- Content_END -->
