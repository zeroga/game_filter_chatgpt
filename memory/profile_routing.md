# Profile Routing

进入任何场景说明、场景推荐、筛选、更新清单或解释场景状态前，先询问 profile code。

提示语：

```text
请先给我你的 profile code。大小写不敏感，只用于选择你的个人偏好层。
```

## 核心原则

```text
profile code 是用户可输入的短代号。
user_key 是系统内部键。
两者不能混用。
```

典型内部键格式：

```text
u_<profile_code>
```

例如：

```text
profile code: 123
user_key: u_123
```

如果用户说“用户123”“我是用户123”“profile 123”“code 123”，语义上的 profile code 仍然是：

```text
123
```

不能把“用户123”创建为新用户，也不能把 `u_123` 再创建成 `u_u_123`。

## profile code 规范化规则

收到用户输入后，必须先生成候选 alias，再查 `public.profile_aliases`。

规范化步骤：

```text
raw_input = trim(profile_code_text)
code_norm = lower(trim(raw_input))
```

然后执行自然语言前缀剥离：

```text
如果 code_norm 形如：用户<ascii_code>
则 profile_alias_candidate = <ascii_code>

如果 code_norm 形如：user<ascii_code>
则 profile_alias_candidate = <ascii_code>

如果 code_norm 形如：profile<ascii_code>
则 profile_alias_candidate = <ascii_code>

如果 code_norm 形如：code<ascii_code>
则 profile_alias_candidate = <ascii_code>
```

其中 `<ascii_code>` 只能包含：

```text
[a-z0-9_]
```

示例：

```text
用户123 -> 123
user123 -> 123
profile_abc -> abc
code abc -> abc
```

## user_key 防重复规则

如果用户输入形如：

```text
u_<alias>
```

必须先把它识别为可能的内部 `user_key`，而不是新 profile code。

处理顺序：

```text
1. 先查 public.memory_users.user_key = code_norm。
2. 如果存在该 user_key，则使用该 user_key，并提示这是内部 user_key 命中。
3. 如果不存在，再取 alias_candidate = 去掉前缀 u_ 的部分。
4. 查 public.profile_aliases.alias_norm = alias_candidate。
5. 如果 alias_candidate 存在，则使用其 user_key。
6. 如果两者都不存在，不得自动创建 u_u_<alias>，必须要求用户确认真实 profile code。
```

禁止自动创建：

```text
user_key = u_u_<alias>
```

除非用户明确说明自己的 profile code 本身就是 `u_<alias>`，并且经过回显确认。

## 查表顺序

身份映射真源：

```text
public.profile_aliases
```

查询顺序：

```text
1. 查询 profile_aliases.alias_norm = code_norm。
2. 查询 profile_aliases.alias_norm = profile_alias_candidate。
3. 若输入形如 u_<alias>，按 user_key 防重复规则处理。
4. 如果多个候选命中不同 user_key，停止并要求用户确认，不得自动选择。
5. 如果没有命中，才进入新 profile 创建流程。
```

映射关系：

```text
alias_norm -> user_key
```

已知初始映射：

```text
zhengkun -> owner_zhengkun
zero -> owner_zhengkun
```

后续所有个人偏好和个人场景查询使用 user_key：

```text
public.user_preference_items
public.user_scenario_items
```

共享游戏资料仍使用：

```text
public.memory_items
```

## 新 profile 创建前检查

如果所有 alias 都不存在，创建新 profile 前必须检查：

```text
code_norm 不得以 u_ 开头。
code_norm 不得包含“用户”“profile”“code”等自然语言标签。
code_norm 只允许短英文、数字、下划线。
code_norm 必须回显给用户确认。
```

创建规则：

```text
user_key = u_<code_norm>
profile_aliases.alias_norm = code_norm
profile_aliases.user_key = user_key
```

复杂代号或疑似自然语言描述，先让用户换一个短 code，不自动建档。
