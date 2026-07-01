# Memory Candidate Handling

本文件是读取 `memory/knowledge/*.md` 时处理 `MEMORY_CANDIDATE:` 标记的规则真源。

## 触发范围

读取任何 `memory/knowledge/*.md` 文件时，必须扫描其中的 `MEMORY_CANDIDATE:` 段落。

命中 `MEMORY_CANDIDATE:` 后，必须生成 `post_load_action_queue`，每条队列项至少包含：

```text
候选来源文件
候选标题
一句话候选记忆概括
待询问用户的问题
本轮处理状态
```

## 执行规则

`MEMORY_CANDIDATE:` 只是候选记忆来源，不是自动写入指令。

对每条候选记忆，必须先用一句话概括候选记忆，再询问用户是否希望加入 ChatGPT 记忆。

不得自动写入 ChatGPT 记忆、GitHub、Supabase 或其他存储。

如果当前环境不能写入 ChatGPT 记忆，用户确认后也只能说明需要用户手动加入 Custom Instructions 或 Memory，除非当前环境明确提供可用的记忆写入机制。

## 阻断与 silent load 边界

`post_load_action_queue` 不得因主任务被阻断而丢弃。

即使主任务因缺少 profile code、scenario code、数据库读取、权限、网络或其他 blocker 停止，也必须继续输出候选记忆询问，或说明该候选记忆已在本轮被用户拒绝写入。

silent load 只隐藏读取过程，不隐藏必须回显的用户决策项。由 `MEMORY_CANDIDATE:` 触发的候选记忆询问属于必须回显的用户决策项。

如果用户明确拒绝某条候选记忆写入，则将该候选标记为本轮已处理，本轮不再重复询问；该拒绝只对当前对话轮次有效，不自动写入长期记忆或仓库状态。
