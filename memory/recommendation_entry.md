# Recommendation Entry Rule

游戏推荐没有默认场景。

在任何推荐、筛选、更新清单、解释场景状态之前，必须先确认两个值：

```text
profile code
scenario code
```

如果缺 profile code，先问：

```text
请先给我你的 profile code。大小写不敏感，只用于选择你的个人偏好层。
```

如果缺 scenario code，继续问：

```text
这次要使用哪个场景？例如 pc_console_coop。没有明确场景时不能开始游戏推荐。
```

禁止行为：

```text
不能默认使用 pc_console_coop。
不能因为某个 profile 只有一个已知场景就自动使用该场景。
不能把 memory/scenarios/index.md 的模板列表当作默认场景。
不能把 memory/profiles/<user_key>/scenarios/ 下已有文件当作本轮默认选择。
```

允许行为：

```text
可以列出该 profile 已有场景供用户选择。
可以解释每个 scenario code 的含义。
用户明确说“继续上次场景”时，只有当前对话中已经确认过场景，才可沿用。
```

读取顺序：

```text
1. 确认 profile code。
2. 根据 profile code 查 user_key。
3. 确认 scenario code。
4. 读取 memory/scenarios/<scenario_code>.md 作为模板。
5. 读取 memory/profiles/<user_key>/scenarios/<scenario_code>.md 作为个人场景。
6. 查询 public.memory_items 共享游戏画像。
7. 查询 public.user_preference_items 用户偏好覆盖。
8. 查询 public.user_scenario_items 用户场景状态。
9. 涉及当前事实时联网核查。
```
