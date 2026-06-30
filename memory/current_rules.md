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
memory/current_rules.md
memory/current_scenario.md
memory/schema_notes.md
memory/current_state.md
memory/project_workdoc.md
```

## 规则存储分工

GitHub 保存：

```text
当前规则
当前场景
工作档
数据库说明
状态记录
变更记录
```

Supabase 保存：

```text
游戏画像
原始 legacy 段落
正负面索引
后续场景条目
后续用户反馈覆盖
```

## 通用硬规则

```text
silent load 只减少对外输出，不减少内部读取、索引、审计和自检。
推荐目标是上限，不是必须填满。
缺游戏数据库时停止推荐或更新。
缺候选审计时阻断推荐。
涉及当前事实、版本、价格、评价、联机结构、DLC、EA 状态时必须联网核查。
用户主观反馈优先于外部主观评价；客观结构仍需外部验证。
```

## 候选审计必填项

```text
game_name
aliases_checked
database_lookup_result
negative_index_result
old_scenario_conclusion_result
user_feedback_conflict_check
current_web_fact_check
scenario_hard_condition_check
why_not_waiting
why_not_investigate
why_not_low_priority
why_not_excluded
final_state
```

## 来源优先级

```text
用户亲身体验
当前对话反馈
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
memory/rules/v4.6_game_filter_preference_library_machine.json
```

该文件只作为来源归档，不再作为后续运行入口。
