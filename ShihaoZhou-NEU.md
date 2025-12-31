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
