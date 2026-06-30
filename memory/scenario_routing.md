# Scenario Routing

本文件定义 scenario code 的确认、查找、创建和真源边界。

## 核心原则

```text
没有默认场景。
scenario code 必须在当前对话中确认。
不能因为某个 profile 只有一个已知场景就自动使用该场景。
不能把自然语言场景描述直接当成新 scenario code。
不能不做二次确认就创建新场景。
```

## 场景相关对象分工

```text
memory/scenario_types/*.md = 场景类型模板。
memory/profiles/<user_key>/scenarios/*.md = 用户个人场景快照。
public.user_scenario_items = 用户场景状态真源。
```

场景类型模板不是实际场景。用户个人场景快照只是可读快照，不是状态真源。

## 确认流程

在任何推荐、筛选、更新清单或解释场景状态之前，必须确认：

```text
profile code
scenario code
```

profile code 的确认由 `memory/profile_routing.md` 处理。拿到已确认的 `user_key` 后，才允许进入 scenario code 确认。

如果缺 scenario code，询问：

```text
这次要使用哪个场景？例如 pc_console_coop。没有明确场景时不能开始游戏推荐。
```

拿到 scenario code 或自然语言场景描述后，按以下顺序处理：

```text
1. 查询已有 memory/scenario_types/*.md。
2. 查询该 user_key 下已有 memory/profiles/<user_key>/scenarios/*.md 快照。
3. 必要时查询 public.user_scenario_items 中该 user_key 已有 scenario_code。
4. 如果输入像已有场景或自然语言别名，先提示可能匹配的已有场景。
5. 用户确认后，回显本次使用的 scenario_code。
6. 用户确认前，不能读取或写入该场景的个人状态。
```

回显格式：

```text
本次使用的 scenario code 是 <scenario_code>。请确认。
```

## 新 scenario 创建规则

只有同时满足以下条件，才允许进入新 scenario 创建流程：

```text
1. profile code 已确认，并已得到 user_key。
2. 已查询已有 scenario_types、用户场景快照和必要的 user_scenario_items。
3. 已提示可能相近或容易混淆的已有场景。
4. 用户明确确认不是已有场景。
5. 用户明确确认要创建新 scenario_code。
```

创建新场景前必须回显：

```text
将创建新 scenario_code: <scenario_code>
将新增或使用场景类型模板: memory/scenario_types/<scenario_code>.md
实际场景状态仍写入 public.user_scenario_items
请确认。
```

## 写入边界

新增或修改场景类型模板属于 GitHub 存档，必须执行 `memory/save_flow.md`。

实际场景状态写入：

```text
public.user_scenario_items
唯一键：user_key + namespace + scenario_code + game_key
数据库字段：state
```

不要把数据库字段写成 `status`。中文可以说“状态”，但涉及数据库字段时必须使用 `state`。

可选导出的 GitHub 场景快照：

```text
memory/profiles/<user_key>/scenarios/<scenario_code>.md
```

快照只作读取辅助，不替代 `public.user_scenario_items`。

## 禁止行为

```text
不能默认使用任何 scenario code。
不能用上一次对话的 scenario code 自动续接新对话。
不能把 memory/scenario_types/index.md 或类型模板列表当作默认场景。
不能把 memory/profiles/<user_key>/scenarios/ 下已有文件当作默认场景。
不能把用户自然语言场景描述直接创建成新 scenario。
不能在用户未确认前读取或写入该场景个人状态。
不能把用户实际场景状态写入 memory/scenario_types/。
不能用 GitHub 场景快照代替 public.user_scenario_items。
```
