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

1. **铺地基**:把 `templates/project-context.md` 复制到游戏项目的
   `harness/project-context.md` 并填实(Godot 版本、目录约定、禁忌)。这是所有 role
   的共享内存,空着则全员失准。
2. **开功能**:在 `harness/features/` 下为该功能建目录 `<NN-slug>`(如 `01-double-jump`),
   从 `templates/HANDOFF.md` 复制一份 `HANDOFF.md` 进去。
3. **激活 role**:开新 session,用 `/role-<name> <NN-slug>` 命令(见下文"用 /role-* 命令激活")。
   role 会读 `harness/project-context.md` + 该功能目录里它需要的上游 artifact,产出自己的
   artifact,并更新 `HANDOFF.md`。
4. **接力**:把功能目录带到下一个 role 的 session(同一仓库里就是换个 /role 命令)。
   规格型 role(Art Spec / Engine Integrator)还要你把执行结果回报给它验收。

## Artifact 命名约定(session 之间真正的"共享状态")

| 文件 | 产出者 | 位置 | 说明 |
|------|--------|------|------|
| `project-context.md`   | 你(人工维护)     | `harness/`(常驻) | Godot 版本、约定、禁忌 |
| `BACKLOG.md`           | Producer          | `harness/`(常驻) | 范围与优先级 |
| `STYLE-BIBLE.md`       | Art Spec          | `harness/`(常驻) | 风格圣经 |
| `FEATURE-DESIGN.md`    | Game Designer     | `features/<slug>/` | 这个机制要带来什么体验 |
| `CONTEXT-FINDINGS.md`  | Explorer          | `features/<slug>/` | 现状勘探(进陌生代码时) |
| `PLAN.md`              | Planner           | `features/<slug>/` | 已批准的实施计划 |
| `CHANGES.md`           | Implementer       | `features/<slug>/` | 改动摘要 + **Wiring Contract**(代码进 src/) |
| `REVIEW.md`            | Reviewer          | `features/<slug>/` | 审查结论与必改项 |
| `ASSET-SPEC.md`        | Art Spec          | `features/<slug>/` | 单次资源规格 + 生成 prompt |
| `ACCEPTANCE.md`        | Art Spec          | `features/<slug>/` | 对你交付资源的验收结论 |
| `INTEGRATION-STEPS.md` | Engine Integrator | `features/<slug>/` | Godot 编辑器里的接线清单 |
| `HANDOFF.md`           | 每个 role 接力更新 | `features/<slug>/` | 该功能的管线状态 + 下一步 |

（`features/<slug>/` 是 `harness/features/<NN-slug>/` 的简写。）

所有 artifact 都在游戏项目的 `harness/` 下并纳入版本控制——每一步决策留痕,出问题能回溯。

## 每个游戏项目的 harness/ 布局

artifact 不放本规范仓库,而是放在**每个游戏项目自己**的 `harness/` 文件夹(纳入版本控制):

```
<game-project>/
  src/  scenes/  assets/ ...        # 游戏本体(Implementer 改的真实代码在这)
  harness/
    project-context.md             # 常驻·共享内存
    BACKLOG.md                     # 常驻·Producer
    STYLE-BIBLE.md                 # 常驻·Art Spec
    features/
      01-double-jump/              # 每个功能一目录:<NN>-<kebab-slug>
        HANDOFF.md                 # 管线状态 + 交接索引
        FEATURE-DESIGN.md  PLAN.md  CHANGES.md  REVIEW.md
        ASSET-SPEC.md  ACCEPTANCE.md  INTEGRATION-STEPS.md
      02-dash/ ...
```

- 常驻文件跨功能共享,放 `harness/` 顶层。
- 每功能的 artifact 收进 `harness/features/<NN-slug>/`,多个功能并行互不覆盖。
- 真实代码仍进游戏的 `src/` 等;CHANGES.md 只是描述 + Wiring Contract。

## 交接范式(handoff):两层

因为 session 之间不共享上下文,交接全靠文件。两个机制保证下一个 session 能冷启动接手:

1. **每个 artifact 顶部统一 frontmatter** —— 标明 `artifact / feature / role / status /
   updated / inputs / next`,一眼看清来历、状态、下一棒交给谁。完整字段见
   `templates/artifact-frontmatter.md`。
2. **每个功能一份 `HANDOFF.md`** —— 管线状态表 +"下一步"+ 决策记录 + 未决 flags。
   人类只看它就知道功能走到哪、该开哪个 role。**每个 role 干完活必须更新它**(已写进
   各 role 的 workflow,是 done 的一部分)。模版见 `templates/HANDOFF.md`。

## 用 /role-* 命令激活(slug 参数规范)

每个 role 对应一个全局 slash 命令(装在 `~/.claude/`,所有项目可用):

```
/role-producer  /role-game-designer
/role-explorer  /role-planner  /role-implementer  /role-reviewer
/role-art-spec  /role-engine-integrator
```

**参数 = 功能 slug**,即 `harness/features/` 下的目录名:

```
/role-planner 01-double-jump                      # Planner 在 01-double-jump 目录里读写
/role-implementer 01-double-jump 先跑一遍测试再开始   # slug 后面可继续跟自由指令
```

规则:
- **每功能 role**(Game Designer / Explorer / Planner / Implementer / Reviewer / Art Spec
  出 ASSET-SPEC 时 / Engine Integrator)应带 slug 指明工作目录。
- **常驻 role**(Producer、Art Spec 维护 STYLE-BIBLE 时)可不带 slug,只动 `harness/` 顶层文件。
- 不带 slug 且 `features/` 下恰好只有一个 `status: in-progress` 的功能时,role 自动选它;
  有多个或没有则停下来问你(防止写错目录)。
- slug 命名建议 `<两位序号>-<kebab>`,如 `01-double-jump`、`02-dash`,兼顾排序与可读。

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
- `role-template.md` ................ 通用 role 模版(造新 role 时复制)
- `templates/` ..................... 复制进游戏项目的骨架
  - `project-context.md` ........... 共享上下文模版(填好放 `harness/`)
  - `HANDOFF.md` ................... 每功能交接文件模版
  - `artifact-frontmatter.md` ...... artifact frontmatter 字段说明
- `roles/direction/` ............... 决定做什么 / 什么体验
  - `producer.md`
  - `game-designer.md`
- `roles/engineering/` ............. coding 四件套(自交付型)
  - `explorer.md`
  - `planner.md`
  - `implementer.md` ............... 含 Wiring Contract
  - `reviewer.md`
- `roles/production/` .............. 规格 + 验收型(需回报闭环)
  - `art-spec.md`
  - `engine-integrator-godot.md` ... Godot 专版

## 还可以加的 role(按需,目前未生成)

- **Game Feel / Playtest** — 把"手感差"翻成可调参数(coyote time、输入缓冲、
  screen shake、tween 曲线、相机阻尼),你调完回报手感再迭代。
- **Audio** — 与 Art Spec 同构;初期可并进 Art Spec,音频量大了再独立。
- **Debugger** — 假设驱动、先取证据再改,与 Implementer 思维不同。
- **Narrative / Writer** — 仅叙事驱动的游戏需要。
