# Scenario Index

本目录按场景分开保存规则。不要再使用单一 `current_scenario` 表示所有场景。

## 默认场景

```text
scenario_code: pc_console_coop
file: memory/scenarios/pc_console_coop.md
name: PC / 主机固定队联机游戏筛选
status: active
```

## 场景文件职责

每个场景文件只保存该场景的规则和状态结构，不保存完整游戏画像。

场景文件应包含：

```text
scenario_code
scenario_name
scope
hard_conditions
user_preference_overlay
candidate_states
candidate_audit_extra_fields
waiting_recheck_policy
scenario_storage_policy
```

## 数据分工

```text
GitHub memory/scenarios/*.md = 场景规则、读取说明、状态字段定义
Supabase memory_items item_type=scenario_entry = 单个游戏在某场景下的状态
Supabase memory_items item_type=structured_profile_record = 跨场景游戏画像
```

## 场景条目 key 规范

```text
<scenario_code>:<game_key>
```

示例：

```text
pc_console_coop:warframe
mobile_non_action:p5x
space_ship_customization:x4_foundations
```

## 后续新增场景

新增场景时：

```text
1. 在 memory/scenarios/ 下新增独立场景文件。
2. 更新本 index。
3. 不修改其他场景的硬规则，除非用户明确要求跨场景规则变化。
4. 场景下的具体游戏状态写入 Supabase scenario_entry。
```
