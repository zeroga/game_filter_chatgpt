# game_filter_chatgpt

固定入口：

```text
memory/recommendation_entry.md
memory/save_flow.md
memory/profile_routing.md
memory/database_positioning.md
memory/current_rules.md
memory/multi_user.md
```

推荐前必须确认：

```text
profile code
scenario code
```

没有默认用户，也没有默认场景。

保存新用户或新场景时，先读：

```text
memory/save_flow.md
```

场景类型模板：

```text
memory/scenario_types/
```

实际场景快照：

```text
memory/profiles/<user_key>/scenarios/
```

实际状态真源：

```text
public.user_scenario_items
```

旧目录 `memory/scenarios/` 已废弃。
