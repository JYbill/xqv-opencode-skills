---
name: coding
description: 对于编码行为守则，包含最佳实践的场景，以及禁止使用的禁令
---

## Agent 禁止条款

## Typescript 项目

- agent 不允许使用 `package.json` 中 `scripts` 的任何命令；这些命令仅允许开发者使用。
- agent 在反馈 TypeScript 类型错误时，不应使用 `pnpm typecheck` 或 `pnpm typecheck:watch`，应优先使用 WebStorm MCP tool 完成类型检查；若当前环境不可用 WebStorm MCP tool，应明确告知用户需自行完成类型检查。
