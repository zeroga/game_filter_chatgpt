# Scenario Template Index

本目录只保存场景模板，不保存任何用户的实际场景状态。

游戏筛选中的“场景”本质上是用户偏好的一部分；同一个场景代码在不同用户下可以有不同推荐、待查、等待、排除和基准线。

## 模板列表

```text
scenario_code: pc_console_coop
file: memory/scenarios/pc_console_coop.md
name: PC / 主机固定队联机游戏筛选
role: global_template_only
```

## 关键规则

```text
不要在 memory/scenarios/*.md 中保存用户实际推荐清单。
不要在 memory/scenarios/*.md 中保存用户实际 waiting 队列。
不要在 memory/scenarios/*.md 中保存用户实际排除清单。
不要把某个用户的场景结论写成全局场景结论。
```

## 实际个人场景存储

个人场景快照保存在：

```text
memory/profiles/<user_key>/scenarios/<scenario_code>.md
```

例如：

```text
memory/profiles/owner_zhengkun/scenarios/pc_console_coop.md
```

数据库中的个人场景状态保存在：

```text
public.user_scenario_items
```

唯一键逻辑：

```text
user_key + namespace + scenario_code + game_key
```

## 共享游戏资料

跨用户共享的游戏画像仍在：

```text
public.memory_items
```

## 场景模板职责

每个 `memory/scenarios/*.md` 只能保存：

```text
scenario_code
scenario_name
scope_template
hard_condition_template
candidate_state_enum
audit_field_template
waiting_recheck_template
storage_policy_template
```

## 新增场景模板

新增一种游戏筛选场景时：

```text
1. 在 memory/scenarios/ 下新增模板文件。
2. 更新本 index。
3. 不写入任何个人推荐或等待清单。
4. 用户实际场景放到 memory/profiles/<user_key>/scenarios/ 或 public.user_scenario_items。
```
