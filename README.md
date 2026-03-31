# Claude Code 源码快照

> 本仓库保存的是一份通过 npm 分发产物中的 source map 暴露出来的 Claude Code `src/` 源码快照，仅用于安全研究、供应链分析、架构学习与教学讨论。

---

## 这是什么

这不是 Anthropic 官方开源仓库，而是一份基于公开可访问构建产物整理出的源码镜像。

- 暴露时间：`2026-03-31`
- 主要内容：Claude Code CLI 的 `src/` 源码快照
- 技术栈：TypeScript + Bun + React + Ink
- 仓库定位：安全研究 / 架构考古 / Agent CLI 设计参考

Claude Code 本质上是一个面向软件工程任务的终端 Agent：

- 能读写文件
- 能执行 Shell
- 能搜索代码库
- 能调工具 / MCP / LSP
- 能跑子代理、技能、计划模式、远程桥接

如果你想研究“现代 Agent CLI 到底怎么组织起来”，这个仓库是很高价值的样本。

---

## 背景说明

2026-03-31，公开分发的 npm 产物中存在 source map 暴露问题，进一步指向了可直接下载的 TypeScript 源码文件，因此这份 `src/` 快照在公网可访问。

公开讨论可参考：

- [Chaofan Shou (@Fried_rice)](https://x.com/Fried_rice/status/2038894956459290963)

本仓库的维护目标是：

- 研究构建产物泄露带来的供应链风险
- 学习成熟 Agent CLI 的模块拆分方式
- 分析权限系统、工具系统、上下文系统的实现思路
- 给安全研究、终端产品、AI 编程工具设计提供真实样本

---

## 仓库边界

本仓库不声称拥有原始代码版权，也不应被理解为官方发布渠道。

请把它理解为一份研究样本，而不是官方 SDK 或正式发行版源码仓库。

适合的用途：

- 安全研究
- 架构分析
- 教学演示
- 终端 Agent 设计参考
- 供应链与发布流程复盘

不适合的用途：

- 伪装成官方仓库对外分发
- 混淆版权归属
- 将快照内容包装成官方支持版本

---

## TCD 视角下怎么读这个仓库

如果你是按 TCD 的方式看代码，不建议一上来全量乱翻，建议按下面三条主线拆：

### 1. Agent 主循环

先看 Claude Code 如何把“用户输入 -> 模型响应 -> 工具调用 -> 权限确认 -> 再回到模型”串起来：

- `src/main.tsx`
- `src/query.ts`
- `src/QueryEngine.ts`
- `src/Tool.ts`
- `src/tools.ts`

这条线解决的是：

- CLI 怎么进入主循环
- 模型工具调用怎么调度
- 流式输出怎么组织
- 工具结果怎么回灌给模型

### 2. 工具与权限系统

再看它如何把“能做什么”和“允许做什么”分开：

- `src/tools/`
- `src/hooks/toolPermission/`
- `src/components/permissions/`
- `src/services/tools/`

这条线最值得研究的是：

- 每个工具的 schema 和执行逻辑如何定义
- 权限弹窗与自动放行模式如何落地
- Shell / 文件 / Web / MCP 工具如何统一抽象

### 3. 扩展能力层

最后看它如何变成一个可扩展平台，而不只是一个单体 CLI：

- `src/skills/`
- `src/plugins/`
- `src/commands/`
- `src/services/mcp/`
- `src/services/lsp/`
- `src/bridge/`
- `src/remote/`

这条线对应的是：

- Skill 如何加载
- Plugin 如何发现与安装
- MCP 如何接入外部系统
- IDE / Remote / Bridge 如何和 CLI 打通

---

## 目录结构速览

```text
src/
├── main.tsx                 # CLI 入口与初始化
├── commands.ts              # 命令注册总表
├── tools.ts                 # 工具注册总表
├── Tool.ts                  # 工具抽象与类型
├── QueryEngine.ts           # 模型调用与工具循环核心
├── query.ts                 # 主查询/主回合编排
├── context.ts               # 上下文收集
├── cost-tracker.ts          # 成本统计
│
├── commands/                # 用户命令
├── tools/                   # 工具实现
├── components/              # Ink UI 组件
├── hooks/                   # React Hook
├── services/                # 服务集成层
├── screens/                 # 全屏界面
├── types/                   # 类型定义
├── utils/                   # 工具函数
│
├── bridge/                  # IDE / 桥接协议
├── remote/                  # 远程会话
├── coordinator/             # 多代理协调
├── plugins/                 # 插件系统
├── skills/                  # 技能系统
├── server/                  # 服务端模式
├── memdir/                  # 持久记忆目录
├── tasks/                   # 任务系统
├── state/                   # 状态管理
├── migrations/              # 配置迁移
├── schemas/                 # Zod Schema
└── upstreamproxy/           # 代理配置
```

---

## 重点模块

### 工具系统

`src/tools/` 是 Claude Code 的核心资产之一。这里能直接看到一个成熟 Agent CLI 常见的工具模型：

- `BashTool`
- `FileReadTool`
- `FileWriteTool`
- `FileEditTool`
- `GlobTool`
- `GrepTool`
- `WebFetchTool`
- `WebSearchTool`
- `AgentTool`
- `SkillTool`
- `MCPTool`
- `LSPTool`
- `TaskCreateTool`
- `TaskUpdateTool`

研究价值在于：

- 工具 schema 如何约束模型输出
- 工具权限如何单独控制
- UI、执行器、结果格式化如何分层

### 命令系统

`src/commands/` 负责 CLI 命令层，能看到它如何把交互命令做成可扩展体系：

- `/review`
- `/config`
- `/doctor`
- `/memory`
- `/skills`
- `/tasks`
- `/mcp`
- `/resume`
- `/diff`

如果你在做自己的 AI CLI，这部分非常值得对照。

### Skill / Plugin / MCP

这是 Claude Code 从“命令行工具”走向“平台”的关键：

- `src/skills/`：技能加载、前置发现、动态激活
- `src/plugins/`：插件装配
- `src/services/mcp/`：MCP 连接、配置、认证、权限

这一层很适合拿来回答两个问题：

1. Agent 平台怎么做可扩展性？
2. 本地 Prompt 技能、插件、外部工具协议，怎么统一到一个 CLI 里？

---

## 建议阅读顺序

如果你第一次看这个仓库，建议按顺序读：

1. `src/main.tsx`
2. `src/query.ts`
3. `src/QueryEngine.ts`
4. `src/Tool.ts`
5. `src/tools/BashTool/`
6. `src/services/tools/`
7. `src/hooks/toolPermission/`
8. `src/skills/loadSkillsDir.ts`
9. `src/services/mcp/`
10. `src/bridge/`

这样能先抓住主干，再看扩展层，不容易迷路。

---

## 这个仓库适合谁

- 想做 Agent CLI / Coding Agent 的工程师
- 想研究 AI 工具调用与权限系统的安全研究者
- 想研究终端 UI + Agent 交互设计的产品/前端工程师
- 想复盘供应链泄露事件的安全从业者
- 想学习大型 TypeScript CLI 项目组织方式的人

---

## 研究重点建议

### 1. 供应链安全

关注：

- source map 暴露
- 构建产物是否带出源码引用
- 静态资源、R2、CDN 与 npm 包之间的边界

### 2. Agent 架构

关注：

- 模型循环
- 工具协议
- 权限判定
- 技能与插件装配
- 远程会话与 IDE 桥接

### 3. 工程实现

关注：

- 大型 TypeScript CLI 如何拆模块
- React + Ink 在终端里的组织方式
- 状态、命令、工具、权限、服务之间的边界

---

## 免责声明

本仓库仅用于研究、教学与防御性分析。

- 不代表官方立场
- 不提供官方支持
- 不保证与任何正式发行版本完全一致
- 不应被当作官方开源项目引用

如果原始权利方提出合理要求，仓库维护方应按实际情况处理。

---

## 一句话总结

这不是“Claude Code 官方源码”，而是一份极有研究价值的真实 Agent CLI 源码快照；如果你关心 AI 编程工具、终端 Agent、安全供应链和可扩展架构，这个仓库值得细读。
