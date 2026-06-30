# Profile Routing

进入任何场景说明或场景推荐前，先询问 profile code。

提示语：

```text
请先给我你的 profile code。大小写不敏感，只用于选择你的个人偏好层。
```

处理规则：

```text
code_norm = lower(trim(profile_code))
```

查表：

```text
public.profile_aliases
```

映射关系：

```text
code_norm -> user_key
```

已知初始映射：

```text
zhengkun -> owner_zhengkun
```

后续所有个人偏好查询使用 user_key：

```text
public.user_preference_items
public.user_scenario_items
```

共享游戏资料仍使用：

```text
public.memory_items
```

如果 code_norm 不存在，创建新 profile：

```text
user_key = u_<code_norm>
```

要求 code_norm 只用短英文、数字、下划线。复杂代号先让用户换一个。
