# Claude Role Harness — Indie Game Dev (Godot)

一套用于"每个 Claude session 扮演一个固定 role"的协作规范,面向 **solo 独游开发**。

## 核心思想

每个 role 都在**独立 session** 里运行,session 之间不共享对话上下文,所以
role 之间不是靠对话协作,而是靠**产物(artifact)**协作。

两类 role 性质不同,务必分清:
- **能自己交付的 role**(coding 四件套):写出代码 = 干完活。
- **规格 + 指导 + 验收型 role**(Art Spec、Engine Integrator):Claude 自己画不了图、
  也进不了 Godot 编辑器点 prefab,所以它们产出的是 **规格 / 可执行步骤 / 验收判据**,
  需要你执行后**把结果(截图 / 报错 / Inspector 状态)回报**,role 再判定对错。
  这一层"回报闭环"是它们能用的前提。

## 完整管线

```
Producer (常驻,管范围) ── BACKLOG.md ──────────────────────────────┐
                                                                    │ 决定做什么/先做什么
Game Designer ── FEATURE-DESIGN.md ──┐                              │
                                     v                              │
project-context.md (常驻,人人读)     Planner ── PLAN.md ── Implementer ── CHANGES.md(含 Wiring Contract) + 代码
        │                              ^                                   │
   Explorer ── CONTEXT-FINDINGS.md ────┘                                   v
        (进入陌生代码时)                                            Reviewer ── REVIEW.md
                                                                            │
Art Spec ── STYLE-BIBLE.md / ASSET-SPEC.md ── (你产出资源) ── ACCEPTANCE.md │
        │                                                                   │
        └──────────────┬────────────────────────────────────────────────┘
                       v
        Engine Integrator (Godot) ── INTEGRATION-STEPS.md ── (你在编辑器里照着接)
```

关键接缝:**Implementer 的 CHANGES.md 里有一节 Wiring Contract**(新脚本暴露了哪些
`@export` 字段、发什么 signal、需要哪些 autoload / input action / group)。Engine
Integrator 正是把这份契约 + 资源规格,翻译成 Godot 编辑器里一步步的接线清单。
没有这节,Engine Integrator 只能猜。

每个 role 只对自己的产物负责,但**永远允许、且应当上报**越界发现的问题(escalation),
而不是闭眼把上游的错误执行完。

## 怎么用

1. 先填 `project-context.md`——所有 role 的"共享内存"(技术栈写 Godot 版本、
   GDScript/C#、目录约定、禁忌)。
2. 开新 session 时,把对应 `roles/<分类>/role-*.md` 作为首条指令贴进去,附上该 role
   `<inputs>` 里要求的 artifact。
3. role 产出 artifact,你带到下一个 session。规格型 role 还要把你的执行结果回报给它验收。

## Artifact 命名约定(session 之间真正的"共享状态")

| 文件 | 产出者 | 说明 |
|------|--------|------|
| `project-context.md`     | 你(人工维护) | 常驻;Godot 版本、约定、禁忌 |
| `BACKLOG.md`             | Producer        | 范围与优先级,常驻维护 |
| `FEATURE-DESIGN.md`      | Game Designer   | 这个机制要带来什么体验 |
| `CONTEXT-FINDINGS.md`    | Explorer        | 现状勘探(进陌生代码时) |
| `PLAN.md`                | Planner         | 已批准的实施计划 |
| `CHANGES.md`             | Implementer     | 改动摘要 + 代码 + **Wiring Contract** |
| `REVIEW.md`              | Reviewer        | 审查结论与必改项 |
| `STYLE-BIBLE.md`         | Art Spec        | 风格圣经,常驻维护 |
| `ASSET-SPEC.md`          | Art Spec        | 单次资源规格 + 生成 prompt |
| `ACCEPTANCE.md`          | Art Spec        | 对你交付资源的验收结论 |
| `INTEGRATION-STEPS.md`   | Engine Integrator | Godot 编辑器里的接线清单 |

这些 artifact 建议随仓库一起纳入版本控制——每一步决策留痕,出问题能回溯。

## 建议的起手顺序(别一上来全铺开)

solo dev 的瓶颈是你切换 role 的精力,不是 role 数量。先把**一个**功能完整跑通一遍:

```
Game Designer → Planner → Implementer → Reviewer → Art Spec → Engine Integrator
```

进陌生代码再加 Explorer;确认这条链顺了,再把 **Producer** 拉起来常驻管范围
(独游死于 scope creep 远多于死于代码烂)。之后按实际痛点扩 Debugger / Game Feel /
Audio 等。

## 文件清单

- `README.md` ........................ 本文件
- `Handoff.md` ...................... session 间交接说明
- `project-context.md` .............. 共享上下文模版(先填这个)
- `role-template.md` ................ 通用 role 模版(造新 role 时复制)
- `roles/direction/` ............... 决定做什么 / 什么体验
  - `role-producer.md`
  - `role-game-designer.md`
- `roles/engineering/` ............. coding 四件套(自交付型)
  - `role-explorer.md`
  - `role-planner.md`
  - `role-implementer.md` .......... 已更新:新增 Wiring Contract
  - `role-reviewer.md`
- `roles/production/` .............. 规格 + 验收型(需回报闭环)
  - `role-art-spec.md`
  - `role-engine-integrator-godot.md` ... Godot 专版

## 还可以加的 role(按需,目前未生成)

- **Game Feel / Playtest** — 把"手感差"翻成可调参数(coyote time、输入缓冲、
  screen shake、tween 曲线、相机阻尼),你调完回报手感再迭代。
- **Audio** — 与 Art Spec 同构;初期可并进 Art Spec,音频量大了再独立。
- **Debugger** — 假设驱动、先取证据再改,与 Implementer 思维不同。
- **Narrative / Writer** — 仅叙事驱动的游戏需要。
