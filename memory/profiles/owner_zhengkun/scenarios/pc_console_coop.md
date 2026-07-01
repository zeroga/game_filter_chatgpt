# Personal Scenario: PC / 主机固定队联机游戏筛选

```text
profile_code: zero
code_norm: zero
user_key: owner_zhengkun
scenario_code: pc_console_coop
source_file: v2.7_game_filter_pc_console_coop_scenario_machine.txt
source_version: v2.7
source_sync_date: 2026-06-28
storage_role: personal_preference_scenario
```

## 读取规则

本文件是 `zero / owner_zhengkun` 的个人场景偏好层。

后续读取顺序：

```text
1. rules/recommendation_entry.md
2. rules/routing/profile_routing.md
3. rules/data/multi_user.md
4. rules/current_rules.md
5. memory/scenario_types/index.md
6. memory/scenario_types/pc_console_coop.md
7. memory/profiles/owner_zhengkun/scenarios/pc_console_coop.md
8. Supabase shared profile: public.memory_items
9. Supabase user overlay: public.user_preference_items where user_key = owner_zhengkun
10. Supabase user scenario state: public.user_scenario_items where user_key = owner_zhengkun and scenario_code = pc_console_coop
```

## 场景来源

来源文件为上传的 `v2.7_game_filter_pc_console_coop_scenario_machine.txt`。

该场景文件为严格 JSON，角色为 `scenario_level_working_doc`，项目为“游戏筛选”，场景为“PC/主机联机游戏”。其 contract 指向项目规则 `v4.6` 和游戏数据库 `v1.0`，并声明推荐目标是上限，不是必须填满。

## 场景硬偏好

优先：

```text
official_coop_pve
formal_release
fixed_party_viable
personal_growth_or_equipment_or_build
low_player_count_complete_experience
```

降权或排除：

```text
early_access
pvp_or_pvpve_main_loop
mod_required_for_coop
guest_cannot_progress_or_customize
governor_shared_resource_base_factory
horror_pressure_core
mobile_only
```

注意风险：

```text
first_person_complex_navigation_or_driving
map_confusion
high_reaction_or_visual_pressure
bullet_or_effect_density
pixel_small_character_readability
late_gear_gate
numerical_scaling_without_build_depth
reskinned_enemies_content_padding
```

任意候选必须确认：

```text
coop_mode
low_player_count_experience
host_guest_progression
operation_visual_pressure
longterm_personal_growth_or_build
```

## 当前状态快照

```text
recommendation_target: 6
investigation_limit: 5
recommendation_target_policy: upper_bound_not_fill_obligation
current_recommendations_count: 4/6
current_investigation_count: 5/5
waiting_count: 27
waiting_info_update_status: incomplete
last_complete_online_update: 2026-05-28
last_incomplete_online_update: 2026-06-28
pending_investigation_task: none
do_not_force_fill_recommendations: true
```

## 当前推荐位

1. `Gloomhaven`
   - state: `recommended_owned_untested`
   - 定位：低操作战术合作 RPG；已拥有，适合正式试玩。
   - 风险：偏难、单局可能长、黑暗奇幻氛围、需短任务测试多人稳定性。

2. `Risk of Rain 2`
   - state: `recommended_short_run_backup`
   - 定位：科幻短局合作肉鸽补位。
   - 风险：长线个人养成弱、后期视觉混乱、不是主坑。

3. `Inkbound`
   - state: `recommended_or_active_positive_observation`
   - 定位：低操作合作战术肉鸽；构筑深度当前观察为正面。
   - 风险：中后期长期动力仍需观察；若已在玩则作为 active positive baseline 而非新坑。

4. `Sunderfolk`
   - state: `recommended_trial_only_pending_full_waiting_recheck`
   - 定位：先免费试玩/低成本验证；不直接买断。
   - 风险：手机控制器/桌游节奏需实测；固定队深度需确认；等待清单全条件复查仍未完成。

