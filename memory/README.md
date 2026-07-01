# Memory Layer

`memory/` 是 GitHub 中的记忆辅助层，不是规则真源。

## 可以保存

```text
memory/scenario_types/*.md = 场景类型模板
memory/profiles/<user_key>/scenarios/*.md = 用户场景快照
memory/snapshots/*.md = 项目状态快照和工作档快照
memory/imports/*.md = 导入说明
memory/changelog/*.md = 非规则性变更记录
```

## 不得保存

```text
规则入口
规则治理逻辑
保存流程真源
推荐流程真源
feedback intake 真源
profile / scenario 路由真源
数据库定位真源
多用户规则真源
```

正式规则入口见：

```text
rules/current_rules.md
```

## 导航索引边界

```text
memory/index.json = 导航索引，不是规则真源。
```

如果 `memory/index.json`、`memory/snapshots/*.md` 与 `rules/current_rules.md` 冲突，必须以 `rules/current_rules.md` 为准。
