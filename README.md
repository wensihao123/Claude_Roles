# Claude Role Harness

一套用于"每个 Claude session 扮演一个固定 role"的协作规范。

## 核心思想

因为每个 role 都在**独立 session** 里运行,session 之间不共享对话上下文,所以
role 之间不是靠对话协作的,而是靠**产物(artifact)**协作的:

```
Explorer  ──> CONTEXT-FINDINGS.md ─┐
                                   ├──> Planner ──> PLAN.md ──> Implementer ──> CHANGES.md + 代码
project-context.md (常驻,人人读) ──┘                                              │
                                                                                  v
                                                            Reviewer <── REVIEW.md
```

每个 role 只对自己的产物负责,但**永远允许、且应当上报**它发现的越界问题
(escalation),而不是闭眼把上游的错误执行完。

## 怎么用

1. 把 `project-context.md` 填好——这是所有 role 的"共享内存",放技术栈、
   目录约定、命名规范、禁止事项。**改一处,所有 role 受益。**
2. 开一个新 Claude session 时,把对应的 `roles/role-*.md` 作为系统/首条指令贴进去,
   再附上该 role 需要读的 artifact(见每个文件的 `<inputs>`)。
3. role 干完后产出它的 artifact(见 `<outputs>`),你把 artifact 带到下一个 session。

## Artifact 命名约定(这是 session 之间真正的"共享状态")

| 文件 | 产出者 | 说明 |
|------|--------|------|
| `project-context.md`   | 你(人工维护) | 常驻;技术栈、约定、禁忌 |
| `CONTEXT-FINDINGS.md`  | Explorer     | 现状勘探报告 |
| `PLAN.md`              | Planner      | 已批准的实施计划 |
| `CHANGES.md`           | Implementer  | 本次改动摘要 + 代码 |
| `REVIEW.md`            | Reviewer     | 审查结论与必改项 |

建议把这些 artifact 放在仓库的 `./.harness/` 目录下,并纳入版本控制——
这样每一步的决策都留痕,出问题能回溯。

## 建议的迭代节奏

先只用 **Planner + Implementer + Reviewer** 三个跑一阵(进入陌生代码库时再加
Explorer)。确认交接顺畅后,再按需扩 Debugger / Tester 等。**不要一上来就铺满
role**,否则维护和切换成本会超过收益。

## 文件清单

- `README.md` ............ 本文件
- `project-context.md` ... 共享上下文模版(先填这个)
- `role-template.md` ..... 通用 role 模版(造新 role 时复制它)
- `roles/role-explorer.md`
- `roles/role-planner.md`
- `roles/role-implementer.md`
- `roles/role-reviewer.md`
