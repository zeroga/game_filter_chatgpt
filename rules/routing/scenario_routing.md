# Scenario Routing

本文件定义 scenario code 的确认、查找、创建、场景对象模型和真源边界。

## 新对话 scenario code 边界

每个新对话中，只要任务涉及推荐、筛选、更新候选清单、解释场景状态、读取用户场景状态、候选审计、等待项复查、排除 / 低优先 / 推荐状态判断，就必须在 profile code 确认后重新获取本轮 `scenario code`。

`scenario code` 必须由用户主动提供，或在用户明确要求“列出现有场景供我选择”后，由用户从列表中选择。ChatGPT 可以提示用户直接提供 scenario code，也可以提示用户要求列出现有场景；但不得把某个已有场景主动设为默认值。

ChatGPT 不得基于已有场景快照、某个 profile 下只有一个场景、上一次对话使用过的 scenario code、场景类型模板、用户历史推荐场景、仓库中存在的 `memory/profiles/<user_key>/scenarios/*.md`、Supabase 中已有的唯一 scenario_code，或当前对话外的记忆推断，主动代替用户选择或建议某个具体 scenario code。

已有场景快照和场景类型模板只能在用户确认 scenario code 后读取，不能用于提前推断本轮场景。只有当用户在当前对话中已经完成 profile code 和 scenario code 确认后，再明确说“继续本轮刚确认过的场景”，才允许沿用当前对话内已确认的 scenario；不能跨新对话沿用。

## 核心原则

```text
没有默认场景。
scenario code 必须在当前对话中确认。
不能因为某个 profile 只有一个已知场景就自动使用该场景。
不能把自然语言场景描述直接当成新 scenario code。

定时周报按已确认订阅的 `user_key` 遍历该 profile 下全部实际场景，是 profile 级批处理，不是“全局场景”，也不代表对交互式推荐默认选择了某个 scenario。必须排除场景类型模板和 `legacy_imported_status`；交互式推荐仍需用户确认单独的 scenario code。
不能不做二次确认就创建新场景。
```

## 场景身份与场景对象模型

场景身份是：

```text
user_key + scenario_code
```

一个 scenario 指：某个 user_key 下的一套游戏筛选目标、适用范围、约束、偏好解释方式、候选审计维度和候选状态空间。

```text
scenario_code = 场景代号，不等于场景全部内容。
scenario_type = 可复用类型模板，不是用户实际场景。
user scenario snapshot = 某用户某场景的可读快照，不是状态真源。
user_scenario_items = 某用户某场景下对具体游戏的状态真源。
```

场景可以包含以下要素：

