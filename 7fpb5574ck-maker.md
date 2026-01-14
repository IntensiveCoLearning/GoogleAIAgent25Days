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
# 2026-01-14
<!-- DAILY_CHECKIN_2026-01-14_START -->
# **Day 18: Self-Improving Agents 学习笔记**

## **1\. 核心概念 (Core Concept)**

-   **自进化闭环 (Self-Correction Loop)**: 让 Agent 具备自我反思和修复错误的能力，减少人工干预。
    
-   **Level 3 Agent**: 并不是指更聪明的模型，而是指更自主的工作流。从只会被动执行的 Copilot，进化为能对自己结果负责、直到做对为止的 Autonomous Agent。
    

## **2\. 架构设计 (Architecture)**

我们构建了一个标准的 "Generator-Evaluator" 循环：

1.  **Generator (生成者)**:
    
    -   角色：负责 "干活"。
        
    -   任务：根据 Prompt 生成初始代码或方案。如果收到反馈，则根据反馈修改代码。
        
2.  **Executor (执行者/沙箱)**:
    
    -   角色：负责 "试错"。
        
    -   任务：在一个安全的环境（Sandbox）中运行代码，捕获标准输出 (stdout) 和错误栈 (stderr)。
        
    -   _关键技术_: 使用Python的
        
        ```
        subprocess
        ```
        
        +
        
        ```
        tempfile
        ```
        
        实现进程级隔离和超时熔断（Timeout），防止 Agent 写出的死循环搞挂主程序。
        
3.  **Evaluator (评估者)**:
    
    -   角色：负责 "找茬"。
        
    -   任务：分析 Executor 的报错信息。它不直接改代码，而是生成具体的
        
        ```
        Feedback
        ```
        
        （例如："第3行语法错误，也就是没有闭合括号"）。
        

## **3\. 关键代码逻辑 (Implementation)**

-   **Loop Structure**:
    
    ```
    python
    ```
    
    while attempts < max\_retries:
    
    code = generator(prompt + feedback)
    
    result = executor(code)
    
    if result.success:
    
    return code
    
    feedback = evaluator(code, result.error)
    
-   **Prompt Engineering**:
    
    -   Generator 的 Prompt 必须是动态的，能够接收
        
        ```
        Previous Failed Attempt
        ```
        
        和
        
        ```
        Feedback
        ```
        
        作为上下文。
        
    -   Evaluator 需要看到完整的 "三位一体"：
        
        ```
        Original Task
        ```
        
        +
        
        ```
        Source Code
        ```
        
        +
        
        ```
        Execution Error
        ```
        
        ，才能给出准确建议。
        

## **4\. 实际落地场景 (Applications)**

结合目前的 Prediction Trader / Copilot 项目：

1.  **合规审查 (One-Shot Correction)**: _最适合 Copilot_
    
    -   在 LLM 生成分析报告后，增加一个
        
        ```
        Evaluator
        ```
        
        (可以是规则代码，也可以是另一个 LLM)。
        
    -   强制检查硬性风控规则（如
        
        ```
        Coverage < 0.4
        ```
        
        必须
        
        ```
        AVOID
        ```
        
        ）。
        
    -   如果违规，自动打回重写，确保展示给用户的永远是合规的。
        
2.  **自愈式爬虫 (Self-Healing Scrapers)**: _最适合 Data 组_
    
    -   这里的 Evaluator 是 Python 异常捕获逻辑。
        
    -   当抓取脚本报
        
        ```
        AttributeError
        ```
        
        时，触发 Generator 读取新的 HTML 源码，自动修复 CSS Selector。
        
3.  **策略参数调优**: _最适合 Trader_
    
    -   Executor 运行回测。
        
    -   Evaluator 分析 PnL 曲线。
        
    -   Generator 调整策略参数（止损位、开仓阈值）。
        

## **5\. 局限与思考 (Trade-offs)**

-   **Token 成本**: 自进化意味着 Token 消耗可能翻倍（3次重试 = 3倍成本）。需要设置合理的
    
    ```
    Max Retries
    ```
    
    (通常 3 次足够)。
    
-   **延迟 (Latency)**: 串行的“生成-检查-再生成”会显著增加用户等待时间。对于实时性要求高的 C 端产品（如 Copilot），需要谨慎使用，或者仅用于后台离线任务。
<!-- DAILY_CHECKIN_2026-01-14_END -->

# 2026-01-13
<!-- DAILY_CHECKIN_2026-01-13_START -->

````markdown
# Day 17 学习笔记：Cloud API Registry + ADK

> **日期**: 2026-01-13
> **主题**: 企业级 Agent 工具治理

---

## 核心收获

### 1. Cloud API Registry 的定位

**一句话定义**：Agent 世界的 API Gateway

| 传统 Web 服务 | Agent 系统 |
|--------------|-----------|
| 管理 HTTP 请求 | 管理 Tool 调用 |
| 用户 → API | Agent → MCP Server |
| Kong / Nginx | Cloud API Registry |

**核心价值**：
- 统一发现：Agent 不需要知道每个服务的细节
- 统一认证：一处配置，全局生效
- 统一权限：集中管理"谁能用什么"
- 统一审计：所有调用有迹可查

### 2. 架构层次

```
┌─────────────────────────────────────────────┐
│  Layer 3: 工具治理层 (Cloud API Registry)    │
│  - 集中管理可用工具                          │
│  - 统一认证与权限控制                        │
│  - 审计与监控                               │
└─────────────────────────────────────────────┘
                     │
┌─────────────────────────────────────────────┐
│  Layer 2: 协议层 (MCP)                       │
│  - 标准化 Agent-Tool 通信                    │
│  - 跨框架兼容                               │
└─────────────────────────────────────────────┘
                     │
┌─────────────────────────────────────────────┐
│  Layer 1: Agent 层 (ADK/LangGraph/CrewAI)   │
│  - 业务逻辑实现                             │
│  - 多 Agent 编排                            │
└─────────────────────────────────────────────┘
```

### 3. 工具过滤 = 最小权限原则

```python
# ❌ 给 Agent 所有权限
tools = api_registry.get_toolset(mcp_server_name="bigquery")

# ✅ 只给必要权限
readonly_tools = api_registry.get_toolset(
    mcp_server_name="bigquery",
    tool_filter=["query", "list_tables"]  # 只读操作
)
```

**关键问题**：如果这个 Agent 被黑了，它能造成多大损失？

### 4. 三层权限模型

| 层级 | IAM 角色 | 控制什么 |
|------|---------|---------|
| Layer 1 | `apiregistry.viewer` | 谁能看到有哪些工具 |
| Layer 2 | `mcp.toolUser` | 谁能调用工具 |
| Layer 3 | `bigquery.dataViewer` | 工具能做什么操作 |

**三层都满足，Agent 才能执行操作。**

### 5. 今日 Key Insight

> **单个 Agent 调用 API 很简单。**
> **但当你有 10 个 Agent 时，真正的问题是：**
> **谁能用什么工具？如何审计？如何统一更新？**

这就是 **Tool Governance（工具治理）** —— 企业级 Agent 的隐藏 Boss。

---

## 关键概念

### ApiRegistry 基础用法

```python
from google.adk.tools.api_registry import ApiRegistry
from google.adk import Agent

# 1. 连接 Registry
api_registry = ApiRegistry(
    api_registry_project_id="my-gcp-project",
    location="global"
)

# 2. 获取工具集
bigquery_tools = api_registry.get_toolset(
    mcp_server_name="projects/xxx/locations/global/mcpServers/bigquery"
)

# 3. 给 Agent 使用
agent = Agent(
    model="gemini-2.0-flash",
    tools=bigquery_tools
)
```

### 工具过滤三种方式

