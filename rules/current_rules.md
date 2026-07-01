# Current Rules

本文件是后续运行入口，不再要求用户指定旧版本号。

## 读取入口

后续只读 GitHub 仓库：

```text
zeroga/game_filter_chatgpt
```

读取顺序：

```text
README.md
rules/current_rules.md
rules/save_flow.md
rules/recommendation_entry.md
rules/feedback_intake.md
rules/routing/profile_routing.md
rules/routing/scenario_routing.md
rules/data/database_positioning.md
rules/data/multi_user.md
rules/data/schema_notes.md
rules/governance/rule_change_issue_only.md
rules/governance/rule_memory_layer_separation.md
rules/tool_basics.md
memory/README.md
memory/snapshots/current_state.md
memory/snapshots/project_workdoc.md
```

没有默认 `current_scenario.md`。实际场景必须通过 profile code + scenario code 确认后读取。

## 规则层 / 记忆层 / 数据层分工

```text
规则层 = rules/
记忆层 = memory/
数据层 = Supabase chatgpt_memory
```

规则真源只放在 `rules/`。记忆辅助内容只放在 `memory/`。游戏筛选运行数据只放在 Supabase。

规则层变更只能 issue 化。ChatGPT 普通对话中不得直接修改规则文件，也不得直接创建规则修改 PR；只能整理规则变更需求，并在用户确认后创建 GitHub issue。Codex 或人工可以根据 issue 驱动任务修改规则文件。

记忆层 / 数据层保存走 `rules/save_flow.md`。规则层不进入 save_flow 写入流程。


## 存档触发规则

写入 GitHub 记忆层文档、Supabase 数据、用户偏好、游玩记录、场景状态、项目状态快照或导入说明，都视为“存档”。

用户说“存档 / 保存 / 写入 / 同步 / 更新到文档 / 写到数据库 / 记录下来”时，只表示进入保存流程，不表示立即写入。

保存流程固定为：汇总待保存内容；输出保存摘要；等待用户确认；确认后才执行写入。

保存摘要只需要包括：

```text
内容摘要
影响范围
```

如果用户只说“保存”或“存档”且没有限定对象，必须汇总当前对话中所有待保存内容。

用户确认前，不能写入 GitHub、Supabase 或其他外部系统。

如果当前对话中已经形成了应保存的结论、游戏记录、场景状态或流程变更需求，但用户还没有明确要求存档，必须提示：

```text
有待写入内容未存档。
```

规则层变更需求不得夹带在普通游戏数据、用户偏好、游玩记录、场景状态或 memory_events 保存中；只能整理为 issue。

## 每轮 profile / scenario 获取边界

每个新对话中，只要任务涉及游戏推荐、游戏筛选、更新候选清单、解释场景状态、读取用户偏好、读取用户游玩记录、读取用户场景状态、候选审计、等待项复查、排除 / 低优先 / 推荐状态判断，就必须重新获取本轮 `profile code` 和 `scenario code`。

`profile code` 必须由用户主动提供。ChatGPT 不得基于已知 alias / user_key 映射、用户昵称、当前账号名、历史 user_key、上一次对话使用过的 profile code、仓库或数据库中已存在的唯一 profile，或当前对话外的记忆推断，主动代替用户选择或建议某个具体 profile code。

ChatGPT 可以提示用户提供 profile code，也可以解释 profile code 用于选择个人偏好层；但不得主动给出某个具体 code 作为本轮默认选项。

用户给出 profile code 后，才能执行 profile routing：规范化 `code_norm = lower(trim(profile_code_text))`，查询 `public.profile_aliases.alias_norm = code_norm`，命中后回显 `profile code / alias_norm / user_key` 并等待用户确认。用户确认前不得读取或写入该用户层数据。

profile 确认后，必须重新获取本轮 `scenario code`。`scenario code` 必须由用户主动提供，或在用户明确要求“列出现有场景供我选择”后，由用户从列表中选择。

