# QA 与 CI 的职责边界：GitHub 工作流调研

> 状态：研究笔记，不是架构决策。  
> 范围：自动化 PR CI、部署后 E2E/视觉/无障碍/性能/安全检查、探索式 QA、人工验收。  
> 来源原则：仅使用产品官方文档和项目自己的仓库/流程。

## 结论摘要

成熟流程通常不是把“测试”归入单一阶段，而是按**可重复性、运行环境、判定者和阻断成本**分层：

| 层 | 典型内容 | GitHub 中的证据/门禁 | 主要权衡 |
|---|---|---|---|
| PR 自动化 CI | build、lint、typecheck、unit、integration、可稳定运行的 E2E；CodeQL、依赖审查 | Check Run / commit status；required status check | 最快、最确定，适合作为每次提交的硬门禁；昂贵或易抖动的套件会拖慢反馈 |
| UI 与非功能回归 | 跨浏览器 E2E、截图差异、组件交互、自动 a11y、性能预算 | 普通或 required check；报告、trace、截图 artifact | 可自动化，但基线审批、性能噪声和浏览器矩阵会增加成本 |
| 预览/测试环境验证 | 部署后的 smoke/E2E、真实集成、迁移、配置和环境相关检查 | deployment status；“部署成功后才可合并”；environment protection | 更接近真实系统，但更慢、依赖外部状态，失败不一定是代码缺陷 |
| 探索式 QA | 新交互、视觉观感、异常路径、设备/辅助技术、无法编码的风险探索 | PR 评论中的结果与证据；也可由 App 回写 check | 能发现测试脚本未设想的问题，但结论依赖技能与证据质量，不天然可重复 |
| 人工接受/发布审批 | 产品意图、UX 取舍、合规例外、风险接受、生产放行 | PR review；environment required reviewer；conversation resolution | 保留真正需要判断/责任承担的门；不宜伪装成机器检查 |

GitHub 本身不规定“验收标准应该写在哪里”。它提供 Issue Form 和 PR Template 来标准化输入；Issue Form 可要求字段，PR Template 会自动填入 PR 正文。[GitHub：Issue 与 PR 模板](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/about-issue-and-pull-request-templates) 因此下面对 Issue/PR 边界的划分是基于工具能力与公开项目实践的综合，而不是 GitHub 的强制规范。

## Issue 验收标准与 PR 验证证据

### 适合放在 Issue 的内容

Issue 应保存跨实现稳定的 **“什么结果才算完成”**：

- 用户可观察行为、业务规则和 Given/When/Then 场景；
- 适用角色、权限、平台/浏览器/设备和关键数据状态；
- 必须满足的非功能约束，例如 WCAG 级别、性能预算、安全或审计要求；
- 明确哪些结果需要人工/产品/QA 判断，以及判断者；
- 不做什么、兼容性边界和可接受的已知限制。

GitHub Issue Form 支持 required 字段、下拉框、复选框、自动标签和项目归属，因此可以把上述信息结构化，但提交后仍只是普通 Markdown Issue。[GitHub：Issue Form 语法](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms)

### PR 可以承载的验证证据

PR 可以汇总针对**这一个实现**的验证结果，但不需要复制测试代码形成另一份测试计划：

- 关联 Issue/验收场景，以及本 PR 覆盖了哪些场景；
- 自动检查的运行结果及其 Check Run、日志或 artifact 链接；
- 无法自动化场景的实际结果、截图、录像、trace、浏览器或设备信息；
- 未运行的检查及原因、已知风险、回滚/监控说明；
- QA/人工验收结果和遗留问题链接。

自动测试的场景和判定逻辑以测试代码为事实来源；PR 只提供审查所需的摘要和证据。GitHub 的 PR Template 可以提示贡献者填写验证摘要，但不会验证填写内容。[GitHub：创建 PR Template](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository)

