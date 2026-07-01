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
rules/governance/rule_change_issue_only.md
rules/governance/rule_memory_layer_separation.md
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
