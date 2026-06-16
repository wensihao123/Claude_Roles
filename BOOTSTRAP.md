# 铺地基 Checklist — 启动一个新 Godot 游戏项目

> 这是 README「怎么用」第 1 步的**可勾选展开版**。一次性、纯人工执行,**不属于任何 role**
> ——role 们假设地基已经铺好。做完这份,就能 `/design-jam 01-<slug>` 或
> `/role-game-designer 01-<slug>` 跑通第一条链。
>
> 建议:把这份复制进游戏项目 `harness/` 里边勾边做,做完留档。

## 阶段 0 · 建 Godot 工程
- [ ] 在 `G:/Claude/projects/<game>/` 用 Godot 新建项目(或确认 `project.godot` 已就位)
- [ ] 建源码骨架:`src/`、`scenes/`、`assets/sprites/`、`assets/audio/`
- [ ] `git init`(所有 artifact 都要纳入版本控制,每步决策留痕可回溯)

## 阶段 1 · 装 harness 骨架(从本规范仓库 `templates/` 复制)
- [ ] `templates/project-context.md` → `<game>/harness/project-context.md`
- [ ] `templates/HANDOFF.md` → `<game>/harness/templates/HANDOFF.md`
      ⚠️ 注意路径:落到**项目内**的 `harness/templates/`(不是 `harness/` 顶层)。
      design-jam 和「手动建新 feature」都从这里 seed 每个功能的 HANDOFF.md。
- [ ] `templates/artifact-frontmatter.md` → `<game>/harness/templates/artifact-frontmatter.md`(各 role 写 artifact 时的字段参考)
- [ ] `BACKLOG.md` / `STYLE-BIBLE.md` **不用手建** —— 分别由 Producer / Art Spec 首次运行时产出到 `harness/` 顶层

## 阶段 2 · 填实 `harness/project-context.md`(最关键 · 空着则全员失准)
逐节填,**去掉所有占位符**:
- [ ] §0 游戏一句话 + 2–4 条设计支柱 + v1 完成定义
- [ ] §1 引擎版本(**写准**!Engine Integrator 按这个版本给编辑器步骤)/ GDScript 或 C# / 目标平台 / 美术基线 / 测试命令
- [ ] §2 目录约定(对齐你阶段 0 实际建的结构)
- [ ] §3 代码约定(命名 / 信号 vs 直调 / 注释取舍)
- [ ] §4 禁止事项(hard NOs)
- [ ] §5 验证一次改动是否 OK 的标准流程(按顺序、全绿才算过)
- [ ] §6 当前已知的坑 / 临时约束

## 阶段 3 · 起第一个 feature
- [ ] 选一个**最小、可独立验证**的功能起手(如 `01-player-move`),slug 用 `<两位序号>-<kebab>`
- [ ] 二选一起链:
  - 想先发散点子 → `/design-jam 01-<slug> <一句话 spark>`(产 `IDEA.md`,自动从 `harness/templates/HANDOFF.md` seed 一份 HANDOFF.md)
  - 点子已清楚 → 手动建 `harness/features/01-<slug>/`,复制 `harness/templates/HANDOFF.md` 进去,再 `/role-game-designer 01-<slug>`

## 阶段 4 · 跑通第一条链(验证脚手架真的通)
按顺序**换 session、换 `/role`**,每步读上游 artifact、产出自己的:
- [ ] Game Designer → `FEATURE-DESIGN.md`
- [ ] (进陌生/既有代码时)Explorer → `CONTEXT-FINDINGS.md`
- [ ] Planner → `PLAN.md`
- [ ] Implementer → `CHANGES.md`(含 **Wiring Contract**)+ 真实代码进 `src/`
- [ ] Reviewer → `REVIEW.md`
- [ ] (需要美术时)Art Spec → `ASSET-SPEC.md` →(你出资源)→ `ACCEPTANCE.md`;Engine Integrator → `INTEGRATION-STEPS.md` →(你在编辑器照着接线)
- [ ] 每步结束确认对应 role 已更新 `HANDOFF.md`(是各 role「done」的一部分)
- [ ] 第一条链顺了,再把 **Producer** 拉起来常驻管范围(scope creep 比代码烂更致命)

## 完成判据(地基算铺好了)
- [ ] `harness/project-context.md` 已无任何占位符
- [ ] `harness/templates/HANDOFF.md` 就位(design-jam 能自动 seed)
- [ ] 至少一个 `harness/features/01-<slug>/` 跑完一条完整链,`HANDOFF.md` 状态一眼可读
