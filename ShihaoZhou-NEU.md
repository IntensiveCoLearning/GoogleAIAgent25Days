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
# 2025-12-29
<!-- DAILY_CHECKIN_2025-12-29_START -->
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

YAML: YAML Ain't Markup Language™，是一种**人类友好的数据序列化格式**

### 核心特点

NaN.  **可读性极高**
      

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
