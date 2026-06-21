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
   的共享内存,空着则全员失准。**完整可勾选步骤见 [`BOOTSTRAP.md`](BOOTSTRAP.md)**
   (含一处易错点:HANDOFF 模版要落到项目内 `harness/templates/`,design-jam 才能自动 seed)。
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
| `INBOX.md`             | 谁都可追加 / Producer 分诊 | `harness/`(常驻) | 中途捕获队列:新想法先丢这,Producer 按优先度分进 BACKLOG |
| `STYLE-BIBLE.md`       | Art Spec          | `harness/`(常驻) | 风格圣经 |
| `FEATURE-DESIGN.md`    | Game Designer     | `features/<slug>/` | 这个机制要带来什么体验 |
| `CONTEXT-FINDINGS.md`  | Explorer          | `features/<slug>/` | 现状勘探(进陌生代码时) |
| `PLAN.md`              | Planner           | `features/<slug>/` | 已批准的实施计划(分阶段·勾选式 `[ ]/[~]/[x]`,每阶段带 Playtest gate) |
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
    INBOX.md                       # 常驻·中途捕获队列(谁都可追加,Producer 分诊后清空)
    STYLE-BIBLE.md                 # 常驻·Art Spec
    features/                      # 只放「活动中」的功能(role 默认只扫这里)
      01-double-jump/              # 每个功能一目录:<NN>-<kebab-slug>
        HANDOFF.md                 # 管线状态 + 交接索引
        FEATURE-DESIGN.md  PLAN.md  CHANGES.md  REVIEW.md
        ASSET-SPEC.md  ACCEPTANCE.md  INTEGRATION-STEPS.md
      02-dash/ ...
    archive/                       # 已完成功能整体挪到这(出默认读取范围,仍纳入版本控制)
      00-player-move/ ...
