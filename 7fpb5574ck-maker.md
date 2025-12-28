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
