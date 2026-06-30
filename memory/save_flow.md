# Save Flow

本文件定义新 profile 和新 scenario 的保存方法。

## 基本原则

```text
没有全局用户。
没有全局场景。
只有 profile code。
只有属于某个 user_key 的实际场景。
scenario_types 只是类型模板，不是场景。
profile code 是用户可输入代号；user_key 是内部键；两者不能混用。
```

## 新 profile 保存方法

用户提供 profile code 后，必须先按 `memory/profile_routing.md` 执行最小规范化和查找。

基本规范化：

```text
code_norm = lower(trim(profile_code_text))
```

创建前必须查询：

```text
public.profile_aliases
```

如果 alias 精确命中，使用已有 user_key，不创建新 profile。

如果 alias 未命中，不能直接创建。必须先提示可能相近或容易混淆的已有账户，例如：

```text
你说的是不是 profile code 123 / user_key u_123？
```

只有用户明确确认不是已有账户，并确认要创建新 profile，才允许创建。

创建前必须回显：

```text
没有命中已有 profile。
将创建新 profile code: <code_norm>
将创建 user_key: u_<code_norm>
请确认。
```

用户确认后：

```text
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

如果 scenario_code 没有类型模板，不能直接创建。必须先列出已有 scenario_types 或相近场景，并提示用户是不是其中之一。

只有用户明确确认不是已有场景，并确认要创建新 scenario_code，才允许新增：

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
不创建全局用户。
不创建全局场景。
不把用户实际场景写到 memory/scenario_types/。
不把用户实际场景写到 memory_items。
不用 memory/profiles/<user_key>/scenarios/*.md 代替 public.user_scenario_items。
不因为某个用户只有一个场景就自动选择该场景。
不因自然语言称呼差异直接创建重复用户。
不因场景名称不完全一致直接创建重复场景。
不做二次确认就创建新 profile 或新 scenario。
```

## 推荐读取方式

```text
1. 确认 profile code。
2. profile_routing 查询并确认 user_key。
3. 确认 scenario code。
4. scenario_types 读取类型模板。
5. user_preference_items 读取用户偏好和 played_record。
6. user_scenario_items 读取实际场景状态。
7. memory_items 读取共享游戏资料。
8. GitHub profile scenario 只作快照参考，不作为状态真源。
```
