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
# 2025-12-31
<!-- DAILY_CHECKIN_2025-12-31_START -->
### DAY4 零技术基础 - 学习笔记与思考 ：

这一课自己看没搞懂在干嘛，开着网页用comet帮忙解释。

Day 1–3 教你在本机把 Agent 跑起来，而 Day 4 开始，用 Agent Engine 做“源码级部署”，让你本地那套代码一行命令就变成真正的云上生产服务。

所以这一课的核心：

-   引入 Agent Engine 的源码部署：不再是你自己 docker（打包和运行） / 手写 CI/CD，而是直接把本地源码“托管”给 Agent Engine，由它在云端跑同一份代码。
    
-   目标是“本地怎么跑，线上就怎么跑”
    

相当于之前两天做的agent，今天就是把现在这份工程，用一行命令丢给 Agent Engine，在云上得到同款运行环境和一个正式的线上 Agent 服务。

1、什么是source-based deployment？

source-based 部署不是在生产环境里直接改代码并发布；而是把仓库里的代码拉到上线环境，当场执行“安装依赖→编译/打包→启动”的流程。简单来说，就是在你自己的电脑上把功能写好后，把这份“代码”带到那台对外营业的正式机器上（云上的服务器也算）。到现场再把它“加工好并启动”，让所有人都能用。

2、CI、CD、GCP是什么？

CI = Continuous Integration（持续集成），你提交代码后，自动帮你测试、打包。

CD = Continuous Delivery / Continuous Deployment（持续交付 / 持续部署），通过测试后，自动部署到服务器。

CI/CD都是自动化流程。

GCP = Google Cloud Platform（谷歌云平台），就是云服务器放在哪个机房（国家/城市）。比如 us-central1 是美国中部。
<!-- DAILY_CHECKIN_2025-12-31_END -->

# 2025-12-30
<!-- DAILY_CHECKIN_2025-12-30_START -->

### DAY3 零技术基础 - 学习笔记与思考 ：

自己跟着文档有看不懂的地方，直接打开vs code，发给gpt，让他搞定，并解释清楚原理。

1、什么是UV？怎么安装？

uv 是一个超快的 Python 包管理与环境工具。它把“创建虚拟环境 + 安装依赖 + 项目初始化”合在一起，用法类似 pip + venv + pip-tools 的组合，但速度更快、体验更简单。简单理解就是更快的包安装器 + 自动虚拟环境管理器。

```Plain
curl -LsSf https://astral.sh/uv/install.sh | sh
```

2、Gemini Starter Kit:

用官方给的脚手架一键生成

```Plain
uvx agent-starter-pack create -y --api-key 
```

```Plain
cd my-agent && make install && make playground
```

