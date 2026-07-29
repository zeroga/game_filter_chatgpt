# Recommendation Entry Rule

普通交互式游戏推荐没有默认用户，也没有默认场景。每个新对话都必须重新获取本轮 `profile code` 和 `scenario code`。已确认创建并启用的 `weekly_game_report` Automation 定时只读生成例外由 `rules/reporting/weekly_report.md` 管理，不得用于交互式推荐。

在任何普通交互式推荐、筛选、更新清单、解释场景状态、读取用户偏好、读取用户游玩记录、读取用户场景状态、候选审计、等待项复查、排除 / 低优先 / 推荐状态判断之前，必须先确认两个值：

```text
profile code
scenario code
```

## profile code 确认规则

如果缺 profile code，必须停止推荐、筛选、更新清单和读取用户层数据，并先问：

```text
请先给我你的 profile code。大小写不敏感，只用于选择你的个人偏好层。
```

拿到 profile code 后，不能直接创建新用户。必须先执行：

```text
rules/routing/profile_routing.md
```

最低要求：

```text
1. code_norm = lower(trim(profile_code_text))。
2. 查询 public.profile_aliases.alias_norm = code_norm。
3. 如果命中，回显 profile code、alias_norm、user_key，等待用户确认。
4. 如果未命中，先提示可能相近或容易混淆的已有账户，询问是不是其中之一。
5. 只有用户明确确认不是已有账户，并确认要创建新 profile，才允许新建。
```

拿到最终 user_key 后，必须回显确认：

```text
本次使用的 profile code 是 <profile_code>，alias_norm = <alias_norm>，user_key = <user_key>。请确认。
```

用户确认前，不能读取或写入该 profile 的个人偏好层。

## 周报两阶段预检

profile 确认后、要求 scenario code 之前，允许且必须按 `rules/reporting/weekly_report.md` 查询该 `user_key` 的 profile 级周报配置。该查询不需要先确认单独的 `scenario_code`。

- 无配置：提示“当前未配置周报”。
- 有配置：只展示启用或暂停状态、推送时间和时区、详细内容范围、新游雷达设置、资讯类别、下次计划运行时间，以及配置与实际定时任务是否同步。尚未确认 scenario code 时不得判断当前场景范围。
- 查询失败：显示降级提示并继续正常推荐，不得把周报配置查询作为推荐阻断项。
- scenario 确认后，再单独展示当前 `scenario_code` 是否命中详细搜索范围，不重复未变化的总体配置。
- 同一对话中同一阶段已展示且配置未变化时，不重复展示。

## scenario code 确认规则

scenario code 的详细确认、查找、自然语言映射和新场景创建规则由以下文件统一管理：

```text
rules/routing/scenario_routing.md
```

如果缺 scenario code，必须停止推荐、筛选、更新清单和读取场景状态，并继续问：

```text
请提供本轮要使用的 scenario code。如果不记得 code，可以让我列出现有场景供你选择。没有明确场景时不能开始游戏推荐。
```

用户确认前，不能读取或写入该场景的个人状态。

## 禁止行为

```text
不能默认使用任何 profile code。
不能默认使用任何 scenario code。
不能把 profile_aliases 中唯一或看似最相关的 alias 当作默认用户。
不能基于用户昵称、当前账号名、历史 user_key、上一次对话的 profile code、唯一 profile 或当前对话外记忆推断，主动建议某个具体 profile code。
不能因为某个 profile 只有一个已知场景就自动使用该场景。
不能把 memory/scenario_types/index.md 的类型列表当作默认场景。
不能把 memory/profiles/<user_key>/scenarios/ 下已有文件当作本轮默认选择。
不能把某个 profile 下唯一 scenario、上一次对话的 scenario code、场景类型模板、用户历史推荐场景、Supabase 中唯一 scenario_code 或当前对话外记忆推断，当作本轮默认场景。
不能把上一次对话的 profile code 或 scenario code 自动带入新对话。
不能只查 public.memory_items 后就给推荐。
不能忽略 played_record 中的已玩、退款、放弃、强正面/强负面参考。
不能因为 alias 未精确命中就直接创建新用户。
不能不做二次确认就创建新 profile 或新 scenario。
不能在收到用户游戏反馈后跳过 rules/feedback_intake.md。
```

## 允许行为

```text
可以询问用户本轮 profile code。
可以解释 profile code 用途。
可以在用户明确要求“列出现有场景供我选择”后，列出该 profile 已有场景供用户选择。
可以解释用户已要求列出的 scenario code 含义。
可以在新用户/新场景创建前提示“是不是某个已有账户/场景”。
用户明确说“继续本轮刚确认过的场景”时，只有当前对话中已经确认过 profile code 和 scenario code，才可沿用。
```

## 推荐读取顺序

```text
1. 询问 profile code。
2. profile_routing 查询并确认 user_key。
3. 回显 profile code、alias_norm、user_key 并等待用户确认。
4. 执行 scenario_routing，确认 scenario code。
5. 读取 memory/scenario_types/<scenario_code>.md 作为场景类型模板。
6. 读取 memory/profiles/<user_key>/scenarios/<scenario_code>.md 作为个人场景快照；该文件只作快照参考，不作为状态真源。
7. 查询 public.memory_items 共享游戏画像。
8. 查询 public.user_preference_items 用户偏好层，至少包括 stable_preference、played_record、game_feedback_overlay、positive_reference_index、negative_reference_index。
9. 查询 public.user_scenario_items 用户场景状态，条件为 user_key + namespace + scenario_code。
10. 如果存在 legacy_imported_status，必要时作为历史状态暂存参考读取，但不能直接当作当前场景结论。
11. 如果当前输入包含游戏反馈，立即执行 rules/feedback_intake.md。
12. 合并当前对话反馈。
13. 涉及当前事实时联网核查；如果第 11 步已经核查过，不得重复制造相反结论。
14. 完成候选审计后才能输出推荐、等待、待查、排除或低优先结论。
```

`rules/feedback_intake.md` 是用户游戏反馈即时理解机制的唯一详细真源；本文件只保留推荐流程中的触发点。

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

当用户在当前对话中提供新的游玩反馈、退款、通关、放弃、回坑、强正面或强负面体验时，先执行：

```text
rules/feedback_intake.md
```

然后按拆分结果写入。

用户游玩记录优先写入：

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
数据库字段：state
```

可公共化的结构事实或机制风险，必须经过联网核对后才写入：

```text
public.memory_items
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
