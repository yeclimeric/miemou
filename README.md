# MIEMOU

> 给职业打工人的 AI 项目初始化 + 自运行开发团队技能。

## 为什么叫 MIEMOU？

`mie` 是“咩”，`mou` 是“哞”。

一声羊叫，一声牛哞，献给每天认真搬砖的打工人。MIEMOU 让 Agent 不只是“能写代码”，而是真正**懂你的项目、按你的规矩干活**。

## 两种模式

### 模式一：项目初始化

为代码仓库初始化 AI 项目环境：

1. **规则配置**——六套成员规则（karpathy 四原则、ponytail 最简原则、诊断循环、TDD、审视法、架构深化）已内置在技能中，无需安装外部 skill、无需在线获取。
2. **编码规则落盘**——karpathy 四原则写入项目根目录 `CLAUDE.md`（已有则合并，保留项目专属指令）。
3. **生成 AGENTS.md**——按「没有这条提示，Agent 是否容易遗漏」的标准，为仓库沉淀高价值指令文件。

### 模式二：自运行 AI 团队

主 agent 化身项目经理（PM），把**操作者指定来源**的开发任务自动跑完「排期确认 → 独立分支 → 派发子 agent（产品/架构/开发/UI/测试）→ 质量门禁 → 合并 → 回报」全流水线。

任务来源由操作者指定，支持四种形态：

- Markdown 任务清单（文件路径，完成后回写勾选）
- 对话中直接下达
- GitHub Issues（保留为可选来源之一）
- 任意结构化来源（表格 / JSON / 需求文档，归一化为统一任务包）

铁律：每个任务必须走完全流程——PM 绝不亲自写代码，绝不在操作者确认的当期队列之外开工，绝不跳过质量门禁直接 commit。

## 使用

安装：把本仓库克隆/放置到你的 Agent skills 目录（SKILL.md 所在目录即技能根目录）。

- 初始化项目：对 Agent 说「初始化当前仓库」或「按 MIEMOU 初始化」。
- 跑团队：对 Agent 说「跑一轮开发任务」「按 `<任务清单路径>` 开发」「组建 AI 团队干活」。

两种模式可串行：先初始化（产出 `AGENTS.md`），团队模式的「项目约定卡」直接从它提取，形成闭环。

## 文件说明

- [SKILL.md](SKILL.md)：技能入口——模式路由、双模式工作流、红线与交付前检查清单。
- [references/init-project.md](references/init-project.md)：模式一细则（调研方式、AGENTS.md 提取与编写规则）。
- [references/member-guidelines.md](references/member-guidelines.md)：六套内置成员规则原文 + 全员行为准则。
- [references/team-model.md](references/team-model.md)：项目约定卡模板、团队角色、派发纪律与回传格式。
- [references/task-package.md](references/task-package.md)：统一任务包格式、四种任务来源适配、优先级裁决。
- [references/quality-gates.md](references/quality-gates.md)：质量门禁、分支与提交纪律、临时产物约定、无约定项目兜底。
