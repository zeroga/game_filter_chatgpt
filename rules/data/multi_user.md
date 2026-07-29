# Multi-user Preference Layer

本文件定义多用户使用时的存储方式。

## 目标

多用户问题不是优先做安全隔离，而是让每个用户拥有各自的偏好、反馈和场景状态。

核心原则：

```text
共享游戏资料库不复制。
用户偏好分层保存。
场景状态按 user_key + scenario_code + game_key 保存。
正式场景按 user_key + scenario_code 的 `__scenario_definition__ / scenario_definition` 记录独立定义，周报不得从其他用户、模板或普通游戏记录推断场景。
周报配置按 user_key + namespace + report_type 独立保存；第一版每个用户只允许一个 `weekly_game_report` 有效配置。
```

## 数据层分工

### 共享层

```text
public.memory_items
```

用途：

```text
跨用户共享的游戏画像
原始 legacy 游戏资料段落
共享正负面索引
通用游戏事实
共享游戏资料导入记录
```

当前共享层已核实的索引 item_type：

```text
positive_reference_index
negative_decision_index
```

不要把共享层负面索引机械改名为 `negative_reference_index`，除非另起迁移方案并明确处理历史数据。

### 用户层

```text
public.memory_users
public.user_preference_items
```

用途：

```text
用户身份键
用户显示名
用户自己的稳定偏好
用户自己的已玩 / 通关 / 退款 / 放弃记录
用户自己的强正面 / 强负面参考
用户自己的游戏反馈覆盖
```

当前用户偏好层已核实的常用 item_type：

```text
user_profile_meta
stable_preference
game_feedback_overlay
positive_reference_index
negative_reference_index
played_record
played_record_library_meta
played_record_schema
legacy_personal_raw_section_archive
```

`positive_reference` / `negative_reference` 不是当前用户偏好层规范 item_type。payload 内部历史词汇可以保留，不做机械替换。

### 用户场景层

```text
public.user_scenario_items
```

用途：

```text
某用户在某场景下对某游戏的状态
推荐 / 待查 / 等待 / 排除 / 低优先 / 参考 / 基准线
场景理由
候选审计结果
waiting 复查状态
最近核查时间
```

数据库字段：

```text
state
```

不要把数据库字段写成 `status`。中文可以说“状态”，但涉及数据库字段时必须使用 `state`。

## 当前初始用户

```text
user_key: owner_zhengkun
display_name: 郑昆
```

该用户是原单用户项目迁移后的 owner 偏好层。

## 推荐读取流程

给某个用户做推荐时：

```text
1. 读取 GitHub 项目规则。
2. 执行 profile_routing 并确认 user_key。
2a. 按 `rules/reporting/weekly_report.md` 预检该用户的周报配置；失败不阻断推荐。
3. 执行 scenario_routing 并确认 scenario_code。
3a. scenario 确认后，才判断该场景是否纳入周报详细搜索范围。
4. 读取对应场景类型模板和用户场景快照。
5. 查询 shared game profile: public.memory_items。
6. 查询 user preference overlay: public.user_preference_items where user_key = <user_key>。
7. 查询 user scenario state: public.user_scenario_items where user_key = <user_key> and scenario_code = <scenario_code>。
8. 合并当前对话反馈。
9. 涉及当前事实时联网核查。
```

## 合并优先级

```text
当前对话明确反馈
用户层场景状态
用户层稳定偏好和反馈覆盖
用户层 played_record
共享游戏画像
共享正负面索引
外部当前事实核查
```

注意：外部当前事实用于客观结构、版本、价格、联机机制和近期评价；用户主观偏好仍以用户层记录为准。

## 写入规则

### 新用户

写入：

```text
public.memory_users
```

建议 user_key：

```text
u_<短英文或数字标识>
```

不要使用容易变化的昵称作为唯一键。昵称可放 display_name。

### 用户稳定偏好、反馈和游玩记录

写入：

```text
public.user_preference_items
```

常用 item_type：

```text
user_profile_meta
stable_preference
game_feedback_overlay
positive_reference_index
negative_reference_index
played_record
```

item_key 示例：

```text
current
pref:coop_pve
feedback:warframe
played:x4_foundations
```

### 用户场景状态

写入：

```text
public.user_scenario_items
```

唯一键逻辑：

```text
user_key + namespace + scenario_code + game_key
```

示例：

```text
user_key: owner_zhengkun
scenario_code: pc_console_coop
game_key: x4_foundations
state: reference_only
```

## 与旧规则的关系

旧规则中已经区分：

```text
project_profile_delta = 跨场景稳定变化
scenario_conclusion_delta = 当前场景结论变化
```

多用户后，这两类 delta 都必须再加 user_key：

```text
user_project_profile_delta
user_scenario_conclusion_delta
```

也就是说：

```text
A 用户不喜欢某游戏，不代表 B 用户也不喜欢。
A 用户在 PC/主机联机场景排除某游戏，不代表 A 用户在单人场景也排除。
```

## 不变规则

```text
同一游戏可以在不同场景有不同结论。
同一游戏可以在不同用户下有不同结论。
共享游戏画像不应写入某个用户的主观排序。
用户场景状态不应反写成共享游戏事实。
```

## 历史数据处理

```text
不做 Supabase 数据迁移。
不改历史 item_type。
不机械替换 payload 内部历史词汇。
文档只明确当前已核实命名和后续写入规范。
```
