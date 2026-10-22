# MCP（Model Context Protocol，模型上下文协议）

MCP 是 Anthropic 2024 年底推出的开源标准协议，被称为 AI 领域的USB‑C 接口：用来标准化大模型（LLM）和外部工具、数据源之间的通信。

## 核心架构

1. **MCP Host**：用户交互入口，如 Claude Desktop、Cursor、VSCode等。

2. **MCP Client**：协议客户端，和 MCP Server 建立一对一连接
> Claude Code 本身也是 MCP Client（内部为每个 MCP Server 维护一个 Client 连接）

3. **MCP Server**：轻量服务程序，对外暴露能力：
    - Resources：读取本地文件、数据库、Notion 文档、企业知识库等上下文数据
    - Tools：执行动作，调用计算器、Git、网页搜索、API、业务系统
    - Prompts：预置提示词模板

Host 提前通过 MCP Client，去 MCP Server 拉取工具清单，把 MCP Server 暴露的工具，包装成 LLM 能识别的 tool 定义，喂给大模型。

> 大模型根本不知道什么是 MCP、JSON-RPC，它只懂有哪些工具、工具入参长什么样。

## 解决的问题

- 传统工具调用：模型工具两两对接，N 个模型 × M 个工具就要 N×M 套集成代码。
- MCP：**一次开发，到处可用**。一个 MCP 服务，可以同时给 Cursor、Claude Code、VSCode 等多个客户端使用


# MCP 的配置

## Claude Code中配置

找到 `~/.claude.json`（比如： `C:\Users\30539\.claude.json`）
然后在对应的项目的 `mcpServers` 中加入配置即可
![[Pasted image 20261020191132.png|529]]
