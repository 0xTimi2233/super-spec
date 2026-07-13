# Super Spec

## Purpose

Super Spec 是运行在 Codex 上、深度集成 GitHub 的软件交付工作流套件。它把需求澄清、项目级事实维护、垂直切片、验收测试、垂直 TDD、审查和合并连接成可恢复的端到端流程，同时让 GitHub 而不是某个 AI 会话持有权威状态。

## Users

- 使用 Codex 维护软件项目的仓库所有者与 Maintainer。
- 负责需求塑形、架构、测试、实现或审查的团队成员。
- 通过公开 Issue 或 PR 参与项目的外部贡献者。
- 需要同时管理公开产品工作与敏感控制工作的团队。

## Problems

- 需求、设计、Issue、测试和代码分别维护，容易形成互相漂移的事实来源。
- 长时间 AI 交付依赖单一对话持续运行，异步 CI 或审查失败后难以恢复正确上下文。
- AI 容易把角色、会话、状态和流程混为一谈，产生重复 PR、重复工作区或隐式状态。
- 产品验收标准经常被扩张成易过期的测试计划，测试设计又可能被遗漏。
- 设计先行与契约可执行性之间缺少安全边界，未来契约可能过早进入默认分支。
- 内部需求、普通外部 Issue 与非受信任外部 PR 需要不同入口，但最终仍应汇入一致的仓库治理。
- GitHub Actions、Runtime 和本地 AI 的责任容易重叠，导致谁负责触发、诊断、修复和合并不清晰。

## Value

- 以规范 Spec、仓库标准、真实 Issue 图、代码与 Active Contract 构成清晰的事实层次。
- 以一个 Issue、一个 Worktree、一个实现 PR 交付可独立验收的垂直切片。
- 由本地 Workflow Runtime 确定性创建或恢复执行会话，让工作台会话可以立即继续其他工作。
- 以测试代码表达 Test BDD，以 PR 证据表达无法自动化的验收，不维护重复测试计划。
- 让设计和 Target Contract 可以领先实现，同时保证 Active Contract 与代码原子交付。
- 通过 GitHub 原生权限、审查、Merge Queue 与安全检查形成可审计的合并门禁。

## Capabilities

### 需求与塑形

- 对计划或需求进行深入澄清，实时维护统一语言，并按需研究或构建可抛弃原型。
- 对外部 Issue 进行 Maintainer 显式触发的本地分诊与澄清，不自动与普通报告者进行高频 AI 对话。
- 原子检查并增量维护产品、设计、架构、领域语言、ADR 与开发规范。
- 把规范 Spec 拆成真实 Parent Issue、垂直切片 Sub-issues 和原生依赖关系。
- 对完整塑形修订版进行 Semantic Integrity 与 Delivery Integrity 审查。

### 开发与验收

- 从 Ready 的垂直切片 Issue 显式启动异步开发工作流。
- 为每个 Issue 创建共享 Worktree、Draft PR、验收测试会话和垂直 TDD 会话。
- 从 Product BDD 推导测试代码中的 Test BDD，并优先自动化验收。
- 逐个解决 Focused RED，在每个验收测试内部执行单元 RED、GREEN、REFACTOR 循环。
- 在契约型切片需要时先执行受限的 Contract Materialization，再进行验收与实现。
- 在 CI、审查或人工验收事件发生后恢复同一 Issue 内对应的持久会话。

### 审查、治理与合并

- 对实现 PR 并行执行 Standards Review 与 Spec Review。
- 支持用户显式调用的本地双轴代码审查，作为云端 PR 审查之外的补充。
- 使用 CI、安全检查、CODEOWNERS、Ruleset、Human Merge Gate 和 Merge Queue 管理合并。
- 对外部 PR 使用非受信任输入边界，同时让通过信任入口的贡献进入相同最终门禁。
- 初始化并幂等维护仓库标签、Project 状态、模板、Actions、安全设置和 Runtime 定义。

## Product Principles

1. **一个事实只维护一次。** 产品行为在 Issue，测试设计在测试代码，无法自动化的证据在 PR，当前实现现实在代码、测试与 Active Contract。
2. **GitHub 持有状态，会话执行流程。** 会话可以恢复或替换，不能成为跨流程事实来源。
3. **流程是持久身份，角色只是 Prompt。** Runtime 调度明确流程，不把人格名称写进状态机。
4. **语义由 AI 处理，调度由机器确定。** Runtime 只消费有限事件、配置和 Process Outcome，不解释自由文本。
5. **垂直交付。** 每个叶子 Issue 必须形成可独立验收的端到端产品行为。
6. **能自动化就写进代码。** 无法自动化时才由验收测试流程执行并在 PR 提供证据。
7. **设计可以领先，生效事实必须原子。** 仓库标准和 Target Contract 可以先行，Active Contract 不能脱离实现单独进入默认分支。
8. **安全边界优先于流程统一。** 不为了复用内部流程而向非受信任代码暴露权限或密钥。
9. **精简但有效。** Skill、状态、标签和配置只保留支撑明确语义所需的最小集合。
10. **缺少前置决定就停止。** 后续流程不能现场补写或猜测上游产品、设计、架构和契约决定。

## Boundaries

- Super Spec 不替代 GitHub 的权限模型、Issue、PR、Project、Actions 或 Merge Queue。
- Workflow Runtime 不进行需求理解、技术诊断、代码审查或合并。
- 云端 Reviewer 不修改实现；第三方 AI Reviewer 只作为用户可选补充。
- 外部 Issue 不由 AI 自动回复、自动接受或自动塑形。
- 外部 PR 的失败不由内部 Runtime 接管，修改责任仍属于提交者。
- Product Reference 不保存 Roadmap 或具体 Issue 汇总。
- Prototype 不成为默认分支的长期事实来源。
- 架构概览不复制可从代码或 Active Contract 推导的当前组件清单。
- 本产品不维护公开与私有两份产品代码镜像。

## Vision

任何一次软件变更都可以从自然语言需求开始，经过充分塑形后成为可审查的规范 Spec 和真实 Issue 图，再由可恢复的 AI 流程在 GitHub 原生治理下完成验收、实现和合并。用户可以持续处理新的需求，而不必守着某个长时间运行的主代理，也不必依赖隐藏会话记忆理解项目当前处于哪里。
