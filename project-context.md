# Project Context (shared by all roles)

> 这是所有 role 的"共享内存"。每个 session 都应先读这一份。
> 保持它简短、事实化、可信——它一错,所有 role 跟着错。

## 1. 项目一句话
[这个项目是做什么的,给谁用。]

## 2. 技术栈
- 语言 / 运行时:[e.g. TypeScript, Node 20]
- 框架 / 关键库:[e.g. React 19, Hono, Drizzle]
- 包管理 / 构建:[e.g. pnpm, vite]
- 测试:[e.g. vitest;运行命令 `pnpm test`]
- 其它工具:[lint / typecheck / format 的具体命令]

## 3. 目录约定
```
[贴一棵精简的目录树,标注每个目录放什么]
src/
  ...
```

## 4. 代码约定
- 命名:[e.g. 文件 kebab-case,组件 PascalCase]
- 风格:[e.g. 函数式优先,避免类;早返回而非深嵌套]
- 错误处理:[e.g. 用 Result 类型,不裸抛]
- 注释:[e.g. 只在"为什么"不显然时注释]

## 5. 禁止事项(hard NOs)
- [e.g. 不要引入新依赖,除非计划里明确批准]
- [e.g. 不要改 `src/legacy/**`]
- [e.g. 不要把密钥写进代码;用环境变量]
- [e.g. 不要做计划之外的"顺手重构"]

## 6. 验证一次改动是否 OK 的标准命令
按顺序跑,全绿才算通过:
```
[e.g. pnpm typecheck]
[e.g. pnpm lint]
[e.g. pnpm test]
```

## 7. 当前已知的坑 / 临时约束
- [e.g. X 模块正在迁移,别动]
- [e.g. CI 对 bundle 体积有上限 200kb]
