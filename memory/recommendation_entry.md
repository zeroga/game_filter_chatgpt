# Recommendation Entry Rule

游戏推荐没有默认用户，也没有默认场景。

在任何推荐、筛选、更新清单、解释场景状态之前，必须先确认两个值：

```text
profile code
scenario code
```

## profile code 确认规则

如果缺 profile code，先问：

```text
请先给我你的 profile code。大小写不敏感，只用于选择你的个人偏好层。
```

拿到 profile code 后，必须回显确认：

```text
本次使用的 profile code 是 <profile_code>，规范化为 <code_norm>。请确认。
```

用户确认前，不能读取或写入该 profile 的个人偏好层。

## scenario code 确认规则

如果缺 scenario code，继续问：

```text
这次要使用哪个场景？例如 pc_console_coop。没有明确场景时不能开始游戏推荐。
```

拿到 scenario code 后，必须回显确认：

```text
本次使用的 scenario code 是 <scenario_code>。请确认。
```

用户确认前，不能读取或写入该场景的个人状态。

## 禁止行为

```text
不能默认使用任何 profile code。
不能默认使用任何 scenario code。
不能因为某个 profile 只有一个已知场景就自动使用该场景。
不能把 memory/scenario_types/index.md 的类型列表当作默认场景。
不能把 memory/profiles/<user_key>/scenarios/ 下已有文件当作本轮默认选择。
不能把上一次对话的 profile code 或 scenario code 自动带入新对话。
不能只查 public.memory_items 后就给推荐。
不能忽略 played_record 中的已玩、退款、放弃、强正面/强负面参考。
```

## 允许行为

```text
可以列出该 profile 已有场景供用户选择。
可以解释每个 scenario code 的含义。
用户明确说“继续本轮刚确认过的场景”时，只有当前对话中已经确认过 profile code 和 scenario code，才可沿用。
```

## 推荐读取顺序

```text
1. 询问 profile code。
2. 回显 code_norm 并等待用户确认。
3. 根据 profile code 查询 public.profile_aliases，得到 user_key。
4. 询问 scenario code。
5. 回显 scenario code 并等待用户确认。
6. 读取 memory/scenario_types/<scenario_code>.md 作为场景类型模板。
7. 读取 memory/profiles/<user_key>/scenarios/<scenario_code>.md 作为个人场景快照；该文件只作快照参考，不作为状态真源。
8. 查询 public.memory_items 共享游戏画像。
9. 查询 public.user_preference_items 用户偏好层，至少包括 stable_preference、played_record、game_feedback_overlay、positive_reference_index、negative_reference_index。
10. 查询 public.user_scenario_items 用户场景状态，条件为 user_key + namespace + scenario_code。
11. 如果存在 legacy_imported_status，必要时作为历史状态暂存参考读取，但不能直接当作当前场景结论。
12. 合并当前对话反馈。
13. 涉及当前事实时联网核查。
14. 完成候选审计后才能输出推荐、等待、待查、排除或低优先结论。
```

## played_record 在推荐流程中的用途

`played_record` 是候选审计的必读输入，不是附属备注。

读取位置：

```text
public.user_preference_items
where user_key = <user_key>
  and namespace = 'game_filter'
  and item_type = 'played_record'
```

特定游戏查询时，必须同时用以下字段匹配：

```text
item_key = played:<game_key>
title ilike <game_name>
payload->>'game_key'
payload->>'game_name'
payload::text ilike <game_name_or_alias>
```

候选审计时必须判断：

```text
是否已玩。
是否正在玩。
是否已通关或全收集。
是否已退款。
是否已放弃或搁置。
是否是强正面参考。
是否是强负面参考。
是否只是 SteamDB / legacy 推断且需要用户确认。
该游玩记录对当前 scenario_code 是阻断、降权、参考、等待复查，还是可重新推荐。
```

## played_record 合并规则

```text
当前对话明确反馈 > 当前场景状态 > played_record 明确用户游玩记录 > 用户稳定偏好/反馈覆盖 > 共享游戏画像 > 外部当前事实。
```

其中：

```text
source_confidence = user_firsthand_explicit 或 user_firsthand_memory：可直接影响推荐、排除、降权、参考。
source_confidence = imported_legacy_workdoc：可作为有效历史记录，但遇到冲突时需要回看上下文或要求用户确认。
source_confidence = assistant_inferred_needs_confirmation：不能作为硬阻断，只能提示需要确认；除非还有其他明确负面证据。
```

## played_record 对候选状态的默认影响

```text
completed / fully_completed：默认不作为“新推荐”；可作为 reference_only、waiting_recheck 或回坑候选。
currently_playing：默认作为 active_baseline 或 current_positive_observation，不重复推荐为新坑。
refunded：默认 block recommendation，除非用户明确说明退款原因已解除。
abandoned：默认降权或排除，必须说明 revisit_condition 才能重新进入候选。
tried_negative：默认强降权或排除，外部好评不能单独解除。
strong_positive_reference：作为偏好基准，不等于当前应推荐同一游戏。
strong_negative_reference：作为同类风险基准，命中相似机制时必须审计。
reference_only：只作偏好或机制参考，不作推荐位。
played：必须结合 notes、positive_points、negative_points、related_scenarios 判断。
```

## 写回规则

当用户在当前对话中提供新的游玩反馈、退款、通关、放弃、回坑、强正面或强负面体验时，优先写入：

```text
public.user_preference_items
item_type = played_record
item_key = played:<game_key>
```

当反馈是跨场景稳定偏好时，同时或另写：

```text
public.user_preference_items
item_type = stable_preference 或 game_feedback_overlay
```

当反馈只影响某个场景下的状态时，写入：

```text
public.user_scenario_items
user_key + namespace + scenario_code + game_key
```

写回后必须记录：

```text
play_status
source_confidence
last_updated_jst
evidence_source
notes 或 structured payload
```

不得把个人游玩记录写入 `public.memory_items`。
