# Legacy / Deprecated

本文件只作为历史规则补丁归档，不是活跃规则真源。当前规则以 `rules/current_rules.md`、`rules/save_flow.md` 和 `rules/governance/` 为准。

# Rule Layer Split Save Policy

更新日期：2026-07-01（JST）

## 规则

存档流程中，如果待保存内容同时包含数据/记忆层变更和规则层变更，必须拆分处理。

第一阶段先处理数据/记忆层变更，包括 Supabase 共享游戏资料、用户偏好、游玩记录、用户场景状态、memory_events，以及 GitHub 场景类型模板或用户场景快照等非规则内容。

第一阶段必须完成保存摘要、用户确认、写入和回查。完成、失败或明确跳过并说明原因后，才允许进入第二阶段。

第二阶段单独处理规则层变更，包括 README、current_rules、save_flow、recommendation_entry、feedback_intake、profile_routing、scenario_routing、database_positioning、multi_user 或其他规则入口、流程规则、治理规则、工作档规则文件。

进入第二阶段时，必须单独询问用户是否继续保存规则层变更，并输出新的规则保存摘要。未获得第二次明确确认前，不得写入任何规则文件。

如果数据/记忆层保存失败或未完成，仍可提出是否继续保存规则层变更，但必须先说明数据/记忆层未完成项和风险，不能把规则写入伪装成数据保存的一部分。

规则层变更不得夹带在普通游戏数据、用户偏好、游玩记录、场景状态或记忆事件保存中。规则变更必须在影响范围中明确标出。

## 待集成位置

- rules/save_flow.md
- rules/current_rules.md

## 当前状态

本文件为独立规则补丁。后续应将本规则并入 save_flow 和 current_rules 的正式入口。