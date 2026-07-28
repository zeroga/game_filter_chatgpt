# 游戏筛选项目工作档

> Snapshot boundary: 本文件是 memory 快照层文件，只作续接和人工阅读辅助。
> 本文件不是规则真源；规则真源以 `rules/current_rules.md` 及其引用文件为准。
> 本文件不是用户场景状态真源；用户场景状态以 `public.user_scenario_items` 为准。
> 若本文件与 `rules/` 下规则冲突，以 `rules/` 为准；若本文件与 Supabase 用户场景状态冲突，以 Supabase 为准。


更新日期：2026-07-01（JST）

## 1. 当前定位

本仓库用于保存 ChatGPT 可读取的项目工作档记忆，不保存完整游戏画像数据库。

当前分工：

| 层 | 职责 |
|---|---|
| GitHub `zeroga/game_filter_chatgpt` / `rules/` | 规则真源、读写规则、数据库定位说明、治理规则、规则入口 |
| GitHub `zeroga/game_filter_chatgpt` / `memory/` | 工作档、当前状态快照、变更记录、场景类型模板、用户场景快照 |
| Supabase `chatgpt_memory` | 游戏画像数据、用户偏好、用户游玩记录、用户场景状态 |
| 上传 TXT / SQL 文件 | 一次性迁移源或备份，不作为日常主读取源 |

本文件定位为项目总览、当前状态快照、数据层索引、续接方式和待办摘要。详细流程规则以 `rules/` 下文件为准。

## 2. 规则入口快照（仅供参考）

本节只保留读取顺序快照；完整读取顺序以 `rules/current_rules.md` 为准。

当前运行入口优先读取：

