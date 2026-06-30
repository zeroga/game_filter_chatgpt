# Supabase 数据记忆说明

更新日期：2026-07-01（JST）

## 1. 数据库定位

Supabase `chatgpt_memory` 是游戏筛选项目的游戏画像数据主库。

```text
project_ref: zpkfrbfgaaojrblcqvvl
schema: public
namespace: game_filter
```

GitHub 不保存完整游戏画像，只保存工作档和数据库使用说明。

## 2. 表结构

### public.memory_items

主数据表。

| 字段 | 用途 |
|---|---|
| `namespace` | 项目命名空间，当前固定为 `game_filter` |
| `item_type` | 数据类型，例如 `structured_profile_record`、`raw_legacy_profile_section` |
| `item_key` | 稳定键，当前结构化画像多用 `record_hash` |
| `title` | 可读标题，通常是游戏名或 section 名 |
| `payload` | 结构化 JSON 数据 |
| `raw_text` | 原始文本，防止结构化提取丢信息 |
| `source_file` | 来源文件 |
| `source_section` | 来源 section |
| `record_hash` | 原始记录 hash |
| `created_at` / `updated_at` | 创建和更新时间 |

### public.memory_events

变更事件表。用于记录导入、upsert、修正、删除等操作。

### public.memory_import_batches

导入批次表。当前记录了 `v1.0_game_filter_game_profile_database_machine.txt` 的导入。

### public.memory_import_file_chunks

预留表。用于未来分块上传或大文件导入。目前为空。

## 3. 当前导入状态

```text
source_file: v1.0_game_filter_game_profile_database_machine.txt
source_version: v1.0
namespace: game_filter
memory_items: 404
structured_profile_record: 368
raw_legacy_profile_section: 28
```

## 4. 常用查询

### 查某个游戏

```sql
select item_type, item_key, title, source_section, record_hash,
       payload,
       raw_text
from public.memory_items
where namespace = 'game_filter'
  and (
    title ilike '%X4%'
    or raw_text ilike '%X4%'
    or payload::text ilike '%X4%'
  )
order by item_type, source_section, title;
```

### 查结构化画像

```sql
select *
from public.memory_items
where namespace = 'game_filter'
  and item_type = 'structured_profile_record'
  and (
    title ilike '%Warframe%'
    or raw_text ilike '%星际战甲%'
  );
```

### 查正面 / 负面索引

```sql
select *
from public.memory_items
where namespace = 'game_filter'
  and item_type in ('positive_reference_index', 'negative_decision_index');
```

### 查导入批次

```sql
select *
from public.memory_import_batches
where namespace = 'game_filter'
order by created_at desc;
```

### 查类型分布

```sql
select item_type, count(*) as rows
from public.memory_items
where namespace = 'game_filter'
group by item_type
order by rows desc, item_type;
```

## 5. 写入约定

后续写游戏画像时，优先使用：

```text
item_type = structured_profile_record
namespace = game_filter
raw_text 必填
payload 必填
source_section 必填
```

如果是当前场景结论，建议后续使用：

```text
item_type = scenario_entry
item_key = <scenario_code>:<game_key>
payload.scenario_code = <scenario_code>
payload.game_key = <game_key>
payload.state = recommended | investigate | waiting | low_priority | excluded | reference_only
```

## 6. 安全边界

当前 RLS 关闭是有意设计，用于降低 ChatGPT / GPT Action 直连授权摩擦。

因此：

- 不写 token。
- 不写 API key。
- 不写 service role key。
- 不写真实身份和私密账号信息。
- 不把 Supabase publishable key 写入 GitHub。
