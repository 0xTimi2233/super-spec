# GitHub 不受信任 PR 工作流调研

> 状态：研究笔记，不是架构决策。  
> 范围：fork PR 相对同仓库分支 PR 的 GitHub 原生行为，以及内部 PR 与不受信任外部 PR 的合流方式。  
> 来源原则：仅使用 GitHub 官方文档和官方 API 文档。

## 结论摘要

**Fork PR 在协作界面上仍是普通 PR，但在 GitHub Actions 中有明确的不受信任边界。** 对公开仓库而言，没有上游写权限的外部贡献者通常通过 fork 提交 PR，因此 fork PR 通常就是这里关心的“不受信任外部 PR”；但“外部 PR”是流程语言，“fork PR”是 GitHub 的仓库拓扑和权限事实，二者不应当在所有语境中机械画等号。

GitHub 会把 fork PR 的 `pull_request`、评论和 review 等事件发送给上游 base repository，PR 也继续使用上游仓库的 Ruleset、required checks、CODEOWNERS 和 merge queue。特殊之处不在“它是不是 PR”，而在**谁控制将被执行的代码、workflow 能否自动运行、token 权限和 secrets 是否可用**。[GitHub：Fork PR 的事件](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#pull-request-events-for-forked-repositories)

平台行为可以概括为：

```text
Trusted PR
  → 固定 PR 门禁自动运行

Untrusted PR（主要是 fork PR）
  → Maintainer Trust Gate：审阅改动并批准运行不受信任代码
  → 无 secrets 的 ci / security
  → 基于可信 base workflow、只把 diff 当数据的 Standards / Spec Reviewer

两条路径随后合流
  → 同一组 required gates
  → CODEOWNERS / Repository Owner 审批
  → Merge Queue
```

因此，需要密钥的 Reviewer **不能直接照搬执行 fork 代码的 `pull_request` job**：前者拿不到密钥；从 base 默认分支运行、只读取 diff 且不执行贡献代码的 `pull_request_target` job 才可能安全取得 base 权限。

## Fork PR 到底有什么特殊

| 能力 | 同仓库可信分支 PR | Fork PR / 不受信任 PR |
|---|---|---|
| GitHub PR、Ruleset、required checks、merge queue | 正常适用 | 同样适用 |
| CODEOWNERS 自动请求 | 使用 base branch 的 CODEOWNERS | 同样使用上游 base branch 的 CODEOWNERS |
| `pull_request` 执行代码 | 可按仓库配置取得权限/secrets | `GITHUB_TOKEN` 只读，其他 Actions secrets 不下发 |
| `pull_request` 是否立即运行 | 通常自动 | 可能先等待有 write 权限的维护者批准 |
| `pull_request_target` | 使用 base 默认分支上下文 | 同样使用 base 默认分支，可自动运行并取得 base 权限/secrets；不得执行 fork 代码 |
| Dependabot PR | 有额外限制 | GitHub 将其按 fork 类似的受限来源处理 |

## 三种相关事件的准确语义

### `pull_request`：执行 PR 代码的低权限通道

`pull_request` workflow 使用 PR merge commit 的 workflow/code；默认 checkout 也是这个 merge commit。对 fork PR，GitHub 将 `GITHUB_TOKEN` 限制为只读、不给其他 secrets，并应用 fork workflow approval policy。[GitHub：安全使用 `pull_request_target`](https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target#the-risks-of-the-pull_request_target-event)

这适合运行会执行贡献者代码的 `ci` 和 `security`。维护者批准 workflow 只是同意消耗 runner 执行该代码，**不会因此把仓库 secrets 交给 fork PR 的 `pull_request` run**。[GitHub：Actions secrets 限制](https://docs.github.com/en/code-security/reference/secret-security/secret-types#actions-secrets)

直接后果：若 `review/standards` 或 `review/spec` 定义在 `pull_request` workflow 中并需要模型 API key，它们对 fork PR 不能照搬内部 PR 的执行方式。

### `pull_request_target`：可信控制面、PR diff 仅作为数据

`pull_request_target` 由同一个 PR 事件触发，但运行 base repository 默认分支上的 workflow；默认 checkout 也取 base 默认分支。它可以取得 base 仓库的 secrets 和读写 token，并且不受 fork workflow approval policy 阻塞，GitHub明确将 labeling、triage、发布认证 status check 列为适用场景。[GitHub：`pull_request_target` 事件](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#pull_request_target) [GitHub：Fork workflow 审批设置](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#controlling-changes-from-forks-to-workflows-in-public-repositories)

它可以承载只读 diff Reviewer：使用默认分支中受信任的 workflow 和 prompt，通过 GitHub API 读取 PR metadata/diff，将 diff 当作数据，然后发布 review/check。

不能在这个高权限 job 中 checkout fork head 后执行 build、测试、安装依赖或项目脚本。GitHub 将这种模式称为典型的“pwn request”风险；同样不能把下载到的 fork artifact 当可信程序执行。[GitHub：安全使用 `pull_request_target`](https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target)

### `workflow_run`：低权限运行完成后的特权后继

`workflow_run` 可以在另一个 workflow 请求或完成后运行；后继 workflow 即使前一个 workflow 没有权限，也可以取得 secrets 和 write token。其 workflow 文件必须存在于默认分支。[GitHub：`workflow_run` 事件](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#workflow_run)

它可以把“不受信任 CI 执行”和“发布有权限的结果/评论”拆成两段，但后继 workflow 必须把前序 artifact 当不受信任数据，不能下载后执行。

## 首次贡献者与外部贡献者审批

公开仓库默认要求首次贡献者的 fork PR workflow 获得维护者批准。仓库可以配置三种策略：只限制 GitHub 新用户中的首次贡献者、限制所有首次贡献者、限制所有外部贡献者。需要审批时，具有 write 权限的维护者先检查 PR，尤其是 `.github/workflows/` 变更，再点击 **Approve workflows to run**；等待超过 30 天的 run 会被删除。[GitHub：管理 Actions 设置](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#controlling-changes-from-forks-to-workflows-in-public-repositories) [GitHub：批准 fork workflow](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/approve-runs-from-forks)

官方 API 也将该策略建模为 `first_time_contributors_new_to_github`、`first_time_contributors` 或 `all_external_contributors`，并提供审批单次 run 的 endpoint。[GitHub：Actions permissions REST API](https://docs.github.com/en/rest/actions/permissions#get-the-fork-pr-contributor-approval-permissions-for-a-repository) [GitHub：Workflow runs REST API](https://docs.github.com/en/rest/actions/workflow-runs#approve-a-workflow-run-for-a-fork-pull-request)

**Approve workflows to run** 不是接受需求、批准合并或 Reviewer 通过；它只表示维护者允许 runner 执行这次不受信任的 PR 代码。具体采用哪一种 approval policy 属于仓库配置决定，不是平台事实。

## Reviewer、CODEOWNERS、Environment 与 required checks

### CODEOWNERS 会自动请求

Fork PR 的 base 如果是上游仓库，GitHub 使用上游 base branch 的 `CODEOWNERS`；PR 修改匹配路径时仍会自动请求相应 reviewer。Draft PR 不会立即通知 code owners，转为 Ready for review 时才通知。[GitHub：CODEOWNERS 与 forks](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners#codeowners-and-forks)

所以“不受信任 PR 不会自动请求任何 reviewer”不成立。正确说法是：**GitHub 原生 CODEOWNERS 正常工作；依赖 secrets 的 AI Reviewer 是否自动工作，取决于 Action trigger 的设计。**

### Environment 不应给不受信任代码补发密钥

Environment 可以要求指定 reviewer 先批准 job，且 environment secrets 只会在保护规则通过后提供给该 job。[GitHub：Deployments 与 environments](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments#environment-secrets)

Environment 审批不改变 fork PR 代码本身不受信任这一事实；如果高权限 job 执行贡献代码，审批后提供 secrets 仍然会扩大暴露面。

### Required checks 保持相同名称

Status check 可以处于 pending、passing 或 failing；required check 未成功时，受保护分支不能合并。若整个 workflow 因路径/分支过滤没有运行，对应 required check 可能一直 Pending，因此外部路径必须确保每个固定门禁最终都报告结果。[GitHub：Status checks](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks) [GitHub：排查 required checks](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/troubleshooting-required-status-checks#handling-skipped-but-required-checks)

内部与外部 PR 可以由不同 trigger/job 产生相同名称的顶层门禁；Ruleset 只观察最终 check 名称和结果，不理解底层采用了哪种触发方式。

## Dependabot 不是普通可信内部 PR

GitHub 明确将 Dependabot 触发的多个事件限制为只读 `GITHUB_TOKEN`，Actions secrets 不可用；需要私有 registry 凭据时使用 Dependabot secrets。即使使用 `pull_request_target`，Dependabot 创建的 PR 仍是只读 token 且无 secrets，这些限制在其他人 rerun 时也不消失。[GitHub：Dependabot on Actions](https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-on-actions#restrictions-when-dependabot-triggers-events)

Dependabot PR 不能因为由 GitHub 自动创建就被当作普通可信内部 PR；其受限 token 与 secrets 语义仍需按官方规则处理。