公开项目也会把自动化与人工验证分开：VS Code 要求“一项 Issue 对应一个 PR”、尽可能带测试，并在月度发布流程中另外创建 Test Plan Items、给需要验证的 Issue 写 verification steps，再由团队完成验证。[VS Code：贡献流程](https://github.com/microsoft/vscode/wiki/How-to-Contribute) [VS Code：发布 Endgame 示例](https://github.com/microsoft/vscode/issues/313587)

## 哪些 UI/QA 检查能够进入自动 CI

“CI 覆盖不了”通常不是测试类型的固定属性，而是当前测试是否具有稳定 oracle、可重复环境、合适成本与安全凭据：

- **E2E / smoke**：Playwright 官方示例在 push 和 PR 上运行测试并上传 HTML report；失败可通过日志和 trace 调试。因此稳定的关键路径 E2E 完全可以是 required check。[Playwright：在 CI 中运行](https://playwright.dev/docs/ci-intro)
- **跨浏览器/跨配置**：Playwright/Cypress 可在 CI 运行浏览器矩阵；Cypress Cloud 可跨机器按历史时长分配 spec，也可按浏览器或子系统分组。代价是机器、时间和外部服务依赖。[Cypress：并行化](https://docs.cypress.io/cloud/features/smart-orchestration/parallelization)
- **视觉回归**：Playwright 支持 screenshot baseline 比较，但提醒不同 OS、版本、硬件和设置会改变渲染，应在与基线一致的环境运行。[Playwright：视觉比较](https://playwright.dev/docs/test-snapshots) Chromatic 将 UI Tests 和需要人工确认差异的 UI Review 分成两个 PR check，两者可分别设为 required；这是“机器发现差异、人判断差异是否正确”的明确分层。[Chromatic：PR 工作流](https://www.chromatic.com/docs/in-pull-request/)
- **自动无障碍检查**：Playwright 可在测试中运行 axe，但官方明确指出自动化只能发现部分问题，建议与手工评估和包容性用户测试结合。[Playwright：无障碍测试](https://playwright.dev/docs/accessibility-testing) Chromatic 也能按组件基线阻止新增的可确定违规，但不报告 axe 的 `Needs Review`/`Incomplete` 情况。[Chromatic：无障碍测试](https://www.chromatic.com/docs/accessibility/)
- **性能**：Lighthouse CI 可在 PR 上发布 status check，以 `warn` 或非零退出码的 `error` 阈值实施性能预算；多次运行与聚合可缓解测量方差，但不能消除环境噪声。[Google：Lighthouse CI](https://web.dev/articles/lighthouse-ci) [Lighthouse CI：配置](https://github.com/GoogleChrome/lighthouse-ci/blob/main/docs/configuration.md)

因此 QA 独立执行仍有价值的区域包括：视觉/交互的“是否符合意图”、未被脚本枚举的异常路径、真实设备和辅助技术、复杂权限/第三方集成、数据迁移和环境故障、以及因成本或凭据不能在 fork PR 中自动运行的场景。Playwright 特别提醒：fork PR 无法访问 secrets，trace、报告和日志也可能含凭据或源码，应只上传可信 artifact store 或加密。[Playwright：CI 与敏感 artifact](https://playwright.dev/docs/ci-intro)

Agent 驱动探索式 QA 可以执行和记录这些步骤，但其产物应被看作**带证据的审查/测试结果**，而不是因为执行者是 AI 就自动成为确定性 CI。若结果能稳定机器判定，可逐步固化为自动测试；若仍需品味、风险接受或业务判断，则保留人工验收。

## GitHub 如何表达门禁

### Required status checks 与 review

受保护分支可要求 PR review、对话已解决、指定 status checks、成功部署和 merge queue。Required check 必须为 `successful`、`skipped` 或 `neutral`；还可限定由指定 GitHub App 报告，降低同名伪造/误报风险。[GitHub：受保护分支](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)

实践上的选项：

- 把快速、稳定、对所有 PR 都有意义的检查设为 required；
- 将慢速、平台限定、实验性或容易受外部状态影响的检查先作为非阻断 check；
- 对允许例外的规则，显式记录批准者和理由，而不是把失败强行改成绿色；
- required job 名必须唯一，否则 GitHub 可能因同名结果歧义而阻塞合并。[GitHub：受保护分支](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)

Merge queue 会在 PR 变化叠加到最新目标分支和队列中其他 PR 后重新要求 required checks，适合合并频繁的分支；CI workflow 必须同时支持 `merge_group`，否则队列会一直等不到检查结果。[GitHub：Merge Queue](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue)

### Deployments、Environments 与人工审批

GitHub 可要求 PR 的 commit 成功部署到指定环境（例如 staging）后才允许合并。[GitHub：可用 Ruleset 规则](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#require-deployments-to-succeed-before-merging) Environment 可配置 required reviewers、防止发起者自批、限制部署分支/标签，并把 environment secrets 延迟到批准后才交给 job。[GitHub：Deployments 与 Environments](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments)

这支持几种不同顺序，而非唯一流程：

1. **PR → preview/staging deploy → 自动 post-deploy E2E → required deployment/check → merge**：更早发现环境集成问题，但每个 PR 都需要可隔离环境。
2. **PR checks → merge → staging deploy → 自动 smoke/E2E → 人工批准 production**：资源较省，但问题发现更晚，需要安全回滚/停止发布。
3. **自动检测 + 人工门**：自动部署后先生成报告，由 environment reviewer 批准下一 job；拒绝会使 workflow 失败。[GitHub：Reviewing deployments](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/review-deployments)
4. **外部系统门禁**：GitHub App custom deployment protection rule 可根据监控、ITSM、安全或合规系统自动批准/拒绝，并可异步等待 webhook 回调；该功能有套餐/仓库可见性限制且仍处 public preview。[GitHub：Custom deployment protection rules](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/configure-custom-protection-rules)

### 安全检查的分层

- **push 前/时**：Secret scanning push protection 可在秘密进入仓库前阻止 push；绕过可要求指定 reviewer，并留下 alert/audit 记录。[GitHub：Push protection](https://docs.github.com/en/code-security/concepts/secret-security/push-protection)
- **PR diff**：Dependency Review 比较 base/head 的依赖变化，默认在发现新增脆弱包时失败；required 后可阻止合并，也能配置严重度和许可证策略。[GitHub：Dependency Review](https://docs.github.com/en/code-security/concepts/supply-chain-security/dependency-review)
- **代码分析**：Code scanning 可在 PR 上显示 check 与 annotation，并可作为 required check。[GitHub：Code scanning alerts](https://docs.github.com/en/code-security/concepts/code-scanning/code-scanning-alerts)
- **部署凭据**：GitHub 推荐以 OIDC 向云提供商换取短期 token，并用 environment protection 限制哪些来源能取得部署权限，避免长期云凭据。[GitHub：部署 OIDC](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-cloud-providers)

这些安全能力覆盖不同时间点，不互相替代；例如 CodeQL 不代替依赖/秘密检查，PR 扫描也不代替运行时配置、DAST 或部署后健康验证。

