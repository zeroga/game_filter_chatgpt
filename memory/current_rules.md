# Current Rules

本文件是后续运行入口，不再要求用户指定旧版本号。

## 读取入口

后续只读 GitHub 仓库：

```text
zeroga/game_filter_chatgpt
```

读取顺序：

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

注意：没有默认 `current_scenario.md`。实际场景必须通过 profile code + scenario code 确认后，从 `memory/scenario_types/<scenario_code>.md`、个人场景快照和 Supabase 用户场景层读取。

## 规则存储分工

GitHub 保存：

```text
当前规则
工作档
数据库说明
状态记录
变更记录
场景类型模板
用户场景快照
```

Supabase 保存：

```text
共享游戏画像
共享游戏事实
用户稳定偏好
用户游玩记录
用户反馈覆盖
用户场景状态
场景条目
待查任务
等待条件
来源摘要
```

## 通用硬规则

```text
silent load 只减少对外输出，不减少内部读取、索引、审计和自检。
推荐目标是上限，不是必须填满。
缺游戏数据库时停止推荐或更新。
缺用户偏好层读取时停止推荐或更新。
缺用户游玩记录读取时停止推荐或更新。
缺用户场景状态读取时停止推荐或更新。
缺候选审计时阻断推荐。
涉及当前事实、版本、价格、评价、联机结构、DLC、EA 状态时必须联网核查。
用户主观反馈优先于外部主观评价；客观结构仍需外部验证。
```

## 推荐前必须读取的数据层

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

`played_record` 不是可选参考。任何推荐、筛选、更新清单、解释候选状态前，都必须读取并合并。

## 候选审计必填项

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

若候选审计中缺少 `played_record_lookup_result` 或 `played_record_status_effect`，不得进入推荐位。

## played_record 状态处理规则

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

## source_confidence 处理规则

```text
user_firsthand_explicit：最高优先级，可直接影响结论。
user_firsthand_memory：高优先级，可直接影响结论；若与当前对话冲突，以当前对话为准。
imported_legacy_workdoc：历史有效记录；遇到冲突时需要回看上下文或标记待确认。
assistant_inferred_needs_confirmation：不能作为硬阻断，只能标记需要用户确认；除非另有明确用户反馈或客观结构证据。
```

## 合并优先级

```text
当前对话明确反馈
当前用户场景状态 public.user_scenario_items
当前用户游玩记录 played_record
当前用户稳定偏好 / 反馈覆盖 / 正负面索引 public.user_preference_items
共享游戏画像 public.memory_items
外部当前事实核查
```

外部当前事实用于客观结构、版本、价格、联机机制和近期评价；用户主观偏好仍以当前对话和用户层记录为准。

## 写回规则

用户提供新的游玩、通关、退款、放弃、回坑、强正面或强负面体验时，写入：

```text
public.user_preference_items
item_type = played_record
item_key = played:<game_key>
```

如果反馈是跨场景稳定偏好，写入或同步更新：

```text
public.user_preference_items
item_type = stable_preference 或 game_feedback_overlay
```

如果反馈只改变某个场景中的推荐、等待、待查、低优先、排除或参考状态，写入：

```text
public.user_scenario_items
user_key + namespace + scenario_code + game_key
```

不得把个人游玩记录、个人正负面索引、个人推荐状态写入 `public.memory_items`。

## 来源优先级

```text
用户当前对话明确反馈
用户 played_record / stable_preference / scenario state
用户亲身体验历史记录
官方机制公告或路线图
官方商店页
近期评价
长差评
实机视频
可信社区
媒体文章硬事实
```

## 旧规则文件定位

```text
memory/rules/v4.6_game_filter_preference_library_machine.json
```

该文件只作为来源归档，不再作为后续运行入口。
