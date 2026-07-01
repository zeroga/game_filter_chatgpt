# Rule Change Issue Only

规则层变更只能 issue 化。

## 规则层变更定义

以下都属于规则层变更：

```text
修改 rules/ 下任何文件
修改 README.md 中的规则入口、规则层说明或治理边界
修改读取顺序
修改保存流程
修改推荐流程
修改 feedback intake 流程
修改 profile / scenario 路由规则
修改数据库定位规则或多用户规则
新增、删除或迁移规则治理文件
```

## ChatGPT 普通对话允许行为

当用户提出规则层变更时，ChatGPT 只能：

```text
1. 识别这是规则层变更。
2. 解释不能直接修改规则文件，也不能创建规则修改 PR。
3. 整理需求。
4. 输出 GitHub issue 草案。
5. 在用户明确要求创建 issue 时，创建 GitHub issue。
```

## ChatGPT 普通对话禁止行为

```text
不得直接修改 rules/ 下文件。
不得直接修改规则入口文件。
不得创建用于修改规则文件的 PR。
不得把规则层变更夹带进记忆层保存。
不得把用户确认保存解释为允许写规则文件。
不得把二次确认解释为允许写规则文件。
不得用 update_file / create_file / delete_file 等工具直接变更规则层文件。
```

## Codex / 人工例外

Codex 或人工可以在明确的 issue 驱动开发任务中修改规则文件并提交 PR。

这种修改必须在 PR 说明中引用对应 issue，并说明完成的规则冲突审计和剩余风险。

## 标准响应

```text
这是规则层变更。
我不能直接修改规则文件，也不能创建规则修改 PR。
我可以把需求整理成 GitHub issue。
请确认是否创建 issue。
```
