# Issue 3 Rule Conflict Audit

> Snapshot boundary: 本文件是 memory 快照层文件，只作续接和人工阅读辅助。
> 本文件不是规则真源；规则真源以 `rules/current_rules.md` 及其引用文件为准。
> 本文件不是用户场景状态真源；用户场景状态以 `public.user_scenario_items` 为准。
> 若本文件与 `rules/` 下规则冲突，以 `rules/` 为准；若本文件与 Supabase 用户场景状态冲突，以 Supabase 为准。


更新日期：2026-07-01（JST）

Issue #3 要求建立三层结构：`rules/` 规则层、`memory/` 记忆辅助层、Supabase 数据层，并审计旧规则是否与三层结构冲突。

## 审计清单

| 文件 | 目标层级 | 冲突点 | 修正方式 | 是否完成 | 备注 |
|---|---|---|---|---|---|
| `README.md` | 项目入口 | 旧版承载大量规则细节，并指向 `memory/` 规则入口 | 重写为三层结构入口，指向 `rules/current_rules.md` | 完成 | 保留项目入口职责，不作为详细规则真源 |
| `memory/current_rules.md` | deleted migrated path | 规则真源位于 `memory/` | 迁移到 `rules/current_rules.md`，旧路径删除 | 完成 | 不再作为规则真源 |
| `memory/save_flow.md` | deleted migrated path | 保存流程真源位于 `memory/`，且可能处理规则层 | 迁移到 `rules/save_flow.md`，旧路径删除 | 完成 | 新 save_flow 明确排除规则层 |
| `memory/recommendation_entry.md` | deleted migrated path | 推荐流程真源位于 `memory/` | 迁移到 `rules/recommendation_entry.md`，旧路径删除 | 完成 | 路径引用已更新 |
| `memory/feedback_intake.md` | deleted migrated path | feedback intake 真源位于 `memory/` | 迁移到 `rules/feedback_intake.md`，旧路径删除 | 完成 | 路径引用已更新 |
| `memory/profile_routing.md` | deleted migrated path | profile 路由规则位于 `memory/` | 迁移到 `rules/routing/profile_routing.md`，旧路径删除 | 完成 | 路径引用已更新 |
| `memory/scenario_routing.md` | deleted migrated path | scenario 路由规则位于 `memory/` | 迁移到 `rules/routing/scenario_routing.md`，旧路径删除 | 完成 | 路径引用已更新 |
| `memory/database_positioning.md` | deleted migrated path | 数据库定位规则位于 `memory/` | 迁移到 `rules/data/database_positioning.md`，旧路径删除 | 完成 | 路径引用已更新 |
| `memory/multi_user.md` | deleted migrated path | 多用户规则位于 `memory/` | 迁移到 `rules/data/multi_user.md`，旧路径删除 | 完成 | 路径引用已更新 |
| `memory/schema_notes.md` | deleted migrated path | 数据库结构说明影响读取和写入边界，位于 `memory/` | 迁移到 `rules/data/schema_notes.md`，旧路径删除 | 完成 | 作为数据层规则说明读取 |
| `memory/rules/*.md` / `memory/rules/*.json` | legacy | 历史规则补丁位于 `memory/` | 迁移到 `rules/legacy/` | 完成 | 仅作来源归档 |
| `memory/current_state.md` | 记忆层快照 | 项目状态快照位于 memory 根目录 | 迁移到 `memory/snapshots/current_state.md` | 完成 | 非规则真源 |
| `memory/project_workdoc.md` | 记忆层快照 | 工作档快照位于 memory 根目录 | 迁移到 `memory/snapshots/project_workdoc.md` | 完成 | 非规则真源 |
| `memory/scenario_types/*.md` | 记忆层 | 无冲突 | 保留 | 完成 | 场景类型模板，不是用户实际场景 |
| `memory/profiles/<user_key>/scenarios/*.md` | 记忆层 | 可能被误当作状态真源 | 保留，并继续声明 `public.user_scenario_items` 是状态真源 | 完成 | 快照只作续接参考 |
| `rules/current_rules.md` | 规则层 | 新入口需避免继续指向 memory 规则真源 | 作为唯一正式运行入口，读取顺序更新到 `rules/` | 完成 | 包含三层分工和 issue 化规则 |
| `rules/save_flow.md` | 规则层 | 旧逻辑可能允许规则层二次确认后写入 | 重构为只处理记忆层 / 数据层保存，并明确规则层只能 issue 化 | 完成 | 不再处理规则层写入 |
| `rules/governance/rule_change_issue_only.md` | 规则层 | 缺少规则层变更流程 | 新增 | 完成 | 明确 ChatGPT 普通对话禁止直接改规则文件 / 规则 PR |
| `rules/governance/rule_memory_layer_separation.md` | 规则层 | 缺少三层混写硬边界 | 新增 | 完成 | 明确 rules / memory / Supabase 禁止混写 |

## 剩余风险

1. 历史 changelog 仍可能保留旧路径文字；如为历史记录可保留，但不得作为当前入口。
2. `memory/snapshots/*.md` 是快照，可能包含历史口径；当前规则以 `rules/current_rules.md` 为准。
3. Supabase 结构未改，本次符合 issue 非目标：不改 Supabase 表结构、不迁移 Supabase 历史数据。
