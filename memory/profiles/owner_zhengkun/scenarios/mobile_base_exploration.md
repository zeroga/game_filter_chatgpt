# Personal Scenario: 移动基地与外出探索

```text
profile_code: zero
alias_norm: zero
user_key: owner_zhengkun
scenario_code: mobile_base_exploration
source_reference: v1.0_game_filter_mobile_base_exploration_scenario_machine.txt
source_version: v1.0
source_index: v4.4_game_filter_preference_library_machine.txt
snapshot_date: 2026-11-02
storage_role: personal_scenario_snapshot_not_source_of_truth
migration_status: minimal_definition_only
source_fidelity: file_name_reference_only
```

## 已确认来源与恢复范围

旧项目偏好库的场景文件索引明确提到上述源文件名；当前检索尚未找到该场景工作档的完整正文。因此仅恢复由文件名与已知场景名称支持的主题“移动基地与外出探索”，不推断原档的硬条件、权重、排序或用户游玩状态。

## 目前未恢复

- 场景硬条件、软偏好及特殊审计字段：原文未获取，不作补全
- 推荐、待查、等待、低优先、排除、参考和基准线：没有完整来源，尚未迁移
- 每条等待的触发条件及最近核查结果：没有完整来源，尚未迁移
- 旧版完整工作档和后续版本变更：需找到并核对，不能把本文件视为复原版本

## 数据真源

```text
project: chatgpt_memory
user_key: owner_zhengkun
namespace: game_filter
scenario_code: mobile_base_exploration
definition: game_key=__scenario_definition__; state=scenario_definition
individual_game_states: public.user_scenario_items
user_preferences_and_played_records: public.user_preference_items
shared_game_profiles: public.memory_items
```

本文件只是迁移缺口快照，不能替代 Supabase 当前场景状态，也不能作为旧候选已经迁移成功的依据。旧原文找到后，应先做差异核查，再经用户确认补迁。
