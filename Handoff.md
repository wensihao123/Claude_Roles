# HANDOFF — Claude Role Harness (独游 / Godot)

> 这份文件是给\*\*下一个 Claude session(在 VS Code + Claude CLI 里)\*\*的交接。 读完它,你应当能在不丢失上下文的情况下继续这套 role 规范的工作。

## 1. 这是什么项目

一套"每个 Claude session 扮演一个固定 role"的协作规范,面向 **solo 独游开发**, 引擎是 **Godot**(具体版本见 `project-context.md`,需用户填准)。

之所以这么设计,是因为每个 role 跑在独立 session 里、**不共享对话上下文**, 所以 role 之间靠\*\*产物(artifact)\*\*协作,而不是靠对话。

## 2. 当前状态:已完成

`./.harness/` 下已有一套可用文件(下面"文件清单"列全)。具体:

* coding 四件套(Explorer / Planner / Implementer / Reviewer)已定义。
* 已加 4 个独游 role:Game Designer、Producer、Art Spec、Engine Integrator(Godot 专版)。
* `project-context.md` 已改成 Godot/独游版(含游戏支柱、v1 定义、引擎版本字段)。
* README 已重写,含完整管线图与 artifact 命名约定。

## 3. 必须知道的关键设计决策(别推翻,除非有意为之)

1. **两类 role 性质不同**:coding 四件套能自交付(写出代码=完成);Art Spec 和 Engine Integrator 是"规格+指导+验收"型——Claude 画不了图、进不了 Godot 编辑器, 所以它们产出规格/可执行步骤/验收判据,**需要用户执行后回报结果(截图/报错/ Inspector 状态)**,role 再判定。这个"回报闭环"是它们能用的前提。
2. **Wiring Contract 是关键接缝**:Implementer 的 `CHANGES.md` 里有一节 Wiring Contract,列出新脚本的 `@export` 字段、emit/接收的 signal、需要的 autoload / input action / group。Engine Integrator 正是把它+资源规格翻成 Godot 接线步骤。 **没有这节,Engine Integrator 只能猜——不要删它。**
3. **每个 role 都有 escalation 机制**:对自己产物单一负责,但发现上游问题必须 STOP + 记到 Flags,而不是闭眼把错误执行完。这是有意为之,别改成"严格只做本职"。
4. **role 模版的固定结构**:identity / objective / inputs / outputs / tools / workflow / definition\_of\_done / escalation / constraints / example。新 role 照 `role-template.md` 来,保持一致。
5. **起手顺序**:别一上来全铺开。先用 `Game Designer → Planner → Implementer → Reviewer → Art Spec → Engine Integrator` 把一个功能完整跑通,进陌生代码再加 Explorer,之后再把 Producer 拉起来常驻管范围。

## 4. 文件清单(都在仓库根目录)

role 按职能分到三个分类子目录下:

```
README.md                                  管线总览 + artifact 命名约定
Handoff.md                                 本文件
project-context.md                         共享上下文(用户需填准 Godot 版本等)
role-template.md                           造新 role 时复制
roles/
  direction/                               决定做什么 / 什么体验
    role-producer.md
    role-game-designer.md
  engineering/                             coding 四件套(自交付型)
    role-explorer.md
    role-planner.md
    role-implementer.md                    已含 Wiring Contract
    role-reviewer.md
  production/                              规格 + 验收型(需回报闭环)
    role-art-spec.md
    role-engine-integrator-godot.md        Godot 专版
```

各 role 的产物对应:Game Designer→FEATURE-DESIGN.md,Producer→BACKLOG.md, Explorer→CONTEXT-FINDINGS.md,Planner→PLAN.md,Implementer→CHANGES.md(+代码), Reviewer→REVIEW.md,Art Spec→STYLE-BIBLE.md/ASSET-SPEC.md/ACCEPTANCE.md, Engine Integrator→INTEGRATION-STEPS.md。

## 5. 待办 / 未完成(下一个 session 可接手的)

* [ ]  **用户需填 `project-context.md`**:Godot 版本、GDScript/C#、目录树、游戏支柱、 v1 完成定义、标准验证命令。这是整套东西的地基,空着则所有 role 失准。
* [ ]  **尚未生成的 role**(用户感兴趣,可按需补):
  * Game Feel / Playtest — 把"手感差"翻成可调参数(coyote time、输入缓冲、 screen shake、tween 曲线、相机阻尼),用户调完回报手感再迭代。
  * Audio — 与 Art Spec 同构;初期可并进 Art Spec。
  * Debugger — 假设驱动、先取证据再改。
* [ ]  可选:把 `.harness/` 纳入版本控制,让每步决策留痕。
* [ ]  可选:跑通第一个真实功能后,回看哪些 role 的 friction 大于收益,做删减。

## 6. 给下一个 session 的第一步建议

1. 先读 `README.md` 和本文件,确认理解管线。
2. 问用户:`project-context.md` 填好了没?没填就先一起填——这是最高优先级。
3. 若用户要继续扩 role,用 `role-template.md`,并保持第 3 节的设计决策一致。
4. 若用户要开始做游戏功能,从 Game Designer 这一环起手,按第 3 节第 5 点的顺序走。

## 7. 一句话状态

规范骨架与全部核心 role 已就绪;**卡点是 `project-context.md` 尚未由用户填实**, 以及第一个真实功能还没跑通来验证整条链。
