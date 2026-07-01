# u_123 / pc_torchlight_like_arpg Scenario Snapshot

更新日期：2026-07-01（JST）

本文件是 GitHub 可读快照，不是状态真源。

状态真源：

```text
public.user_scenario_items
where user_key = 'u_123'
  and namespace = 'game_filter'
  and scenario_code = 'pc_torchlight_like_arpg'
```

用户偏好与游玩记录真源：

```text
public.user_preference_items
where user_key = 'u_123'
  and namespace = 'game_filter'
```

## 场景身份

```text
user_key: u_123
scenario_code: pc_torchlight_like_arpg
```

## 当前场景口径

```text
查找 PC 上类似《火炬之光》的动作 RPG / 刷装 ARPG。
```

## 已确认字段

```text
平台：PC
类型范围：类火炬之光、俯视角、刷装、Build 型 ARPG
赛季口径：排除必须追赛季、强制重开、非赛季体验残缺的游戏；不自动排除永恒服 / 标准服 / 非赛季服体验可接受的游戏。
其他条件：用户当前表示无所谓。
```

## 可后续提示但不阻断

```text
是否要求联机
是否排除 MMO / 强在线服务
是否要求长期终局
画面 / 操作排除项
```

以上字段未限定时不参与筛选，不阻断推荐或保存，不由模型自行补齐。

## 本轮关键用户反馈

### Diablo 4 / 暗黑破坏神 4

```text
用户有玩 Diablo 4。
虽然 Diablo 4 是赛季制游戏，但永恒服体验可以接受。
因此 Diablo 4 不作为“赛季制绝对排除”的负例。
```

### 赛季制偏好修正

```text
用户反感的是：
- 必须追赛季
- 必须重开
- 不玩赛季就明显损失主要内容体验
- 非赛季 / 永恒服 / 标准服体验残缺

不自动排除：
- 有赛季，但非赛季 / 永恒服 / 标准服体验可接受的游戏。
```

## 本轮推荐参考

```text
Grim Dawn：优先推荐。
Chronicon：优先推荐，前提是接受像素风。
The Slormancer：候补推荐。
Titan Quest II：观察，当前不急开。

Diablo 4：已玩 / 基准参考，不作为新推荐。
Path of Exile、Torchlight: Infinite、Last Epoch 等后续不能仅因存在 season / league / cycle 自动排除，必须核查非赛季模式体验是否足够完整。
```

## 后续执行要求

```text
1. 推荐前必须读取 u_123 的 user_preference_items，包括 played_record、stable_preference、game_feedback_overlay。
2. 推荐前必须读取该 scenario_code 下的 user_scenario_items。
3. 未限定字段不参与筛选。
4. 发现游戏存在赛季 / league / cycle 时，必须进一步核查非赛季 / 标准服 / 永恒服是否能长期正常游玩。
5. 收到新的游戏反馈时，先执行 rules/feedback_intake.md。
```
