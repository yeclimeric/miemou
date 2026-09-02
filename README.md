# MIEMOU

一个适合职业打工人的 AI 项目初始化 Prompt。

## 唯一入口

项目初始化只通过 Agent 执行 [MIEMOU.md](MIEMOU.md)。不要根据 README 手动创建脚手架、安装依赖或编写项目文件。

在 Agent 中打开本仓库后，直接发送：

```text
请阅读并严格执行 MIEMOU.md，完成当前仓库的项目初始化。
```

Agent 会以 `MIEMOU.md` 为准，完成 skill 安装确认、编码规则配置和 `AGENTS.md` 初始化。后续如果项目技术栈或团队约定发生变化，也应先更新 `MIEMOU.md`，再让 Agent 重新执行。

## 文件说明

- [MIEMOU.md](MIEMOU.md)：项目初始化的唯一执行规范。
- [README.md](README.md)：说明仓库用途和使用入口，不承载额外初始化流程。
