# Deprecated: Current Scenario

本文件已废弃，不再作为场景规则入口。

原因：场景必须分场景存储，不能用单一 `current_scenario` 表示所有场景。

后续读取场景时，先读场景索引：

```text
memory/scenarios/index.md
```

当前默认场景文件：

```text
memory/scenarios/pc_console_coop.md
```

后续新增场景时，在 `memory/scenarios/` 下新建独立文件，并更新 `memory/scenarios/index.md`。