ChatGPT 不得基于已有场景快照、某个 profile 下只有一个场景、上一次对话使用过的 scenario code、场景类型模板、用户历史推荐场景、仓库中存在的 `memory/profiles/<user_key>/scenarios/*.md`、Supabase 中已有的唯一 scenario_code，或当前对话外的记忆推断，主动代替用户选择或建议某个具体 scenario code。

ChatGPT 可以提示用户直接提供 scenario code，也可以提示“如果不记得 code，可以让我列出现有场景供你选择”；但不得把某个已有场景主动设为默认值。

只有当用户在当前对话中已经完成 profile code 和 scenario code 确认后，再明确说“继续本轮刚确认过的场景”，才允许沿用当前对话内已确认的 profile / scenario。不能跨新对话沿用。

已知 alias / user_key 映射只能用于用户输入后的匹配和回显，不能用于主动发起选择。已有场景快照和场景类型模板只能在用户确认 scenario code 后读取，不能用于提前推断本轮场景。

## profile / scenario 确认规则

任何读取或写入用户层数据前，必须先执行：

```text
rules/routing/profile_routing.md
```

任何读取或写入用户场景状态前，必须先执行：

```text
rules/routing/scenario_routing.md
```

`rules/routing/scenario_routing.md` 负责 scenario code 的确认、查找、创建、场景对象模型和真源边界。

核心控制点：

```text
1. 先精确查询 public.profile_aliases。
2. 命中则回显 profile code、alias_norm、user_key，等待确认。
3. 未命中时，不直接创建新用户。
4. 先提示可能相近或容易混淆的已有账户，询问是不是其中之一。
5. 只有用户明确确认不是已有账户，并确认要创建新 profile，才允许新建。
6. 创建新 scenario 前必须二次确认；先提示相近已有场景，确认不是已有场景后才新建。
7. 场景身份是 user_key + scenario_code。
8. 场景字段不要求完整；未定义字段不阻断场景成立、保存或推荐。
9. 未定义字段只用于后续提示用户是否补充，不得由模型自行补齐。
10. 用户明确说“无所谓”的字段视为放宽约束，不视为缺失。
11. 不使用 complete / incomplete / draft 作为实际业务状态。
```

## 规则存储分工

GitHub 的 `rules/` 保存当前规则；GitHub 的 `memory/` 保存场景类型模板、用户场景快照、项目状态快照和导入说明。

Supabase 保存共享游戏画像、共享游戏事实、用户稳定偏好、用户游玩记录、用户反馈覆盖、用户场景状态、场景条目、待查任务、等待条件、来源摘要。

## 通用硬规则

```text
silent load 只减少对外输出，不减少内部读取、索引、审计和自检。
推荐目标是上限，不是必须填满。
缺 profile 路由确认时停止推荐或更新。
缺 scenario 路由确认时停止推荐或更新。
缺游戏数据库时停止推荐或更新。
缺用户偏好层读取时停止推荐或更新。
缺用户游玩记录读取时停止推荐或更新。
缺用户场景状态读取时停止推荐或更新。
缺候选审计时阻断推荐。
涉及当前事实、版本、价格、评价、联机结构、DLC、EA 状态时必须联网核查。
用户主观反馈优先于外部主观评价；客观结构仍需外部验证。
用户对游戏发表意见时，必须立即执行 rules/feedback_intake.md。
场景相关存档必须按 rules/save_flow.md 生成分层写入清单并逐项回查。
```

## 输出粒度规则

```text
非游戏内容分析、游戏画像、推荐理由、候选审计类回复时，默认尽量精简。
只输出必要结论、当前状态和下一步动作。
游戏内容、推荐分析、规则审计和存档摘要不受该精简规则强制压缩。
```

## 用户游戏反馈即时理解规则

用户对游戏发表意见时，必须先执行：

```text
rules/feedback_intake.md
```

`rules/feedback_intake.md` 是即时理解机制的唯一详细真源。本文只保留硬触发和入口指针，不重复维护完整三步流程。

## 场景对象与场景存档完整性

```text
rules/routing/scenario_routing.md = 场景身份、场景对象模型、scenario code 确认、查找、创建和真源边界。
rules/save_flow.md = 场景相关存档的分层写入清单、执行和回查规则。
```