```

- 常驻文件跨功能共享,放 `harness/` 顶层。
- 每功能的 artifact 收进 `harness/features/<NN-slug>/`,多个功能并行互不覆盖。
- 真实代码仍进游戏的 `src/` 等;CHANGES.md 只是描述 + Wiring Contract。
- 功能 `done` 后整目录挪进 `harness/archive/`(见下文「防上下文噪音」)。

## 交接范式(handoff):两层

因为 session 之间不共享上下文,交接全靠文件。两个机制保证下一个 session 能冷启动接手:

1. **每个 artifact 顶部统一 frontmatter** —— 标明 `artifact / feature / role / status /
   updated / inputs / next`,一眼看清来历、状态、下一棒交给谁。完整字段见
   `templates/artifact-frontmatter.md`。
2. **每个功能一份 `HANDOFF.md`** —— 管线状态表 +"下一步"+ 决策记录 + 未决 flags。
   人类只看它就知道功能走到哪、该开哪个 role。**每个 role 干完活必须更新它**(已写进
   各 role 的 workflow,是 done 的一部分)。模版见 `templates/HANDOFF.md`。

## 功能进度记号 + 验收回环(写死,每功能严格照走)

全 harness 用**同一套三态记号** `[ ]` 未开始 · `[~]` 进行中 · `[x]` 完成或确认不需要——
PLAN 步骤、REVIEW must-fix、ACCEPTANCE 修复项、INTEGRATION-STEPS 步骤、HANDOFF 管线状态表
的每个阶段都用它,一眼可读、可冷启动接手。

会来回跑的环有**三处**,结构相同:出规格/产物 → 对方执行 → 验收 → 不过则「带记号的待办 + 退回」
→ 复验 → 闭合。**通用闭合原则:验收方那一行 `[x]` 当且仅当没有 open 待办**(待办用三态记号,
执行方逐条翻 `[x]`,复验只核待办是否真消 + 有无新问题;环可多轮)。三处分别是:

1. **实现 ↔ 审查**:Reviewer REQUEST CHANGES → 审查 `[~]`、实现退回 `[~]`,must-fix 当返工单;
   Implementer 只改 must-fix(不重跑全 PLAN)→ 实现 `[x]` → Reviewer 复审。APPROVE 才 审查 `[x]`。
2. **美术 ↔ 人**:Art Spec 出 ACCEPTANCE(每条 fail 带记号)→ 你按 fix 重做资源 → Art Spec 复验。
   全 pass 才 美术 `[x]`。
3. **接线 ↔ 人**:Engine Integrator 的 INTEGRATION-STEPS 步骤带记号,你照做回报。某步失败:编辑器侧
   就改步骤重做;若是代码缺口(Wiring Contract 漏项)则 flag back Implementer(退回实现),别在
   编辑器糊。你确认最终 Run 通过才 接线 `[x]`。

**功能完成判据:** 所有需要的阶段都 `[x]` 且 verdict 非 REQUEST CHANGES → 完成最后一个必需阶段的
role 把 frontmatter `status` 翻 `done`,交 Producer 归档。**哪些阶段不需要由 Planner 在 PLAN 阶段
一次裁定**(把跳过的可选阶段直接标 `[x] 不需要`),免得 `[ ]` 永远卡住完成。

## 防上下文噪音(状态优先 + 归档)

项目跑久了,artifact 会把「当前状态」和「历史账本」越堆越混,role 每次冷启动整篇读,
就吸进一堆陈年噪音。两条约定把它压住:

1. **状态在前、历史在后/外;role 先读状态,要历史才下钻。**
   - 每份会长大的文件,顶部是「当前真相」(小、必读),历史/账本沉到底部或单独文件
     (按需读)。HANDOFF 的「管线状态 + 下一步」必读,「决策记录」是参考;BACKLOG 的
     v1 scope + Now/Next 必读,「Decision log」是参考。
   - **活的事实源**(ARCHITECTURE / BALANCE / STATE-MACHINES / UX-MAP)**只写当前形态,
     不在里面记变更史**;每次变更的来龙去脉归各自的 `harness/{arch,balance,state,ux}/
     *-CHANGE-NN.md`。事实源要小到能一眼看清"现在长什么样"。
2. **功能 `done` 后归档。** 把 `harness/features/<NN-slug>/` 整目录挪到
   `harness/archive/<NN-slug>/`。role 默认只扫 `harness/features/`(活动中),归档的出
   读取范围但仍在版本控制里——要翻历史用 git / grep,无损。
   - 谁来归档:人工随手做,或交给常驻的 **Producer** 顺带 compaction(剪 BACKLOG 的
     Decision log、收已发布项、查事实源有没有偷偷嵌历史)。

完整约定见 `templates/artifact-frontmatter.md` 末尾的「状态 vs 历史」一节。

## 中途捕获·延迟分诊(INBOX)

开发途中你总会冒出新需求。过去这逼着你切回 Planner/Producer 去改要求文件,打断在跑的
流程、把 PLAN/HANDOFF 的状态搅乱——这也是文件系统变乱的一个根因。新范式把**「捕获」和
「决定」分开**:

1. **中途只捕获,不改要求。** 你随口提的新想法,当前 role 追加一行到 `harness/INBOX.md`
   (`- [日期][来自<feature>/<role> 或 用户][优先级] <想法>`)、回显给你、继续手头活——
   **不动 FEATURE-DESIGN/PLAN 等要求文件,不中断当前 feature**。你也可以自己直接往 INBOX 追加。
2. **只有 Producer 分诊。** 其它 role 只捕获,不排序、不做 scope 决定(避免越权)。本流程跑
   完后开 `/role-producer`,它把 INBOX 按优先度分进 BACKLOG 的 Now/Next/Later/Cut,再清空
   INBOX。`[优先级]`(高/中/低)是你的粗略提示,Producer 参考但终裁;role 替你记时不自编,
   你没说就留 `[?]`。
3. **例外:当前任务已走错。** 若你的新输入意味着在跑的计划/设计已失效,role 不该闷头记进
   INBOX,而是按各自 escalation 停下上报——把明知错的活干完比中断更糟。

这样单个 feature 跑的过程中要求稳定、不被反复改,想法也不丢。模版见 `templates/INBOX.md`。

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
/role-implementer 01-double-jump auto              # Auto 模式:不在阶段边界停,自验+续跑到底
```

