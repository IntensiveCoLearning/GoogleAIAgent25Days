---
timezone: UTC+8
---

# Sherlei

**GitHub ID:** hello2sherlei

**Telegram:** @sherlei030

## Self-introduction

大家好，我是Sherlei，来自成都，目前以独立创业者身份探索 AI 在产品与运营中的落地。我有 9 年中国本土 HR 经验，近半年尝试用主流各类AI工具，搭建自己的工具站和设计产品。我没有技术基础，但在AI的辅助下已经做出了几个小工具和静态网站，对AI的发展和能力很有信心，因此想要更深入的学习。

我擅长中文表达，也能用英文沟通，期待与大家多交流，一起把 Agent 做“能用、好用、可规模”的产品。

## Notes

<!-- Content_START -->
# 2025-12-29
<!-- DAILY_CHECKIN_2025-12-29_START -->
### DAY2 零技术基础 - 学习笔记与思考 ：

1、什么是YAML：

YAML 是一种给人看的（而不是给电脑看的）配置文件格式/小语言，用来用非常简洁、缩进式的文本来描述结构化数据，经常被用作各种应用的“配置文件”。比 JSON、XML 更容易读写，很多现代工具（包括 ADK 的 agents 配置）都会用它来写配置。

2、如何创建搜索智能体 AI agent with Google Search(YAML)

-   在第一天安装好的虚拟环境终端，创建自己的智能体文件夹：
    

```Plain
adk create --type=config my_agent
```

-   选择root agent - Gemini-2.5-flash，model - google AI
    
-   输入自己的API KEY
    

完成后终端会创建一个my agent folder，包含.env和root\_agent.yaml

-   打开创建的root agent，创建tools , Ctrl o保存，回车后ctrl x退出
    

```Plain
nano root_agent.yaml
```

```Plain
name: search_assistant_agent
model: gemini-2.5-flash
description: An agent who can do Google search and provide accurate answers to uese queries.
instruction: You are an assistant agent. Use your search abliities to find relevant information and deliver clear,consice answers to user questions.
tools:
  - name: google_search
```

-   然后adk web运行测试
    

```Plain
adk web
```

-   终端会返回一个网址，复制粘贴打开网页，选择my\_agent，就可以啦！
    