```python
# 方式 1：白名单
tool_filter=["query", "list_tables"]

# 方式 2：谓词函数
def is_safe(name):
    return "delete" not in name.lower()
tool_filter=is_safe

# 方式 3：前缀命名（避免冲突）
tool_name_prefix="bq_"  # bq_query, bq_list_tables
```

### 自定义认证

```python
def custom_header_provider() -> dict:
    return {
        "Authorization": f"Bearer {get_access_token()}",
    }

api_registry = ApiRegistry(
    api_registry_project_id="...",
    header_provider=custom_header_provider
)
```

---

## 与我项目的关联

### 当前状态

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Copilot    │  │    Trader    │  │     Data     │
│  .env 各自配  │  │  .env 各自配  │  │  .env 各自配  │
│  权限靠人脑   │  │  权限靠人脑   │  │  权限靠人脑   │
└──────────────┘  └──────────────┘  └──────────────┘
```

### 潜在风险检查

| 问题 | 当前状态 | 风险等级 |
|------|---------|---------|
| Trader Agent 被黑能造成多大损失？ | 有交易权限 | ⚠️ 中 |
| API Key 是否各自独立？ | 部分共享 | ⚠️ 中 |
| 有没有调用日志审计？ | 无 | ⚠️ 低 |

### 未来演进方向（不急，心里有数）

```
┌─────────────────────────────────────────────────┐
│           统一 Tool Registry                     │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐     │
│  │Polymarket │ │ Database  │ │ Notifier  │     │
│  │   API     │ │   API     │ │   API     │     │
│  └───────────┘ └───────────┘ └───────────┘     │
│                                                 │
│  权限配置：                                      │
│  - Copilot: 只读市场数据                        │
│  - Trader: 读市场 + 执行交易（限额）             │
│  - Data: 读写数据库                             │
└─────────────────────────────────────────────────┘
```

### 现阶段行动项

- [ ] 检查 Trader 的 API 权限范围
- [ ] 确认各项目 API Key 是否隔离
- [ ] 考虑添加关键操作日志

---

## 课程脉络回顾

| Day | 主题 | 与 Day 17 的关系 |
|-----|------|-----------------|
| Day 11 | Google Managed MCP | MCP 是基础协议 |
| Day 14 | A2A Protocol | Agent 间通信 |
| Day 16 | 跨框架编排 | 多框架统一管理 |
| **Day 17** | **Cloud API Registry** | **工具治理层** |

**演进路径**：
```
单工具调用 → MCP 协议 → 多 Agent 编排 → 统一工具治理
```

---

## 金句记录

> "单个 Agent 调 API 很简单，但当你有 10 个 Agent 时，真正的问题是工具治理。"

> "Cloud API Registry = Agent 世界的 API Gateway"

> "你不需要现在就用企业级方案，但心里要有这张架构图。"

---

## 明日预告

**Day 18**: Self-Improving Agents
- 学习如何让 Agent 从反馈中自我改进

---

*记录者: Leo + Claude*
*最后更新: 2026-01-13*
````
<!-- DAILY_CHECKIN_2026-01-13_END -->

# 2026-01-12
<!-- DAILY_CHECKIN_2026-01-12_START -->


````markdown
# Day 16 学习笔记：LangGraph + A2A (Cross-Framework Orchestration)

> **日期**: 2026-01-12
> **主题**: 跨框架编排与 Agent 架构选型

---

## 核心收获

### 1. 主流 Agent 框架对比

| 框架 | 出品方 | 核心理念 | 适用场景 |
|------|--------|----------|----------|
| **Google ADK** | Google | 原生 Vertex AI 集成 | 企业级、Google 生态 |
| **LangGraph** | LangChain | 状态图工作流 | 复杂工作流、需要回滚 |
| **CrewAI** | CrewAI Inc | 角色扮演多代理 | 快速原型、业务建模 |
| **AutoGen** | Microsoft | 多代理对话 | 研究、对话实验 |
| **Claude SDK** | Anthropic | 强模型 + 开放协议 | 推理密集型任务 |

**策略差异**：
- Google: 重框架（ADK）+ A2A 协议
- Anthropic: 轻框架（SDK）+ MCP 协议

### 2. A2A vs MCP

| 协议 | 主导方 | 定位 | 特点 |
|------|--------|------|------|
| **A2A** | Google | Agent 间通信 | 框架互通、分布式 |
| **MCP** | Anthropic | AI 连接工具/数据 | 更通用、已成事实标准 |

### 3. 多 Agent vs 单 Agent + 多工具

```
❌ 常见误区：每个功能一个 Agent
   数据Agent → 分析Agent → 风控Agent → 执行Agent
   问题：协调复杂、延迟高、调试难

✅ 务实方案：1 个 Agent + 多工具
   主 Agent (Claude/Gemini)
     ├── 数据工具
     ├── 分析工具
     ├── 风控工具
     └── 执行工具
   优势：简单、快速、可控
```

**什么时候才需要多 Agent**：
- 需要不同"观点"辩论（Bull vs Bear）
- 任务可并行且相互独立
- 需要分布式部署

### 4. MCP 的正确定位

```
MCP 适用场景：
✅ 本地开发（Claude Code、Cursor）
✅ 内部工具、个人助手
✅ 原型验证

MCP 不适合：
❌ 高并发生产环境
❌ Web 产品后端

生产环境应该用：
→ 直接 Tool Use / Function Calling
→ 函数直接定义在代码里
```

### 5. 预测市场 Agent 选型建议

**推荐方案**：Claude SDK + Tool Use

理由：
1. 预测市场核心是"判断准"，Claude 推理最强
2. 不需要复杂框架，工具调用足够
3. 灵活、可控、好调试

**架构建议**：
```
当前：Copilot (Gemini) → 用户看 UI
建议：保持现状，geminiServiceV2.ts 做法正确

未来演进（如果需要）：
1. 拆分 geminiServiceV2.ts（63KB 太大）
2. 统一 AI 分析接口
3. 考虑 Claude 做深度分析（推理更强）
```

---

## 关键概念

### LangGraph 核心

```python
# State：在节点之间传递的数据
class AgentState(TypedDict):
    messages: list
    task: str
    result: str

# Node：执行具体操作的函数
def analyzer_node(state): ...
def executor_node(state): ...

# Edge：定义流转逻辑
graph.add_edge("analyzer", "executor")
graph.add_conditional_edges("executor", should_continue, {...})
```

### A2A 协议核心

```python
# Agent Card：自描述能力（类似 OpenAPI）
AgentCard(
    name="researcher",
    capabilities=["market_research", "analysis"],
    input_schema={...},
    output_schema={...}
)

# Task：工作单元
A2ATask(
    input_data={"query": "..."},
    state=TaskState.RUNNING
)
```

---

## 与我项目的关联

### 现有架构（已经很清晰）

```
Polymarket Data → Supabase → prediction-copilot (Gemini 分析)
                           → prediction-trader (套利策略，规则驱动)
