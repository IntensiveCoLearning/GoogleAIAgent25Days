---
timezone: UTC+8
---

# David Zhou

**GitHub ID:** ShihaoZhou-NEU

**Telegram:** @Ox221Eight

## Self-introduction

Full-Stack Developer

## Notes

<!-- Content_START -->
# 2026-01-09
<!-- DAILY_CHECKIN_2026-01-09_START -->
# Day 13

**AI 开发的格局正从无状态的“请求-响应”循环，转向有状态的多轮 Agent 工作流。** 随着 **Interactions API** 开启测试，Google 提供了一个专为此新时代设计的统一接口——它是连接原始模型与全托管 **Gemini Deep Research Agent** 的单一网关。

### **今日即可使用的两种方式：**

1.  “引擎”升级 (ADK)：
    
    通过在你的 ADK Agent 中设置 use\_interactions\_api=True，你可以将对话历史和状态管理移至服务端。这支持后台执行——非常适合处理长耗时任务，避免客户端在模型“思考”时因超时而断开。
    
2.  “桥接”模式 (A2A)：
    
    如果你拥有一组运行 Agent2Agent (A2A) 协议的 Agent 集群，现在可以将 Google 托管的 Agent 视为远程对等节点。全新的 InteractionsApiTransport 可将 A2A 消息直接映射到 Interactions API，让你现有的客户端无需编写自定义 API 封装，即可向 Deep Research Agent 发送任务。
    

### **核心变革：从生成文本到自主工作流**

Interactions API 标志着从无状态文本生成到有状态、自主工作流的基础性转变：

-   **状态外包**：开发者无需再手动处理复杂的上下文窗口和脆弱的客户端循环，而是将整个推理状态和历史管理卸载到 Google 的基础架构中。
    
-   **离线执行**：为长耗时任务开启“即发即弃”的后台执行模式，解决了复杂 Agent 链中常见的超时问题。
    
-   **统一原语**：提供了一个统一的原语，用于访问原始模型以及像 Gemini Deep Research 这样的全托管 Agent。
<!-- DAILY_CHECKIN_2026-01-09_END -->

# 2026-01-08
<!-- DAILY_CHECKIN_2026-01-08_START -->

# Day11

**魔法不在于提示词（Prompt），而在于事件循环（Event Loop）。**

大多数 AI Agent 依赖 HTTP 协议（请求 → 等待 → 响应），这会产生延迟，且让“打断” AI 变得不可能。我们通过 **双向流（Bi-Directional Streaming）** 打破了这一循环，也就是大家熟知的 **ADK Bidi-streaming**。通过与 Gemini 建立持久的 WebSocket 连接，我们创建了一个客户端输入（音频/视频/文本）与服务端输出（音频/文本/工具调用）同步流动的会话。我们可以利用 ADK 来控制 Gemini Live 并为其配备强大的工具。

### **工作原理：**

-   **应用初始化**：启动时创建 Agent、SessionService 和 Runner。
    
-   **会话初始化**：为每个连接建立 Session、RunConfig 和 LiveRequestQueue。
    
-   **双向流传输**：并发执行上行（客户端 → 队列）和下行（事件 → 客户端）任务。
    
-   **优雅终止**：确保 LiveRequestQueue 和 WebSocket 连接的妥善关闭。
    

### **核心功能：**

-   **WebSocket 通信**：通过 `/ws/{user_id}/{session_id}` 实现实时双向流。
    
-   **多模态请求**：支持文本、音频及图像/视频输入，并具备自动语音转录功能。
    
-   **灵活响应**：根据模型架构自动确定输出文本或音频。
    
-   **会话恢复**：通过 RunConfig 配置支持断线重连。
    
-   **并发任务**：独立的异步上/下行任务，实现最优性能。
    
-   **交互式 UI**：带有事件控制台的 Web 界面，用于监控 Live API 事件。
    
-   **Google 搜索集成**：为 Agent 配备 `Google Search` 工具。
<!-- DAILY_CHECKIN_2026-01-08_END -->