![](https://zcnq1p1e7hrx.feishu.cn/space/api/box/stream/download/asynccode/?code=MDg1NWIzZjAxMDQwZGExYzA0MjU5MmIyMGUyMDk5NjBfSEpOWVJxczVHblJ1OVBsN3V6N291d2FiWXpKMzczTFdfVG9rZW46TThtOGJQazdrb3VrYXF4Z3hBUGNHQktIblBkXzE3NjcxMDgwMjI6MTc2NzExMTYyMl9WNA)

3、我怎么判断我的agent需要哪些第三方工具？

我自己的想法很简单，直接问AI，让TA给我答案。一般来讲基础的肯定会有Google\_search。附官方支持的[Google工具和第三方工具](https://google.github.io/adk-docs/tools/#google-search)。

4、这些小练习都可以直接用大模型获得答案，为什么要自己搭agent？意义在哪？

从练习的几个agent来看，搭的工具都比较简单，只是提问的话，确实感觉也就那样，回答的界面还不好看。所以产生了这个疑问。问了AI之后，得到答案如下：

单聊大模型更像“临时找人问路”，而搭一个 agent 更像“招了一个长期帮你干活的员工”，两者定位其实不一样。

## 先说区别到底在哪

-   大模型（直接问 Gemini / GPT）
    
    -   每次你问，它回一句就结束，默认不会自己连着干后续步骤。
        
    -   它本身只会“说话”和“算”，不会主动去调 API、查数据库、点网页、写完代码再自动跑测试这些动作。
        
-   Agent（你现在在看的这种 demo）
    
    -   把大模型当“脑子”，外面再套一层：目标、记忆、工具、环境，比如 HTTP API、数据库、浏览器、代码执行环境等。
        
    -   给一个稍微宏观的目标，它可以自己拆解步骤、反思结果、循环执行，直到把活干完，而不是你一句一句手把手喂指令。
        

## 那什么时候不是“自嗨”

下面这种场景，直接问 Gemini / GPT 会很累，Agent 就有价值了：

-   需要多步连续动作的任务
    
    -   比如“帮我把这 50 篇文章爬一下、抽关键信息存进我的 Notion，然后生成一份汇总报告发到邮箱”。
        
    -   这里涉及：爬网页 → 解析 → 抽取 → 写数据库/Notion API → 生成报告 → 发邮件，全是工具链，Agent 可以自动串起来。
        
-   需要长期运行或定时跑的东西
    
    -   例如每天早上自动：看一遍几家网站/内部系统 → 拉业务数据 → 分析 → 生成日报。
        
    -   单聊大模型做不到“自己每天 9 点起来干活”，但 Agent 可以挂在后端像服务一样跑。
        
-   需要强绑定你自己数据 / 业务规则
    
    -   Agent 可以直接访问你公司的数据库、CRM、知识库、项目管理系统，按照你的业务规则做动作（比如自动建工单、更新状态）。
        
    -   这已经超出了“问问题 → 文本回答”的范畴，更像一个半自动员工。
        
-   需要多人协作 / 多角色分工
    
    -   比如一个 multi-agent：一个负责编程，一个负责测试，一个负责报告，一个负责和外部 API 打交道，由一个“协调者 Agent”调度。
        
    -   单轮对话模型可以模拟这种对话，但工程上维护起来很麻烦，专门的 agent 框架就方便很多。
        

## 对“个人用户”来说有什么现实价值

如果只是偶尔问问题、写点文案、改改代码片段，确实直接开 Gemini / GPT 聊天就够用，不必为了“追热点”硬上 Agent 框架，那确实有点自嗨。

但只要你出现下面任意一个想法，就已经接近 Agent 的典型落地场景了：

-   “我有一堆重复、机械但带点判断力的工作，能不能让 AI 帮我自动化？”
    
-   “我不想每天 copy paste 东西给 AI，让它帮我干，我想让它自己连 API、自己跑。”
    
-   “我有一套自己的工具 / 内部系统，想让 AI 像人一样在这里面来回操作。”
    

一旦目标从“聊天问答案”升级成“让 AI 实打实帮我干活”，你在搭的就不再是一个聊天机器人，而是一个真正的 agent。

## 那你现在做这个 demo 意义在哪

结合你开的这个 Gemini 3 Pro agent demo 仓库，它的意义更偏这几类：

-   学会怎么用框架把模型、工具、状态管理组装在一起（为以后做真正的自动化打基础）。
    
-   顺手把自己一些个人工作/副业里的重复流程，逐步改造成半自动/全自动流水线。
    
-   提前踩坑：观察“让模型自己计划、调用工具”到底哪里容易翻车，怎么加 guardrail、回滚、日志等。
    

如果你目前没有任何想自动化的真实工作流，那短期确实更像“自嗨练手”；但一旦你哪天想把一块“每天都要做、又不想做”的事情交给 AI，这些 agent 的尝试就会立刻变成“早期技术储备”，而不是无意义的玩具。

5、带有agent功能的vibe coding工具（codex, claude code skills）有什么区别？

你说的这些 vibe coding 工具（Claude Code、Codex、Cursor 之类）本质上是“超强 IDE+脚本执行器”，你用自然语言描述功能，它帮你生成/改写大段代码，通常直接在 IDE 里改文件、跑测试、跑命令。而你现在看的这种 agent，更偏“长期挂着、可以接业务系统和多种工具的自动化服务”。

## Agent 相比 vibe coding，多了哪些维度

和 vibe coding 相比，agent 更多是面向“在真实环境里执行任务”，而不仅是“在代码仓库里写代码”。

-   环境范围不同
    
    -   vibe coding：主要操作的是代码库 + 终端/测试环境。
        
    -   agent：可以被部署成长期在线服务，调 HTTP API、查数据库、操作第三方 SaaS、发邮件、跑定时任务等。
        
-   触发方式不同
    
    -   vibe coding：一般是“你在 IDE 里打字 → 它响应”，强交互、强同步，你不在场它就停了。
        
    -   agent：可以由外部事件驱动（webhook、消息队列、cron），在你不在线的时候自己运行，比如每天定时跑报表、监听新订单自动处理。
        
-   目标形态不同
    
    -   vibe coding：目标通常是“写完一段代码 / 修好一个 issue / 生成一个 PR”。
        
    -   agent：目标可以是“持续监控 + 自动决策 + 自动执行”，例如：监控库存 → 自动下单补货；监控日志 → 自动拉起工单和告警。
        

## 也有重叠：vibe coding 里的“agent 化趋势”

你提到的“我输入一个命令，它自己去跑”，其实是 vibe coding 正在往 agent 化进化的表现，比如：Claude Code 支持更自主地拆任务、跑子 agent、自动跑测试、做 checkpoint。

很多人现在做法是：

-   在一个“开发容器”里让 Claude Code 全自动从 GitHub issue 干到 PR，你只在最后 review 一下，这已经很接近“专职开发 Agent”。
    
-   Cursor、Copilot 等也开始引入“agent 模式”，可以自己创建文件、搭项目骨架、连数据库、跑 server，再回报告。
    

所以可以粗暴理解为：

## 什么时候还要自己搭 agent，而不是只用 Claude Code

如果你的需求只是在“写代码 / 改代码 / 出 PR”层面，那直接用 Claude Code 这类 vibe coding 工具已经非常顶，用不着折腾 agent 框架，这点完全可以“摆烂”。

但一旦出现这些想法，就会明显感觉 agent 更合适：

-   **需要跨代码和非代码世界**：写完代码后还要自动部署、调用业务 API、通知 Slack/飞书、更新 Notion、写回数据库。
    
-   **需要多人/多系统协同**：比如接收客户请求 → 查 CRM → 查库存 → 调用支付 → 回写订单 → 通知客服，这一整条链路基本没代码编辑器什么事了。
    
-   **需要“你不在的时候它也自己跑”**：定时任务、监听事件、自动恢复流程，这都是 agent 的原生地盘。
    

换句话说：

-   你现在用的 Claude Code / vibe coding，是把“写代码这块”自动化到极致。
    
-   你在看的 Gemini 3 Pro agent 这类东西，是在探索“把写完的代码 + 现有工具变成一个长期跑的自动化员工”。
    

——————

明白了。刚开始学，还没有到自己跑自动化的那一步，所以容易有这样的困惑。后面可以尝试接入自己的实际任务，带着真实场景来练习。
<!-- DAILY_CHECKIN_2025-12-30_END -->

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
