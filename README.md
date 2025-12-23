# Google 25天AI Agent课程

## 介绍

**Google Cloud Advent of Agents 学习计划 (25 Days)**

**Advent of Agents (Google Cloud Edition)** 是一个专注于在 Google Cloud 上使用 **ADK (Agent Development Kit)** 和 **Gemini 3** 模型构建生产级 Agent 的系列挑战，从零开始构建生产级 Google Cloud AI Agent 的完整路径。

課程亮點
* 🎯 每天一項功能，每項功能只需不到 5 分鐘即可體驗
* 📋 開箱即用的複製貼上命令
* 📚 官方文件鏈接
*  🆓 100% 免費

更多资讯请看Google Advent of Agents 2025（网站预计开放到2026三月）：https://adventofagents.com/
## 关键词

AI Agent
## 面向人群

喜欢AI Agent、想做产品以及企业
## 报名时间

- 报名开始时间：2025-12-25
- 报名结束时间：2025-12-27
## 共学时间

- 共学开始时间：2025-12-28
- 共学结束时间：2026-01-21
## 发起人

- 姓名：馬鈴薯來囉
- GitHub ID：Toby1009
- Telegram：Yunizero
- Email：st99005912@gmail.com
## 发起组织

  [LXDAO](https://lxdao.io/) <img alt="organization-logo" height="60px" width="60px" src="https://cdn.lxdao.io/bafkreiay6vxsvv3ksxr75lzzt3iqy3zja3o2epuxh47ivs24p2xs3awexm.png" />


## 请假规则

每周请假 3 次


## 社群

Telegram：https://t.me/+3wGYldnFuFZkZjE1
## 学习资料/课程安排

课程中文翻译：https://www.notion.so/Google-25-AI-Agent-2d26fba5a2d38037ad5dcb5141935cc2?source=copy_link

以下模块僅供參考，以[Google Advent of Agents](https://adventofagents.com/)為準
#### **模块 0：环境与工具准备 (Days 1, 2, 4, 6)**

**目标**

* 完成 Google Cloud 基础环境配置。
* 熟悉 **ADK (Agent Development Kit)** 和 **Agent Starter Pack**。
* 能够不写代码跑通第一个 Hello World Agent，并理解基于源码的部署流程。

**关键知识点**

* **Gemini 3**：Google 最新一代模型的基础认知。
* **YAML 配置**：使用配置文件快速定义 Agent 行为。
* **IDE 集成**：在 Cursor、Antigravity 或 Gemini CLI 中配置开发环境。
* **Source-Based Deployment**：直接从源码部署 Agent (Agent Engine)。
---

#### **模块 1：Agent 核心能力开发 (Days 3, 7, 8, 9, 10)**

**目标**

* 掌握 Agent 的核心逻辑：代码执行、上下文管理和记忆机制。
* 实现具有“自主解决问题”能力的 Agent。
* 解决 Agent 开发中常见的 Context 丢失和 Token 成本问题。

**关键知识点**

* **Code Execution**：LLM 不仅生成文本，还能编写并自主执行代码来解决问题。
* **Context Management**：理解 ADK 的上下文分层设计 (Layers)，像编译视图一样管理 Context。
* **Context Caching & Compaction**：解决长对话中的“Lost in the middle”问题，优化延迟和成本。
* **Time Travel (Undo)**：利用 ADK 内置机制实现对话的“撤回”和“重成”，无需复杂的数据库回滚。
---

#### **模块 2：连接、协议与多模态 (Days 11, 12, 13, 14)**

**目标**

* 让 Agent 能够“看、听、说” (多模态)。
* 学会使用标准协议连接外部工具和其他 Agent。
* 理解从“无状态文本生成”到“有状态工作流”的转变。

**关键知识点**

* **Managed MCP (Model Context Protocol)**：使用 Google 托管的 MCP 标准连接外部服务与数据。
* **Gemini Live API (Bi-Directional Streaming)**：通过 WebSocket 实现实时的语音/视频多模态交互。
* **Interactions API**：构建有状态的自动化工作流。
* **A2A (Agent-to-Agent) Protocol**：让不同的 Agent 之间可以互相发现、通信和协作。

---

#### **模块 3：进阶交互与生态集成 (Days 15, 16, 17, 20)**

**目标**

* 打破传统 Chat 界面限制，生成动态 UI。
* 将 LangGraph 等流行框架与 Google 的架构集成。
* 处理复杂的自定义数据结构。

**关键知识点**

* **A2UI (Agent-to-User Interface)**：Agent 直接生成 JSONL payload 来渲染动态 UI，而不只是输出文字。
* **LangGraph + A2A**：将 LangGraph 构建的 Agent 接入 A2A 协议，使其具备跨生态互操作性。
* **Gemini 3 Flash**：使用更快的模型，并配置 Thinking Levels (思考层级)。
* **A2A Extensions (Sidecar)**：在标准协议下扩展自定义数据字段 (Sidecar pattern)。

---

#### **模块 4：企业级生产、运维与安全 (Days 5, 18, 19, 21, 22)**

**目标**

* 将 Demo 转化为生产级应用，具备完整的可观测性。
* 掌握企业级 Agent 的注册、发现与安全防护。
* **最终挑战**：参考 Kaggle 获奖案例，构建并加固自己的 Agent。

**关键知识点**

* **Observability (可观测性)**：集成 Cloud Trace, Log Analytics 和 BigQuery，实现零配置监控。
* **Cloud API Registry**：使用集中式的工具库，解决企业开发中工具复用的难题。
* **Gemini Enterprise**：将你的 Agent 注册到企业目录，使组织内全员可见。
* **Model Armor & Security**：学习 Guardrails (护栏)，不仅靠 Prompt 提示，而是用策略层拦截 PII (个人隐私信息) 和恶意攻击。


---
#### 补充
* 已有大神做了个笔记：https://github.com/anxiong2025/25-Day-Agents-Course-by-Google?tab=readme-ov-file
* Google AI Agent 5天基础学习：https://github.com/sdivyanshu90/5-Day-AI-Agents-Intensive-Course-with-Google

祝你在 Google Cloud 的 Agent 探险中好运！


## 共学激励

学到知识就是最大的福利！


## 更多信息

更多信息内容


## 报名和打卡规则

- 报名：https://www.notion.so/lxdao/232dceffe40b8030993ad26f2eb6bed2

- 打卡：https://www.notion.so/lxdao/232dceffe40b80508330c5ee936d4dab

## 残酷共学打卡记录表

✅ = Done ⭕️ = Missed ❌ = Failed

<!-- START_COMMIT_TABLE -->
| Name | 12.28 | 12.29 | 12.30 | 12.31 | 1.01 | 1.02 | 1.03 | 1.04 | 1.05 | 1.06 | 1.07 | 1.08 | 1.09 | 1.10 | 1.11 | 1.12 | 1.13 | 1.14 | 1.15 | 1.16 | 1.17 | 1.18 | 1.19 | 1.20 | 1.21 |
| ------------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
<!-- END_COMMIT_TABLE -->








<!-- STATISTICALDATA_START -->
## 统计数据

- 总参与人数: 0
- 完成人数: 0
- 完成用户: 
- 全勤用户: 
- 淘汰人数: 0
- 淘汰率: 0.00%
<!-- STATISTICALDATA_END -->
