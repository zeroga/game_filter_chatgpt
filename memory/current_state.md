# 当前状态

更新日期：2026-07-01（JST）

## 当前阶段

项目已完成从“巨大工作档记忆”到“双层记忆”的基础切换：

```text
GitHub = 工作档记忆
Supabase = 游戏数据记忆
```

## 已完成事项

- 已确认 GitHub 仓库：`zeroga/game_filter_chatgpt`。
- 已确认 Supabase 项目：`chatgpt_memory` / `zpkfrbfgaaojrblcqvvl`。
- 已建立 Supabase 记忆表：
  - `public.memory_items`
  - `public.memory_events`
  - `public.memory_import_batches`
  - `public.memory_import_file_chunks`
- 已将 `v1.0_game_filter_game_profile_database_machine.txt` 迁入 Supabase。
- 已验证导入行数：`memory_items = 404`。
- 已读取项目规则 `v4.6_game_filter_preference_library_machine.txt`。
- 已验证数据库直连查询可用。
- 已用数据库形式查询 `X4: Foundations`，确认其结论：单人太空 / 舰队经营强正面，但因无官方联机不进入 PC/主机联机场景推荐。

## 当前主数据源

```text
Supabase project_ref: zpkfrbfgaaojrblcqvvl
schema: public
table: memory_items
namespace: game_filter
```

## 当前规则源

```text
v4.6_game_filter_preference_library_machine.txt
```

当前规则文件未迁入数据库；已在对话中读取，并在本仓库记录为当前项目规则基线。

## 重要约束

- 后续游戏画像查询默认走 Supabase，不回读巨大 TXT。
- GitHub 不保存完整游戏画像，只保存工作档和状态。
- 推荐前必须查旧数据、负面索引、用户反馈冲突和当前事实。
- waiting 项更新必须全量核查；不完整时必须标记 `waiting_info_update_incomplete`。
- 当前数据库 RLS 有意关闭，只允许存放低风险游戏筛选数据。

## 待处理事项

1. 将当前场景档迁入 GitHub 或 Supabase。
2. 明确 `scenario_entry` 的最小 JSON 格式。
3. 为常用查询建立固定 SQL 模板。
4. 后续如有新游戏反馈，应写入 Supabase，并同步写 `memory_events`。
5. 若要开放给其他用户，需要另行设计权限和白名单，不应把真实权限数据放入 RLS 关闭的库。
