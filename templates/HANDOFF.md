---
feature: NN-slug                 # e.g. 01-double-jump
status: in-progress              # backlog | in-progress | done | parked
updated: YYYY-MM-DD
---
# HANDOFF — <Feature 名>

> 每个功能一份,放在 `harness/features/<NN-slug>/HANDOFF.md`。
> 它是这个功能的"单一事实来源":人类只看它就知道走到哪、下一步开哪个 role session。
> 每个 role 干完活必须更新自己那一行的状态 + "下一步"。
>
> 读取顺序:**「管线状态」+「下一步」是必读状态**;「决策记录」「未决 flags」是参考/账本,
> 需要追溯才往下看。功能 `done` 后,整目录(含本文件)挪进 `harness/archive/`。

## 管线状态
| 阶段 | Role | Artifact | 状态 |
|------|------|----------|------|
| 设计 | Game Designer | FEATURE-DESIGN.md | [ ] |
| 勘探 | Explorer | CONTEXT-FINDINGS.md | [ ] |
| 计划 | Planner | PLAN.md | [ ] |
| 实现 | Implementer | CHANGES.md | [ ] |
| 审查 | Reviewer | REVIEW.md | [ ] |
| 美术 | Art Spec | ASSET-SPEC.md / ACCEPTANCE.md | [ ] |
| 接线 | Engine Integrator | INTEGRATION-STEPS.md | [ ] |

> 状态记号(与 PLAN 步骤、REVIEW must-fix 同一套,全 harness 一致):
> `[ ]` 未开始 · `[~]` 进行中(含产出 draft、含被打回返工中) · `[x]` 完成或确认不需要(此阶段已了结)。
> Implementer 跑分阶段 PLAN 时可在「实现」行后缀阶段进度,如 `[~] phase 2/3`
> (细到步骤看 PLAN.md 的 `[~]/[x]`)。
> 阻塞不是第四种状态:阶段留在 `[~]`,把阻塞项写进下面「未决 flags」。
> 各 role 收尾自己这行的映射:artifact 还没动 = `[ ]`;在产出 / 还是 draft / 被打回返工 = `[~]`;
> artifact 定稿(accepted)或本阶段确认不需要 = `[x]`。(artifact 文件自身 frontmatter 的
> `status: draft|accepted|…` 是它自己的生命周期,与这里的行记号分开,别混。)

## 验收回环(三处人机来回 · 写死,严格照走)
> 会来回跑的环有三处,结构相同:**出规格/产物 → 对方执行 → 验收 → 不过则「带记号的待办 + 退回」→ 复验 → 闭合**。
> 通用闭合原则:验收方那一行 `[x]` 当且仅当**没有 open 待办**——待办用 `[ ]/[~]/[x]`,执行方逐条翻 `[x]`,复验只核待办是否真消 + 有无新问题。环可多轮。
>
> 1. **实现 ↔ 审查**:Implementer 完工 → 实现 `[x]`、审查 `[ ]`、开 Reviewer。Reviewer 出 REVIEW.md:
>    APPROVE / APPROVE WITH NITS → 审查 `[x]`;REQUEST CHANGES → 审查 `[~]`、实现退回 `[~]`,
>    must-fix 每条带记号当返工单,Implementer **只针对 must-fix** 逐条改完 → 实现 `[x]`、回 Reviewer 复审。
> 2. **美术 ↔ 人**:Art Spec 出 ASSET-SPEC →(image-prompt 编译)→ 你出资源 → Art Spec 验收出 ACCEPTANCE
>    (每条 fail 带记号)。全 pass → 美术 `[x]`;有 fail → 美术 `[~]`,你按 fix 重做资源 → 回 Art Spec 复验。
> 3. **接线 ↔ 人**:Engine Integrator 出 INTEGRATION-STEPS(每组带 Verify、末尾 Run & expected)→ 你在编辑器照做并回报。
>    全通过 → 接线 `[x]`;某步失败 → 接线 `[~]`:编辑器侧步骤错就改步骤、你再做;若暴露代码缺口
>    (Wiring Contract 漏项)则 flag back Implementer(退回实现/审查),别在编辑器糊。

## 功能完成判据(写死)
> 所有**需要的**阶段都 `[x]`,且审查 verdict 非 REQUEST CHANGES。
> 哪些阶段不需要:**Planner 在 PLAN 阶段一次裁定**——把本 feature 不走的可选阶段(勘探/美术/接线)
> 直接标 `[x]` 并在行内注「不需要」,使判据可达(别留 `[ ]` 卡住完成)。
> 满足时:完成最后一个必需阶段的那个 role 把 frontmatter `status` 翻 `done`,
> 「下一步」写「功能完成 → Producer 归档 + 记 Shipped」。归档动作由 Producer 做。

## 下一步
<开哪个 role、喂哪些 artifact。例:开 `/role-planner NN-slug`,喂 FEATURE-DESIGN.md>

## 决策记录(账本·按需读)
> 只记**仍有约束力**的关键决策;被推翻/过期的删掉或压成一句话。别让它无限长。
- YYYY-MM-DD <关键决策 + 来源 artifact>

## 未决 flags
- <从各 artifact 的 "Flags / Open questions" 汇总的阻塞项;**解决了就删**,清空了写"无">
