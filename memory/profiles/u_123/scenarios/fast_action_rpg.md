# u_123 / fast_action_rpg Scenario Snapshot

更新日期：2026-07-01（JST）

本文件是 GitHub 可读快照，不是状态真源。

状态真源：

```text
public.user_scenario_items
where user_key = 'u_123'
  and namespace = 'game_filter'
  and scenario_code = 'fast_action_rpg'
```

用户偏好与游玩记录真源：

```text
public.user_preference_items
where user_key = 'u_123'
  and namespace = 'game_filter'
```

## 场景口径

```text
高速即时战斗。
有 RPG 成长、职业、装备、Build 或角色差异。
优先正式版、PC / 主机。
不优先 EA、纯刷日常 MMO、浅清版动作小品。
需特别避开终盘重复乏味、联机不可用或联机结构不可靠的项目。
```

## 本轮用户反馈拆分结果

### Granblue Fantasy: Relink

Supabase state:

```text
public.user_scenario_items
state = reference_only
game_key = granblue_fantasy_relink
```

当前结论：

```text
用户已玩过。
不作为 fast_action_rpg 新推荐。
可作为高速动作 RPG、角色差异、Boss 共斗和刷装结构的正面机制参考。
```

### Stranger of Paradise: Final Fantasy Origin / FF 起源

Supabase state:

```text
public.user_scenario_items
state = excluded
game_key = stranger_of_paradise_final_fantasy_origin
```

当前结论：

```text
用户已玩过。
用户反馈：终盘战斗比较无聊，实际体验为没法联机。
在 fast_action_rpg 中不作为新推荐。
作为负面参考：终盘重复/乏味风险，联机可用性需按平台和场景核查。
```

公共画像拆分：

```text
可公共化部分：终盘 / 支线 / 刷装阶段存在重复和乏味风险。
不可直接公共化为事实的部分：不能写成“无联机”，因为公开商店信息仍标注 Online Co-op。
联机问题应记录为：官方支持联机，但实际可用性需按平台、任务、进度、DLC、网络或匹配限制审计。
```

## 当前新推荐候选状态

```text
GBF Relink：reference_only，已玩，不进新推荐。
FF 起源：excluded，已玩且负面，不进新推荐。
Nioh 3：待后续重新联网核查正式版状态、联机/发售/试玩情况、Build 深度和动作压力。
Ys X: Proud Nordics：待后续重新联网核查平台、版本差异和是否仅作为单人补位。
```

## 后续执行要求

收到新的游戏反馈时，先执行：

```text
rules/feedback_intake.md
```

不得把用户反馈整句直接塞进 played_record，也不得跳过公共画像候选、用户游玩记录、用户偏好/反馈覆盖、用户场景状态的拆分。

涉及当前事实、版本、联机状态、评价趋势、DLC 或发售状态时必须联网核查。
