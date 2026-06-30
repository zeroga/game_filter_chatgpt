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

存档规则允许在 README、`memory/current_rules.md`、本文件中保留安全冗余。该冗余用于防止精读规则后误写入 GitHub / Supabase。

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

如果 alias 精确命中，使用已有 user_key，不创建新 profile。

如果 alias 未命中，不能直接创建。必须先提示可能相近或容易混淆的已有账户。

只有用户明确确认不是已有账户，并确认要创建新 profile，才允许创建。

创建后身份映射真源写入：

```text
public.memory_users
public.profile_aliases
```

可选写入 GitHub 导航页：

```text
memory/profiles/<user_key>/index.md
```

GitHub profile index 只是导航，不是身份映射真源。

## 新 scenario 保存方法

新 scenario 必须先属于某个已确认 user_key。

保存前必须确认：

```text
profile code
scenario code
```

新 scenario 的确认、已有场景查找、自然语言映射、对象模型和创建前回显由以下文件统一管理：

```text
memory/scenario_routing.md
```

如果 scenario_code 没有类型模板，不能直接创建。必须先列出已有 scenario_types、用户场景快照或相近场景，并提示用户是不是其中之一。

只有用户明确确认不是已有场景，并确认要创建新 scenario_code，才允许新增：

```text
memory/scenario_types/<scenario_code>.md
```

实际场景状态真源写入 Supabase：

```text
public.user_scenario_items
唯一键：user_key + namespace + scenario_code + game_key
数据库字段：state
```

可选导出 GitHub 快照：

```text
memory/profiles/<user_key>/scenarios/<scenario_code>.md
```

GitHub 场景快照只是可读快照，不是状态真源。

## 场景相关存档完整性规则

保存内容只要涉及新 scenario_code、场景对象模型、场景口径、场景类型模板、用户场景快照、场景候选或场景状态，必须先生成分层写入清单。

分层写入清单至少覆盖：

```text
1. memory/scenario_types/<scenario_code>.md
2. memory/profiles/<user_key>/scenarios/<scenario_code>.md
3. public.user_scenario_items
4. public.user_preference_items
5. public.memory_items
6. public.memory_events
```

每一层都必须判断：

```text
需要写入 / 需要更新 / 不写入并说明原因
```

保存摘要仍只对用户展示：

```text
内容摘要
影响范围
```

但内部执行不得省略分层写入清单。保存摘要中的影响范围必须能映射到清单中的具体层。

用户确认写入后，必须按清单逐项执行，并在写入后逐项回查：

```text
GitHub 文件：fetch_file 确认文件存在且内容已更新。
Supabase 记录：select 确认目标行存在且字段和值符合写入目的。
```

只要清单中任一应写层没有写入或没有回查，不能声明保存已完成；必须明确列为未完成、失败或跳过并说明原因。

如果只写 Supabase 状态真源，但漏写应有 GitHub 场景类型模板或用户场景快照，视为场景存档不完整。

## 用户反馈和游玩记录保存方法

用户提供新的游玩、通关、退款、放弃、回坑、强正面或强负面体验时，必须先执行：

```text
memory/feedback_intake.md
```

`memory/feedback_intake.md` 是用户游戏反馈即时理解机制的唯一详细真源。本文件只记录保存前置依赖，不重复维护完整拆分流程。

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
字段：state
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
不在场景相关存档中跳过分层写入清单和写入后回查。
```

## 推荐读取方式

```text
1. 确认 profile code。
2. profile_routing 查询并确认 user_key。
3. 执行 scenario_routing 并确认 scenario code。
4. scenario_types 读取类型模板。
5. user_preference_items 读取用户偏好和 played_record。
6. user_scenario_items 读取实际场景状态。
7. memory_items 读取共享游戏资料。
8. GitHub profile scenario 只作快照参考，不作为状态真源。
9. 收到当前对话游戏反馈时，立即执行 feedback_intake。
```