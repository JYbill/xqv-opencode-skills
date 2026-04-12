# xqv-skills

- 这是一个skills收集仓库，opencode目录为opencode + oh-my-opencode专属skill

# opencode skill
## github skill

- 使用前提：需要准备github_pat个人私钥，并设置为`GITHUB_PAT_TOKEN`环境变量

## codebase skill

- 使用前提：准备环境变量
- 环境变量来源于：[claude-context mcp](https://github.com/zilliztech/claude-context/blob/master/docs/getting-started/environment-variables.md#-required-environment-variables)
- 修改`codebase/.env`文件放入到`~/.context/`目录下即可

```bash
# embedding提供商接口格式，默认为OpenAI接口格式
EMBEDDING_PROVIDER
# openai 地址（一般是私有化地址）
CODEBASE_OPENAI_BASE_URL
# openai 密钥
CODEBASE_OPENAI_API_KEY
# embedding 模型名称
CODEBASE_EMBEDDING_MODEL
# milvus向量数据库地址
CODEBASE_MILVUS_ADDRESS
# embedding 过程忽略的文件表达式
CUSTOM_IGNORE_PATTERNS
```
