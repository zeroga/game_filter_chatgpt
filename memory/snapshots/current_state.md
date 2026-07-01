# 当前状态

> Snapshot boundary: 本文件是 memory 快照层文件，只作续接和人工阅读辅助。
> 本文件不是规则真源；规则真源以 `rules/current_rules.md` 及其引用文件为准。
> 本文件不是用户场景状态真源；用户场景状态以 `public.user_scenario_items` 为准。
> 若本文件与 `rules/` 下规则冲突，以 `rules/` 为准；若本文件与 Supabase 用户场景状态冲突，以 Supabase 为准。


更新日期：2026-07-01（JST）

## 当前阶段

项目已完成从“巨大工作档记忆”到“双层记忆”的基础切换：

```text
GitHub rules/ = 规则入口；GitHub memory/ = 工作档记忆 / 场景类型模板 / 用户场景快照
Supabase = 游戏数据记忆 / 用户偏好 / 游玩记录 / 用户场景状态
```

## 已完成事项

- 已确认 GitHub 仓库：`zeroga/game_filter_chatgpt`。
- 已确认 Supabase 项目：`chatgpt_memory` / `zpkfrbfgaaojrblcqvvl`。
- 已建立 Supabase 记忆表：
  - `public.memory_items`
  - `public.memory_events`
  - `public.memory_import_batches`
  - `public.memory_import_file_chunks`
- 已将 `v1.0_game_filter_game_profile_database_machine.txt` 迁入 Supabase。
- 已验证导入行数：`memory_items = 404`。
- 历史迁移时曾读取旧项目规则 `v4.6_game_filter_preference_library_machine.txt`；该文件现在只作 legacy 归档来源，不是当前规则入口。
- 已验证数据库直连查询可用。
- 已用数据库形式查询 `X4: Foundations`，确认其结论：单人太空 / 舰队经营强正面，但因无官方联机不进入 PC/主机联机场景推荐。
- 已新增 `rules/feedback_intake.md`，作为用户游戏反馈即时理解机制的唯一详细真源。
- 已新增 `rules/routing/scenario_routing.md`，并补充场景对象模型、字段完整性边界、场景相关存储边界。
- 已在 `rules/save_flow.md` 中补充场景相关存档完整性规则：场景相关保存必须生成分层写入清单并逐项回查。

## 当前主数据源

```text
Supabase project_ref: zpkfrbfgaaojrblcqvvl
schema: public
namespace: game_filter
```

主要表：

```text
public.memory_items
public.user_preference_items
public.user_scenario_items
public.profile_aliases
public.memory_users
public.memory_events
```

## 当前规则入口快照（仅供参考）

规则入口快照如下；如与 `rules/current_rules.md` 冲突，以 `rules/current_rules.md` 为准：

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
rules/data/schema_notes.md
rules/governance/rule_change_issue_only.md
rules/governance/rule_memory_layer_separation.md
memory/README.md
memory/snapshots/current_state.md
memory/snapshots/project_workdoc.md
```

旧规则归档：

```text
rules/legacy/v4.6_game_filter_preference_library_machine.json
```

旧规则只作为来源归档，不再作为后续运行入口。

## 重要约束

- 后续游戏画像查询默认走 Supabase，不回读巨大 TXT。
- GitHub 不保存完整游戏画像；rules/ 保存规则，memory/ 保存工作档、状态快照、场景类型模板和用户场景快照。
- 推荐前必须查旧数据、负面索引、用户反馈冲突和当前事实。
- 用户对游戏发表意见时，必须立即执行 `rules/feedback_intake.md`。
- 新场景保存或推荐时，只能按用户已确认字段执行；未定义字段不由模型补齐，且不阻断已确认 scenario_code 下的推荐或保存。若场景目标、平台范围或硬排除条件较粗，只能说明“场景定义仍较粗”。正式规则以 `rules/routing/scenario_routing.md` 和 `rules/save_flow.md` 为准。
- 场景相关存档必须按 `rules/save_flow.md` 生成分层写入清单并逐项回查。
- `public.user_scenario_items` 的数据库字段是 `state`，不要写成 `status`。
- waiting 项更新摘要只作历史参考；实际候选状态必须查询 `public.user_scenario_items`。不完整核查的正式处理以 `rules/recommendation_entry.md` 和 `rules/save_flow.md` 为准。
- 当前数据库 RLS 有意关闭，只允许存放低风险游戏筛选数据。

## 待处理事项

1. 为常用查询建立固定 SQL 模板。
2. 后续如有新游戏反馈，应先执行 `feedback_intake`，再进入推荐/解释/保存流程。
3. 后续如涉及场景保存，必须先执行场景分层写入清单并回查。
4. 若要开放给其他用户，需要另行设计权限和白名单，不应把真实权限数据放入 RLS 关闭的库。
