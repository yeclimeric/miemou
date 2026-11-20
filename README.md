# MIEMOU

> 给职业打工人的 AI 项目初始化 + 自运行开发团队技能。

## 为什么叫 MIEMOU？

`mie` 是“咩”，`mou` 是“哞”。

一声羊叫，一声牛哞，献给每天认真搬砖的打工人。MIEMOU 让 Agent 不只是“能写代码”，而是真正**懂你的项目、按你的规矩干活**。

## 两种模式

### 模式一：项目初始化

为代码仓库初始化 AI 项目环境：

1. **规则配置**——六套成员规则（karpathy 四原则、ponytail 最简原则、诊断循环、TDD、审视法、架构深化）已内置在技能中，无需安装外部 skill、无需在线获取。
2. **编码规则落盘**——karpathy 四原则落盘 `CONTEXTS.md`（已有则合并，保留项目专属指令）；与 `AGENTS.md` 分工：前者承载行为基线，后者只写仓库专属指引，两者均不绑定特定 agent 工具链。
3. **生成 AGENTS.md**——按「没有这条提示，Agent 是否容易遗漏」的标准，为仓库沉淀高价值指令文件。

### 模式二：自运行 AI 团队

主 agent 化身项目经理（PM），把**操作者指定来源**的开发任务自动跑完「排期确认 → 独立分支 → 派发子 agent（产品/架构/开发/UI/测试）→ 质量门禁 → 合并 → 回报」全流水线。

**授权档位（L0/L1/L2）**：确认门的行为由档位统一决定，默认 L1（队列确认一次、本轮自动跑完）；「自主跑」切 L2 后所有确认门降级为 PM 按推荐答案自决 + 裁决日志落盘 + 收尾批量呈报，无人值守不中断——质量门禁与打回纪律在任何档位下不降级。

**不中断三件套**：

- 在途状态落盘 `.miemou/state.md`（任务状态台账，每次状态跃迁即时更新）——会话中断后读台账断点续跑；
- 门禁基线口径——建分支时捕获 lint/测试基线快照，门禁标准是「不新增失败」，主分支存量红灯不冻结新任务；
- 红灯修复回路 + 轮级熔断——测试失败附全文再派原开发子 agent，预算 3 轮；连续 2 个任务受阻自动停轮收尾。

**单工作树约束**：实现与自测类任务默认串行；只读勘察类可并行（上限 3）；确需并行实现时用 `git worktree` 隔离，合并后清理。

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
- [references/karpathy-examples.md](references/karpathy-examples.md)：karpathy 四原则配套——逐原则真实代码示例（对错对照、反模式速查、关键洞见）。
- [references/team-model.md](references/team-model.md)：项目约定卡模板、团队角色、派发纪律与回传格式。
- [references/task-package.md](references/task-package.md)：统一任务包格式、四种任务来源适配、优先级裁决。
- [references/quality-gates.md](references/quality-gates.md)：质量门禁（基线口径）、分支与提交纪律、任务状态台账（.miemou/state.md）、红灯修复回路与熔断、无约定项目兜底。
- [references/codebase-design.md](references/codebase-design.md)：架构深化配套——深模块设计词汇（模块/接口/深度/接缝/适配器/杠杆/局部性）、深化指南与「设计两次」并行探索模式。
- [references/architecture-report.md](references/architecture-report.md)：架构深化配套——候选呈报 HTML 报告格式、脚手架与图示模式。
- [references/scripts/hitl-loop.template.sh](references/scripts/hitl-loop.template.sh)：诊断循环配套——HITL（human-in-the-loop）复现回路模板脚本。

## 致谢（Acknowledgments）

本技能内置的成员规则整合自以下开源 agent skill——感谢原作者的杰出工作。整合时完整保留了各自的规则、清单与示例，并统一为中文：

| 本仓库位置 | 来源 skill | 整合内容 | 上游许可证 |
|---|---|---|---|
| [member-guidelines.md](references/member-guidelines.md) §1 + [karpathy-examples.md](references/karpathy-examples.md) | [karpathy-guidelines](https://github.com/multica-ai/andrej-karpathy-skills)（andrej-karpathy-skills） | karpathy 四原则：三思而后写、简单优先、外科手术式改动、目标驱动执行；配套逐原则代码示例与反模式速查 | [MIT](https://github.com/multica-ai/andrej-karpathy-skills#license) |
| [member-guidelines.md](references/member-guidelines.md) §2 | [ponytail](https://github.com/DietrichGebert/ponytail) | 最简完整改动：取舍顺序、绝不削减清单、lite/full/ultra 级别 | [MIT](https://github.com/DietrichGebert/ponytail/blob/main/LICENSE) |
| [member-guidelines.md](references/member-guidelines.md) §4 | diagnosing-bugs | 诊断循环六阶段：反馈回路、脱敏、复现最小化、假设、插桩、修复回归、清理 | 未标注 |
| [member-guidelines.md](references/member-guidelines.md) §5 | grilling | 审视法：设计树、前沿轮次、事实与决策分工 | 未标注 |
| [member-guidelines.md](references/member-guidelines.md) §6 | improve-codebase-architecture | 架构深化：摩擦勘察、删除测试、HTML 候选报告、审视循环 | 未标注 |
| [codebase-design.md](references/codebase-design.md) | codebase-design | 深模块设计词汇、深化指南、「设计两次」并行探索模式 | 未标注 |
| [architecture-report.md](references/architecture-report.md) | improve-codebase-architecture（HTML-REPORT.md） | 架构审视 HTML 报告格式与图示模式 | 未标注 |
| [scripts/hitl-loop.template.sh](references/scripts/hitl-loop.template.sh) | diagnosing-bugs（scripts/） | HITL 复现回路模板脚本 | 未标注 |

其余一套规则（§3 TDD）为本项目原生内容。

- karpathy-guidelines 与 ponytail 均以 [MIT License](https://opensource.org/licenses/MIT) 发布：前者无独立 LICENSE 文件，MIT 声明于上游 README「License」节与 skill 元数据 `license` 字段；后者有上游 LICENSE 文件（版权人 DietrichGebert）。完整许可证文本见 [LICENSE-THIRD-PARTY.md](LICENSE-THIRD-PARTY.md)。
- 其余来源 skill 未在本地元数据中标注许可证，以各上游项目实际声明为准；如你有上游仓库地址，欢迎补充链接完善本节。
