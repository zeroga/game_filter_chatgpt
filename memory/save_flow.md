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

用户提供 profile code 后，必须先按 `memory/profile_routing.md` 执行规范化和查重。

### 1. 规范化

```text
raw_input = trim(profile_code_text)
code_norm = lower(trim(raw_input))
```

自然语言前缀必须剥离：

```text
用户123 -> 123
user123 -> 123
profile123 -> 123
code123 -> 123
```

如果输入形如：

```text
u_<alias>
```

必须先识别为可能的内部 user_key 或已存在 profile 的派生写法，不能直接创建：

```text
user_key = u_u_<alias>
```

### 2. 查重

创建前必须查询：

```text
public.profile_aliases
public.memory_users
```

查重顺序：

```text
1. profile_aliases.alias_norm = code_norm
2. profile_aliases.alias_norm = prefix_stripped_candidate
3. 如果 code_norm 形如 u_<alias>，查 memory_users.user_key = code_norm
4. 如果 code_norm 形如 u_<alias>，再查 profile_aliases.alias_norm = <alias>
```

只要任何一步命中已有 user_key，就使用已有 user_key，不创建新 profile。

如果不同候选命中不同 user_key，停止并要求用户确认，不能自动保存。

### 3. 创建前回显

只有在所有候选均未命中时，才允许进入新 profile 创建。创建前必须回显：

```text
原始输入: <raw_input>
规范化 profile code: <code_norm>
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

## 禁止的新 profile 创建方式

```text
不要把“用户123”创建为 u_用户123 或 u_u_123。
不要把“user123”创建为 u_user123，除非用户明确确认 profile code 就是 user123。
不要把内部 user_key 当成 profile code 再包一层。
不要在 alias 查重前创建新用户。
不要因为 profile_aliases 没有精确命中 raw_input 就直接创建新用户。
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
不创建全局用户。
不创建全局场景。
不把用户实际场景写到 memory/scenario_types/。
不把用户实际场景写到 memory_items。
不用 memory/profiles/<user_key>/scenarios/*.md 代替 public.user_scenario_items。
不因为某个用户只有一个场景就自动选择该场景。
不因自然语言称呼差异创建重复用户。
```

## 推荐读取方式

```text
1. 确认 profile code。
2. 按 profile_routing 规范化 profile code 并查重。
3. profile_aliases 查 user_key。
4. 确认 scenario code。
5. scenario_types 读取类型模板。
6. user_preference_items 读取用户偏好和 played_record。
7. user_scenario_items 读取实际场景状态。
8. memory_items 读取共享游戏资料。
9. GitHub profile scenario 只作快照参考，不作为状态真源。
```
