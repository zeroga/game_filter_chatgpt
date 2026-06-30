## 运行硬约束

### README 优先级

```text
本 README 是进入项目后的最高入口。

阅读和执行优先级：

1. 只读优先约束。
2. 必须精读约束。
3. 存档机制。
4. 用户游戏反馈即时理解机制。
5. 其他项目规则按任务目标、风险、影响范围和已读规则判断执行。
```

### 只读优先约束

```text
进入本仓库后，默认只能执行只读操作：读取仓库元数据、读取文件、搜索文件、读取 issue / PR、读取评论。
除非用户明确要求修改、提交、创建、删除、关闭、评论或其他写入动作，否则不得调用任何会改变 GitHub / Supabase / 外部系统状态的接口。
读目录、查结构、验证文件存在性时，也必须使用只读接口；不得用 create_tree、create_file、update_file、delete_file、create_commit 等写类接口进行探测。
若误触写类接口，必须立即停止、说明影响范围，并在后续只使用只读接口。
```

### 必须精读约束

```text
读取本项目规则时，必须逐条精读入口文件、规则文件、数据库定位文件、多用户规则、保存流程和当前状态文件。
不得只读取 README、搜索命中片段、摘要、文件头、索引或局部规则后就下结论。
不得把 silent load 理解为跳过读取；silent load 只减少对外输出，不减少内部读取、索引、审计和自检。
涉及推荐、筛选、更新清单、解释场景状态、同步工作档、迁移数据、清理公共库、用户层/共享层边界判断时，必须先完成规则精读和执行化。
精读完成前，只能输出“已读到的局部结论”和“未完成读取范围”，不得声称已完整读取。
```

### 存档机制

```text
写入 GitHub 文档、Supabase 数据、用户偏好、游玩记录、场景状态、规则文件或工作档，都视为“存档”。
默认不主动存档。用户说“保存 / 存档 / 写入 / 同步 / 记录下来”等，只表示进入保存流程，不表示立即写入。
保存流程固定为：汇总待保存内容 -> 输出保存摘要 -> 等待用户确认 -> 用户确认后才执行写入。
保存摘要只需要包括：内容摘要、影响范围。
如果当前对话中已经形成了应保存的规则、结论、游戏记录、场景状态或流程变更，但用户还没有明确要求存档，必须提示：有待写入内容未存档。
```

### 用户游戏反馈即时理解机制

```text
用户对游戏发表意见时，必须立刻执行 memory/feedback_intake.md。
先联网核对对象、版本、平台、机制、联机状态、近期评价和是否过时。
再拆分为公共画像候选、用户游玩记录、用户偏好 / 反馈覆盖、用户场景状态。
这一步早于推荐、解释、更新清单、候选审计和保存流程。
不能等到用户说保存时才核对事实或拆分数据层。
```

# game_filter_chatgpt

固定入口：

```text
memory/recommendation_entry.md
memory/feedback_intake.md
memory/save_flow.md
memory/profile_routing.md
memory/database_positioning.md
memory/current_rules.md
memory/multi_user.md
```

推荐前必须确认：

```text
profile code
scenario code
```

没有默认用户，也没有默认场景。

保存新用户或新场景时，先读：

```text
memory/save_flow.md
```

场景类型模板：

```text
memory/scenario_types/
```

实际场景快照：

```text
memory/profiles/<user_key>/scenarios/
```

实际状态真源：

```text
public.user_scenario_items
```

旧目录 `memory/scenarios/` 已废弃。