5. `empty_slot`
   - 理由：无足够干净且风险闭合候选；不为凑满推荐位提升低优先项。

6. `empty_slot`
   - 理由：无足够干净且风险闭合候选；不为凑满推荐位提升低优先项。

## 当前待查队列

1. `Void Crew`
   - priority: high
   - 关键问题：2–3 人是否顺；共享飞船资源是否形成执政官模式；第一人称驾驶/维修/外出压力；客机权益/进度/跨平台稳定性；长期升级是否足够。

2. `Warhammer 40K: Rogue Trader`
   - priority: high_after_waiting_trigger
   - 关键问题：co-op 共享队伍/主机存档是否可接受；DLC/完整版是否适合完整一周目；联机稳定性；角色构筑意义。

3. `Demeo x Dungeons & Dragons: Battlemarked`
   - priority: medium
   - 关键问题：成长/Build 是否过浅；战役和 one-shot 是否足够；重复遭遇/RNG/误伤是否挫败；是否只是轻量 D&D 桌游。

4. `Rotwood`
   - priority: medium_action_risk
   - 关键问题：是否堆怪；手感是否成立；武器/装备/升级差异是否实质；少人体验是否清楚。

5. `Gatekeeper`
   - priority: medium
   - 关键问题：角色小/特效密度能否接受；是否只是俯视角 Risk of Rain 2；局外成长是否足够；2–3 人固定队是否清晰不乱。

## 低优先 / 降权

```text
Deep Rock Galactic: low_priority_mature_backup; 挖矿/洞穴导航为核心循环，成长偏横向，不再占正式推荐位。
Stolen Realm: low_priority_do_not_recommend; checked_user_rejected，画风不行、动态难度压缩构筑，误上推荐已撤回。
Towerborne: low_priority; 堆怪、养成单调、Build 深度不足。
HELLCARD: low_priority; 少人体验问题明显。
Absolum: strong_downrank; 用户实测手感不行。
TMNT: Splintered Fate: low_priority; 清版动作肉鸽子类降温。
Guntouchables: low_priority; 低价轻量小品，不当长期主坑。
Crimson Night: removed_from_investigation; 评价样本极少，hardcore 风险，不占待查位。
```

## Active baselines / Reference

```text
Diablo 4: active_baseline; 暗黑-like 基准线，新刷子必须明显比继续 D4 更值得开。
Warframe: reference_only; 近 3000 小时，科幻 PvE / 长期 Build / 装备收集正面参考。
Destiny 2: reference_only; 近 3000 小时，服务型 PvE 和装备成长参考。
X4: Foundations: single_player_reference_only; 太空 / 舰船 / 经济 / 势力 / 长期目标正面，但不能联机。
```

## Waiting 复查队列

状态总则：`waiting_info_update_status = incomplete`。所有 waiting 项仍保持 `not_completed_recheck_required`，不能写成已复查完成。