```

### 不需要改的

- 不需要多 Agent
- 不需要 A2A 协议
- 不需要复杂状态图
- 现有 Tool Use 做法正确

### 可以优化的（P1）

1. 拆分 `geminiServiceV2.ts`（63KB 太大）
2. 统一 AI 分析工具函数

---

## 代码练习

今日创建的示例：
- [01_langgraph_basics.py](../day16/01_langgraph_basics.py) - LangGraph 基础
- [02_langgraph_as_tool.py](../day16/02_langgraph_as_tool.py) - LangGraph 作为 ADK Tool
- [03_a2a_cross_framework.py](../day16/03_a2a_cross_framework.py) - A2A 跨框架通信

---

## 金句记录

> "预测市场的胜负在于'判断准'，不在于'框架炫'。"

> "先用最简单的架构跑通闭环，有了真实数据反馈后，再决定是否拆分 Agent。"

> "过早的复杂架构是预测市场项目的头号杀手。"

---

## 明日预告

**Day 17**: Cloud API Registry + ADK
- 学习如何将云端 API 注册并集成到 ADK Agent

---

*记录者: Leo + Claude*
*最后更新: 2026-01-12*
````
<!-- DAILY_CHECKIN_2026-01-12_END -->

# 2026-01-11
<!-- DAILY_CHECKIN_2026-01-11_START -->



````markdown
# Day 15: A2UI (Generative UIs) 学习笔记

> **日期**: 2026-01-11
> **主题**: Agent-to-User Interface - 让 Agent "说 UI"

---

## 核心概念

### A2UI 是什么？

Google 2025年12月发布的开源协议，让 Agent 生成声明式 JSON 来描述 UI，而非返回纯文本。

**一句话**：Agent 不再只是"回答问题"，而是"提供工具界面"。

### 三大设计原则

| 原则 | 含义 |
|------|------|
| **Security First** | 声明式 JSON，非可执行代码；Component Catalog 白名单 |
| **LLM-Friendly** | 扁平化组件结构，易于增量生成 |
| **Cross-Platform** | 支持 React/Flutter/Angular 等原生渲染 |

### 为什么不直接让 LLM 生成 HTML？

**安全性**。LLM 可能被注入攻击，生成恶意代码。A2UI 只能请求预定义组件，无法执行任意代码。

类比：餐厅菜单 vs 开放式厨房
- 传统方式 = 客人进厨房随便做（危险）
- A2UI = 客人只能从菜单点菜（安全）

---

## 协议要点

### 四种消息类型

1. `createSurface` - 创建界面
2. `updateComponents` - 更新组件
3. `updateDataModel` - 更新数据
4. `deleteSurface` - 删除界面

### 组件格式（扁平结构）

```json
{
  "updateComponents": {
    "surfaceId": "main",
    "components": [
      {"id": "root", "component": "Column", "children": ["title"]},
      {"id": "title", "component": "Text", "text": "Hello"}
    ]
  }
}
```

**为什么扁平？** LLM 可以逐个组件生成，支持流式渲染和增量更新。

---

## 协议栈关系

```
A2UI (Agent → User)     ← 今天学的
   ↑ 建立在
A2A  (Agent → Agent)    ← Day 14
   ↑ 建立在
ADK  (开发框架)          ← Week 1-2
```

---

## 对我项目的价值评估

### Prediction Copilot

- **潜在应用**: 分析结果动态生成卡片/图表
- **当前优先级**: 中等（V2 引擎 + 固定 UI 已能满足需求）
- **适用场景**: 如果未来要做更灵活的分析结果展示

### Prediction Trader

- **不适用**: 量化执行需要确定性，不需要动态 UI

---

## 关键认知

### "应用课程所学"的三个层次

| 层次 | 含义 |
|------|------|
| Level 1 | 有类似代码 ❌（碰巧相似，不是应用）|
| Level 2 | 理解设计意图 ✅（能解释为什么这样设计）|
| Level 3 | 有意识的选择 ✅✅（知道何时用/不用）|

### Agent vs AI 调用

| | AI 调用 | Agent |
|---|---|---|
| 模式 | 输入→LLM→输出（一次性）| 自主规划→多步执行→迭代 |
| 控制 | 你控制流程 | Agent 自己决定下一步 |

Copilot V2 引擎目前是 **AI 调用**模式，不是 Agent 模式。

---

## 代码实践

```bash
cd day15
python3 concepts.py      # A2UI 消息格式演示
python3 a2ui_schema.py   # Pydantic Schema 定义
python3 simple_agent.py  # Agent 示例
```

---

## 参考资料

- [Google Developers Blog - Introducing A2UI](https://developers.googleblog.com/introducing-a2ui-an-open-project-for-agent-driven-interfaces/)
- [A2UI GitHub](https://github.com/google/A2UI)
- [A2UI Specification v0.9](https://a2ui.org/specification/v0.9-a2ui/)

---

## 一句话总结

A2UI 让 Agent 能安全地生成动态 UI，但对我当前项目来说是"好知道"而非"必须用"。核心价值在于理解**声明式 UI 生成**的设计模式。

---
*Week 3 Day 1 完成。进入 Advanced Architectures 阶段。*
````
<!-- DAILY_CHECKIN_2026-01-11_END -->

# 2026-01-10
<!-- DAILY_CHECKIN_2026-01-10_START -->




````markdown
# Day 14: Connecting Agents with A2A (Agent2Agent Protocol)

> **日期**: 2026-01-10
> **主题**: A2A Protocol, 分布式 Agent 通信, 跨服务协作
> **状态**: ✅ 完成

---

## 🎯 核心目标

1.  **理解 A2A 协议**: 掌握 Agent-to-Agent 通信的核心概念和架构。
2.  **暴露 Agent 服务**: 使用 `to_a2a()` 将本地 Agent 发布为网络服务。
3.  **消费远程 Agent**: 使用 `RemoteA2aAgent` 调用远程 A2A 服务。
4.  **实践多服务架构**: 构建一个基于 A2A 的分布式 Agent 系统。

---

## 🧠 概念地图

### 1. 为什么需要 A2A？

在复杂的企业级应用中，Agent 可能需要：
- 跨**不同服务器**部署（微服务架构）
- 跨**不同编程语言**协作（Python Agent ↔ Go Agent）
- 与**第三方 Agent** 集成（不同供应商）
- 建立**正式的 API 契约**（而非内部代码依赖）

```
┌─────────────────────────────────────────────────────────────┐
│                    A2A vs Local Sub-Agents                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Local Sub-Agents (Day 1-13)      A2A Protocol (Day 14)    │
│   ─────────────────────────────    ─────────────────────    │
│   同一进程内通信                    跨网络 HTTP 通信          │
│   共享内存 (session.state)         JSON-RPC / SSE           │
│   无需序列化                        需要序列化/反序列化        │
│   高性能、低延迟                    支持分布式、可扩展         │
│   适合: 内部模块组织                适合: 微服务、第三方集成    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2. A2A 协议核心概念

| 概念 | 说明 |
|------|------|
| **Agent Card** | `.well-known/agent.json` — Agent 的"名片"，描述能力和端点 |
| **Client Agent** | 发起请求的 Agent（消费者） |
| **Remote Agent** | 提供服务的 Agent（生产者） |
| **Task** | A2A 通信的基本单位，包含请求和响应 |
| **Artifact** | 任务执行的产物（文本、文件等） |

### 3. A2A 通信流程

```
┌──────────────┐           ┌──────────────┐           ┌──────────────┐
│   Client     │  1. GET   │    A2A       │           │   Remote     │
│   Agent      │ ─────────>│   Server     │           │   Agent      │
│              │agent.json │              │           │              │
│              │<─────────-│              │           │              │
│              │           │              │           │              │
│              │ 2. POST   │              │ 3. Run    │              │
│              │ ─────────>│   /tasks     │ ─────────>│              │
│              │  (Task)   │              │           │              │
│              │           │              │ 4. Yield  │              │
│              │ 5. SSE    │              │ <─────────│              │
│              │ <─────────│  (Events)    │           │              │
│              │           │              │           │              │
└──────────────┘           └──────────────┘           └──────────────┘
```

> **核心技术栈**: HTTP + JSON-RPC + Server-Sent Events (SSE)

---

## ⚒️ 实践示例

### 示例 1: 暴露 Agent 为 A2A 服务 (`to_a2a()`)

**目标**: 将一个简单的 Agent 发布为可被远程调用的服务。

