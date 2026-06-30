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

## 2. 当前规则基线

项目规则基线为：

```text
v4.6_game_filter_preference_library_machine.txt
```

其角色为：

```text
rules_schema_only
```

关键规则：

- v4.6 只定义规则、schema、状态机、审计要求、输出约束。
- 完整游戏画像不存放在规则文件中。
- 游戏画像数据必须从数据库读取。
- 自然语言说明只是注释，机器字段优先。
- silent load 只控制外部输出，不减少内部读取、索引、审计和自检。
- 推荐目标是上限，不是填满义务；干净候选不足时保留空位。
- 推荐前必须执行候选审计。
- 若缺游戏数据库、用户偏好层、用户游玩记录、用户场景状态或候选审计，阻断推荐。

## 3. 当前数据主库

数据主库：Supabase `chatgpt_memory`

```text
project_ref: zpkfrbfgaaojrblcqvvl
schema: public
namespace: game_filter
main_table: memory_items
event_table: memory_events
import_batch_table: memory_import_batches
```

核心数据层：

```text
public.memory_items = 共享游戏资料层
public.profile_aliases + public.memory_users = profile 路由层
public.user_preference_items = 用户偏好与个人游玩记录层
public.user_scenario_items = 用户场景状态层
```

## 4. 端到端推荐流程

### 4.1 身份和场景确认

任何推荐、筛选、更新清单、解释场景状态前，先确认：

```text
profile code
scenario code
```

然后执行：

```text
1. code_norm = lower(trim(profile_code))
2. 查询 public.profile_aliases 得到 user_key
3. 确认 scenario_code
4. 读取 memory/scenario_types/<scenario_code>.md
5. 读取 memory/profiles/<user_key>/scenarios/<scenario_code>.md 作为快照参考
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

当前已建立的库元数据：

```text
item_type = played_record_library_meta
item_key = current
```

当前已建立的字段规范：

```text
item_type = played_record_schema
item_key = v1
```

每个游戏的个人游玩记录使用：

```text
item_type = played_record
item_key = played:<game_key>
```

### 5.1 `played_record` 必填字段

```text
game_key
game_name
play_status
source_confidence
last_updated_jst
```

### 5.2 `played_record` 建议字段

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

### 5.3 `play_status` 枚举

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

### 5.4 `source_confidence` 枚举

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

### 8.1 用户提供新游玩反馈

当用户提供新的游玩、通关、退款、放弃、回坑、强正面或强负面体验时，写入或更新：

```text
public.user_preference_items
item_type = played_record
item_key = played:<game_key>
```

必须记录：

```text
play_status
source_confidence
last_updated_jst
evidence_source
notes 或 structured payload
```

### 8.2 用户提供稳定偏好或反馈覆盖

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

### 8.3 用户提供场景内结论

如果反馈只改变某个场景中的推荐、等待、待查、低优先、排除或参考状态，写入：

```text
public.user_scenario_items
user_key + namespace + scenario_code + game_key
```

写入后，推荐流程必须以用户场景状态和 played_record 共同覆盖共享画像。

## 9. 已验证样本

数据库直连已验证可用。样本查询结果包括：

- `Warframe`：正面参考，长线 PvE / Mod / 武器和战甲收集。
- `No Man's Sky`：单人正面、联机同步负面。
- `Stolen Realm`：用户已拒绝，负面参考。
- `Inkbound`：当前正面观察。
- `Granblue Fantasy: Relink`：已通关等待大型内容。
- `X4: Foundations`：单人太空 / 舰队经营强正面；因无官方联机，不进入 PC/主机联机场景推荐。

这些样本如果涉及个人体验、已玩状态、正负面参考或场景结论，应从用户层合并读取，而不是只看共享游戏资料层。

## 10. 安全边界

`chatgpt_memory` 为降低直连授权摩擦，当前 RLS 有意关闭。

这意味着：

- 数据库只应存放可公开或低风险的游戏筛选数据。
- 不应存放密钥、token、私密账户数据或不可公开的个人信息。
- GitHub 中不保存 Supabase publishable key 或 anon key。
- 用户层可以保存低风险游戏偏好和游玩记录，但不能保存账号、交易、好友、真实身份、密钥或平台隐私。

## 11. 下次续接方式

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