```text
README.md
rules/current_rules.md
rules/save_flow.md
rules/reporting/weekly_report.md
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

专项分工：

```text
README.md = 最高入口、项目边界、只读优先、精读约束、存档安全护栏。
rules/current_rules.md = 当前规则入口和总规则。
rules/feedback_intake.md = 用户游戏反馈即时理解机制的唯一详细真源。
rules/save_flow.md = 保存 / 存档 / 写入确认流程，以及场景相关存档分层写入清单和回查规则。
rules/reporting/weekly_report.md = profile 级周报的配置、正文、审计、增量、调度、同步和自动写入边界唯一专项真源。
rules/routing/profile_routing.md = profile code 到 user_key 的确认与路由。
rules/routing/scenario_routing.md = scenario code 的确认、查找、创建、场景对象模型和真源边界。
rules/data/database_positioning.md = 数据库分层定位。
rules/data/multi_user.md = 多用户偏好层与场景层分工。
rules/data/schema_notes.md = 数据库结构说明和旧定位纠偏。
rules/governance/rule_change_issue_only.md = 规则层变更只能 issue 化。
rules/governance/rule_memory_layer_separation.md = 规则层 / 记忆层 / 数据层分离边界。
memory/README.md = 记忆层职责说明。
memory/snapshots/current_state.md = 当前阶段、已完成事项、待处理事项。
```

存档安全护栏允许在 README、`rules/current_rules.md`、`rules/save_flow.md` 中保留必要冗余，避免精读后误写入 GitHub / Supabase。

即时理解机制不在多处维护完整流程；详细规则只放在 `rules/feedback_intake.md`。

场景对象模型不在多处维护完整流程；详细规则只放在 `rules/routing/scenario_routing.md`。

场景相关存档完整性不在多处维护完整流程；详细规则只放在 `rules/save_flow.md`。

## 3. 核心数据层

```text
public.memory_items = 共享游戏资料层
public.profile_aliases + public.memory_users = profile 路由层
public.user_preference_items = 用户偏好与个人游玩记录层
public.user_scenario_items = 用户场景状态层
public.user_report_subscriptions = profile 级周报配置与运行快照层
```

GitHub 场景相关文件定位：

```text
memory/scenario_types/*.md = 场景类型模板，不是实际场景。
memory/profiles/<user_key>/scenarios/*.md = 用户个人场景快照，不是状态真源。
public.user_scenario_items = 用户场景状态真源。
```

## 4. 场景对象模型摘要

完整规则见：

```text
rules/routing/scenario_routing.md
```

一个 scenario 指某个 user_key 下的一套游戏筛选目标、适用范围、约束、偏好解释方式、候选审计维度和候选状态空间。

新场景字段完整性摘要：

```text
confirmed user_key
confirmed scenario_code
按用户已确认字段执行
未定义字段不由模型补齐
未定义字段不阻断已确认 scenario_code 下的推荐或保存
```

如果缺少场景目标、适用范围或硬性排除条件，只能说明“场景定义仍较粗”，不得声称场景无效；正式规则以 `rules/routing/scenario_routing.md` 和 `rules/save_flow.md` 为准。

## 5. 场景相关存档完整性摘要

完整规则见：

```text
rules/save_flow.md
```

保存内容涉及新 scenario_code、场景对象模型、场景口径、场景类型模板、用户场景快照、场景候选或场景状态时，必须先生成分层写入清单。

分层写入清单至少覆盖：

```text
memory/scenario_types/<scenario_code>.md
memory/profiles/<user_key>/scenarios/<scenario_code>.md
public.user_scenario_items
public.user_preference_items
public.memory_items
public.memory_events
```

写入后必须逐项回查。只要应写层没有写入或没有回查，不能声明保存完成。

## 6. 已核实数据库字段和命名

### 6.1 user_scenario_items

实际数据库字段为：

```text
state
```

不要把数据库字段写成 `status`。中文可以说“状态”，但涉及数据库字段时必须使用 `state`。

当前已见 state 值包括：

```text
recommended
waiting
investigate
low_priority
excluded
reference_only
```

### 6.2 正负面索引 item_type

不做 Supabase 数据迁移，不改历史 `item_type`，不机械替换 payload 内部历史词汇。

用户偏好层 `public.user_preference_items` 当前使用：

```text
positive_reference_index
negative_reference_index
```

共享资料层 `public.memory_items` 当前使用：

```text
positive_reference_index
negative_decision_index
```

后续文档不要把 `positive_reference / negative_reference` 写成当前规范 `item_type`。如只是在 payload 内部出现历史词汇，可保留。

## 7. 推荐前最低读取要求

推荐、筛选、更新清单、解释场景状态之前，必须完成：

```text
1. 按 profile_routing 确认 profile code -> user_key。
2. 按 scenario_routing 确认 scenario code。
3. 读取场景类型模板。
4. 读取用户个人场景快照作为参考。
5. 查询 public.memory_items。
6. 查询 public.user_preference_items，包括 played_record。
7. 查询 public.user_scenario_items，字段为 state。
8. 如果当前输入包含游戏反馈，立即执行 feedback_intake。
9. 涉及当前事实时联网核查。
10. 完成候选审计后才能输出推荐、等待、待查、排除或低优先结论。
```

`played_record` 不是可选参考。缺少 `played_record_lookup_result` 或 `played_record_status_effect` 时，候选不得进入推荐位。

## 8. 特定游戏查询模板

共享游戏资料层：

```sql
select *
from public.memory_items
where namespace = 'game_filter'
  and (
    title ilike '%<game_name>%'
    or item_key = '<game_key>'
    or payload::text ilike '%<game_name_or_alias>%'
  );
```

用户偏好与游玩记录层：

```sql
select *
from public.user_preference_items
where user_key = '<user_key>'
  and namespace = 'game_filter'
  and (
    item_key = 'played:<game_key>'
    or title ilike '%<game_name>%'
    or payload::text ilike '%<game_name_or_alias>%'
  );
```

用户场景状态层：

```sql
select *
from public.user_scenario_items
where user_key = '<user_key>'
  and namespace = 'game_filter'
  and (
    scenario_code = '<scenario_code>'
    or scenario_code = 'legacy_imported_status'
  )
  and (
    game_key = '<game_key>'
    or title ilike '%<game_name>%'
    or payload::text ilike '%<game_name_or_alias>%'
  );
```

`legacy_imported_status` 只作历史状态暂存参考，不能直接替代当前 `scenario_code` 的结论。

## 9. 安全边界

`chatgpt_memory` 为降低直连授权摩擦，当前 RLS 有意关闭。

这意味着：

```text
数据库只应存放可公开或低风险的游戏筛选数据。
不应存放密钥、token、私密账户数据或不可公开的个人信息。
GitHub 中不保存 Supabase publishable key 或 anon key。
用户层可以保存低风险游戏偏好和游玩记录，但不能保存账号、交易、好友、真实身份、密钥或平台隐私。
```

## 10. 下次续接方式

新对话继续游戏筛选时，优先读取：

```text
README.md
rules/current_rules.md
rules/save_flow.md
rules/reporting/weekly_report.md
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

随后按 profile code 和 scenario code 读取对应场景模板与个人场景快照。

然后查询 Supabase 三层：

```sql
-- 共享游戏资料层
select *
from public.memory_items
where namespace = 'game_filter';
```

```sql
-- 用户偏好与游玩记录层
select *
from public.user_preference_items
where user_key = '<user_key>'
  and namespace = 'game_filter';
```

```sql
-- 用户场景状态层
select *
from public.user_scenario_items
where user_key = '<user_key>'
  and namespace = 'game_filter';
```

如果是特定游戏，三个层都要按 `title / game_key / item_key / payload` 查询，不能只查 `public.memory_items`。

## 11. 当前待办

```text
1. 后续如新增场景，按 `rules/routing/scenario_routing.md` 执行二次确认；只按用户已确认字段执行，未定义字段不阻断已确认 scenario_code 下的推荐或保存。
2. 后续如有新游戏反馈，先执行 feedback_intake，再进入 save_flow。
3. 场景相关存档必须按 save_flow 生成分层写入清单并回查。
4. waiting 项更新摘要只作历史参考；实际候选状态必须查询 `public.user_scenario_items`，正式处理以 `rules/recommendation_entry.md` 和 `rules/save_flow.md` 为准。
5. 如需要迁移或重命名 Supabase 历史 item_type，必须另起迁移方案，不在文档修正中顺手改数据。
6. 若要开放给其他用户，需要另行设计权限和白名单，不应把真实权限数据放入 RLS 关闭的库。
7. 按 Issue #13 的 schema 文档在 Supabase 创建 `public.user_report_subscriptions`，从创建时启用 RLS 并验证客户端角色无权限。
8. 用户确认具体 profile 周报配置后，创建 ChatGPT Automation，保存 `automation_id` 并完成配置与任务双向回查。
```