保存内容涉及新 scenario_code、场景口径、场景候选、场景状态、场景类型模板或用户场景快照时，不能只写 Supabase 状态真源。必须检查 GitHub 场景类型模板、GitHub 用户场景快照、Supabase 用户场景状态、用户偏好、公共画像和 memory_events 的写入或跳过原因。

场景字段可以不完整；保存时必须有 `scenario_code`，然后按当前已知字段写入。未限定字段可留空或作为后续提示项记录，不因字段不完整阻断保存。

## 推荐前必须读取的数据层

```text
1. public.memory_items
   - 共享游戏画像
   - 共享游戏事实
   - positive_reference_index
   - negative_decision_index

2. public.user_preference_items
   - stable_preference
   - played_record
   - game_feedback_overlay
   - positive_reference_index
   - negative_reference_index

3. public.user_scenario_items
   - 当前 user_key + scenario_code 下的推荐、待查、等待、排除、低优先、参考、基准线状态
   - 数据库字段为 state
```

`played_record` 不是可选参考。任何推荐、筛选、更新清单、解释候选状态前，都必须读取并合并。

## 候选审计必填项

```text
game_name
aliases_checked
database_lookup_result
played_record_lookup_result
played_record_status_effect
positive_reference_index_result
negative_index_result
old_scenario_conclusion_result
user_feedback_conflict_check
current_web_fact_check
scenario_hard_condition_check
why_not_already_played_or_completed
why_not_waiting
why_not_investigate
why_not_low_priority
why_not_excluded
final_state
```

若候选审计中缺少 `played_record_lookup_result` 或 `played_record_status_effect`，不得进入推荐位。

## played_record 状态处理规则

```text
completed / fully_completed：默认不作为新推荐；可作为 reference_only、active_baseline、waiting_recheck 或回坑候选。
currently_playing：默认作为 active_baseline 或 current_positive_observation，不重复推荐为新坑。
refunded：默认 block recommendation；除非用户明确说明退款原因已解除。
abandoned：默认降权或排除；必须有 revisit_condition 才能重新进入候选。
tried_negative：默认强降权或排除；外部好评不能单独解除。
strong_positive_reference：作为偏好基准和同类正面参考，不等于当前推荐同一游戏。
strong_negative_reference：作为同类风险基准；命中相似机制时必须审计。
reference_only：只作偏好或机制参考，不作推荐位。
played：必须结合 notes、positive_points、negative_points、related_scenarios 判断。
```

## 合并优先级

```text
当前对话明确反馈
当前用户场景状态 public.user_scenario_items
当前用户游玩记录 played_record
当前用户稳定偏好 / 反馈覆盖 / 正负面索引 public.user_preference_items
共享游戏画像 public.memory_items
外部当前事实核查
```

## 写回规则

用户提供新的游玩、通关、退款、放弃、回坑、强正面或强负面体验时，先执行 `rules/feedback_intake.md`，再按拆分结果写入记忆层或数据层。

用户游玩记录写入 `public.user_preference_items` 的 `played_record`。

跨场景稳定偏好写入 `stable_preference` 或 `game_feedback_overlay`。

只改变某个场景结论的反馈写入 `public.user_scenario_items`，数据库字段为 `state`。

经外部核对后可公共化的游戏结构、机制风险、版本状态、联机结构、终局问题等，写入 `public.memory_items`。

不迁移历史数据，不机械替换 payload 内部历史词汇。

不得把个人游玩记录、个人正负面索引、个人推荐状态写入 `public.memory_items`。

## 来源优先级

```text
用户当前对话明确反馈
用户 played_record / stable_preference / scenario state
用户亲身体验历史记录
官方机制公告或路线图
官方商店页
近期评价
长差评
实机视频
可信社区
媒体文章硬事实
```

## 旧规则文件定位

```text
rules/legacy/v4.6_game_filter_preference_library_machine.json
```

该文件只作为来源归档，不再作为后续运行入口。
