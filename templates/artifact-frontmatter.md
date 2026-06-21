# Artifact Frontmatter 约定

每个 **per-feature** artifact(FEATURE-DESIGN / CONTEXT-FINDINGS / PLAN / CHANGES /
REVIEW / ASSET-SPEC / ACCEPTANCE / INTEGRATION-STEPS)文件**开头**都带这段 YAML
frontmatter,让任何冷启动的 session 一眼知道来历、状态、下一棒交给谁:

```yaml
---
artifact: PLAN                 # 类型:FEATURE-DESIGN | CONTEXT-FINDINGS | PLAN |
                               #       CHANGES | REVIEW | ASSET-SPEC | ACCEPTANCE |
                               #       INTEGRATION-STEPS
feature: 01-double-jump        # 所属功能 slug,= 目录名
role: Planner                  # 产出这份 artifact 的 role
status: draft                  # draft | accepted | superseded | blocked
updated: 2026-06-16            # 最后更新日期
inputs: [FEATURE-DESIGN.md, CONTEXT-FINDINGS.md]   # 实际消费的上游 artifact
next: Implementer              # 下一棒应由谁接手
---
```

字段语义:
- `status`:`draft` 刚产出待验收;`accepted` 人类/下游已采纳;`blocked` 卡在某个
  flag;`superseded` 被新版本取代(保留留痕,不删)。
- `inputs`:写你**真的读过并据以决策**的上游文件,不是"理论上相关"的。
- `next`:配合 HANDOFF.md 的"下一步",让交接无需口头说明。

**标准** 文件(project-context.md / BACKLOG.md / STYLE-BIBLE.md)是常驻、跨功能的,
不走这套 per-feature frontmatter,只需保留一行 `updated: YYYY-MM-DD`。

## 状态 vs 历史(防上下文噪音)

项目跑久了,文件会把「当前状态」和「历史账本」混在一起,role 整篇读就吸进噪音。
两条硬约定:

1. **状态在前、历史在后/外;读取时先看状态,需要历史才往下钻。**
   - **状态**=回答"现在是什么样、下一步做什么"的部分,小而新,role 必读:
     HANDOFF 的「管线状态 + 下一步」、BACKLOG 的「v1 scope + Now/Next」、事实源的当前形态。
   - **历史/账本**=只增的留痕,默认不读、要追溯才读:HANDOFF 的「决策记录」、
     BACKLOG 的「Decision log / Cut」、各 `*-CHANGE-NN.md`、CHANGES.md。把它们放在文件
     底部或单独文件里,别插在状态前面。
   - 写状态部分时**就地更新**(覆盖旧值),不要用追加日志的方式记当前状态。
   - 历史部分定期滚动收口:陈旧、不再有约束力的条目压成一句话摘要或随功能归档。
2. **活的事实源只写当前真相。** ARCHITECTURE.md / BALANCE.md / STATE-MACHINES.md /
   UX-MAP.md 只描述"现在长什么样",**不在里面记变更史**;每次变更的诊断与取舍归各自的
   `harness/{arch,balance,state,ux}/*-CHANGE-NN.md`。事实源要小到能一眼看清现状。
3. **功能 done 后归档。** `harness/features/<NN-slug>/` 整目录挪到
   `harness/archive/<NN-slug>/`;role 默认只扫 `harness/features/`,归档的出读取范围、
   仍在版本控制里。详见 README「防上下文噪音」。