Implementer 有两档执行模式(默认 Manual):**Manual** 在每个阶段边界停下、呈现该阶段
的 Playtest gate 等你回报;**Auto** 一口气跑完整份 PLAN,自己在阶段边界做力所能及的
验证、把"需在编辑器里人工确认"的部分记进 CHANGES,只在安全边界(验证修两次仍挂 /
计划有错 / 要动架构·数值·状态机·依赖 / 破坏性操作)停下。在激活指令里写 `auto` 切换。

规则:
- **每功能 role**(Game Designer / Explorer / Planner / Implementer / Reviewer / Art Spec
  出 ASSET-SPEC 时 / Engine Integrator)应带 slug 指明工作目录。
- **常驻 role**(Producer、Art Spec 维护 STYLE-BIBLE 时)可不带 slug,只动 `harness/` 顶层文件。
- 不带 slug 且 `features/` 下恰好只有一个 `status: in-progress` 的功能时,role 自动选它;
  有多个或没有则停下来问你(防止写错目录)。
- slug 命名建议 `<两位序号>-<kebab>`,如 `01-double-jump`、`02-dash`,兼顾排序与可读。

## 换棒:一 session 一 role,`/clear` 再激活下一个

**一个 session 只扮演一个 role/skill**,换棒不靠在同一对话里切角色(两份契约会打架、
Reviewer 的 fresh eyes 会失效、上下文会爆),而是**清空上下文当新 session 用**:

```
/clear                      # 同一个终端里清空上下文 = 干净的新 session(不用关窗口重开)
/role-reviewer 01-xxx       # 紧接着激活下一棒;它自己去读该读的上游 artifact
```

- **为什么零成本**:所有进度都在 artifact 里(HANDOFF、PLAN/REVIEW 的 `[ ]/[~]/[x]` 记号、
  各产物),`/clear` 清掉的只是会被新 role 从文件重新读回来的东西——这正是"靠 artifact 不靠
  对话记忆"的回报。
- **用 `/clear`,别用 `/compact`**:compact 会残留上一个角色的上下文,正好犯"契约打架 /
  fresh eyes 失效"的毛病;clear 才是干净切。
- **回上游验收也照此办**:复审 / re-accept 就是"该上游 role 的又一次 session"——`/clear` →
  `/role-reviewer <slug>`,新 Reviewer 读带 `[x]` 的 REVIEW.md 即识别为复审,从断点接判。
- **同一 role 续做**(长 PLAN 没跑完)同理:`/clear` → 再开同名 role,靠 `[~]/[x]` 记号冷启动。
- **两条命令不能合并成一次提交**(slash 命令只在消息开头生效),但 `/clear` 是即时的,连敲
  `/clear`↵ 再 `/role-*`↵ 基本就是一个连贯动作,没有可感知停顿。

为减少记忆负担,**每个 role/skill 干完都会在末尾打印一条明确的接棒命令**,形如:
`已完成 — 下一步:/role-xxx 02-xxx(切换前先 /clear)`。照着复制即可,不用自己回忆该开谁。

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
- `BOOTSTRAP.md` .................... 启动新游戏项目的「铺地基」可勾选 checklist(README 第 1 步的展开版)
- `role-template.md` ................ 通用 role 模版(造新 role 时复制)
- `templates/` ..................... 复制进游戏项目的骨架
  - `project-context.md` ........... 共享上下文模版(填好放 `harness/`)
  - `style-basic-2d.md` ............ 通用 2D 图形工程纪律(与玩法/画风无关,复制进 `harness/` 常驻;Art Spec 引用、Engine Integrator 认领其〔EI〕节)
  - `HANDOFF.md` ................... 每功能交接文件模版
  - `INBOX.md` ..................... 中途捕获队列模版(复制进 `harness/` 常驻;Producer 分诊清空)
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