```python
# day14/prime_server/agent.py
from google.adk.agents import Agent

def check_prime(number: int) -> dict:
    """检查一个数是否为质数。
    
    Args:
        number: 要检查的整数
        
    Returns:
        包含检查结果的字典
    """
    if number < 2:
        return {"number": number, "is_prime": False, "reason": "小于2"}
    for i in range(2, int(number**0.5) + 1):
        if number % i == 0:
            return {"number": number, "is_prime": False, "divisor": i}
    return {"number": number, "is_prime": True}

root_agent = Agent(
    name="prime_checker",
    model="gemini-2.5-flash",
    description="一个专门检查质数的 Agent。提供数字，告诉你是否为质数。",
    instruction="你是一个数学助手，专门检查用户提供的数字是否为质数。使用 check_prime 工具。",
    tools=[check_prime],
)
```

```python
# day14/prime_server/main.py
from google.adk.a2a.utils.agent_to_a2a import to_a2a
from agent import root_agent

# 一行代码将 Agent 转换为 A2A 服务！
a2a_app = to_a2a(root_agent, port=8001)

# 启动命令: uvicorn main:a2a_app --host localhost --port 8001
```

**启动后可访问**:
- Agent Card: `http://localhost:8001/.well-known/agent.json`
- A2A 端点: `http://localhost:8001/a2a/prime_checker/`

### 示例 2: 消费远程 A2A Agent (`RemoteA2aAgent`)

**目标**: 从另一个 Agent 中调用远程的质数检查服务。

```python
# day14/math_client/agent.py
from google.adk.agents import Agent
from google.adk.a2a.remote_a2a_agent import RemoteA2aAgent

# 创建远程 Agent 的本地代理
prime_checker = RemoteA2aAgent(
    name="remote_prime_checker",
    description="远程质数检查服务。可以检查任意整数是否为质数。",
    agent_card="http://localhost:8001/.well-known/agent.json"
)

# 将远程 Agent 作为 sub_agent 使用
root_agent = Agent(
    name="math_tutor",
    model="gemini-2.5-flash",
    description="数学辅导助手，可以帮助解答各种数学问题。",
    instruction="""你是一个数学辅导老师。
    
当用户询问关于质数的问题时，你应该将任务委托给 remote_prime_checker。
对于其他数学问题，你可以直接回答。
""",
    sub_agents=[prime_checker],  # 像本地 sub_agent 一样使用！
)
```

### 示例 3: 完整的分布式系统

**架构图**:

```
                    ┌─────────────────┐
                    │   Coordinator   │  Port: 8000
                    │   (主协调器)     │
                    └────────┬────────┘
                             │ A2A
            ┌────────────────┼────────────────┐
            │                │                │
            ▼                ▼                ▼
   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
   │ Math Agent  │  │ Weather     │  │ Translator  │
   │ Port: 8001  │  │ Agent: 8002 │  │ Agent: 8003 │
   └─────────────┘  └─────────────┘  └─────────────┘
```

**Coordinator Agent**:

```python
# day14/coordinator/agent.py
from google.adk.agents import Agent
from google.adk.a2a.remote_a2a_agent import RemoteA2aAgent

# 连接多个远程服务
math_agent = RemoteA2aAgent(
    name="math_service",
    description="数学计算服务",
    agent_card="http://localhost:8001/.well-known/agent.json"
)

weather_agent = RemoteA2aAgent(
    name="weather_service", 
    description="天气查询服务",
    agent_card="http://localhost:8002/.well-known/agent.json"
)

translator_agent = RemoteA2aAgent(
    name="translation_service",
    description="多语言翻译服务",
    agent_card="http://localhost:8003/.well-known/agent.json"
)

root_agent = Agent(
    name="smart_coordinator",
    model="gemini-2.5-flash",
    description="智能协调器，可以调用多种专业服务。",
    instruction="""你是一个智能助手，可以协调多个专业服务来回答用户问题。

可用服务:
- math_service: 处理数学计算和质数检查
- weather_service: 查询天气信息
- translation_service: 翻译文本

根据用户需求，将任务委托给合适的服务。
""",
    sub_agents=[math_agent, weather_agent, translator_agent],
)
```

---

## 🔧 CLI 工具

### 使用 `adk api_server --a2a`

除了 `to_a2a()` 方法，ADK 还提供 CLI 方式启动 A2A 服务：

```bash
# 需要先创建 agent.json 文件
adk api_server --a2a ./my_agent_folder
```

**手动创建 Agent Card** (`agent.json`):

```json
{
  "name": "prime_checker",
  "description": "检查数字是否为质数的 Agent",
  "url": "http://localhost:8001/a2a/prime_checker/",
  "version": "1.0.0",
  "capabilities": {
    "streaming": true,
    "pushNotifications": false
  },
  "skills": [
    {
      "id": "check_prime",
      "name": "质数检查",
      "description": "检查给定的整数是否为质数"
    }
  ]
}
```

---

## 🔑 关键收获

| 知识点 | 一句话总结 |
|--------|-----------|
| **A2A Protocol** | 网络层的 Agent 通信标准，基于 HTTP + JSON-RPC + SSE |
| **Agent Card** | Agent 的"名片"，描述能力和端点，位于 `.well-known/agent.json` |
| **`to_a2a()`** | 一行代码将 ADK Agent 转换为 A2A 可访问的 FastAPI 服务 |
| **`RemoteA2aAgent`** | 远程 Agent 的本地代理，可像本地 sub_agent 一样使用 |
| **适用场景** | 微服务架构、跨语言协作、第三方集成、正式 API 契约 |
| **vs Local Sub-Agents** | A2A 适合分布式；本地 sub_agents 适合单进程内模块化 |

---

## 🧪 动手练习

### 练习 1: 启动质数检查服务
1. 创建 `day14/prime_server/` 目录
2. 实现 `agent.py` 和 `main.py`
3. 运行 `uvicorn main:a2a_app --port 8001`
4. 访问 `http://localhost:8001/.well-known/agent.json` 验证

### 练习 2: 创建客户端 Agent
1. 创建 `day14/math_client/` 目录
2. 使用 `RemoteA2aAgent` 连接质数服务
3. 通过 `adk web` 测试委托功能

### 练习 3 (挑战): 构建微服务生态
1. 启动 3 个不同功能的 A2A 服务
2. 创建一个 Coordinator 统一调度
3. 测试跨服务的复杂查询

---

## 🔗 参考资源

