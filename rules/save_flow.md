# Save Flow

本文件定义新 profile、新 scenario、用户反馈、游玩记录、记忆层快照和 Supabase 数据的保存方法。规则层变更不进入 save_flow。

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

如果当前对话中已经形成了应保存的结论、游戏记录、场景状态或流程变更需求，但用户还没有明确要求存档，必须提示：

```text
有待写入内容未存档。
```

存档安全护栏允许在 README、`rules/current_rules.md`、本文件中保留必要冗余。该冗余用于防止精读规则后误写入 GitHub / Supabase。


## 规则层排除规则

`save_flow` 只处理记忆层和数据层保存，不处理规则层写入。

允许处理的对象：

```text
public.memory_items
public.user_preference_items
public.user_scenario_items
public.memory_events
memory/scenario_types/*.md
memory/profiles/<user_key>/scenarios/*.md
memory/snapshots/*.md
memory/imports/*.md
```

不允许处理的对象：

```text
rules/**
README.md 中的规则入口部分
任何规则治理文件
任何运行流程文件
任何读取顺序文件
```

当待保存内容同时包含记忆层 / 数据层变更和规则层变更需求时：

```text
1. 先按本文件处理记忆层 / 数据层保存。
2. 规则层需求不得写入规则文件。
3. 记忆层 / 数据层保存完成、失败或明确跳过后，说明规则层需求只能 issue 化。
4. 整理规则层 issue 草案；仅在用户明确要求时创建 GitHub issue。
```

不得把用户确认保存、二次确认或保存摘要解释为允许写入规则文件。

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

用户提供 profile code 后，必须先按 `rules/routing/profile_routing.md` 执行最小规范化和查找。

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
rules/routing/scenario_routing.md
```

场景身份是：

```text
user_key + scenario_code
```

场景不要求字段完整。

```text
场景要素可以只定义一部分。
未定义字段不阻断场景成立。
未定义字段不阻断保存。
未定义字段不阻断推荐。
未定义字段只用于后续提示用户是否补充。
用户明确说“无所谓”的字段，视为放宽约束，不视为缺失。
模型不得替用户补齐未定义字段。
不使用 complete / incomplete / draft 作为实际业务状态。
```

涉及新场景保存时：

```text
1. 如果没有 scenario_code，先让用户确认 scenario_code。
2. 确认后保存当前已知场景要素。
3. 未限定字段可留空或作为后续提示项记录。
4. 不因字段不完整阻断保存。
```

实际场景状态真源写入 Supabase：

```text
public.user_scenario_items
唯一键：user_key + namespace + scenario_code + game_key
数据库字段：state
```

场景定义记录可使用：

```text
game_key = __scenario_definition__
state = scenario_definition
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

如果某层不写入，必须说明原因。示例：

```text
memory/scenario_types/<scenario_code>.md 不写入：该场景目前只有用户个人口径，没有可复用模板价值。
public.memory_items 不写入：本次内容是用户个人偏好或个人场景状态，不是共享游戏事实。
```

## 用户反馈和游玩记录保存方法

用户提供新的游玩、通关、退款、放弃、回坑、强正面或强负面体验时，必须先执行：

```text
rules/feedback_intake.md
```

`rules/feedback_intake.md` 是用户游戏反馈即时理解机制的唯一详细真源。本文件只记录保存前置依赖，不重复维护完整拆分流程。

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

## 周报配置与推荐变动保存方法

周报配置的创建、修改、暂停、恢复和删除，以及周报提出的推荐变动，都属于存档，必须先输出内容摘要和影响范围并等待用户确认。

周报配置保存摘要必须列出：`user_key`、启用状态、时间与时区、投递模式、详细场景/平台范围、新游雷达、EA/试玩/测试与中文范围、资讯类别、价格促销、传闻开关、将创建或同步的 Automation，以及是否授权定时任务更新运行元数据。

用户确认后：

1. 按唯一逻辑键 `user_key + namespace + report_type` 写入 `public.user_report_subscriptions`；第一版不得创建多个有效周报。
2. 创建或更新 ChatGPT Automation，并把实际标识写回 `automation_id`。
3. 分别回查 Supabase 配置和实际 Automation 的时间、时区与启停状态。
4. 一致时更新 `sync_state`；不一致或部分失败时记录错误并明确告警，不得声称同步完成。

用户可在确认创建周报时一并授权未来定时运行自动更新 `last_snapshot`、`last_run_at`、`last_success_at`、`last_error`、`sync_state`。此授权不扩展到其他表或 GitHub 文件。

推荐变动建议必须明确标记“尚未写入”。用户确认保存后，仍须按本文件的场景分层写入清单执行并逐项回查；不能把订阅创建确认、运行元数据授权或 Automation 授权解释为接受推荐变动。

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
不因场景字段未完整就阻断已确认 scenario_code 的保存。
不使用 complete / incomplete / draft 作为实际业务状态。
不把未定义字段当成用户确认字段。
```

## 推荐读取方式

```text
1. 确认 profile code。
2. profile_routing 查询并确认 user_key。
3. 执行 scenario_routing 并确认 scenario code。
4. scenario_types 读取类型模板；没有可复用模板时说明跳过原因。
5. user_preference_items 读取用户偏好和 played_record。
6. user_scenario_items 读取实际场景状态和必要的场景定义记录。
7. memory_items 读取共享游戏资料。
8. GitHub profile scenario 只作快照参考，不作为状态真源。
9. 收到当前对话游戏反馈时，立即执行 feedback_intake。
```
