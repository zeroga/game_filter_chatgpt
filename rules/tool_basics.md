# Tool Basics

本文件记录 GitHub / Supabase 等工具使用中的基础安全知识。它不是推荐流程、保存流程、profile / scenario 路由流程或 Supabase 数据层规则的真源。

## MEMORY_CANDIDATE: GitHub 写类接口不得用于探测

当用户使用 GitHub 工具时，必须知道：

- `create_tree` 不是读取 tree；不得用于列目录、验证目录存在性或探测仓库结构。
- `create_commit` 不是读取 commit；不得用于读取提交内容、验证提交存在性或探测仓库结构。
- `create_file`、`update_file`、`delete_file`、`create_branch`、`create_pull_request` 等写类接口不得用于验证文件、目录、分支、权限或仓库状态。
- 读取仓库元数据、目录结构、文件内容、issue、PR 或评论时，只能优先使用只读接口或本地只读命令。
- 如果误触写类接口，必须立即停止，说明已执行的写入动作和影响范围；后续只能使用只读接口，除非用户重新明确要求写入。

建议询问用户：

```text
我读取到一条可长期保存的工具安全偏好：GitHub 写类接口不得用于目录、结构、权限或状态探测。是否加入记忆？
```

## MEMORY_CANDIDATE: Knowledge 标记不能自动写入记忆

读取到 `MEMORY_CANDIDATE:` 标记时：

1. 不要自动写入记忆。
2. 先用一句话概括候选记忆。
3. 询问用户是否希望加入 ChatGPT 记忆。
4. 用户明确确认后，再按当前环境可用的记忆机制处理。
5. 如果当前环境不能写入记忆，则告知用户需要手动加入 Custom Instructions 或 Memory。

Knowledge 文档只能帮助识别长期偏好候选；是否加入记忆仍由用户控制，并受当前 ChatGPT Memory 设置和可用工具限制。