- [ADK A2A 官方文档](https://google.github.io/adk-docs/a2a/)
- [A2A Protocol 规范](https://a2aprotocol.org/)
- [Google Developers Blog: A2A 发布公告](https://developers.googleblog.com/)
- GEMINI.md Section 4.A (Distributed Communication)
- ADK Python Cheatsheet Section 4 (Multi-Agent Systems)

---

## ✅ 实践验证记录

### 环境准备

```bash
# 安装 A2A SDK
uv pip install a2a-sdk
# 安装版本: a2a-sdk==0.3.22
```

### 服务端验证

**启动命令**:
```bash
cd day14/prime_server
python main.py
```

**输出**:
```
🚀 正在启动 A2A 质数检查服务...
📋 Agent Card: http://localhost:8001/.well-known/agent.json
🔗 A2A 端点: http://localhost:8001/a2a/prime_checker/
INFO:     Uvicorn running on http://localhost:8001 (Press CTRL+C to quit)
```

### Agent Card 验证

**请求**: `curl http://localhost:8001/.well-known/agent.json`

**响应** (格式化后):
```json
{
  "name": "prime_checker",
  "description": "一个专门检查质数的 Agent...",
  "protocolVersion": "0.3.0",
  "preferredTransport": "JSONRPC",
  "url": "http://localhost:8001",
  "defaultInputModes": ["text/plain"],
  "defaultOutputModes": ["text/plain"],
  "skills": [
    {
      "id": "prime_checker",
      "name": "model",
      "tags": ["llm"]
    },
    {
      "id": "prime_checker-check_prime",
      "name": "check_prime",
      "description": "检查一个数是否为质数...",
      "tags": ["llm", "tools"]
    }
  ]
}
```

### 注意事项

1. **实验性功能**: ADK 的 A2A 实现目前标记为 `[EXPERIMENTAL]`，API 可能会有变化
2. **Agent Card 端点更新**: 新版推荐使用 `/.well-known/agent-card.json`（旧的 `agent.json` 已弃用）
3. **依赖关系**: 需要单独安装 `a2a-sdk` 包

### Week 2 完成总结

| Day | 主题 | 核心知识点 |
|-----|------|-----------|
| Day 08 | Context Management | ADK 上下文层级、State Prefix |
| Day 09 | Time Travel | Session 快照、状态回滚 |
| Day 10 | Caching & Compaction | Context Cache、Events Compaction |
| Day 11 | MCP | MCPToolset、外部服务集成 |
| Day 12 | Multimodal | Live API、Audio/Video 流式处理 |
| Day 13 | Interactions API | Callbacks、Tool Confirmation |
| Day 14 | A2A Protocol | 分布式 Agent 通信、微服务架构 |

**🎉 恭喜完成 Week 2！你已掌握 ADK 的上下文管理与编排能力。**

````
<!-- DAILY_CHECKIN_2026-01-10_END -->

# 2026-01-09
<!-- DAILY_CHECKIN_2026-01-09_START -->





````markdown
# Day 13: Interactions API (Stateful Workflows)

> **日期**: 2026-01-09
> **主题**: Callbacks, Tool Confirmation, Human-in-the-Loop
> **状态**: ✅ 完成

---

## 🎯 核心目标

1.  **理解 Callbacks 机制**: 掌握 Agent 生命周期中的 6 大 Hook 点。
2.  **Human-in-the-Loop**: 使用 `require_confirmation` 实现敏感操作的人工确认。
3.  **Interactions API 概念**: 了解 Google 最新推出的有状态工作流 API。

---

## 🧠 概念地图

### 1. Callbacks: Agent 的拦截器

Callbacks 是 ADK 提供的**拦截与控制机制**，允许开发者在 Agent 执行的关键节点注入自定义逻辑。

```
┌──────────────────────────────────────────────────────┐
│                  Agent 执行生命周期                    │
├──────────────────────────────────────────────────────┤
│  User Message                                        │
│       ↓                                              │
│  [before_agent_callback] ── 返回 Content 可跳过整个 Agent │
│       ↓                                              │
│  Agent Logic (e.g., LlmAgent)                        │
│       ↓                                              │
│      ┌─ [before_model_callback] ── 返回 LlmResponse 可跳过 LLM │
│      │       ↓                                       │
│      │   LLM Call (e.g., Gemini)                     │
│      │       ↓                                       │
│      └─ [after_model_callback] ── 可替换 LLM 响应     │
│       ↓                                              │
│      ┌─ [before_tool_callback] ── 返回 dict 可跳过工具执行 │
│      │       ↓                                       │
│      │   Tool Execution                              │
│      │       ↓                                       │
│      └─ [after_tool_callback] ── 可替换工具结果       │
│       ↓                                              │
│  [after_agent_callback] ── 可替换最终 Agent 输出      │
│       ↓                                              │
│  Final Response                                      │
└──────────────────────────────────────────────────────┘
```

**核心原则**:
- `return None`: 允许默认行为继续执行。
- `return <特定对象>`: **覆盖**默认行为，跳过后续步骤。

### 2. Tool Confirmation (Human-in-the-Loop)

当 Agent 需要执行敏感操作（如删除数据、转账等）时，可以暂停执行，等待人工确认。

**三种确认模式**:

| 模式 | 使用方式 | 适用场景 |
|------|----------|----------|
| **Boolean** | `FunctionTool(my_func, require_confirmation=True)` | 所有调用都需确认 |
| **Dynamic** | `FunctionTool(my_func, require_confirmation=lambda amount: amount > 10000)` | 根据参数动态判定 |
| **Payload** | `tool_context.request_confirmation(hint, payload)` | 需用户提供额外选项 |

### 3. Interactions API (Google 2025 新推出)

Interactions API 是 Google 从**无状态 request-response** 转向**有状态 multi-turn 工作流**的关键基础设施。

**核心特性**:
- **原生状态管理**: 自动处理对话历史 (Conversation History)。
- **后台执行**: 支持长时间运行的任务 (Long-running tasks)。
- **统一接口**: 可同时与原始模型 (Raw Model) 和托管 Agent (e.g., Deep Research) 交互。
- **支持模型**: Gemini 3.0, Gemini 2.5, Deep Research Preview。

> **与 ADK 的关系**: Interactions API 可作为 ADK Agent 的 Inference Engine，是 `generateContent` API 的有状态进化版本。

---

## ⚒️ 实践示例

### 示例 1: `before_model_callback` - 输入拦截

**场景**: 检测到敏感关键词时，阻止 LLM 调用。

```python
from google.adk.agents import Agent
from google.adk.agents.callback_context import CallbackContext
from google.adk.models import LlmResponse, LlmRequest
from google.genai import types
from typing import Optional

def block_sensitive_input(
    callback_context: CallbackContext, 
    llm_request: LlmRequest
) -> Optional[LlmResponse]:
    """如果用户输入包含 'BLOCK'，则跳过 LLM 调用。"""
    last_user_msg = ""
    if llm_request.contents and llm_request.contents[-1].role == 'user':
        if llm_request.contents[-1].parts:
            last_user_msg = llm_request.contents[-1].parts[0].text
    
    if "BLOCK" in last_user_msg.upper():
        print("[Callback] 'BLOCK' 关键词检测到，跳过 LLM 调用。")
        return LlmResponse(
            content=types.Content(
                role="model",
                parts=[types.Part(text="此请求因安全策略被拦截。")],
            )
        )
    return None  # 允许 LLM 调用继续

my_agent = Agent(
    name="safe_agent",
    model="gemini-2.5-flash",
    instruction="你是一个安全的助手。",
    before_model_callback=block_sensitive_input,
)
```

### 示例 2: Tool Confirmation - 动态确认

**场景**: 转账金额超过 10000 时需要人工确认。

```python
from google.adk.tools import FunctionTool, ToolContext

def wire_money(amount: float, recipient: str, tool_context: ToolContext) -> dict:
    """转账到指定账户。"""
    # 执行转账逻辑...
    return {"status": "success", "amount": amount, "to": recipient}

# 金额 > 10000 时触发确认流程
def needs_approval(amount: float, **kwargs) -> bool:
    return amount > 10000

transfer_tool = FunctionTool(wire_money, require_confirmation=needs_approval)
```

### 示例 3: Payload Confirmation - 高级交互

**场景**: 预订机票时，让用户选择座位等级。

```python
from google.adk.tools import ToolContext

def book_flight(destination: str, price: float, tool_context: ToolContext) -> dict:
    """预订机票，需用户确认并选择座位等级。"""
    tool_context.request_confirmation(
        hint="请确认预订并选择座位等级。",
        payload={"seat_class": ["economy", "business", "first"]}
    )
    return {"status": "pending_confirmation"}
```

---

## 🔑 关键收获

| 知识点 | 一句话总结 |
|--------|-----------|
| **Callbacks** | Agent 的 AOP (面向切面编程)，拦截并控制执行流 |
| **返回值语义** | `None` = 继续执行；`对象` = 覆盖跳过 |
| **Tool Confirmation** | 敏感操作的"保险丝"，支持 Boolean/Dynamic/Payload 三种模式 |
| **Interactions API** | Google 新一代有状态 API，ADK 的底层引擎可选项 |
| **Plugins** | 全局 Callbacks，应用于 Runner 级别的所有 Agent |

---

## 🔗 参考资源

- [ADK Callbacks 官方文档](https://google.github.io/adk-docs/callbacks/)
- [Google Interactions API 发布博客](https://developers.googleblog.com/)
- ADK Python Cheatsheet Section 10 (Control Flow with Callbacks)
- GEMINI.md Section 7.4 (Tool Confirmation)
````
<!-- DAILY_CHECKIN_2026-01-09_END -->

# 2026-01-08
<!-- DAILY_CHECKIN_2026-01-08_START -->






# Day 12: Multimodal Agents (Gemini Live API & Streaming)

> **日期**: 2026-01-08 **主题**: Streaming Responses & Multimodal Inputs **状态**: ✅ 完成

* * *

## 🎯 核心目标

1.  **理解 Streaming**: 如何在 ADK 中开启流式输出，提升用户体验。
    
2.  **多模态处理 (Multimodal)**: 让 Agent 不仅能读文字，还能“看”图片和“听”音频。
    
3.  **Gemini Live API (BIDI)**: 初探双向实时交互机制。
    

* * *

## 🧠 概念预热

### 1\. Streaming (流式输出)

通过 `RunConfig(streaming_mode=StreamingMode.SSE)`，Agent 的响应将分块返回。这在长文本生成或复杂工具调用时能显著降低感知延迟。

### 2\. Multimodal (多模态)

Gemini 原生支持文本、图像、音频和视频。在 ADK 中，我们可以通过 `runner.run_async` 发送包含 `genai_types.Part` 数据的 `Content`。

### 3\. Gemini Live API

支持毫秒级延迟的双向音视频交互。ADK 通过 `StreamingMode.BIDI` 和 `LiveRequestQueue` 来管理实时流。

* * *

## ⚒️ 实践记录

### 1\. 实现 Agent (`day12/agent.py`)

创建了名为 `multimodal_explorer` 的 Agent，使用 `gemini-2.0-flash` 模型。

### 2\. 流式输出验证 (`day12/test_stream.py`)

-   **关键点**: 必须使用 `StreamingMode.SSE` (Server-Sent Events)。
    
-   **验证**: 实现了异步迭代捕获分块输出。
    

### 3\. 多模态验证 (`day12/test_multimodal.py`)

-   **发现**: 脚本调用 100% 成功。
    
-   **发送图片**: 使用 `genai_types.Part.from_bytes(data=image_bytes, mime_type="image/png")`。
    

* * *

## 🔍 故障排查 (Troubleshooting)

### 🚨 案例: 网页端上传图片报 400 错误

**症状**: 在 ADK Web UI (Playground) 拖入图片发送时，返回 `400 INVALID_ARGUMENT: Provided image is not valid.`。

**排查与结论**:

-   **现象**: 脚本调用正常，但 UI 发起的请求长度只有 ~86KB（实际图片 ~560KB）。
    
-   **原因**: ADK Web UI 存在截断大文件或前端编码 Bug。
    
-   **解决方案**: 在开发阶段优先使用 Python 脚本验证，UI 测试建议使用极小图片（<60KB）。
    

* * *

## 🔗 参考资源

-   ADK Bidi Demo
    
-   [Gemini Live API Docs](https://ai.google.dev/gemini-api/docs/live)
    
-   ADK Python Cheatsheet Section 15
<!-- DAILY_CHECKIN_2026-01-08_END -->

# 2026-01-07
<!-- DAILY_CHECKIN_2026-01-07_START -->







````markdown
# Day 11: Google Managed MCP (Connecting to Services)

> **日期**: 2026-01-07
> **主题**: Model Context Protocol (MCP) & Google Cloud Action API
> **状态**: ✅ 完成

---

## 🎯 核心目标

1.  **理解 MCP (Model Context Protocol)**: 为什么它是 AI Agent 连接世界的标准？
2.  **Google Managed MCP**: 如何不写复杂的 Server 代码，直接让 Agent 拥有安全连接数据的能力。

---

## 🧠 概念预热

### 1. 什么是 MCP？(The "USB" for AI)

在 MCP 出现之前，每个 Agent 都要自己写特定的 integration 代码来连接 Notion, Github, Slack...
MCP 定义了一种**标准协议**：
*   **MCP Server**: 提供资源 (Resources)、提示 (Prompts) 和工具 (Tools)。
*   **MCP Client (Your Agent)**: 通过标准方式消费这些能力。

这就好比 **USB 接口**：你不需要为每个鼠标单独焊死在电脑主板上，只要它是 USB 的，插上就能用。

### 2. 这个 Protocol 解决了什么问题？

*   **Fragmentation**: 不再需要 `langchain-notion`, `llama-index-notion`... 只需要一个 `mcp-server-notion`。
*   **User Context**: 让 AI 安全地访问你的本地文件、数据库或私有 SaaS 数据。

### 3. "Google Managed" 是什么意思？

通常运行 MCP Server 需要你自己在本地或服务器跑一个进程 (`npx @modelcontextprotocol/server-notion`)。
Google 的 Managed MCP (通常指 Vertex AI Extensions 或其生态集成) 试图让这个过程更“云原生”，或者就在 ADK 层面无缝支持。

*(注：由于 MCP 是 Anthropic 发起的开源标准，Google ADK 的支持方式可能是通过通用 Tool 接口兼容，或者特定的 Adapter。我们将探索 ADK 如何作为 MCP Client 使用)*

---

## 💻 代码实现 (day11/agent.py)

利用 `McpToolset` 和 `uvx` 运行 Python 版 MCP SQLite Server：

```python
from google.adk.tools import McpToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams, StdioServerParameters

sqlite_mcp_toolset = McpToolset(
    connection_params=StdioConnectionParams(
        server_params=StdioServerParameters(
            command="uvx",
            args=["mcp-server-sqlite", "--db-path", db_path],
        )
    ),
    tool_name_prefix="db_" 
)
```

## 📊 验证结果

运行 `day11/test_mcp.py` 结果表明 Agent 完美继承了 MCP 工具：

1.  **动态发现**: Agent 自动识别并调用了 `db_list_tables`。
2.  **数据交互**: Agent 成功从 SQLite `users` 表中读取了初始化的测试数据。

---

## 🔗 参考资源
- [Model Context Protocol (MCP) Official Site](https://modelcontextprotocol.io/)
- [ADK McpToolset Documentation](https://google.github.io/adk-python/tools/mcp/)
````
<!-- DAILY_CHECKIN_2026-01-07_END -->

# 2026-01-06
<!-- DAILY_CHECKIN_2026-01-06_START -->








````markdown
# Day 10: Big Context ≠ Better Memory (Caching & Compaction)

> **日期**: 2026-01-06
> **主题**: Context Caching + Event Compaction
> **状态**: ✅ 完成

---

## 🎯 一句话总结

> **用"缓存"换取速度，用"压缩"换取记忆。**

---

## 🧠 核心概念：通俗理解

### 1. Context Caching (上下文缓存) —— "书摊开在桌上"

*   **问题**: 就像每次回答问题前都要重新从头阅读一本 500 页的书（Token 计费贵，处理慢）。
*   **解决**: 把处理过的长文档（Context）**缓存**在服务端（书摊开放在桌上）。
*   **效果**: 下次提问时，直接基于缓存回答，**延迟极低，成本大幅下降**。
*   **适用**: 法律文档问答、长篇小说续写、拥有复杂 System Prompt 的角色扮演。

| 配置项 | 作用 | 推荐值 |
|:---|:---|:---|
| `min_tokens` | 只有超过这个长度才触发缓存 | 32,768 (32k) |
| `ttl_seconds` | 缓存多久不因使用而失效 | 3600 (1小时) |

### 2. Event Compaction (历史压缩) —— "定期写会议纪要"

*   **问题**: Agent 的脑容量（Context Window）有限。如果对话无限变长，最早的记忆（如用户的核心指令）会被挤出并遗忘。
*   **解决**: 定期暂停，回顾最近的 N 轮对话，将其**总结**成一段摘要（Summary），替换掉原始的废话。
*   **效果**: 腾出了脑容量，同时**保留了关键信息**。
*   **核心机制**: 原始对话 A, B, C, D -> 压缩为 [Summary(A,B)] + C, D。

| 配置项 | 作用 | 推荐值 |
|:---|:---|:---|
| `compaction_interval` | 每隔几轮对话触发一次压缩 | 5~10 |
| `overlap_size` | 压缩时保留最近几条原始消息 | 2 (保证上下文连贯) |

---

## 💻 代码实现 (day10/agent.py)

```python
from google.adk.apps import App
from google.adk.apps.app import EventsCompactionConfig
from google.adk.agents.context_cache_config import ContextCacheConfig

day10_app = App(
    name="day10_app",
    root_agent=reader_agent,
    
    # 🌟 启用缓存
    context_cache_config=ContextCacheConfig(
        min_tokens=32768, 
        ttl_seconds=3600
    ),
    
    # 🌟 启用压缩
    events_compaction_config=EventsCompactionConfig(
        compaction_interval=5,
        overlap_size=2
    )
)
```

---

## 📊 验证结果

通过 `day10/test_compaction.py` 我们验证了：

1.  **Caching 生效**: 日志显示 Agent 检查了 context 长度（当前测试文本较短，提示 `Previous request too small`，证明检查逻辑在线）。
2.  **Compaction 生效**: 
    - 模拟了 5 轮对话。
    - 在第 3 轮附近，观察到了额外的 LLM 请求（摘要生成）。
    - 最终历史记录 (`session.events`) 包含了 Summary 事件，而非全是原始 Message。

---

## 🔗 参考资源
- [ADK Context Caching](https://google.github.io/adk-python/context-caching/)
- [ADK Event Compaction](https://google.github.io/adk-python/memory/compaction/)
````
<!-- DAILY_CHECKIN_2026-01-06_END -->

# 2026-01-05
<!-- DAILY_CHECKIN_2026-01-05_START -->









````markdown
# Day 09: Undo Buttons for Agents (Time Travel)

> **日期**: 2026-01-05  
> **主题**: Session Rewind + Resumability  
> **状态**: ✅ 完成

---

## 🎯 一句话总结

> **给 Agent 装上「撤销」和「断点续传」按钮**

---

## 🧠 核心概念：两个比喻

| 比喻 | ADK 概念 | 作用 | 代码入口 |
|------|----------|------|----------|
| ⏪ 撤销 | `rewind_async()` | 回滚到历史某一轮对话 | `runner.rewind_async()` |
| 💾 断点续传 | `ResumabilityConfig` | 中断后从失败点恢复 | `App(resumability_config=...)` |

---

## ⏪ 撤销：Session Rewind

**场景**：用户说"刚才那个答案不对，帮我重新回答"

### 工作原理

1. **指定回滚点**：通过 `invocation_id` 标识要撤销的那一轮对话
2. **计算逆向 Delta**：ADK 遍历事件历史，计算 State 和 Artifact 的回滚差值
3. **追加 Rewind Event**：不删除历史，而是追加一个 `rewind_event`，后续 LLM 不再「看」被撤销的内容

```python
from google.adk import Runner

# 回滚到某个 invocation 之前
await runner.rewind_async(
    user_id="user_123",
    session_id="session_abc",
    rewind_before_invocation_id="inv_to_undo"  # ← 要撤销的 invocation_id
)
```

### 限制 (重要!)

| 限制 | 说明 |
|------|------|
| ❌ 不回滚全局状态 | `app:` 和 `user:` 前缀的 State 不会被撤销 |
| ❌ 不回滚外部系统 | 如果 tool 调用了外部 API（如下单），需自己处理回滚 |
| ⚠️ 非原子操作 | 回滚中途若有其他写入，可能不一致 |

---

## 💾 断点续传：Resumability

**场景**：Agent 执行到一半，网络断了或服务重启了

### 启用方式

```python
from google.adk.apps import App, ResumabilityConfig

app = App(
    name="my_resumable_app",
    root_agent=my_agent,
    resumability_config=ResumabilityConfig(is_resumable=True)  # ← 核心配置
)
```

### 恢复执行

```python
# 通过 invocation_id 恢复之前中断的执行
async for event in runner.run_async(
    user_id="user_123",
    session_id="session_abc",
    invocation_id="interrupted_inv_id"  # ← 指定要恢复的 invocation
):
    print(event)
```

### 限制

| 限制 | 说明 |
|------|------|
| ⚠️ 至少一次执行 | 恢复时 tool 可能被重复调用，需保证幂等性 |
| ❌ 内存状态丢失 | 临时变量、`temp:` 前缀 State 会丢失 |

---

## 📊 对比总结

| 特性 | Rewind | Resumability |
|------|--------|--------------|
| 目的 | 回滚历史 | 从中断恢复 |
| 触发方式 | 用户主动 | 系统自动检测 |
| 数据影响 | 追加逆向事件 | 继续追加事件 |
| tool 行为 | 无重复调用 | 可能重复调用 |

---

## 💡 最佳实践

1. **Tool 设计幂等性**：恢复时 tool 可能被重复调用
2. **避免回滚活跃 Session**：确保没有并发写入
3. **全局状态用 `app:` / `user:`**：这些不会被 rewind 影响

---

## ✏️ 自测题（明天复习用）

1. 如何让用户"撤销"上一轮对话？
2. `ResumabilityConfig` 在哪里配置？
3. Rewind 不会回滚哪些 State？

---

## 📂 今日产出

- `day09/agent.py` - Resumable App 示例
- `day09/test_rewind.py` - Rewind 测试脚本 (概念性)
- `logs/day09/learning_notes.md` - 本文件

---

## 🔗 参考资源

- [ADK Rewind Sessions](https://google.github.io/adk-python/sessions/rewind-sessions/)
- [ADK Resuming Agents](https://google.github.io/adk-python/resuming-agents/)
````
<!-- DAILY_CHECKIN_2026-01-05_END -->

# 2026-01-04
<!-- DAILY_CHECKIN_2026-01-04_START -->










````markdown
# Day 08: Effective Context Management (ADK Layers)

> **日期**: 2026-01-04  
> **主题**: ADK 的上下文分层管理  
> **状态**: ✅ 完成

---

## 🎯 一句话总结

> **教 Agent 用便签纸传话 + 用笔记本记长期偏好 + 用文件柜存文档**

---

## 🧠 核心概念：三个比喻

| 比喻 | ADK 概念 | 代码示例 | 生命周期 |
|------|----------|----------|----------|
| 📝 便签 | `output_key` + `{state_key}` | Agent A 写，Agent B 读 | 当前会话 |
| 📓 笔记本 | `user:` 前缀 | `state["user:preference"]` | **跨会话永久** |
| 🗄️ 文件柜 | Artifacts | `save_artifact()` / `load_artifact()` | 按需配置 |

---

## 📝 便签：Agent 之间传话

**场景**：Summarizer 生成摘要 → Translator 读取并翻译

```python
# Agent A: 写便签
summarizer = Agent(
    name="summarizer",
    instruction="Summarize the text.",
    output_key="summary"  # ← 输出自动保存到 state["summary"]
)

# Agent B: 读便签
translator = Agent(
    name="translator",
    instruction="Translate this: {summary}"  # ← 自动从 state 注入
)

# 组合为流水线
pipeline = SequentialAgent(sub_agents=[summarizer, translator])
```

---

## 📓 笔记本：跨会话记忆

**场景**：记住用户偏好，下次对话还知道

```python
def remember_preference(preference: str, tool_context: ToolContext):
    # user: 前缀 = 跨会话持久化
    tool_context.state["user:preference"] = preference
    return {"status": "saved"}
```

**State 前缀速查**：
| 前缀 | 作用域 | 用途 |
|------|--------|------|
| (无) | 当前会话 | 工作流临时数据 |
| `user:` | 跨会话 | 用户偏好 |
| `app:` | 全局 | 应用设置 |
| `temp:` | 仅当前执行 | 一次性中间结果 |

---

## 🗄️ 文件柜：存储二进制文件

**场景**：生成报告并保存为 Artifact

```python
async def generate_report(topic: str, tool_context: ToolContext):
    content = f"# Report: {topic}\n..."
    artifact = types.Part.from_bytes(data=content.encode(), mime_type="text/markdown")
    
    # ⚠️ 异步方法，必须 await
    version = await tool_context.save_artifact(filename="report.md", artifact=artifact)
    return {"version": version}
```

---

## ⚠️ 今日踩坑

1. **`adk web` 启动位置**：必须从项目根目录运行，否则找不到 agent
2. **异步方法忘记 `await`**：`save_artifact()` 和 `load_artifact()` 是异步的，不加 `await` 会返回 `coroutine` 对象导致序列化错误

---

## ✏️ 自测题（明天复习用）

1. Agent A 的输出怎么让 Agent B 看到？
2. 怎么让数据在用户换个对话后还记得？
3. 存文件用什么方法？要注意什么？

---

## 📂 今日产出

- `day08/agent.py` - SequentialAgent Pipeline 示例
- `day08/tools.py` - User State + Artifacts 工具
- `day08/.env` - API 配置

---

## 🔗 参考资源

- [ADK Session & State 官方文档](https://google.github.io/adk-python/sessions/)
- [ADK Artifacts 官方文档](https://google.github.io/adk-python/artifacts/)
````
<!-- DAILY_CHECKIN_2026-01-04_END -->

# 2026-01-03
<!-- DAILY_CHECKIN_2026-01-03_START -->











# **📅 2026-01-03 Day 07 学习日记**

## **📝 学习主题：LLMs Can Execute Code (Autonomous Problem Solving)**

今天是 Week 1 的最后一天，我们将学习如何让 AI Agent **自主编写并执行代码** 来解决问题。通过

```
BuiltInCodeExecutor
```

，LLM 获得了一个强大的计算器和逻辑执行引擎，真正实现了 "Neuro-Symbolic"（神经符号）AI。

### **🧠 核心概念**

**1\. 为什么需要代码执行能力？**

LLM 的"思考"是基于概率的，擅长语言生成，但有以下先天局限：

-   **精确计算**: 复杂的数学运算容易出错（LLM 是"文科生"）。
    
-   **数据处理**: 处理大量 CSV/JSON 数据时，Token 上下文有限且容易产生幻想。
    
-   **可复现性**: 自然语言推理是不可控的，而代码逻辑是确定性的。
    
-   **复杂算法**: 排序、搜索、图遍历等算法用代码实现远比 Prompt 高效。
    

**关键洞见**:

-   **Prompt**: 负责 "Intent" (意图) 和 "Design" (设计)。
    
-   **Code**: 负责 "Action" (行动) 和 "Precision" (精确)。
    

**2\. ADK 中的 Code Executor 机制**

ADK 通过

```
CodeExecutor
```

接口将 LLM 生成的代码发送到沙盒环境运行。

| 执行器 | 底层机制 | 适用场景 | 安全性 |
| --- | --- | --- | --- |
| BuiltInCodeExecutor | Docker 容器或本地受限进程 (gVisor) | 开发测试、简单脚本、本地工具 | ⭐⭐⭐ (依赖环境配置) |
| AgentEngineSandboxCodeExecutor | Google Cloud 全托管沙盒 (MicroVMs) | 生产环境、企业级应用、高并发 | ⭐⭐⭐⭐⭐ (硬件级隔离) |

> **_⚠️ 安全警示_**_: 即使是本地沙盒，也绝对_**_禁止_**_将 Code Executor 开放在未受信任的输入源（如允许用户直接让 Agent 执行任意代码）。这是 "Prompt Injection" 的高危区。_

**3\. 进阶知识点 (Deep Dive)**

🔄 The "Self-Correction" Loop (自我纠错循环)

这是 Agent 最强大的能力之一。当代码执行报错时，Agent 能够：

1.  **捕获 Stderr**: 读取 Traceback 错误信息。
    
2.  **Analysis**: 即时分析错误原因（是语法错误、库缺失，还是逻辑漏洞？）。
    
3.  **Rewrite**: 重新编写代码。
    
4.  **Retry**: 再次执行。
    

这种"试错"能力让 Agent 能够解决它从未见过的复杂问题。

💾 State Persistence (状态持久化)

代码执行通常是**无状态**的（Stateless），特别是在

```
BuiltInCodeExecutor
```

中。

-   **Block A**:
    
    ```
    x = 5
    ```
    
-   **Block B**:
    
    ```
    print(x)
    ```
    
    \-> **NameError!**
    

**解决方案**:

-   **单次执行**: 尽量将逻辑封装在一个完整的代码块中。
    
-   **Session Context**: 高级执行器（如 Agent Engine Sandbox）支持 Session 级别的内存持久化，允许跨代码块共享变量。
    

### **💡 Week 1 总结 & Week 2 展望**

回顾这一周，我们完成了从"零"到"准生产"的旅程：

-   Day 01-02: 基础与配置
    
-   Day 03-04: Gemini 模型与云端部署
    
-   Day 05-06: 可观测性与工具集成
    
-   **Day 07**: 自主代码执行 (Agent 的"大脑"长出了"手")
    

**Next (Week 2): Context & Orchestration** 我们将解决 Agent 的"记忆"问题。如何让 Agent 记住长对话？如何管理庞大的文档上下文？Day 08 将深入 **Effective Context Management**。
<!-- DAILY_CHECKIN_2026-01-03_END -->

# 2026-01-02
<!-- DAILY_CHECKIN_2026-01-02_START -->












**📅 Day 06 打卡：ADK Ready & Context Engineering**

**📝 核心收获** 今天不写代码，而是“磨刀”。从手搓代码转向了 **Agent 工程化** 思维。

**🚀 工具链解锁**

1.  **Antigravity (IDE)**: 不止是编辑器，而是能自主规划任务的 AI 伙伴。
    
2.  **Context Files (**
    
    [**GEMINI.md**](http://GEMINI.md)**)**: 本日MVP。这是写给 AI 看的项目说明书（System Prompt 扩展包）。只要维护好这个文件，新来的 AI 助手立马就能懂我的代码规范和技术栈。
    
3.  **Agent Starter Pack**: Google 官方的 Agent 脚手架，生产级标准，开箱即用。
    

**💡 认知升级 (Aha Moment)**

-   **Neuro-Symbolic**: 用代码逻辑（Symbolic）去约束发散的大模型（Neuro）。
    
-   未来的核心竞争力：**Context Engineering**。谁能给 AI 提供更精准的上下文，谁就能驾驭更强的 Agent。
    

**🔜 Next Step**: Day 07，赋予 AI 执行代码的能力！(LLMs Can Execute Code)

#GoogleAI #Agent开发 #GEMINI #100DaysOfCode
<!-- DAILY_CHECKIN_2026-01-02_END -->

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
