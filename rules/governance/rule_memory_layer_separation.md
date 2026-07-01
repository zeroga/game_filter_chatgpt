# Rule / Memory Layer Separation

## 三层结构

```text
规则层 = rules/
记忆层 = memory/
数据层 = Supabase chatgpt_memory
```

## 规则层职责

`rules/` 保存规则真源，包括：

```text
读取顺序
运行规则
保存流程
推荐流程
feedback intake 流程
profile / scenario 路由规则
数据库定位说明
多用户规则
规则治理规则
变更流程
历史规则归档
```

`rules/` 不得保存：

```text
具体游戏画像
用户偏好
用户游玩记录
用户场景状态
候选清单
推荐结论
等待清单
排除清单
用户个人反馈
场景候选审计结果
```

## 记忆层职责

`memory/` 是 GitHub 中的记忆辅助层，只保存：

```text
场景类型模板
用户场景快照
项目状态快照
导入说明
非规则性工作档
```

`memory/` 不得保存：

```text
规则入口
规则治理逻辑
保存流程真源
推荐流程真源
profile / scenario 路由真源
数据库定位真源
多用户规则真源
```

## Supabase 数据层职责

Supabase 保存游戏筛选运行数据，包括：

```text
共享游戏画像
共享游戏事实
用户稳定偏好
用户游玩记录
用户反馈覆盖
用户场景状态
候选状态
等待项
排除项
低优先项
推荐项
来源摘要
事件记录
```

Supabase 不得保存：

```text
规则真源
读取顺序
保存流程
规则治理逻辑
规则文件正文
```

## 硬性禁止

```text
不得把规则层内容写入记忆层。
不得把记忆层内容写入规则层。
不得把用户个人偏好、游戏反馈、推荐结论写入规则文件。
不得把规则治理逻辑、保存流程、读取顺序写入 Supabase 记忆表。
不得用 memory_items / user_preference_items / user_scenario_items 代替规则文件。
不得用 README / current_rules / save_flow 代替游戏记忆数据库。
```
