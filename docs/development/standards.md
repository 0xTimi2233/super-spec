# Development Standards

## Language and Naming

- 项目自有文档、Skill 和用户可见流程说明使用中文；代码标识符、GitHub 固定术语与行业通用名称按其原始拼写使用。
- 项目自有 Reference 使用小写文件名。只有平台约定文件保留规定大小写，例如 `AGENTS.md`、`SECURITY.md` 和 `.github/CODEOWNERS`。
- 使用统一语言中的规范术语。涉及模块设计时固定使用 Module、Interface、Seam、Adapter、Depth、Leverage 与 Locality。
- 不把流程称为角色。QA、DEV 或 Reviewer 只可以出现在 Prompt 的角色语气中，持久身份使用验收测试、垂直 TDD 或审查等流程名称。

## Skill Design

- 每个 Skill 必须代表用户或模型有意义地调用的一项能力，不能把不可调用的内部说明伪装成 Skill。
- `SKILL.md` 只保留触发语义、核心顺序、硬边界和渐进加载入口；条件执行说明放入 `references/`，实际输出模板放入 `assets/`，可重复的确定性操作优先放入脚本或 CLI。
- Skill 保持短小、指令明确、容易记忆。不要复制调用方已经知道的上下文，也不要同时提供多个含义重叠的入口。
- 模型可调用性与用户可调用性分开判断；禁止模型自动调用的 Skill 必须有真实的副作用或流程门槛理由。
- 工作流使用到的上游 Skill 必须复制为受控版本并连同实际依赖闭包维护；运行时不得引用本机上游目录。
- Skill 之间按用户可理解的流程组合，不把 Runtime 的机器状态机复制到多个 Skill 中。

## Module Design

- 优先设计深 Module：以小 Interface 隐藏复杂实现，为调用方提供 Leverage，为维护者提供 Locality。
- 调用方与测试穿过同一个最高可行 Seam；不要仅为测试暴露内部步骤或状态。
- 只有存在至少两个合理 Adapter 时才引入抽象 Seam。生产与内存测试 Adapter 可以构成真实 Seam。
- 依赖由外部注入，核心逻辑返回明确结果；I/O 与第三方系统位于 Adapter 中。
- Runtime Core 必须保持确定性。相同 Workflow State、事件与 Runtime Definition 必须产生相同转换和操作意图。

## Workflow Definitions and Prompts

- Runtime Definition 只允许声明事件匹配、内建步骤、流程调度和状态转换，不允许任意 shell。
- Runtime Job 保持最小，不复制固定状态机或事件路由。输入 Job 是 gitignored 的本地临时材料，提交后由 Runtime 持久化。
- 首次 Prompt 指示会话读取对应流程 Skill 和 Issue；恢复 Prompt 提供稳定 Trigger Reference，并让会话自行读取 GitHub 当前状态。
- Codex 执行会话通过 app-server 或 SDK 的 Thread Interface 创建和恢复。Runtime 必须记录 Thread ID 及请求的模型与 Reasoning Effort；恢复时沿用原执行配置，替换会话时显式重建同一配置。
- Prompt 不承担机器路由。流程结束必须同时提供人类可读 Report Summary 与有限 Process Outcome。
- Runtime 失败必须产生可按仓库与 Issue 查询的证据；初始化使用 ensure 语义，重试不得复制已有资源。

## Test-Driven Delivery

- Issue 只保存 Product BDD，不保存测试计划。
- 验收测试流程从 Product BDD 推导 Test BDD，并将可自动化场景写入测试代码。Given、When、Then 注释只在能提高场景精度时使用。
- 无法自动化的场景由验收测试流程执行，并在 PR 发布包含环境、步骤、观察结果及必要附件的证据。
- 垂直 TDD 每次选择一个 Focused RED，先确认失败原因有效，再执行所需的单元 RED、GREEN、REFACTOR 循环，直到该验收测试 GREEN，随后重构并选择下一个。
- 可以预先存在多个失败验收测试，但禁止并行实现多个 Focused RED。
- 测试只观察公开 Interface 行为，不锁定内部步骤；重构实现不应迫使行为测试重写。
- 缺陷修复必须先建立能够证明诊断的失败测试，除非该行为无法可靠自动化。

## Contracts

- Target Contract 可以在塑形中领先代码；默认分支的 Active Contract 必须始终可生成、可编译并与实现一致。
- Contract Materialization 只能创建候选 Active Contract、生成代码和最小可编译未实现骨架，不得实现产品行为。
- 契约格式、兼容性、生成一致性及可自动化的 Consumer 或 Provider Contract Test 纳入 `ci`。
- 契约和生成代码不得脱离对应测试与实现单独合并。
- 执行流程发现 Target Contract 语义错误时必须请求 Human Decision，不得在开发阶段擅自改变产品语义。

## Reviews

- 塑形变更并行审查 Semantic Integrity 与 Delivery Integrity；实现 PR 并行审查 Repository Standards 与 Canonical Spec。
- Reviewer 只读取、判断并发布结论，不修改被审查实现。
- 审查结论必须绑定明确修订版或 PR Head；语义内容变化后旧审批必须失效。
- 云端 Reviewer Prompt 由对应 GitHub Action 定义并遵守其事件信任模型；需要权限处理外部 PR 时必须使用默认分支中的受信任配置。
- 本地 `code-review` 只能由用户显式调用，不作为云端 PR 门禁的隐式替代。
- 第三方 AI Reviewer 可以补充核心流程，但核心状态与合并条件不得依赖其专有结果。

## CI and Security

- Required Checks 使用稳定顶层名称：`ci`、`security`、`review/standards`、`review/spec`。
- `ci` 至少覆盖项目实际存在的 lint、typecheck、unit 和 integration；只有存在真实 E2E 配置时才加入 E2E，不创建空测试凑门禁。
- `security` 按仓库技术栈聚合适用的 CodeQL、Dependency Review、Secret Scanning、容器或 IaC 检查。
- `merge_group` 重跑 `ci` 与 `security`；基于 PR Head 的 AI Review 不重复执行。
- 外部 PR 的代码不得在具有仓库 Secret 或写权限的上下文中执行。需要权限的审查从受信任默认分支读取 diff。
- GitHub 权限遵循最小权限；CODEOWNERS、Ruleset、Environment Required Reviewer 和 Merge Queue 使用平台原生能力。

## Documentation and Change Discipline

- 一个事实只维护在其权威载体中，不建立同步副本：产品行为在 Issue，测试设计在代码，手工证据在 PR，Active Contract 在 `contracts/`。
- Product、Design、Architecture 与 Development References 只保存跨 Issue 稳定事实，不汇总 Roadmap 或当前任务列表。
- 架构概览只描述目标结构、Interface、Seam、依赖方向和运行边界，不复制代码清单。
- ADR 只有在决定难以逆转、缺少上下文会显得反直觉、且来自真实取舍时才创建。
- 后续流程发现上游 Reference 不足或相互冲突时必须停止并返回正确流程，不得现场补写整套文档。
- 所有修改在提交前执行与风险相称的格式、静态检查和测试；生成文件必须通过生成一致性检查。