![](https://zcnq1p1e7hrx.feishu.cn/space/api/box/stream/download/asynccode/?code=MmUzNmQ4YWZhNWUzNmFjY2E4MzQyYjNmMjQ5YzM0NzZfRzZhcmVrMm5VdjdFaDNsVzlENzlCYU9haWlzRzRDbEdfVG9rZW46T294UWJaMmQxb1JIcWh4OGRqamN0UW9Sbm5oXzE3NjcwMDQwODY6MTc2NzAwNzY4Nl9WNA)

-   哦豁，貌似tools创建失败了。重来
    
    -   坑点！虽然终端让输了API KEY，这只是创建向导。
        
    -   但运行时是否可用，取决于两点：
        
        -   .env 是否包含正确的变量和值，并且与选择的后端匹配。
            
        -   运行 adk web / adk run 的目录是否能加载到该 .env。
            
        -   adk run的目录一定是在父目录里也就是my\_agent所在的目录文件夹
            
    -   所以，要进入.env把API写进去
        

```Plain
nano .env
```

```Plain
GOOGLE_GENAI_USE_VERTEXAI=FALSE
GOOGLE_API_KEY=xxxxx
```

-   提示：修改了 .env 或 YAML 后，必须重启 `adk web` 才会生效。
    
    -   可以**检查两个文件是否按之前复制粘贴的写正确的**，我上次失误就是发现yaml文件写错了。有可能是nano的格式写错了，可以直接进入文件夹点开文件查看，确保无误后重新运营项目。
        
-   成功啦！tada
    

![](https://zcnq1p1e7hrx.feishu.cn/space/api/box/stream/download/asynccode/?code=NDJmNTk3ODM0MTE1NTRlYWU4OTIxNzlkZjkxZGVkZmRfQ3A3QkdjeUZWelQ1OUVtUGNrYUxuaDM1NE1qa2VZZzhfVG9rZW46UWVUTGJCNEJsb2YzOGR4a0lSYmNOMm9kbkRlXzE3NjcwMDQwODY6MTc2NzAwNzY4Nl9WNA)

3、什么是MCP：

全称 Model Context Protocol，就是“模型上下文协议”，简单说它像 AI 的“通用插头”，让 AI 程序能轻松连接外部工具和数据，不用每次都重新发明轮子。也就是外联的一个工具。

4、什么是firecrawl/爬虫：

爬虫就是网络上的“自动搜集机”，程序自动访问网站，像机器人一样抓取网页上的文字、图片或数据，用来分析或存储。Firecrawl 是个 AI 网页爬虫工具，能自动抓取网站内容，自动识别重点内容，绕过 JavaScript 动态页、反爬虫，输出 AI 能直接读懂的格式，专为 AI 使用设计。

5、如何构建能调用外部工具的多智能体app？Build a Multi-agent app with MCP （YAML）

-   先安装工具包
    

```Plain
pip install -U google-adk
pip install -U firecrawl-py
```

-   创建智能体文件夹
    

```Plain
adk create --type=config my_first_agent
```

-   同样的选择root agent - gemini, model - goole AI，先后输入自己的API KEY
    
-   打开.env文件保存自己的API KEY
    
-   更新root\_agent.yaml文件
    

```JavaScript
name: "web_research_coordinator"
model: "gemini-2.5-flash"
description: "A coordinator agent that manages web research using Firecrawl for scraping and two specialized sub-agents for research and summarization."
instruction: |
  You are a web research coordinator agent. Your job is to:
  1. Coordinate web research tasks using two sub-agents:
     - research_agent: Handles web search and scraping using the Firecrawl MCP tool, and analyzes content for insights and patterns
     - summary_agent: Creates comprehensive summaries and reports
  2. Synthesize findings from both agents into actionable insights

  Important: when delegating to research_agent, provide clear, specific instructions:
  For URLs: "Please scrape and analyze the content from [URL]"
  For research topics: "Please search for and analyze information about [Topic]"
  Do NOT pass complex objects or arrays to the research_agent. Use simple, clear text instructions.

  When given a URL or research topic:
  - Pass a clear, simple instruction to the research_agent (e.g., "Scrape and analyze https://example.com" or "Research AI trends")
  - The research_agent will use appropriate Firecrawl tools with correct parameters
  - The research_agent will analyze the content and return key findings
  - Delegate summarization of the research_agent's analysis to the summary_agent
  - Combine outputs from both agents into a final comprehensive report

sub_agents:
  - config_path: "research_agent.yaml"
  - config_path: "summary_agent.yaml"
```

-   创建子agent
    
    -   **注意格式一定要对，包括大小写、空格！**
        
    -   github注册一个firecrawl，获取API，存入.env
        
    
    ```Plain
    FIRECRAWL_API_KEY=xxxxx
    ```
    
    -   创建research\_agent.yaml
        
    
    ```Plain
    # research_agent.yaml
    version: "1.0"
    
    agent:
      name: "research_agent"
      model:
        provider: "google"
        name: "gemini-2.5-flash"
      description: "Specialized agent for analyzing web content and extracting insights, patterns, and key information."
      instructions: |
        You are a research analysis agent with access to Firecrawl web scraping tools. Your job is to:
        1. Use Firecrawl tools to scrape and search web content
        2. Analyze scraped content for key insights and patterns
        3. Identify important facts, trends, and relationships
        4. Extract relevant quotes and data points
        5. Provide structured analysis of the content
        6. Highlight any inconsistencies or gaps in information
    
        Firecrawl Tool Usage (examples):
        - For URLs: use firecrawl_scrape with parameter: {"url": "https://example.com"}
        - For search queries: use firecrawl_search with parameter: {"query": "search term"}
        - Always use simple object parameters, not arrays or complex structures
    
        Always provide your analysis in a structured format with clear sections for:
        - Key Findings
        - Important Data Points
        - Trends and Patterns
        - Notable Quotes
        - Areas for Further Investigation
    
    tools:
      - type: "mcp"
        name: "firecrawl"
        stdio_server_params:
          command: "npx"
          args:
            - "-y"
            - "firecrawl-mcp"
        env:
          FIRECRAWL_API_KEY: "${FIRECRAWL_API_KEY}"
    ```
    
    -   创建summary\_agent.yaml
        
    
    ```YAML
    # summary_agent.yaml
    version: "1.0"
    
    agent:
      name: "summary_agent"
      model:
        provider: "google"
        name: "gemini-2.5-flash"
      description: "Specialized agent for creating comprehensive summaries and reports from research findings."
      instructions: |
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
    ```
    
-   回到父目录cd /Users/shuranzi ， run adk web
    

（如果有选定的文件夹，进入自己想进入的文件夹可以 cd 文件夹名）

6、坑点总结

-   API要写进.env文件里
    
-   Adk run要回到父目录
    
-   yaml文件格式一定要对，让AI反复检查比对
<!-- DAILY_CHECKIN_2025-12-29_END -->

# 2025-12-28
<!-- DAILY_CHECKIN_2025-12-28_START -->

### 零技术基础 - 学习笔记与思考：

**1、什么是Agent/智能体：**

-   智能体可以理解为：一个会自己思考、自己规划步骤、自己调用工具完成任务的 AI 助手，而不是只回答一句话就结束的普通聊天机器人。
    
-   例如：一个“旅行智能体”可以自动查机票、看酒店、算预算、生成行程，而不是只回你一段说明文字。
    

**2、什么是ADK：**

[Agent Development Kit - Google](https://google.github.io/adk-docs/)推出的一套工具箱，专门用来帮人搭建和运行 AI 智能体应用的框架，简单来说就是一个智能体开发的套件。

**附 - ADK的安装：**

-   首先确保自己电脑安装了python（官网下载[https://www.python.org/downloads/](https://www.python.org/downloads/) ）
    
-   终端输入代码，先创建虚拟环境，给每个项目准备一套独立的“Python 小房间”，不和系统里别的项目混在一起。
    

```Plain
python3 -m venv .venv       # 创建虚拟环境
source .venv/bin/activate   # 激活虚拟环境（Mac / Linux）
```

-   然后安装ADK
    

```Plain
pip install google-adk
```

**3、什么是Agent Engine on Vertex AI：**

Google Cloud 里有一个云端平台，名字叫 [Vertex AI Agent Builder](https://console.cloud.google.com/vertex-ai/agents/agent-garden?pli=1)，可以理解成在云上帮你搭建、托管和管理很多智能体系统的总控制台。

Agent Engine是 Vertex AI Agent Builder 里面的一个子服务名字，用来专门负责在云端运行和管理智能体，是一套托管服务，帮你在生产环境里部署、管理和扩展 AI 智能体，不用自己搭后台服务器。

以后要把自己的智能体真正放上线对外提供服务，就会用到。

**4、什么是Agent Starter Pack：**

[ASP](https://github.com/GoogleCloudPlatform/agent-starter-pack)是Google 准备好的一套现成模板工程 + 自动化脚手架，包含了很多模板，用来把 AI 智能体快速搬到 Google Cloud 上跑，尤其是配合 Agent Engine 用。

三个工具的关系顺序：从你的电脑 (ADK) → 项目模板 (ASP) → 云端运行 (Agent Engine)，也就是从写智能体代码、本地调试测试 → 打包成云项目、一键部署脚本→ 变成在线服务、 自动扩容监控。

**5、《Introduction to Agents Whitepaper》核心讲了什么：**

-   我们使用AI，会希望对效率的提升越来越快，预期AI能实在的帮我们解决问题完成任务，甚至在部分场景中能代替自己进行自主决策。所以智能体的发展是大语言模型的必然趋势。
    
-   我们之所以能信任一个智能体，把任务交给这个智能体去完成，是预期智能体具备优秀的任务理解能力、规划能力、调用工具与资源的能力、整合解决方案的能力，这些综合能力会快速高效且高质量的帮我们完成任务。
    
-   智能体能实现自主系统的核心能力有4个部分，通过持续的周期的思考、行动、观察的循环完成目标。
    
    -   **模型（“大脑”）/ The model (the brain)**：核心语言模型或基础模型，作为智能体的中央推理引擎。我们常用的大语言模型，比如GPT、Gemini都是属于这一类。也就是思维、推理和决策中心。
        
    -   **工具（“双手”）/ Tools(the hands)**：连接智能体推理与外部世界的机制，使其能够检索信息（如RAG - 检索增强生成以获得最新信息）和执行动作（如调用API、发送电子邮件）。也就是工具外联中心。
        
    -   **编排层（“神经系统”）/ The Orchestration Layer (the Nervous System)**：管理智能体的控制过程，负责规划、记忆和推理策略的执行，运用提示词框架和推理技巧拆解步骤并执行。也就是运营中心。
        
    -   **部署（“身体和腿”）/ Deployment(the body and legs):**将智能体托管在安全、可扩展的服务器上，并将其与必要的生产服务（如监控、日志和管理）集成，使其成为可靠和可访问服务的环节。也就是大规模服务的云服务中心。
<!-- DAILY_CHECKIN_2025-12-28_END -->
<!-- Content_END -->