# 2026-01-07
<!-- DAILY_CHECKIN_2026-01-07_START -->


# Day11

## MCP

### **统一且可控的接口**

无需再解析脆弱的命令行输出，你的 Agent 现在拥有了针对复杂系统的结构化、可发现接口：

-   **Google Maps**：提供“轻量级落地 (Grounding Lite)”，为 Agent 提供新鲜的地理空间数据和路线详情，防止对物理位置产生幻觉。
    
-   **BigQuery**：允许 Agent 直接解析 Schema 并执行查询，无需将海量数据集移入上下文窗口，从而降低延迟和安全风险。
    
-   **GKE & Compute Engine**：将基础架构管理（预配、调整大小、诊断）转变为可发现的工具，实现真正的“Day-2”自主运维。
    

### **原生安全，而非事后补丁**

由于这些是托管端点，你不会失去控制权。管理员可以通过 **Google Cloud IAM** 管理访问权限，并使用 **Google Cloud Model Armor** 防御间接提示词注入。
<!-- DAILY_CHECKIN_2026-01-07_END -->

# 2026-01-06
<!-- DAILY_CHECKIN_2026-01-06_START -->



# Day10

**长周期的 Agent 会话面临两大敌人：延迟和“迷失在中间 (Lost in the middle)”综合征。** 随着对话历史的增长，重复发送庞大的系统指令变得昂贵，且模型难以在近期的噪音中优先处理早期的规则。

ADK 通过双管齐下的方式解决了这个问题：

1.  **上下文缓存 (Context Caching)**：允许你缓存 Prompt 中不可变的部分（如系统指令、Few-shot 示例），这样你就无需为每一轮对话重复支付计算成本。
    
2.  **上下文压缩 (Context Compaction)**：防止历史记录膨胀。
    

**ADK 不会无休止地追加原始消息。** 它采用滑动窗口机制，将较早的事件总结为简练的“记忆”块，同时保留最近的交互原件以确保精准度。

## Code

