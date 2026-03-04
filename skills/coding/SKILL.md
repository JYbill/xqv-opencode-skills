---
name: coding
description: 对于编码行为守则，包含最佳实践的场景，以及禁止使用的禁令
---

# 通用编码规范

- 当访问对象属性时，特别是对象层级过多，比如 `config.module.curriculum.xxx`，最多只能使用两层属性访问，所以前面的例子应该使用如下方式来定义

```typescript
const curriculum = config.module;
curriculum.xxx;
```

# 提交规范

- 每次提交代码都应该将所有改动作为一次commit，然后做出总结性的债要作为commit message

# Agent 禁止条款

## Typescript 项目

- agent 不允许使用 `package.json` 中 `scripts` 的任何命令；这些命令仅允许开发者使用。
- agent 在反馈 TypeScript 类型错误时，不应使用 `pnpm typecheck` 或 `pnpm typecheck:watch`，应优先使用 WebStorm MCP tool 完成类型检查；若当前环境不可用 WebStorm MCP tool，应明确告知用户需自行完成类型检查。
