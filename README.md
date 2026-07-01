# game_filter_chatgpt

本仓库是“游戏筛选”项目给 ChatGPT / Codex 读取的项目入口。

## 三层结构

```text
规则层 = rules/
记忆层 = memory/
数据层 = Supabase chatgpt_memory
```

| 层 | 位置 | 职责 |
|---|---|---|
| 规则层 | `rules/` | 规则真源、流程规则、治理规则、读取顺序、写入边界、profile / scenario 路由规则、数据库定位说明 |
| 记忆层 | `memory/` | GitHub 中的记忆辅助层，只保存场景类型模板、用户场景快照、项目状态快照、导入说明等非规则内容 |
| 数据层 | Supabase `chatgpt_memory` | 游戏筛选运行数据，包括共享游戏画像、用户偏好、游玩记录、用户场景状态、事件记录 |

## 顶层执行护栏

以下规则是进入本仓库后的最高执行护栏，必须先于任何具体任务执行。

### 只读优先

进入本仓库后，默认只能执行只读操作：

```text
读取仓库元数据
读取文件
搜索文件
读取 issue / PR
读取评论
```

除非用户明确要求修改、提交、创建、删除、关闭、评论或其他写入动作，否则不得调用任何会改变 GitHub、Supabase 或外部系统状态的接口。

读目录、查结构、验证文件存在性时，也必须使用只读接口；不得用 `create_file`、`update_file`、`delete_file`、`create_branch`、`create_pull_request`、`create_commit` 等写类接口进行探测。

若误触写类接口，必须立即停止，说明影响范围，并在后续只使用只读接口，除非用户重新明确要求写入。

### 必须精读

读取本项目规则时，必须逐条精读以下文件：

```text
README.md
rules/current_rules.md
rules/save_flow.md
rules/recommendation_entry.md
rules/feedback_intake.md
rules/routing/profile_routing.md
rules/routing/scenario_routing.md
rules/data/database_positioning.md
rules/data/multi_user.md
rules/memory_candidate_handling.md
rules/governance/rule_change_issue_only.md
rules/governance/rule_memory_layer_separation.md
memory/knowledge/tool_basics.md
memory/README.md
memory/snapshots/current_state.md
memory/snapshots/project_workdoc.md
```

不得只读取 README、搜索命中片段、摘要、文件头、索引或局部规则后就下结论。

不得把 silent load 理解为跳过读取；silent load 只减少对外输出，不减少内部读取、索引、审计和自检。

涉及推荐、筛选、更新清单、解释场景状态、同步工作档、迁移数据、清理公共库、用户层 / 共享层边界判断时，必须先完成规则精读和执行化。

精读完成前，只能输出“已读到的局部结论”和“未完成读取范围”，不得声称已完整读取。

### 存档确认

用户说“保存 / 存档 / 写入 / 同步 / 记录下来”时，只表示进入保存流程，不表示立即写入。

记忆层 / 数据层保存必须按 `rules/save_flow.md` 执行：

```text
1. 汇总待保存内容。
2. 输出保存摘要。
3. 等待用户确认。
4. 用户确认后才执行写入。
5. 写入后回查。
```

保存摘要至少包括：

```text
内容摘要
影响范围
```

用户确认前，不能写入 GitHub、Supabase 或其他外部系统。

规则层变更不进入保存写入流程。规则层变更只能整理为 GitHub issue，由 Codex 或人工根据 issue 修改规则文件并提交 PR。


## 当前读取入口

正式规则入口是：

```text
rules/current_rules.md
```

推荐读取顺序：

```text
README.md
rules/current_rules.md
rules/save_flow.md
rules/recommendation_entry.md
rules/feedback_intake.md
rules/routing/profile_routing.md
rules/routing/scenario_routing.md
rules/data/database_positioning.md
rules/data/multi_user.md
rules/memory_candidate_handling.md
rules/governance/rule_change_issue_only.md
rules/governance/rule_memory_layer_separation.md
memory/knowledge/tool_basics.md
memory/README.md
memory/snapshots/current_state.md
memory/snapshots/project_workdoc.md
```

## 规则层变更流程

规则层变更只能 issue 化。

```text
ChatGPT 在普通对话中不得直接修改 rules/ 下的规则文件。
ChatGPT 在普通对话中不得直接创建规则修改 PR。
ChatGPT 只能识别规则层变更、整理需求、输出 issue 草案，并在用户明确要求时创建 GitHub issue。
Codex 或人工可以根据 issue 驱动的任务修改规则文件并提交 PR。
```

## 记忆层 / 数据层保存流程

记忆层和数据层保存走：

```text
rules/save_flow.md
```

保存对象包括：

```text
public.memory_items
public.user_preference_items
public.user_scenario_items
public.memory_events
memory/scenario_types/*.md
memory/profiles/<user_key>/scenarios/*.md
memory/snapshots/*.md
```

保存流程仍是：汇总待保存内容、输出保存摘要、等待用户确认、写入、写入后回查。

## 禁止混写边界

```text
rules/ 不保存具体游戏画像、用户偏好、用户游玩记录、用户场景状态、候选清单、推荐结论、等待清单、排除清单或用户个人反馈。
memory/ 不保存规则真源、规则治理逻辑、保存流程真源、推荐流程真源、profile / scenario 路由真源、数据库定位真源或多用户规则真源。
Supabase 不保存规则真源、读取顺序、保存流程、规则治理逻辑或规则文件正文。
```

旧的 memory 根目录规则文件已删除；旧规则补丁只保留在 `rules/legacy/` 作为来源归档。
