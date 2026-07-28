# Database Positioning

当前数据库定位：

```text
Supabase chatgpt_memory = 游戏筛选项目的数据运行库
```

它分三层：

```text
public.memory_items = 共享游戏资料层
public.profile_aliases + public.memory_users = profile 路由层
public.user_preference_items + public.user_scenario_items = 用户个人偏好与个人场景层
public.user_report_subscriptions = profile 级周报配置与运行快照层
```

## 共享层

```text
public.memory_items
```

保存跨用户共享的游戏画像、原始段落、共享正负面索引和共享游戏事实。

不要保存某个用户的实际推荐、等待、排除、已玩反馈或个人排序。

## profile 路由层

```text
public.profile_aliases
```

保存：

```text
code_norm -> user_key
```

其中：

```text
code_norm = lower(trim(profile_code))
```

当前已知：

```text
zero -> owner_zhengkun
zhengkun -> owner_zhengkun
```

## 用户层

```text
public.user_preference_items
public.user_scenario_items
```

保存用户自己的稳定偏好、反馈覆盖、已玩记录，以及某个用户在某个场景下对某个游戏的状态。

唯一状态键：

```text
user_key + namespace + scenario_code + game_key
```

## GitHub 对应关系

```text
memory/scenario_types/*.md = 场景类型模板，不是场景。
memory/profiles/<user_key>/scenarios/*.md = 用户个人场景快照。
```

周报业务配置、调度映射、同步状态和上期快照存放在 `public.user_report_subscriptions`。实际定时计划存放在 ChatGPT Automations；规则正文仍只存放在 `rules/`，周报配置不得代替用户偏好或场景状态。

## 推荐前提

游戏推荐前必须先确认：

```text
profile code
scenario code
```

不能默认用户，不能默认场景。

## 数据结构说明入口

完整 Supabase 表结构、常用字段、`item_type` / `state` 命名纠偏和写入边界见：

```text
rules/data/schema_notes.md
```
