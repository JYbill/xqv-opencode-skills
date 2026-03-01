---
name: codebase
description: 如果需要获取本地代码相关优先使用本SKILL.md，本skill能够直接提升本地代码的查询效率
mcp:
  codebase:
    command: ["pnpx", "@zilliz/claude-context-mcp@latest"]
    environment:
      EMBEDDING_PROVIDER: OpenAI
      OPENAI_BASE_URL: ${OPENAI_BASE_URL}
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      EMBEDDING_MODEL: ${EMBEDDING_MODEL}
      MILVUS_ADDRESS: ${MILVUS_ADDRESS}
      CUSTOM_IGNORE_PATTERNS: tmp/**,logs/**,.ai/**,.aiassistant/**,.cache/**,.claude/**,.github/**,.husky/**,.idea/**,.sisphus/**,public/**,vendor/**,*.env
---

# 预备操作

- 使用`skill_mcp`工具加载本SKILL.md所有的mcp工具

# codebase MCP

- index_codebase：为milvus混合搜索（BM25 + 稠密向量）建立代码库目录索引
- search_code：使用自然语言查询配合混合搜索（BM25 + 稠密向量）检索已建立索引的代码库。
- clear_index：清除代码库的搜索索引。
- get_indexing_status：获取代码库的当前索引状态。显示正在索引的代码库的进度百分比以及已索引代码库的完成状态。

# 场景

- 如果用户询问本地代码相关问题时，而且你发现本代码仓库并没有进行embedding时，应该询问用户是否同意使用`index_codebase`工具来完成embedding整个本地代码！禁止在没有询问用户的情况下，私自调用`index_codebase`工具！！！

- 如果用户询问本地代码相关问题时，发现本地仓库已经embedding过，那么正常去使用`search_code`工具查询本地代码信息

- 以下工具禁止在没有询问用户意愿后私自调用：`clear_index`、`index_codebase`
