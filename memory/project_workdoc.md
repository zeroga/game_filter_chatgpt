# 游戏筛选项目工作档

更新日期：2026-07-01（JST）

## 1. 当前定位

本仓库用于保存 ChatGPT 可读取的项目工作档记忆，不保存完整游戏画像数据库。

当前分工：

| 层 | 职责 |
|---|---|
| GitHub `zeroga/game_filter_chatgpt` | 工作档、当前状态、读写规则、数据库说明、变更记录 |
| Supabase `chatgpt_memory` | 游戏画像数据主库，供 ChatGPT / GPT Action 直接查询和写入 |
| 上传 TXT / SQL 文件 | 一次性迁移源或备份，不作为日常主读取源 |

## 2. 当前规则基线

项目规则基线为：

```text
v4.6_game_filter_preference_library_machine.txt
```

其角色为：

```text
rules_schema_only
```

关键规则：

- v4.6 只定义规则、schema、状态机、审计要求、输出约束。
- 完整游戏画像不存放在规则文件中。
- 游戏画像数据必须从数据库读取。
- 自然语言说明只是注释，机器字段优先。
- silent load 只控制外部输出，不减少内部读取、索引、审计和自检。
- 推荐目标是上限，不是填满义务；干净候选不足时保留空位。
- 推荐前必须执行候选审计。
- 若缺游戏数据库或候选审计，阻断推荐。

## 3. 当前数据主库

数据主库：Supabase `chatgpt_memory`

```text
project_ref: zpkfrbfgaaojrblcqvvl
schema: public
namespace: game_filter
main_table: memory_items
event_table: memory_events
import_batch_table: memory_import_batches
```

已迁入：

```text
v1.0_game_filter_game_profile_database_machine.txt
```

导入结果：

```text
memory_items: 404 rows
structured_profile_record: 368 rows
raw_legacy_profile_section: 28 rows
memory_events: 1 row
memory_import_batches: 1 row
```

## 4. 直接连库读写原则

日常游戏筛选时：

1. 先读取本工作档入口和当前状态。
2. 按规则确认需要的数据范围。
3. 直接查询 Supabase `public.memory_items`。
4. 同时查询结构化记录和 `raw_text`，避免结构化字段遗漏旧结论。
5. 只在涉及当前事实、价格、版本、评价、发售状态、DLC、联机结构时联网核查。
6. 不把数据库全量内容回写 GitHub。

## 5. 写入原则

写入 GitHub：

- 项目状态变化
- 当前任务续接点
- 规则说明变化
- 数据库结构说明变化
- 变更日志

写入 Supabase：

- 游戏画像
- 场景条目
- 待查任务
- 等待条件
- 用户反馈覆盖
- 来源摘要

禁止写入：

- OAuth token
- API key
- service role key
- 私密账号信息
- 真实身份信息
- 不应公开的用户隐私

## 6. 已验证样本

数据库直连已验证可用。样本查询结果包括：

- `Warframe`：正面参考，长线 PvE / Mod / 武器和战甲收集。
- `No Man's Sky`：单人正面、联机同步负面。
- `Stolen Realm`：用户已拒绝，负面参考。
- `Inkbound`：当前正面观察。
- `Granblue Fantasy: Relink`：已通关等待大型内容。
- `X4: Foundations`：单人太空 / 舰队经营强正面；因无官方联机，不进入 PC/主机联机场景推荐。

## 7. 安全边界

`chatgpt_memory` 为降低直连授权摩擦，当前 RLS 有意关闭。

这意味着：

- 数据库只应存放可公开或低风险的游戏筛选数据。
- 不应存放密钥、token、私密账户数据或不可公开的个人信息。
- GitHub 中不保存 Supabase publishable key 或 anon key。

## 8. 下次续接方式

新对话继续游戏筛选时，优先读取：

```text
memory/index.json
memory/current_state.md
memory/schema_notes.md
```

然后查询 Supabase：

```sql
select *
from public.memory_items
where namespace = 'game_filter'
  and item_type in ('structured_profile_record', 'raw_legacy_profile_section');
```

如果是特定游戏，优先查：

```sql
select *
from public.memory_items
where namespace = 'game_filter'
  and (
    title ilike '%<game_name>%'
    or raw_text ilike '%<game_name>%'
    or payload::text ilike '%<game_name>%'
  );
```
