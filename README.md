# game_filter_chatgpt

本仓库是“游戏筛选”项目给 ChatGPT 使用的固定工作档入口。

## 后续读取方式

新对话中只需要要求：

```text
读取 GitHub 仓库 zeroga/game_filter_chatgpt
```

然后按以下顺序读取：

```text
1. memory/current_rules.md
2. memory/scenarios/index.md
3. 按场景索引读取具体场景文件，例如 memory/scenarios/pc_console_coop.md
4. memory/schema_notes.md
5. memory/current_state.md
6. memory/project_workdoc.md
7. 必要时读取 memory/changelog/*.md
```

## 重要分工

```text
GitHub = 当前规则、分场景规则、工作档、数据库说明、当前状态
Supabase = 游戏画像数据主库、后续场景条目
```

完整游戏画像不存放在 GitHub。游戏数据应通过 Supabase 查询：

```text
Supabase project_ref: zpkfrbfgaaojrblcqvvl
schema: public
table: memory_items
namespace: game_filter
```

## 当前规则入口

```text
memory/current_rules.md
```

旧的版本化规则文件已经降级为来源归档：

```text
memory/rules/v4.6_game_filter_preference_library_machine.json
```

后续不要求用户指定规则版本号。

## 场景入口

```text
memory/scenarios/index.md
```

当前默认场景：

```text
memory/scenarios/pc_console_coop.md
```

`memory/current_scenario.md` 已废弃，仅保留跳转说明。

单个游戏的场景状态后续应写入 Supabase 的 `scenario_entry` 类型。

## 当前数据库基线

```text
v1.0_game_filter_game_profile_database_machine.txt
已迁入 Supabase memory_items，共 404 条。
```

## 存储边界

GitHub 不保存完整游戏画像数据库，也不保存私密凭据或账号身份材料。
