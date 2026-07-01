# 当前状态

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
- 已读取项目规则 `v4.6_game_filter_preference_library_machine.txt`。
- 已验证数据库直连查询可用。
- 已用数据库形式查询 `X4: Foundations`，确认其结论：单人太空 / 舰队经营强正面，但因无官方联机不进入 PC/主机联机场景推荐。
- 已新增 `rules/feedback_intake.md`，作为用户游戏反馈即时理解机制的唯一详细真源。
- 已新增 `rules/routing/scenario_routing.md`，并补充场景对象模型、最小定义、场景相关存储边界。
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

## 当前规则源

当前运行主入口：

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
- 新场景必须先满足 `rules/routing/scenario_routing.md` 的场景对象模型最小定义：confirmed user_key、confirmed scenario_code、场景目标、适用平台 / 类型范围、硬性排除条件。
- 场景相关存档必须按 `rules/save_flow.md` 生成分层写入清单并逐项回查。
- `public.user_scenario_items` 的数据库字段是 `state`，不要写成 `status`。
- waiting 项更新必须全量核查；不完整时必须标记 `waiting_info_update_incomplete`。
- 当前数据库 RLS 有意关闭，只允许存放低风险游戏筛选数据。

## 待处理事项

1. 为常用查询建立固定 SQL 模板。
2. 后续如有新游戏反馈，应先执行 `feedback_intake`，再进入推荐/解释/保存流程。
3. 后续如涉及场景保存，必须先执行场景分层写入清单并回查。
4. 若要开放给其他用户，需要另行设计权限和白名单，不应把真实权限数据放入 RLS 关闭的库。
