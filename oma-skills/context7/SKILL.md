---
name: context7
description: 如果要获取最新的代码或技术文档（如：node.js、nx、nest.js、express、koa、react、vue、go、python3、rust、c++...）则需要读取本SKILL.md
mcp:
  context7:
    url: https://mcp.context7.com/mcp
---

# 预备操作

- 使用`skill_mcp`工具加载本SKILL.md所有的mcp工具

# context7 MCP

- resolve-library-id：将通用库名称解析为与 Context7 兼容的库 ID
  - query：必填，用户的问题或任务（用于根据相关性对结果进行排序）
  - libraryName：必填，要搜索的相关库的名称

- query-docs：使用 Context7 兼容的库 ID 获取库的文档
  - libraryId：必填，精确的 Context7 兼容库 ID（例如 /mongodb/docs 、 /vercel/next.js ）
  - query：必填，需要获取相关文档的问题或任务
