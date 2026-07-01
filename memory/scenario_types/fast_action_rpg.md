# Fast Action RPG Scenario Type

scenario_code: `fast_action_rpg`

本文件是场景类型模板，不是某个用户的实际场景状态。

实际状态真源仍为：

```text
public.user_scenario_items
```

## 场景目标

筛选高速即时动作 RPG / 动作刷装 RPG / 高速角色 Build 动作游戏。

该场景优先寻找：

```text
高速即时战斗
RPG 成长、职业、装备、Build 或角色差异
正式版优先
PC / 主机优先
可长期游玩或至少有明确中后期成长目标
```

## 正向条件

```text
1. 动作节奏快，战斗输入、职业/角色差异或技能循环有实际意义。
2. 有角色成长、装备成长、Build 调整、职业系统、刷装或可持续提升目标。
3. 终盘或中后期内容不应只靠纯数值堆叠延长时间。
4. 如果主打联机，必须核查官方联机结构、平台限制、任务范围、进度/奖励同步和当前可用性。
5. 如果是单人游戏，必须说明其不能作为长期联机主坑，只能作为单人高速动作 RPG 候选。
```

## 负向条件

```text
Early Access 默认排除或等待。
浅清版动作小品默认降权。
纯日常 MMO / 纯数值刷装 / 终盘重复感强默认降权。
联机不可用、联机范围窄、进度奖励不同步或需要特殊条件才能联机时必须降权或待查。
已玩、通关、退款、弃坑或强负面反馈的游戏不得作为新推荐。
```

## 用户反馈处理

收到用户对本场景相关游戏的反馈时，必须先执行：

```text
rules/feedback_intake.md
```

顺序固定为：

```text
1. 立即联网确认游戏对象、版本、平台、机制、联机状态、近期评价和是否过时。
2. 将反馈拆分为公共画像候选、用户游玩记录、用户偏好/反馈覆盖、用户场景状态。
3. 再进入推荐、解释、更新清单、候选审计或保存流程。
```

## 当前已知参考点

```text
Granblue Fantasy: Relink
- 高速动作、角色差异、Boss 共斗、刷装结构可作正面机制参考。
- 对 u_123 已是已玩项目，不作为新推荐。

Stranger of Paradise: Final Fantasy Origin
- 可作负面参考：终盘 / 支线 / 刷装重复和乏味风险；联机可用性需要按平台、任务、进度、DLC、网络或匹配限制审计。
- 对 u_123 已是已玩负面项目，不作为新推荐。
```

## 候选审计补充字段

除通用候选审计字段外，本场景应特别检查：

```text
combat_speed_and_input_load
rpg_growth_depth
build_or_class_depth
endgame_loop_quality
endgame_repetition_risk
coop_structure_if_any
coop_progress_and_reward_sync
platform_or_region_restrictions
already_played_or_reference_status
```
