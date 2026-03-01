---
name: ide-edit
description: 需要使用编辑器编辑时，需要使用本SKILL.md，本skill专门负责处理jetbrains webstorm IDE的编码工作
mcp:
  webstorm:
    url: http://localhost:64342/sse
---

# 预备操作

- 使用`skill_mcp`工具加载本SKILL.md所有的mcp工具

# webstorm MCP

- webstorm MCP 包含多种使用webstorm IDE对项目读取、编辑等操作的tools，使用这些tools能方便编辑内容并给用户给予编辑反馈

# 场景

- 优先使用webstorm MCP工具的处理。如果webstorm MCP连接失败，则需要回退到普通的编辑tool处理
