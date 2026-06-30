# Save Flow

本文件定义新 profile 和新 scenario 的保存方法。

## 基本原则

```text
没有全局用户。
没有全局场景。
只有 profile code。
只有属于某个 user_key 的实际场景。
scenario_types 只是类型模板，不是场景。
```

## 新 profile 保存方法

用户提供 profile code 后，必须先回显并等待确认。

确认后：

```text
code_norm = lower(trim(profile_code))
user_key = u_<code_norm>
```

写入 Supabase：

```text
public.memory_users
public.profile_aliases
```

其中：

```text
profile_aliases.alias_norm = code_norm
profile_aliases.user_key = user_key
```

可选写入 GitHub 导航页：

```text
memory/profiles/<user_key>/index.md
```

GitHub profile index 只是导航，不是身份映射真源。

身份映射真源永远是：

```text
public.profile_aliases
```

## 新 scenario 保存方法

新 scenario 必须先属于某个已确认 user_key。

保存前必须确认：

```text
profile code
scenario code
```

如果 scenario_code 没有类型模板，可新增：

```text
memory/scenario_types/<scenario_code>.md
```

注意：这是类型模板，不是实际场景。

实际场景状态写入 Supabase：

```text
public.user_scenario_items
```

唯一键：

```text
user_key + namespace + scenario_code + game_key
```

可选导出 GitHub 快照：

```text
memory/profiles/<user_key>/scenarios/<scenario_code>.md
```

GitHub 场景快照只是可读快照，不是状态真源。

状态真源永远是：

```text
public.user_scenario_items
```

## 禁止保存方式

```text
不要创建全局场景。
不要把用户实际场景写到 memory/scenario_types/。
不要把用户实际场景写到 memory_items。
不要用 memory/profiles/<user_key>/scenarios/*.md 代替 public.user_scenario_items。
不要因为某个用户只有一个场景就自动选择该场景。
```

## 推荐读取方式

```text
1. 确认 profile code。
2. 确认 scenario code。
3. profile_aliases 查 user_key。
4. scenario_types 读取类型模板。
5. user_preference_items 读取用户偏好。
6. user_scenario_items 读取实际场景状态。
7. memory_items 读取共享游戏资料。
8. GitHub profile scenario 只作快照参考，不作为状态真源。
```
