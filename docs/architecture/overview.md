# 架构概览

## 架构驱动因素

- GitHub 必须是跨会话、跨进程和跨机器可恢复的 Workflow State 权威来源。
- 工作台会话提交开发工作后必须立即释放，不能等待 CI、Reviewer 或长期交付流程。
- Runtime 必须确定性调度，不能依赖 AI 解释自由文本或隐式推断责任人。
- 一个垂直切片 Issue 必须共享一个 Worktree 和一个实现 PR，同时保留验收测试与实现的独立会话上下文。
- 仓库标准可以领先实现，Active Contract 必须始终代表默认分支可执行现实。
- 非受信任外部 PR 不能接触仓库密钥或让特权工作流执行其代码。
- Skill 套件必须自包含，不依赖开发机上可变的上游 Skill 目录。

## Bounded Context

当前仓库只有一个 Bounded Context：**GitHub 交付工作流**。其统一语言维护在 `glossary.md`。未来只有出现真正独立、拥有不同语言和模型的上下文时，才引入 Context Map；不会按代码目录预先拆分上下文。

## Modules

### Workbench Skills

面向用户或模型提供小而明确的流程 Interface，包括澄清、塑形、分诊、规范化、Issue 拆分、审查、开发提交和诊断。Skill 负责语义工作或指导调用工具，不持有权威 Workflow State。

### Repository Standards

由 Product、Design、Architecture、Development References、领域 Glossary 和 ADR 组成。它们表达对后续变更持续生效的目标语义，可以领先当前代码，但所有 Conformance Gap 必须被开放 Issue 图覆盖。

### GitHub Workflow State

由 Issue 正文与关系、Project Status、PR、Review、Check、Merge Queue 和受控评论共同组成，是流程阶段与审查事实的权威来源。评论中的机器标记只承载有限 Process Outcome 或当前审查结果，不替代人类可读内容。

### Workflow Runtime

本地后台运行的深 Module。它观察 GitHub 事件，根据 Runtime Definition 和 Session Registry 确定性创建、确保或恢复工作区及执行会话，并记录可按仓库与 Issue 查询的 Runtime Log。它不理解需求、不诊断失败、不审查代码，也不合并 PR。

Runtime 的核心 Interface 接收当前 Workflow State 与一个标准化事件，返回新的状态投影和一组待执行操作。状态转换、幂等资源确保、会话复用和错误路由隐藏在该 Interface 之后。

Codex Session Adapter 使用 Codex app-server 或其 SDK 创建和恢复独立 Thread。Runtime 在创建时传入流程所需的模型与 Reasoning Effort，保存返回的 Thread ID 和执行配置；后续事件通过同一 Thread ID 启动新 Turn。Thread 丢失或不可恢复时，Runtime 可以用同一执行配置创建替代 Thread，而不改变 GitHub Workflow State。

### Execution Processes

验收测试、垂直 TDD 和本地代码审查是相互独立的 AI 流程。每个 Issue 生命周期内，验收测试与垂直 TDD 各有一个可恢复会话并共享 Worktree；Reviewer 会话只审查并报告，不修改实现。

### GitHub Review and Merge Workflows

GitHub Actions 承载云端 Semantic、Delivery、Standards 与 Spec Reviewer，以及 CI、安全检查和 Human Merge Gate。Action 按其事件和信任模型选择配置来源；需要权限处理外部 PR 时只运行受信任配置。GitHub Merge Queue 负责最终合并；Runtime 不参与合并决定。

### Active Contracts

根目录 `contracts/` 保存当前默认分支实际生成、编译、发布或供下游消费的机器契约。Target Contract 属于塑形语义；只有进入对应 Issue Worktree 后，才能通过 Contract Materialization 成为候选 Active Contract，并与生成代码、测试和实现一起合并。

## Interfaces 与 Seams

### Dispatch Seam

`implement` 组合 `dispatch-workflow`，通过最小 CLI Interface 提交本地 Runtime Job。Job 只标识 Issue、工作流和可配置首次 Prompt；固定状态机、事件路由和恢复规则由 Runtime Definition 持有。

### Transition Seam

Runtime 的最高测试 seam 是“Workflow State + Event → Transition + Operations”。生产 Adapter 读取和修改 GitHub、Codex 会话与本地工作区；测试 Adapter 使用内存状态记录操作。调用方与测试都不穿透该 Interface 检查 Runtime 内部步骤。

### GitHub Event Seam

Runtime 接收标准化 GitHub 事件和稳定 Trigger Reference。恢复的执行会话自行读取对应 Issue、PR、评论和检查结果，Runtime 不复制正文，也不从正文猜测路由。

### Process Outcome Seam

执行流程发布人类可读 Report Summary，并附有限的机器可读 Process Outcome。GitHub 身份确认发布者；Runtime 根据结果码和当前 Workflow State 确定下一步。

### Review Seam

塑形审查以 Parent Issue 为根，对同一 Review Revision 下的规范 Spec、Issue 图和可选 Standards PR 进行双轴审查。实现审查以 PR diff 为根，对 Standards 与 Spec 分别给出结论。

## 依赖方向

1. Workbench Skills 可以读取 Repository Standards 和 GitHub Workflow State。
2. `implement` 只能通过 Dispatch Seam 向 Runtime 提交工作，不能直接操作 Session Registry。
3. Runtime Core 依赖抽象的 GitHub、Session 与 Workspace Interfaces；生产 Adapter 依赖外部系统，核心状态机不反向依赖 Adapter。
4. Execution Processes 读取当前 GitHub 对象和 Worktree，通过 Process Outcome Seam 返回结果，不控制 Runtime。
5. GitHub Review and Merge Workflows 按事件信任模型读取配置；需要权限的外部 PR 流程只读取默认分支受信任配置。合并结果反向成为 Runtime 可观察事件。
6. Active Contract 可以由代码生成或驱动生成，但其变更必须与相应测试和实现处于同一交付原子中。

## Runtime 与部署边界

- Workflow Runtime 是本地长期后台进程，与工作台 Codex 会话解耦。
- Execution Session 是可替换执行器；Session Registry 只是优化恢复成本的本地索引。
- Session Registry 保存流程对应的 Codex Thread ID、模型与 Reasoning Effort，但这些元数据不成为业务状态。
- GitHub Actions 是云端公网执行环境，负责自动审查、CI、安全和合并门禁。
- Product Repository 保存产品代码和非敏感协作；Control Repository 保存敏感工作；Portfolio Project 只做跨仓库可见性汇总。
- Runtime 丢失后必须能从 GitHub、Worktree 和持久日志重建，不允许只有会话记忆知道流程状态。

## 架构不变量

- 一个垂直切片 Issue 至多有一个由 Runtime 管理的活跃 Worktree 和实现 PR。
- 同一 Issue 内按流程复用会话，不跨 Issue 复用验收测试或垂直 TDD 会话。
- Project Status 不编码审查结论、合并队列状态或阻塞原因。
- Runtime 不解释自由文本、不执行任意仓库脚本、不自动消费全部 Ready Issue。
- Review Revision 只绑定语义内容，不因评论和普通执行元数据变化而失效。
- 外部 PR 的非受信任代码永远不在拥有仓库密钥的上下文中执行。
- 默认分支中的 Active Contract 必须可生成、可编译并与实现一致。

## 决策

长期且非显然的取舍记录在 `adr/`。本概览不复制 ADR 的理由，也不维护当前实现组件清单。
