---
name: search
description: 按问题场景使用 GitHub MCP、Context7 MCP、Exa WebSearch MCP 搜索并回答。用户要求联网检索、查 GitHub 项目、核对源码、查框架或 SDK 文档、查 API 用法、查全网资料、官网、博客、产品信息、新闻或对比资料时，必须使用这个 skill。这个 skill 强制禁止 Bash、subagent 和普通文件检索工具参与外部搜索。
---

# search

这个 skill 用于把搜索任务限定在 GitHub MCP、Context7 MCP、Exa WebSearch MCP 内完成。目标是按信息来源选择正确 MCP，避免用错误工具查错地方。

## 禁止令

禁止使用 Bash 工具完成搜索、检索、网页访问、GitHub 查询、文档查询或任何替代 MCP 的信息获取行为。

禁止使用 subagent、Agent、Explore、general-purpose 或任何代理工具完成本 skill 的搜索任务。

禁止使用 Glob、Grep、Read 等普通文件工具替代外部搜索。只有当用户明确要求处理本地文件，或需要读取用户指定的本地输入文件时，才允许读取本地文件。

禁止把未检索到的信息猜成事实。无法通过 MCP 查证时，明确说明未查到。

## GitHub MCP

GitHub MCP 用于 GitHub 站内信息。

当问题指向 GitHub 仓库、源码、README、issue、PR、commit、release、workflow、action、discussion，或需要核对某个开源项目的真实实现时，优先使用 GitHub MCP。

它适合确认代码事实、项目状态、版本变更、仓库讨论、维护者回应和 issue 进展。

它不适合做泛网页搜索，也不适合查非 GitHub 来源的官网、博客、新闻和产品页面。

回答时说明结论来自具体仓库页面、issue、PR、commit、release 或文件位置。

## Context7 MCP

Context7 MCP 用于库、框架、SDK、API 的官方文档和稳定用法。

当问题涉及某个依赖的接口、配置项、示例代码、版本行为、迁移说明、最佳实践或 API 参数时，优先使用 Context7 MCP。

使用前先确认库名和主题。如果需要库 ID，先解析库 ID，再拉取对应文档。

它适合查稳定技术文档和官方用法。

它不适合查 GitHub issue、PR、源码实现细节、社区新闻、产品资讯或泛网页资料。

回答时说明结论来自哪个库的文档，并区分文档事实和自己的整理判断。

## Exa WebSearch MCP

Exa WebSearch MCP 用于开放网页搜索。

当问题需要全网资料、官网页面、博客文章、产品信息、新闻、对比资料，或 GitHub MCP 与 Context7 MCP 都覆盖不到时，使用 Exa WebSearch MCP。

优先打开官方来源、高可信来源和能回溯原始信息的页面。

它适合补充背景资料、最新网页信息、产品边界和跨来源对比。

它不适合替代 GitHub MCP 核对仓库内部事实，也不适合替代 Context7 MCP 查询库文档细节。

回答时保留可回溯链接，并说明哪些结论来自网页资料。

## 组合使用

如果一个问题同时涉及多个来源，就组合使用 MCP。

源码真实性和仓库状态用 GitHub MCP 核对。

API 用法、配置项和 SDK 行为用 Context7 MCP 核对。

背景资料、官网说明、新闻、博客和产品对比用 Exa WebSearch MCP 补齐。

最终回答必须说明信息来源，不能把不同来源混成一个未标注的事实。
