# Rule Layer Split Save Policy Integration PR

更新日期：2026-07-01（JST）

## 目的

本文件用于在 PR 分支中形成可审阅差异，推动将已存在的规则补丁正式接入入口链。

已存在规则补丁文件：

```text
memory/rules/rule_layer_split_save_policy_2026_07_01.md
```

## 需要并入的正式入口改动

### 1. current_rules 读取顺序

在 `memory/current_rules.md` 的读取顺序中，`memory/save_flow.md` 后增加：

```text
memory/rules/rule_layer_split_save_policy_2026_07_01.md
```

### 2. current_rules 通用硬规则

在 `memory/current_rules.md` 的通用硬规则块中增加：

```text
数据/记忆层变更和规则层变更必须拆分保存；规则层写入必须在数据/记忆层保存完成后单独确认。
```

### 3. save_flow 正文

在 `memory/save_flow.md` 的“保存流程”后、“基本原则”前，合并 `memory/rules/rule_layer_split_save_policy_2026_07_01.md` 的规则正文。

## 当前说明

此前直接完整替换 `memory/current_rules.md` 和 `memory/save_flow.md` 被工具安全层拦截。本文件作为 PR 中的最小差异，明确记录需要人工或后续工具完成的正式入口集成。