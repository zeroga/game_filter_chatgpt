# Scenario Type Index

本目录只保存“场景类型模板”，不保存任何实际游戏场景。

游戏筛选里的实际场景一定属于某个 profile。

## 类型列表

```text
scenario_code: pc_console_coop
file: memory/scenario_types/pc_console_coop.md
name: PC / 主机固定队联机游戏筛选
type_role: scenario_type_template
```

## 关键规则

```text
这里没有全局场景。
这里没有默认场景。
这里不保存任何用户的推荐清单。
这里不保存任何用户的 waiting 队列。
这里不保存任何用户的排除清单。
```

## 实际场景位置

实际场景快照必须挂在 profile 下：

```text
memory/profiles/<user_key>/scenarios/<scenario_code>.md
```

例如：

```text
memory/profiles/owner_zhengkun/scenarios/pc_console_coop.md
```

数据库里的实际场景状态保存到：

```text
public.user_scenario_items
```

唯一键：

```text
user_key + namespace + scenario_code + game_key
```

## 场景类型模板职责

每个 `memory/scenario_types/*.md` 只能保存：

```text
scenario_code
scenario_type_name
scope_template
condition_template
candidate_state_enum
audit_field_template
waiting_recheck_template
storage_policy_template
```
