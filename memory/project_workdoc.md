# 游戏筛选项目工作档

更新日期：2026-07-01（JST）

## 1. 当前定位

本仓库用于保存 ChatGPT 可读取的项目工作档记忆，不保存完整游戏画像数据库。

当前分工：

| 层 | 职责 |
|---|---|
| GitHub `zeroga/game_filter_chatgpt` | 工作档、当前状态、读写规则、数据库说明、变更记录 |
| Supabase `chatgpt_memory` | 游戏画像数据、用户偏好、用户游玩记录、用户场景状态 |
| 上传 TXT / SQL 文件 | 一次性迁移源或备份，不作为日常主读取源 |

## 2. 核心数据层

```text
public.memory_items = 共享游戏资料层
public.profile_aliases + public.memory_users = profile 路由层
public.user_preference_items = 用户偏好与个人游玩记录层
public.user_scenario_items = 用户场景状态层
```

当前运行入口优先读取：

```text
README.md
memory/current_rules.md
memory/schema_notes.md
memory/current_state.md
memory/project_workdoc.md
memory/recommendation_entry.md
memory/save_flow.md
memory/profile_routing.md
memory/database_positioning.md
memory/multi_user.md
```

## 3. profile / scenario 确认机制

任何读取或写入用户层数据前，必须先确认：

```text
profile code
scenario code
```

profile code 的身份映射真源是：

```text
public.profile_aliases
```

基本流程：

```text
1. code_norm = lower(trim(profile_code_text))。
2. 查询 public.profile_aliases.alias_norm = code_norm。
3. 如果命中，回显 profile code、alias_norm、user_key，等待用户确认。
4. 如果未命中，不能直接创建新用户。
5. 先提示可能相近或容易混淆的已有账户，询问是不是其中之一。
6. 只有用户明确确认不是已有账户，并确认要创建新 profile，才允许新建。
```

示例：

```text
用户说“用户123”，而已有 123 -> u_123 时，应提示：
你是不是指 profile code 123 / user_key u_123？确认后我会使用这个账户。
```

新 scenario 也使用同一原则：

```text
1. 先查已有 scenario_code / scenario_types。
2. 如果输入像已有场景的自然语言说法，提示用户是不是某个已有场景。
3. 只有用户明确确认不是已有场景，并确认要创建新 scenario，才允许新建。
```

不需要复杂解析规则。核心控制点是：

```text
新用户 / 新场景必须二次确认。
疑似已有账户 / 已有场景时，先提示“是不是某个账户/场景”。
```

## 4. 端到端推荐流程

### 4.1 身份和场景确认

```text
1. 确认 profile code。
2. profile_routing 查询并确认 user_key。
3. 确认 scenario_code。
4. 读取 memory/scenario_types/<scenario_code>.md。
5. 读取 memory/profiles/<user_key>/scenarios/<scenario_code>.md 作为快照参考。
```

个人场景快照不是状态真源。状态真源永远是：

```text
public.user_scenario_items
```

### 4.2 数据读取

推荐流程必须同时读取三层数据：

```text
1. public.memory_items
   - 共享游戏画像
   - 共享游戏事实

2. public.user_preference_items
   - stable_preference
   - played_record
   - game_feedback_overlay
   - positive_reference_index
   - negative_reference_index

3. public.user_scenario_items
   - 当前 user_key + scenario_code 下的推荐、待查、等待、排除、低优先、参考、基准线状态
```

不能只查 `public.memory_items`。

### 4.3 特定游戏查询

如果用户询问特定游戏，必须在三层中同时查询。

共享游戏资料层：

