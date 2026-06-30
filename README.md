# game_filter_chatgpt

本仓库是“游戏筛选”项目给 ChatGPT 使用的工作档入口。

## 后续读取方式

新对话中只需要要求：

```text
读取 GitHub 仓库 zeroga/game_filter_chatgpt
```

然后按以下顺序读取：

```text
1. memory/index.json
2. memory/current_state.md
3. memory/schema_notes.md
4. memory/project_workdoc.md
5. 必要时读取 memory/changelog/*.md
```

## 重要分工

```text
GitHub = 工作档记忆、当前状态、规则入口、数据库说明
Supabase = 游戏画像数据主库
```

完整游戏画像不存放在 GitHub。游戏数据应通过 Supabase 查询：

```text
Supabase project_ref: zpkfrbfgaaojrblcqvvl
schema: public
table: memory_items
namespace: game_filter
```

## 当前规则基线

```text
v4.6_game_filter_preference_library_machine.txt
```

## 当前数据库基线

```text
v1.0_game_filter_game_profile_database_machine.txt
已迁入 Supabase memory_items，共 404 条。
```

## 安全边界

不要把 token、API key、service role key、真实账号、私密身份信息写入本仓库。