-   context\_[config.py](http://config.py)
    

```python
from google.adk.apps import App, EventsCompactionConfig
from google.adk.agents.context_cache_config import ContextCacheConfig

# Configure your App with both Caching and Compaction
app = App(
    name='long-memory-agent',
    root_agent=my_agent,
    
    # 1. Cache heavy instructions
    context_cache_config=ContextCacheConfig(
        min_tokens=2048,    # Only cache if prompt is heavy
        ttl_seconds=1800,   # Keep cache alive for 30 mins
        cache_intervals=10  # Refresh after 10 uses
    ),

    # 2. Compress history to prevent "Context Rot"
    events_compaction_config=EventsCompactionConfig(
        compaction_interval=3, # Summarize every 3 turns
        overlap_size=1         # Keep 1 turn of context overlap
    )
)
```
<!-- DAILY_CHECKIN_2026-01-06_END -->

# 2026-01-05
<!-- DAILY_CHECKIN_2026-01-05_START -->




# Day09

ADK 支持**时光倒流 (Time Travel) 与检查点 (Checkpointing)** 功能了！

不同于直接删除历史记录的破坏性做法，运行器（Runner）会计算“现在”与“当时”之间的差异（即状态与工件的增量数据 State & Artifact Deltas），并在日志中追加一个“倒回事件”。这让你可以将应用状态恢复到特定的时间戳或调用 ID (Invocation ID)，同时保留完整的审计轨迹以供查证。

### **化繁为简的修复方案**

你不仅是“回到过去”——而是将整个世界还原到当时完全一致的状态，并保留被倒回的那条路径以备不时之需。

## Code

-   rewind\_[example.py](http://example.py)
    

```
 import asyncio
 from adk.api.agents.in_memory_runner import InMemoryRunner
 ​
 # 1. Initialize the runner
 runner = InMemoryRunner(...)
 ​
 # 2. Something goes wrong? (e.g., hallucination or error at 'invocation_456')
 # Instead of clearing the session, we request a rewind.
 ​
 # 3. Rewind (Time Travel)
 # This asynchronously restores session state and artifacts to the target moment.
 asyncio.run(
     runner.rewind_async(
         session_id='session_123',
         before_invocation_id='invocation_456'
     )
 )
 ​
 # 4. Resume the conversation from that exact point
 asyncio.run(runner.run(query="Let's try that request again with these constraints..."))
```
<!-- DAILY_CHECKIN_2026-01-05_END -->

# 2026-01-04
<!-- DAILY_CHECKIN_2026-01-04_START -->





# Day08

**别再往你 Agent 的 LLM 里塞乱七八糟的上下文了。**

“追加一切”的策略是导致延迟激增和“迷失在中间 (Lost in the middle)”幻觉的单程票。Google ADK 改变了这一模式，将上下文视为**编译视图 (Compiled view)**，而非一串巨大的字符串。ADK 不再将原始历史记录生搬硬套地塞进窗口，而是通过一系列处理器从结构化、持久化的会话状态中动态过滤、压缩并格式化出简洁的“工作上下文”。作为开发者，你可以控制这个处理流水线，按需定制行为。

**真正的工程化需要粒度化的控制，而不仅仅是更大的上下文窗口。**

ADK 的分层架构将“存储”与“展示”分离，允许你通过“句柄模式 (Handle pattern)”将大文件外部化为 **Artifacts（工件）**，并仅在绝对必要时通过可搜索的 **Memory（记忆）** 检索长期数据。无论你是在管理严谨的多 Agent 移交，还是在调试工具交互，**ToolContext** 等专用对象都能确保你的 Agent 仅访问其所需的作用域——不多不少，恰到好处。

**利用优化模型能力的模式，为生产规模而构建。**

ADK 通过强制分离“静态指令”（不变的策略和 Schema）与“轮次指令”（动态的、由控制器拥有的引导），让 **Context Caching（上下文缓存）** 得以工程化运行。这种设计保持了沉重的系统头部信息稳定，从而大幅削减成本和延迟，同时确保每一轮的指令在逻辑上与用户输入分离，从而获得更好的安全性和验证效果。

As described in [**Day 3 of our Kaggle 5 Day intensive course**](https://www.kaggle.com/whitepaper-context-engineering-sessions-and-memory), Agents are largely "context management".

## Code

-   app\_[setup.py](http://setup.py)
    

```
 # An Example of Static Context Policy 
 from google.adk.apps import App
 from google.adk.agents import Agent
 from google.adk.agents.context_cache_config import ContextCacheConfig
 ​
 STATIC_POLICY_HEADER = """You are a strict policy assistant for internal compliance Q&A.
 ​
 Follow this exact JSON schema in every response:
 {"answer": str, "citations": [str], "confidence": float}
 ​
 Safety:
 - Never provide medical or legal advice; refuse with a brief explanation.
 - Never invent policy numbers or sections; ask for the missing reference.
 ​
 Style:
 - Use short sentences.
 - Prefer active voice.
 - If uncertain, say so and request the missing input.
 ​
 Tools:
 - search: use for public web facts.
 - bq: use for internal policy tables (read-only).
 """
 ​
 agent = Agent(
     name="policy_agent",
     static_instruction=STATIC_POLICY_HEADER,
     instruction="Default: be concise and include at most two citations."
 )
 ​
 app = App(
     name="policy_qa_app",
     context_cache_config=ContextCacheConfig(
         ttl_seconds=3600,     # cache the header for 1 hour
         cache_intervals=5,    # force a refresh every 5 requests (guardrail)
         min_tokens=1000       # only cache if header is “worth it”
     ),
     root_agent=agent
 )
```

-   [**steering.py**](http://steering.py)
    

```
 # An Example of a runtime controller generating each turn instructions
 from dataclasses import dataclass
 from typing import Optional, Tuple
 ​
 @dataclass
 class SteeringInputs:
     goal: str                                  # this turn’s objective
     style: str = "concise"                     # terse, detailed, crisp, etc.
     max_cites: int = 2                         # runtime knob
     tenant_hint: Optional[str] = None          # "Answer for EU employees only"
     corrective: Optional[str] = None           # "Last reply missed field X; include it"
     confidence_range: Tuple[float, float] = (0.6, 0.9)
 ​
 def build_turn_instruction(s: SteeringInputs) -> str:
     parts = [
         f"Goal: {s.goal}",
         f"Style: {s.style}",
         (
             "Constraints: "
             f"include at most {s.max_cites} citations; "
             "refuse medical/legal advice; "
             "if info is missing, ask one targeted question; "
             f"return 'confidence' between {s.confidence_range[0]} and {s.confidence_range[1]}."
         )
     ]
     if s.tenant_hint:
         parts.append(f"Tenant: {s.tenant_hint}")
     if s.corrective:
         parts.append(f"Correction: {s.corrective}")
     return "
 ".join(parts)
```

-   chat\_[handler.py](http://handler.py)
    

```
 # An Example of a chat handler which composes the turn instruction
 from steering import SteeringInputs, build_turn_instruction
 from google.adk.agents import Agent
 ​
 # agent imported from app_startup.py
 ​
 def route_intent(user_message: str) -> str:
     text = user_message.lower()
     if "compare" in text: return "compare"
     if "list" in text and "control" in text: return "list_controls"
     if "summarize" in text: return "summarize"
     return "answer"
 ​
 INTENT_TO_GOAL = {
     "summarize": "Summarize ACME-42 in plain English.",
     "list_controls": "List mandatory controls from ACME-42 with one-line rationales.",
     "compare": "Compare ACME-42 to ISO 27001 at a high level, return a short markdown table inside the JSON 'answer'.",
     "answer": "Answer the user directly."
 }
 ​
 def chat(session_id: str, user_message: str, ui_style: str | None = None):
     intent = route_intent(user_message)
     goal = INTENT_TO_GOAL.get(intent, f"Answer the user: {user_message[:120]}")
 ​
     style = ui_style or get_flag(session_id, "style", default="concise")
     max_cites = get_flag(session_id, "max_citations", default=2)
     tenant_hint = get_tenant_hint(session_id)     # e.g., "EU employees only" or None
     corrective = get_last_validation_error(session_id)  # None or short string
 ​
     turn_instruction = build_turn_instruction(
         SteeringInputs(
             goal=goal,
             style=style,
             max_cites=max_cites,
             tenant_hint=tenant_hint,
             corrective=(f"Your last reply failed validation: {corrective}. Fix it this turn." if corrective else None)
         )
     )
 ​
     agent.instruction = turn_instruction
     response = agent.run(user_message=user_message)
     validate_and_record(session_id, response)     # optional schema check + feedback
     return response
```
<!-- DAILY_CHECKIN_2026-01-04_END -->

# 2026-01-03
<!-- DAILY_CHECKIN_2026-01-03_START -->






# Day07

### **通过 ADK 的代码执行器 (Code Executor) 你将获得：**

-   **BuiltInCodeExecutor**：开箱即用。
    
-   **灵活运行**：可在你的本地机器或托管云沙箱中运行。
    
-   **云沙箱兼容**：适配 Agent Engine、Google Kubernetes Engine (GKE) 和 Daytona。
    
-   **全生命周期开发**：Agent 可以编写、测试、调试并迭代代码。
    
-   **最终产出**：既可以是代码执行的结果，也可以是代码本身。
    
-   **零复杂度**：无需复杂设置，只需启用该工具即可。
<!-- DAILY_CHECKIN_2026-01-03_END -->

# 2026-01-02
<!-- DAILY_CHECKIN_2026-01-02_START -->







# Day06

在随附的视频中，你可以看到 **Antigravity** 在零配置的情况下就能理解 ADK 的工作原理。这要归功于入门套件中默认开启的 **ADK 备忘单 (Cheatsheet)**，它适用于大多数 IDE。

如果你目前没有使用 Agent Starter Pack，这里有一些关于如何配置常用 IDE 的建议。

**超短启动指令：**

```
 uvx agent-starter-pack create deep_search --adk
```

```
 # Using ADK and Agent Starter Pack within Antigravity
 uvx agent-starter-pack create deep_search --adk
 # nothing else
```

```
 # Install dependencies
 pip install pydantic-ai
 gemini extensions install https://github.com/derailed-dash/adk-docs-ext
 gemini
```
<!-- DAILY_CHECKIN_2026-01-02_END -->

# 2026-01-01
<!-- DAILY_CHECKIN_2026-01-01_START -->








# Day05

**自动配置的两个层级：**

1.  **Agent 遥测 (Telemetry)**
    
    -   **Cloud Trace** 捕获每一次执行。
        
    -   **LLM 调用**及其延迟细分。
        
    -   **工具执行**耗时。
        
    -   完整的**对话流**可视化。
        
2.  **提示词-响应日志 (Prompt-Response Logging)（自动开启）**
    
    -   通过 **Terraform** 预配完整的端到端 (E2E) 流程。
        
    -   **日志分析 (Log Analytics)** + 具有自定义保留期的**日志桶 (Log Buckets)**。
        
    -   **BigQuery Delta Lake** 配合自定义视图，方便进行数据查询。
        
    -   **数据安全**：日志中不包含敏感数据——所有内容均写入 **GCS (Google Cloud Storage)**。
        

**只需一条指令即可部署：**

```
 uvx agent-starter-pack create my-agent -a adk_base -d agent_engine
 make deploy
 ​
```
<!-- DAILY_CHECKIN_2026-01-01_END -->

# 2025-12-31
<!-- DAILY_CHECKIN_2025-12-31_START -->









# Day04

**gent Engine 现已支持基于源码的部署（Source-based deployment）。** 你的源代码可以直接部署到生产环境，彻底消除了常见的部署阻碍。

**告别以下烦恼：**

-   **Pickle 序列化错误**
    
-   **复杂的序列化难题**
    
-   **让人头疼的模块路径问题**
    

现在，本地运行的效果与生产环境完全一致。开发与部署之间的鸿沟已不复存在。

```
 uvx agent-starter-pack create my-agent -a adk_base -d agent_engine
 ​
 ​
```

随后只需运行

```
 make deploy
```

即可完成部署。

-   Create new project with Agent Starter Pack
    

```
 uvx agent-starter-pack create my-agent -a adk_base -d agent_engine
 cd my-agent && make deploy
```

-   Enhance existing ADK agent
    

```
 uvx agent-starter-pack enhance --d agent_engine
```

### uvx遇到安装问题就不要用uvx，直接

```
 # 1. Create and activate a virtual environment
 python -m venv .venv
 source .venv/bin/activate
 ​
 # 2. Install the package
 pip install agent-starter-pack
 ​
 # 3. Run the create command
 agent-starter-pack create
 ​
 # You can also pass flags to skip the prompts
 agent-starter-pack create my-adk-agent -a adk_base -d agent_engine
 ​
 cd <your-project> && make install && make playground
```
<!-- DAILY_CHECKIN_2025-12-31_END -->

# 2025-12-30
<!-- DAILY_CHECKIN_2025-12-30_START -->










# Day03

-   **Google 搜索落地 (Grounding)**：原生接入实时网络数据。
    
-   **电脑操作控制 (Computer use)**：让 Agent 能够导航并与用户界面 (UI) 进行交互。
    
-   **实时 API (Live API)**：为语音和视频 Agent 提供实时流式传输支持。
    
-   **原生可观测性**：全面监控 Gemini 调用、工具使用以及 Agent 的推理过程。
    

uvx agent-starter-pack create -y --api-key YOUR\_GEMINI\_API\_KEY

## Code

-   One liner with Agent Starter Pack
    

```
 uvx agent-starter-pack create -y --api-key YOUR_GEMINI_API_KEY
```

-   Using ADK CLI
    

```
 uv init
 uv add google-adk
 uv add google-genai
 export GOOGLE_API_KEY="YOUR_API_KEY"
 set GOOGLE_API_KEY="YOUR_API_KEY" #CMD
 source .venv/bin/activate
 .venv\Scripts\activate #CMD
 adk create my_agent
```

-   Download sample and run locally
    

```
 curl '<https://raw.githubusercontent.com/GoogleCloudPlatform/devrel-demos/refs/heads/main/ai-ml/agent-labs/gemini-3-pro-agent-demo/my_agent/agent.py>' > my_agent/agent.py
 adk web
```

## A Multi-agent app with MCP

```
 pip install google-adk
 pip install firecrawl-py
```

```
 # yaml-language-server: $schema=https://raw.githubusercontent.com/google/adk-python/refs/heads/main/src/google/adk/agents/config_schemas/AgentConfig.json
 name: web_research_coordinator
 model: gemini-2.5-flash
 description: A coordinator agent that manages web research using Firecrawl for scraping and two specialized sub-agents for research and summarization.
 instruction: |
   You are a web research coordinator agent. Your job is to:
   1. Coordinate web research tasks using two sub-agents:
       - Research Agent: Uses Firecrawl to scrape web content based on given queries.
       - Summarization Agent: Summarizes the scraped content into concise reports.
   2. Synthesize findings from both sub-agents into actionable insights.
   Important: when delegating to research_agent, provide clear and specific instrucations:
   For URLs: "Please scrape and analyze the content from [URL]"
   For research topics: "Please search for and analyze information about [TOPIC]"
   DO NOT pass complex objects or arrays to the research_agent. Use simple, clear text instrucations.
   When given a URL or research topic:
   - Pass a clear, simple instruction to the research_agent.(e.g. "Scrape and analyze https://example.com" or "Research the latest trends in AI.")
   - The research_agent will use appropriate Firecrawl tools with correct parameters. 
   - The research_agent will analyze the content and return key findings.
   - Delegate summarization of the research_agent's analysis to the summary_agent.
   - Combine outputs from both agents into a final comprehensive report.
 sub_agents:
   - config_path: research_agent.yaml
   - config_path: summary_agent.yaml
 ​
```

```
 name: research_agent
 model: gemini-2.5-flash
 description: "Specialized agent for analyzing web content and extracting insights, patterns, and key information."
 instruction: |
   You are a research analysis agent with access to Firecrawl web scraping tools. Your job is to:1. Use Firecrawl tools to scrape and search web content
   2. Analyze scraped content for key insights and patterns
   3. Identify important facts, trends, and relationships4. Extract relevant quotes and data points
   5. Provide structured analysis of the content
   6. Highlight any inconsistencies or gaps in information
 ​
   Firecrawl Tool Usage:
   - For URLs:Use 'firecrawl_scrape' with parameter:{"url":"https://example.com"}
   - For search queries:Use 'firecrawl_search' with parameter:{"query":"search term"}
   - Always use simple object parameters, not arrays or complex structures
 ​
   Always provide your analysis in a structured format with clear sections for:
   - Key Findings
   - Important Data Points
   - Trends and Patterns
   - Notable Quotes
   - Areas for Further Investigation
 tools:
   - name: MCPToolset
     args:
       stdio_server_params:
         command: "npx"
         args:
           - "-y"
           - "firecrawl-mcp"
         env:
           FIRECRAWL_API_KEY: "${FIRECRAWL_API_KEY}"
 ​
```

```
 name: summary_agent
 model: gemini-2.5-flash
 description: "Specialized agent for creating comprehensive summaries and reports from research findings."
 instruction: |
   You are a summarization agent. Your job is to:
   1. Create clear, concise summaries of research findings
   2. Organize information into logical sections
   3. Generate executive summaries for quick understanding
   4. Create detailed reports with proper formatting
   5. Ensure all important information is captured and presented clearly
   Always structure your output with:
   - Executive Summary (2-3 sentences)
   - Detailed Summary (organized by topic)
   - Key Takeaways (bullet points)
   - Recommendations (if applicable)
   When creating summaries, ensure:
   - Information is accurate and well-organized
   - Key points are highlighted and easy to find
   - Complex information is simplified without losing meaning
   - Recommendations are actionable and specific
   - The summary is comprehensive yet concise
 ​
```
<!-- DAILY_CHECKIN_2025-12-30_END -->

# 2025-12-29
<!-- DAILY_CHECKIN_2025-12-29_START -->











# Day01

准备阶段，环境安装

-   **Gemini 3**：上下文工程 (Context Engineering)、电脑操作控制 (Computer Use)、实时 API (Live API) 以及进阶模式。
    
-   **Google ADK**：Agent 开发套件 (Python 版)。
    
-   **Vertex AI Agent Engine**：数分钟内即可完成 Agent 部署。
    
-   **Agent 入门套件**：端到端 (E2E) 的生产级就绪范本。
    

## Installation[**¶**](https://google.github.io/adk-docs/get-started/python/#installation)

Install ADK by running the following command:

```
 pip install google-adk
```

Create a Python virtual environment:

```
 python -m venv .venv
```

## Create an agent project[**¶**](https://google.github.io/adk-docs/get-started/python/#create-an-agent-project)

Run the `adk create` command to start a new agent project.

```
 adk create my_agent
```

Gemini API: [**https://aistudio.google.com/api-keys**](https://aistudio.google.com/api-keys)

# Day02

使用YAML构造AI Agent

只需 4 行文本 = 1 个可运行的 AI Agent。

无需 Python，无需编程，只需 YAML。

我所说的并不是那些功能受限的无代码工具，而是**代码优先 (Code-first)** 的 Agent——即开发者为真实生产环境所构建的架构。

**通过 4 行 YAML，你将获得：**

-   一个由 **Gemini 3** 驱动的可运行 AI Agent
    
-   **Google 搜索**功能集成
    
-   随时可部署的状态
    
-   无需任何编程基础
    

**复制、粘贴、使用内置 Web UI 运行。就这么简单。**

## YAML

YAML: YAML Ain’t Markup Language™，是一种**人类友好的数据序列化格式**

### 核心特点

NaN. **可读性极高**

YAML 使用缩进和简洁的语法，比 JSON 和 XML 更易读：

```
 # YAML
 person:
   name: 张三
   age: 25
   hobbies:
     - 阅读
     - 编程
     - 游戏
     
 // 等效的 JSON
 {
   "person": {
     "name": "张三",
     "age": 25,
     "hobbies": ["阅读", "编程", "游戏"]
   }
 }
```

2.  **使用缩进代替括号**
    

-   使用**空格**（通常是 2 个）表示层级
    
-   **不能用 Tab 键**，必须用空格
    
-   缩进必须一致
    

### 使用场景

1.  配置文件
    
    -   Docker Compose (docker-compose.yaml)
        
    -   Kubernetes (deployment.yaml, service.yaml)
        
    -   CI/CD (GitHub Actions, GitLab CI)
        
    -   应用配置 (Spring Boot, Django)
        

### **进阶可能：**

一旦你拥有了 YAML Agent，你可以进一步添加：

-   **内置工具**（如 `Google Search`、`code_execution` 代码执行）
    
-   **自定义 Python 工具**（当你准备好进阶时）
    
-   **子 Agent**（用于处理复杂工作流）
    
-   **多 Agent 编排**
    

**从简单开始，按需扩展。**

### AI Agent with Google Search

```
 adk create --type=config david_agent #（包含yaml用于配置agents）
 adk create david_agent
 ​
```

Add Google Search tool to the agent file by simple adding the tools section.

```
 name: search_assistant_agent
 model: gemini-2.5-flash
 description: An agent who can do Google search and provide accurate answers to user queries.
 instruction: You are an assistant agent. Use your search capabilities to find accurate and relevant information to answer user queries.
 tools:
   - name: google_search
```
<!-- DAILY_CHECKIN_2025-12-29_END -->

# 2025-12-28
<!-- DAILY_CHECKIN_2025-12-28_START -->













Day 1
<!-- DAILY_CHECKIN_2025-12-28_END -->
<!-- Content_END -->
