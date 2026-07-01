# Supabase 数据说明

本文件的旧版数据库定位已过时。

当前有效数据库定位以以下文件为准：

```text
rules/data/database_positioning.md
```

关键修正：

```text
Supabase chatgpt_memory 不是单纯的游戏画像主库。
它是游戏筛选项目的数据运行库。
```

当前分层：

```text
public.memory_items = 共享游戏资料层
public.profile_aliases + public.memory_users = profile 路由层
public.user_preference_items + public.user_scenario_items = 用户个人偏好与个人场景层
```

后续不要把用户个人场景状态写入 `public.memory_items`。

用户个人场景状态应写入：

```text
public.user_scenario_items
```

共享游戏资料仍写入：

```text
public.memory_items
```
