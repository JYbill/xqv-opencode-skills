# xqv-opencode-skills

- 这是一个opencode推荐skills仓库，包含所有本人推荐以及可用的skills + mcp，如果要使用本skills，务必满足skills支持mcp功能，能满足该功能的agent client有：opencode...

## github skill

- 使用前提：需要准备github_pat个人私钥，并设置为`GITHUB_PAT_TOKEN`环境变量

## codebase skill

- 使用前提：准备环境变量
- 环境变量来源于：[claude-context mcp](https://github.com/zilliztech/claude-context/blob/master/docs/getting-started/environment-variables.md#-required-environment-variables)

```bash
# embedding提供商接口格式，默认为OpenAI接口格式
EMBEDDING_PROVIDER
# openai 地址（一般是私有化地址）
OPENAI_BASE_URL
# openai 密钥
OPENAI_API_KEY
# embedding 模型名称
EMBEDDING_MODEL
# milvus向量数据库地址
MILVUS_ADDRESS
# embedding 过程忽略的文件表达式
CUSTOM_IGNORE_PATTERNS
```