```text
confirmed user_key
confirmed scenario_code
场景目标
适用平台 / 类型范围
硬性排除条件
软偏好 / 加分项
候选审计补充字段
用户明确表示“无所谓”的字段
未限定但可后续提示的字段
是否需要 GitHub scenario_type 模板
是否需要 GitHub user scenario snapshot
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

推荐、筛选和候选审计时，只按已定义约束执行；未定义字段不参与筛选，也不得被模型自行补齐。

## 新自然语言场景意图流程

当用户拒绝已有场景，并提出新的自然语言场景意图时：

```text
1. 先承认这是一个新场景意图。
2. 列出当前已知场景要素。
3. 列出未限定字段，并提示用户可以补充。
4. 如果用户说“无所谓”，对应字段视为放宽约束。
5. 不能直接替用户生成完整场景。
6. 不能把模型自拟字段当成用户确认字段。
7. 如果用户要求推荐，则按当前已知约束推荐，未限定字段不参与筛选。
8. 如果用户要求保存，则必须先确认 scenario_code。
9. scenario_code 确认后，按当前已知字段保存场景；字段不完整不影响保存。
```

## 场景相关对象分工

```text
memory/scenario_types/*.md = 场景类型模板。
memory/profiles/<user_key>/scenarios/*.md = 用户个人场景快照。
public.user_scenario_items = 用户场景状态真源。
```

场景类型模板不是实际场景。用户个人场景快照只是可读快照，不是状态真源。

### memory/scenario_types/<scenario_code>.md 存储内容

存可复用场景模板，通常包括：

```text
scenario_code
scenario_type_name
scope_template
condition_template
candidate_state_enum
audit_field_template
waiting_recheck_template
storage_policy_template
```

不得在 scenario_type 中保存某个用户的实际推荐、等待、排除、已玩记录或个人排序。

如果一个用户场景只有用户个人口径，没有可复用模板价值，可以不新增 scenario_type；但保存摘要或执行结果必须说明跳过原因。

### memory/profiles/<user_key>/scenarios/<scenario_code>.md 存储内容

存该用户该场景的 GitHub 可读快照，通常包括：

```text
user_key
scenario_code
当前场景口径摘要
已确认字段
用户明确放宽的字段
可后续提示但不阻断的字段
重要候选摘要
本轮关键用户反馈摘要
Supabase 状态真源查询位置
最后同步时间
```

快照只用于续接和人工阅读；不能替代 `public.user_scenario_items`。

### public.user_scenario_items 存储内容

存某个用户在某个场景下对具体游戏的真实状态，或该场景的定义记录。

```text
user_key + namespace + scenario_code + game_key
state
reason
payload
last_checked_at
updated_at
```

数据库字段必须使用 `state`，不要写成 `status`。

场景定义记录可使用：

```text
game_key = __scenario_definition__
state = scenario_definition
```

该记录只保存场景口径，不保存具体游戏推荐结果。

### public.user_preference_items 存储内容

存用户已玩、通关、弃坑、退款、稳定偏好、反馈覆盖、正负面参考等用户层信息。

### public.memory_items 存储内容

只存可跨用户复用的共享游戏画像、共享游戏事实和共享正负面索引；不得写入个人游玩记录或个人场景状态。

## 确认流程

在任何推荐、筛选、更新清单或解释场景状态之前，必须确认：

```text
profile code
scenario code
```

profile code 的确认由 `rules/routing/profile_routing.md` 处理。拿到已确认的 `user_key` 后，才允许进入 scenario code 确认。

如果缺 scenario code，询问：

```text
请提供本轮要使用的 scenario code。如果不记得 code，可以让我列出现有场景供你选择。没有明确场景时不能开始游戏推荐。
```

拿到 scenario code 或自然语言场景描述后，按以下顺序处理：

```text
1. 查询已有 memory/scenario_types/*.md。
2. 查询该 user_key 下已有 memory/profiles/<user_key>/scenarios/*.md 快照。
3. 必要时查询 public.user_scenario_items 中该 user_key 已有 scenario_code。
4. 如果输入像已有场景或自然语言别名，先提示可能匹配的已有场景。
5. 用户确认后，回显本次使用的 scenario_code。
6. 用户确认前，不能读取或写入该场景的个人状态。
```

回显格式：

```text
本次使用的 scenario code 是 <scenario_code>。请确认。
```

## 新 scenario 创建规则

只有同时满足以下条件，才允许进入新 scenario 创建流程：

```text
1. profile code 已确认，并已得到 user_key。
2. 已查询已有 scenario_types、用户场景快照和必要的 user_scenario_items。
3. 已提示可能相近或容易混淆的已有场景。
4. 用户明确确认不是已有场景。
5. 用户明确确认要创建新 scenario_code。
```

新 scenario 可以只保存当前已知字段。字段未完整不影响创建或保存，但未定义字段必须只作为后续提示项，不能被模型补齐。

创建新场景前必须回显：

```text
将创建新 scenario_code: <scenario_code>
实际场景状态仍写入 public.user_scenario_items
可选导出 GitHub 场景快照: memory/profiles/<user_key>/scenarios/<scenario_code>.md
请确认。
```

如果需要新增可复用模板，再额外回显：

```text
将新增或使用场景类型模板: memory/scenario_types/<scenario_code>.md
```

## 写入边界

新增或修改场景类型模板、用户场景快照或 Supabase 场景状态都属于存档，必须执行 `rules/save_flow.md`。

涉及场景的写入清单、分层执行和写入后回查由以下文件管理：

```text
rules/save_flow.md
```

实际场景状态写入：

```text
public.user_scenario_items
唯一键：user_key + namespace + scenario_code + game_key
数据库字段：state
```

不要把数据库字段写成 `status`。中文可以说“状态”，但涉及数据库字段时必须使用 `state`。

可选导出的 GitHub 场景快照：

```text
memory/profiles/<user_key>/scenarios/<scenario_code>.md
```

快照只作读取辅助，不替代 `public.user_scenario_items`。

## 禁止行为

```text
不能默认使用任何 scenario code。
不能用上一次对话的 scenario code 自动续接新对话。
不能把 memory/scenario_types/index.md 或类型模板列表当作默认场景。
不能把 memory/profiles/<user_key>/scenarios/ 下已有文件当作默认场景。
不能把用户自然语言场景描述直接创建成新 scenario。
不能在用户未确认前读取或写入该场景个人状态。
不能把用户实际场景状态写入 memory/scenario_types/。
不能用 GitHub 场景快照代替 public.user_scenario_items。
不能因场景要素未完整就阻断已确认 scenario_code 的保存或推荐。
不能使用 complete / incomplete / draft 作为实际业务状态。
不能把未定义字段当成用户确认字段。
```
