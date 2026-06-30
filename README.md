## 运行硬约束

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

# game_filter_chatgpt

固定入口：

```text
memory/recommendation_entry.md
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
