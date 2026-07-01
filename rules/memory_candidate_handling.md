# Memory Candidate Handling

本文件是 `MEMORY_CANDIDATE:` 后处理规则真源。

`memory/knowledge/*.md` 仍是公共知识存档，不是规则真源；其中的 `MEMORY_CANDIDATE:` 只表示可询问用户是否加入记忆的候选来源。

## 扫描规则

读取任何 `memory/knowledge/*.md` 文件时，必须扫描 `MEMORY_CANDIDATE:` 标记。

命中后，必须生成 `post_load_action_queue`。队列项至少包括：

```text
source_file
candidate_heading
one_sentence_summary
status
```

## 队列处理规则

`post_load_action_queue` 不得因主任务 blocker 被丢弃。

即使主任务因缺少 profile code、scenario code、数据库读取、外部事实核查或其他 blocker 停止，也必须继续处理队列中的用户决策项。

对候选记忆，不能自动写入 ChatGPT 记忆、GitHub、Supabase 或其他存储。必须先用一句话概括候选记忆，并询问用户是否希望加入记忆。

如果用户明确拒绝写入，将该候选项标记为本轮已处理，本轮不再重复询问。

如果当前环境不能写入 ChatGPT 记忆，应说明需要用户手动加入 Custom Instructions 或 Memory，或等待可用记忆工具。

## silent load 边界

silent load 只隐藏读取过程，不隐藏必须回显的用户决策项。

读取 memory / knowledge 产生的 `post_load_action_queue` 属于必须回显的用户决策项，不得被 silent load 吞掉。