```text
Sunderfolk — 查当前售价/免费试玩、正式版评价、固定队深度、低操作战术价值。
Borderlands 4 / 无主之地4 — 查当前各区价格、版本/DLC/Ultimate、PC 优化、近期评价、联机/跨平台、是否值得从等待上移。
Granblue Fantasy: Relink — 查是否有实质新内容、DLC/角色/高难、当前价格、是否值得回坑。
Monster Hunter Wilds / 怪物猎人：荒野 — 查资料片/更新、PC 性能、当前价格、是否已到回坑条件。
Across the Obelisk / 横跨方尖碑 — 查 DLC/新角色/新职业/Bundle 折扣；若无实质构筑池扩展则继续等待。
红色警戒3：日冕 Mod — 查最新可玩地图、联机可用性、是否有新合作内容。
Warhammer 40K: Rogue Trader — 查 Void Shadows/Lex Imperialis/后续 DLC、完整版价格、补丁稳定性、是否适合完整一周目。
Solasta II — 查是否 1.0、内容量、联机稳定、评价。
Age of Wonders 4 / AOW4 — 查新 DLC、Bundle/折扣、联机变化。
Heroes of Might and Magic: Olden Era — 查 1.0/EA、co-op 模式、评价和长局压力。
Stellaris / 群星 — 查官方 DLC、订阅/Bundle、联机大版本、是否值得回坑。
Slay the Spire II / 杀戮尖塔2 — 查 1.0、合作模式、近期评价、构筑自由度。
Rogue Point — 查 1.0、合作结构、FPS 压力、评价。
Deep Rock Galactic: Rogue Core — 查 1.0、共享升级、timer、评价。
Romestead — 查 1.0、资源共管、基地/城镇共享压力。
Lost Castle 2 — 查 1.0 正式版、近期评价、联机稳定、少人体验、成长深度、价格；不得继续写“等正式版”。
Mecha BREAK / 解限机 — 查新赛季 PvE 内容量、循环、机体/武器养成。
BlazBlue Entropy Effect / 苍翼混沌效应 — 查新角色/构筑/DLC/价格。
Once Human / 七日世界 — 查新 PvE 剧本、大型内容、联机循环。
The First Descendant / 第一后裔 — 查大型系统重构、装备/Build、后期循环、评价。
Broken Arrow / 断箭 — 查新模式、官方合作战役、PvE 任务链。
Neon Abyss 2 / 霓虹深渊2 — 查 1.0、系统完善、联机、Build 和内容稳定性。
NeverLight — 查是否发售、联机、长期成长、黑暗/类魂/基地风险。
Cardtographer — 查发售、真联机、队伍构筑、局外成长。
Eternally — 查内容量、合作战役、PvP 比重。
Fellowship — 查 EA 完整度、装备系统、难度/副本、价格、评价。
Jump Space — 查 EA 完整度、武器/飞船保留、少人体验、升级深度、第一人称船员协作风险。
```

## 场景负面索引

```text
Stolen Realm: block recommendation; checked_user_rejected / low_priority; 画风不行，动态难度可能压缩后期构筑。
Deep Rock Galactic: block formal_recommendation; risk_confirmed / low_priority; 挖矿/找路核心循环和成长偏横向。
Towerborne: block recommendation; low_priority; 堆怪、养成单调。
HELLCARD: block recommendation; low_priority; 少人体验问题。
Absolum: block recommendation; strong_downrank; 手感不行。
Terrinoth: Heroes of Descent: block recommendation; strong_downrank; 深度太浅、构筑简单、游戏简单。
Heroes of Hammerwatch II: block recommendation; excluded; 像素风角色太小、看不清。
```

## 审计要求

加入推荐前必须：

```text
check_project_negative_index
check_scenario_negative_index
check_old_scenario_conclusion
complete_candidate_audit_record
answer_why_not_waiting_investigate_low_priority_excluded
```

若任一检查缺失：不得加入推荐。

若命中低优先或负面：保持低优先或排除，除非完成负面解除记录。

若干净候选少于 6：保留空位。

## 近期变化

```text
2026-06-28: 场景档改为严格 JSON，不再复制项目级自然语言流程规则。
2026-06-28: Deep Rock Galactic 从推荐降为低优先成熟补位；Stolen Realm 恢复 checked_user_rejected / 低优先 / 不推荐。
2026-06-28: 等待清单仍识别为未完成复查；所有 waiting 项保持 not_completed_recheck_required，不能写成已复查完成。
2026-06-28: 推荐目标 6 个是上限，不是必须填满；空位 5-6 保留。
2026-06-28: Split project rules and game database; scenario now references v4.6 rules and v1.0 game database; waiting list remains incomplete.
```

## 自检继承

```text
json_parse_validated: true
filename_ascii: true
waiting_info_update_status: incomplete
all_waiting_items_not_completed_recheck_required: true
deep_rock_downgraded: true
stolen_realm_blocked_from_recommendation: true
recommendation_slots_not_force_filled: true
conforms_to_project_library_v4_6: true
references_game_database_v1_0: true
json_valid: true
```
