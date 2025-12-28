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
