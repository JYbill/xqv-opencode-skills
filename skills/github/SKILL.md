---
name: github
description: 如果进行GitHub相关操作，比如：github issue操作、github release操作、github discussion操作...等。务必使用本SKILL.md
mcp:
  github:
    url: https://api.githubcopilot.com/mcp
    headers:
      Authorization: Bearer ${GITHUB_PAT_TOKEN}
      X-MCP-Toolsets: repos,issues
      X-MCP-Readonly: true
---

# 预备操作

- 使用`skill_mcp`工具加载本SKILL.md所有的mcp工具

# github MCP

- 下面展示常用tool，以MCP tool具体为准

## 文件与仓库操作

- get_file_contents：获取文件或目录内容
- get_repository_tree：获取仓库目录树结构
- get_repository：获取仓库详细信息
- search_code：全仓库代码搜索

## 提交历史

- get_commit：获取提交详情
- list_commits：列出提交历史

## 发布版本

- get_latest_release：获取最新发布版本
- get_release：获取指定发布版本
- list_releases：列出所有发布版本

## Issue 相关工具

- issue_read：获取 issue 详情（支持多种操作）
- list_issues：列出仓库的 issues
- search_issues：搜索 issues（支持复杂查询）

## 标签管理（只读）

- get_label：获取特定标签
- list_label：列出所有标签
- list_issue_types：列出 issue 类型（组织级）
