# Profile Routing

进入任何场景说明、场景推荐、筛选、更新清单或解释场景状态前，先询问 profile code。

提示语：

```text
请先给我你的 profile code。大小写不敏感，只用于选择你的个人偏好层。
```

## 新对话 profile code 边界

每个新对话中，只要任务涉及推荐、筛选、更新候选清单、解释场景状态、读取用户偏好、读取用户游玩记录、读取用户场景状态、候选审计、等待项复查、排除 / 低优先 / 推荐状态判断，就必须由用户主动提供本轮 `profile code`。

ChatGPT 可以询问“请先给我本轮要使用的 profile code”，也可以解释 profile code 用于选择个人偏好层；但不得基于已知 alias / user_key 映射、用户昵称、当前账号名、历史 user_key、上一次对话的 profile code、唯一 profile 或当前对话外的记忆推断，主动代替用户选择或建议某个具体 profile code。

已知 alias / user_key 映射只能在用户输入 profile code 后用于匹配、回显和确认，不能用于主动发起选择。用户确认前，不得读取或写入该用户层数据。

## 核心原则

```text
profile code 是用户可输入的短代号。
user_key 是系统内部键。
两者不能混用。
```

身份映射真源永远是：

```text
public.profile_aliases
```

映射关系：

```text
profile_aliases.alias_norm -> user_key
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

## 基本规范化

收到用户输入后，先做最小规范化：

```text
code_norm = lower(trim(profile_code_text))
```

不要设计复杂解析规则。profile code 的主规则是“短、明确、可回显确认”。

## 查找流程

```text
1. 查询 public.profile_aliases.alias_norm = code_norm。
2. 如果命中，回显 profile code、alias_norm、user_key，等待用户确认。
3. 用户确认后，才能读取或写入该 user_key 的用户层数据。
   用户确认后也允许读取该 `user_key` 在 `public.user_report_subscriptions` 中的 profile 级周报配置，不需要先确认 scenario code；配置处理遵守 `rules/reporting/weekly_report.md`。
4. 如果未命中，不能立刻创建新用户。
5. 先列出可能相近或容易混淆的已知 alias / user_key，询问用户是不是其中之一。
6. 只有用户明确确认“不是已有用户，要创建新 profile”，才进入新 profile 创建流程。
```

相近提示不需要复杂算法。可用人工可解释的启发式：

```text
- 输入只差“用户”“profile”“code”等自然语言词。
- 输入看起来像已有 user_key，例如 u_123 与 123。
- 输入包含已有 alias 的主体数字或主体英文。
- 用户表达为“我是用户123”时，可提示“是不是 profile code 123？”
```

示例：

```text
用户输入：用户123
已存在 alias：123 -> u_123
应提示：你是不是指 profile code 123 / user_key u_123？确认后我会使用这个账户。
```

## 新 profile 创建规则

只有在以下条件都满足时，才创建新 profile：

```text
1. profile_aliases 精确查询未命中。
2. 已提示可能的相近账户。
3. 用户明确确认不是已有账户。
4. 用户确认要创建的新 profile code。
```

创建前必须回显：

```text
没有命中已有 profile。
将创建新 profile code: <code_norm>
将创建 user_key: u_<code_norm>
请确认。
```

用户确认后才可写入：

```text
public.memory_users
public.profile_aliases
```

其中：

```text
profile_aliases.alias_norm = code_norm
profile_aliases.user_key = u_<code_norm>
```

## 禁止行为

```text
不要因为 alias 精确未命中就直接创建新用户。
不要把用户口语中的“用户123”直接当成新 profile。
不要把内部 user_key 当成新的 profile code 再包一层。
不要创建新 profile 而不做二次确认。
不要在用户未确认前读取或写入该 profile 的用户层数据。
```
