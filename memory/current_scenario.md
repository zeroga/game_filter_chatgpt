# Current Scenario

当前默认场景：PC / 主机固定队联机游戏筛选。

## 场景定位

目标不是泛游戏推荐，而是筛选适合固定队长期玩的 PC / 主机合作游戏。

## 场景硬条件

```text
优先官方联机，不依赖 Mod 联机。
优先 PvE 合作。
强制 PvP 或 PvP 主轴默认排除。
Early Access 默认等待，不进推荐。
需要确认客机或队友是否有稳定进度、奖励、解锁和成长。
需要确认联机是否是主循环，不只是附属模式。
需要确认少人固定队体验是否成立。
```

## 用户偏好约束

```text
偏科幻、机甲、舰船、装备、职业、角色、Build、长期账号成长。
排斥纯数值堆叠但 Build 浅。
排斥强反应、强眼力压力、地图迷路压力过高。
排斥恐怖氛围过重。
排斥强制共管同一基地、工厂、资源链或舰船的执政官模式。
方块或模块建造不是硬排，但要警惕审美疲劳和长期视觉问题。
```

## 场景候选状态

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

## 推荐前必须确认

```text
旧游戏数据库是否已有结论。
负面索引是否命中。
当前场景是否已有旧结论。
是否与用户已有反馈冲突。
当前发售状态、EA 状态、DLC、价格、近期评价是否变化。
官方联机结构是否符合固定队 PvE。
为什么不是 waiting。
为什么不是 investigate。
为什么不是 low_priority。
为什么不是 excluded。
```

## waiting 项复查

更新游戏清单时，waiting 项不能只沿用旧文字。必须重新核查：

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

## 场景数据存储

场景规则保存在 GitHub：

```text
memory/current_scenario.md
```

单个游戏在本场景下的状态，后续应写入 Supabase：

```text
item_type: scenario_entry
namespace: game_filter
```

建议 key：

```text
pc_console_coop:<game_key>
```

建议 payload 字段：

```text
scenario_code
canonical_name
state
reason
hard_condition_result
candidate_audit
waiting_check_status
last_checked_at
source_summary
```
