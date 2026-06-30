# Scenario Type: PC / 主机固定队联机游戏筛选

```text
scenario_code: pc_console_coop
type_role: scenario_type_template
```

## 类型定位

这是一个场景类型模板，不是实际场景。

实际场景必须属于某个 profile，例如：

```text
memory/profiles/owner_zhengkun/scenarios/pc_console_coop.md
```

## 范围模板

用于筛选适合固定队长期玩的 PC / 主机合作游戏。

## 条件模板

```text
优先官方联机，不依赖 Mod 联机。
优先 PvE 合作。
强制 PvP 或 PvP 主轴默认排除。
Early Access 默认等待，不进推荐。
必须确认客机或队友是否有稳定进度、奖励、解锁和成长。
必须确认联机是否是主循环，不只是附属模式。
必须确认少人固定队体验是否成立。
```

## 个人偏好来源

实际偏好不写在本模板中，应来自：

```text
public.user_preference_items
memory/profiles/<user_key>/scenarios/pc_console_coop.md
public.user_scenario_items
```

## 候选状态枚举

```text
recommended
recommended_trial_only
investigate
waiting
low_priority
excluded
active_baseline
reference_only
empty_slot
unknown
```

## 推荐前审计模板

```text
共享游戏数据库是否已有结论。
共享负面索引是否命中。
当前用户在该场景是否已有旧结论。
是否与当前用户已有反馈冲突。
当前发售状态、EA 状态、DLC、价格、近期评价是否变化。
官方联机结构是否符合固定队 PvE。
为什么不是 waiting。
为什么不是 investigate。
为什么不是 low_priority。
为什么不是 excluded。
```

## waiting 复查模板

```text
是否正式版或仍是 EA。
是否已有 DLC、资料片、赛季或大型内容变化。
是否有折扣、试玩、Demo 或打包版本变化。
近期多语言玩家评价是否改变。
联机结构和客机进度是否改变。
原先命中的用户风险点是否解除。
```

若没有全部查完，必须标记：

```text
waiting_info_update_incomplete
```

## 实际状态存储

```text
public.user_scenario_items
```

唯一键：

```text
user_key + namespace + scenario_code + game_key
```
