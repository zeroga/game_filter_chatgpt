# Save Flow

本文件定义新 profile、新 scenario、用户反馈、游玩记录和规则变更的保存方法。

## 保存流程

“保存 / 存档 / 写入 / 同步 / 记录下来”表示用户要求进入保存流程，不表示立即写入。

保存流程固定为三步：

```text
1. 汇总待保存内容。
2. 输出保存摘要并等待用户确认。
3. 用户确认后才执行写入。
```

保存摘要只需要包括：

```text
内容摘要
影响范围
```

如果用户只说“保存”或“存档”，但没有限定保存对象，必须汇总当前对话中所有待保存内容。

如果用户明确限定保存对象，只汇总该限定范围内的待保存内容。

用户确认前，不能写入 GitHub、Supabase 或其他外部系统。

只有用户在保存摘要之后明确回复“确认 / 确认写入 / 确认保存 / 按摘要写入”等确认表达，才允许执行写入。

如果当前对话中已经形成了应保存的规则、结论、游戏记录、场景状态或流程变更，但用户还没有明确要求存档，必须提示：

```text
有待写入内容未存档。
```

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

## 用户反馈和游玩记录保存方法

用户提供新的游玩、通关、退款、放弃、回坑、强正面或强负面体验时，必须先执行：

```text
memory/feedback_intake.md
```

完成对象核对和数据层拆分后，再按拆分结果写入或更新。

用户游玩记录写入或更新：

```text
public.user_preference_items
item_type = played_record
item_key = played:<game_key>
```

用户提供跨场景稳定偏好或反馈覆盖时，写入或更新：

```text
public.user_preference_items
item_type = stable_preference 或 game_feedback_overlay
```

用户提供只影响某个场景的结论时，写入或更新：

```text
public.user_scenario_items
user_key + namespace + scenario_code + game_key
```

可公共化的游戏结构、机制风险、版本状态、联机结构、终局问题等，必须经过联网核对后，才允许写入共享游戏资料层：

```text
public.memory_items
```

以上写入都属于存档，必须遵守本文件的保存流程。

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
不在用户没有明确要求存档时写 GitHub 或 Supabase。
不把用户个人体验未经核对直接写入 public.memory_items。
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
9. 收到当前对话游戏反馈时，立即执行 feedback_intake。
```