```sql
select *
from public.memory_items
where namespace = 'game_filter'
  and item_type = 'structured_profile_record'
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

### 4.4 合并优先级

```text
当前对话明确反馈
当前用户场景状态 public.user_scenario_items
当前用户游玩记录 played_record
当前用户稳定偏好 / 反馈覆盖 / 正负面索引 public.user_preference_items
共享游戏画像 public.memory_items
外部当前事实核查
```

外部当前事实用于客观结构、版本、价格、联机机制和近期评价。用户主观偏好仍以当前对话和用户层记录为准。

### 4.5 候选审计

每个候选进入推荐、等待、待查、低优先、排除或参考前，必须有审计记录。

必填项：

```text
game_name
aliases_checked
database_lookup_result
played_record_lookup_result
played_record_status_effect
positive_reference_index_result
negative_index_result
old_scenario_conclusion_result
user_feedback_conflict_check
current_web_fact_check
scenario_hard_condition_check
why_not_already_played_or_completed
why_not_waiting
why_not_investigate
why_not_low_priority
why_not_excluded
final_state
```

若缺少 `played_record_lookup_result` 或 `played_record_status_effect`，候选不得进入推荐位。

## 5. 个人游玩记录库机制

个人游玩记录库位于用户偏好层：

```text
public.user_preference_items
user_key = owner_zhengkun
namespace = game_filter
```

每个游戏的个人游玩记录使用：

```text
item_type = played_record
item_key = played:<game_key>
```

必填字段：

```text
game_key
game_name
play_status
source_confidence
last_updated_jst
```

建议字段：

```text
platforms_played
hours_band
completion_state
coop_experience
positive_points
negative_points
drop_reason
revisit_condition
related_scenarios
evidence_source
notes
```

`play_status` 枚举：

```text
played
currently_playing
completed
fully_completed
abandoned
refunded
tried_negative
strong_positive_reference
strong_negative_reference
reference_only
```

`source_confidence` 枚举：

```text
user_firsthand_explicit
user_firsthand_memory
imported_legacy_workdoc
assistant_inferred_needs_confirmation
```

SteamDB 占位记录或未确认的历史推断记录必须标记：

```text
source_confidence = assistant_inferred_needs_confirmation
needs_user_confirmation = true
```

## 6. played_record 的使用规则

```text
completed / fully_completed：默认不作为新推荐；可作为 reference_only、active_baseline、waiting_recheck 或回坑候选。
currently_playing：默认作为 active_baseline 或 current_positive_observation，不重复推荐为新坑。
refunded：默认 block recommendation；除非用户明确说明退款原因已解除。
abandoned：默认降权或排除；必须有 revisit_condition 才能重新进入候选。
tried_negative：默认强降权或排除；外部好评不能单独解除。
strong_positive_reference：作为偏好基准和同类正面参考，不等于当前推荐同一游戏。
strong_negative_reference：作为同类风险基准；命中相似机制时必须审计。
reference_only：只作偏好或机制参考，不作推荐位。
played：必须结合 notes、positive_points、negative_points、related_scenarios 判断。
```

`source_confidence` 的处理：

```text
user_firsthand_explicit：最高优先级，可直接影响结论。
user_firsthand_memory：高优先级，可直接影响结论；若与当前对话冲突，以当前对话为准。
imported_legacy_workdoc：历史有效记录；遇到冲突时需要回看上下文或标记待确认。
assistant_inferred_needs_confirmation：不能作为硬阻断，只能标记需要用户确认；除非另有明确用户反馈或客观结构证据。
```

## 7. 用户场景状态机制

用户在某个场景下对某游戏的推荐、待查、等待、排除、低优先、参考、基准线状态写入：

```text
public.user_scenario_items
user_key = owner_zhengkun
namespace = game_filter
scenario_code = <scenario_code>
game_key = <game_key>
```

当前存在一个历史导入暂存场景：

```text
scenario_code = legacy_imported_status
```

`legacy_imported_status` 只表示历史状态暂存区，不是正式推荐场景。后续需要按真实 `scenario_code` 重新分类。

## 8. 更新规则

用户提供新的游玩、通关、退款、放弃、回坑、强正面或强负面体验时，写入或更新：

```text
public.user_preference_items
item_type = played_record
item_key = played:<game_key>
```

如果反馈是跨场景稳定偏好，写入或更新：

```text
public.user_preference_items
item_type = stable_preference 或 game_feedback_overlay
```

如果反馈形成正负面参考索引，写入或更新：

```text
public.user_preference_items
item_type = positive_reference_index 或 negative_reference_index
```

如果反馈只改变某个场景中的推荐、等待、待查、低优先、排除或参考状态，写入：

```text
public.user_scenario_items
user_key + namespace + scenario_code + game_key
```

写入后，推荐流程必须以用户场景状态和 played_record 共同覆盖共享画像。

## 9. 安全边界

`chatgpt_memory` 为降低直连授权摩擦，当前 RLS 有意关闭。

这意味着：

- 数据库只应存放可公开或低风险的游戏筛选数据。
- 不应存放密钥、token、私密账户数据或不可公开的个人信息。
- GitHub 中不保存 Supabase publishable key 或 anon key。
- 用户层可以保存低风险游戏偏好和游玩记录，但不能保存账号、交易、好友、真实身份、密钥或平台隐私。

## 10. 下次续接方式

新对话继续游戏筛选时，优先读取：

```text
README.md
memory/current_rules.md
memory/schema_notes.md
memory/current_state.md
memory/project_workdoc.md
memory/recommendation_entry.md
memory/save_flow.md
memory/profile_routing.md
memory/database_positioning.md
memory/multi_user.md
```

随后按 profile code 和 scenario code 读取对应场景模板与个人场景快照。

然后查询 Supabase 三层：

```sql
-- 共享游戏资料层
select *
from public.memory_items
where namespace = 'game_filter'
  and item_type = 'structured_profile_record';
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
