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
