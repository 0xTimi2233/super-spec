# GitHub 交付工作流

用于描述从需求澄清到代码合并的统一语言。这里仅定义领域术语，不记录流程、配置或实现方案。

## 工作单元

**Issue 生命周期（Issue Lifecycle）**：
一个 Issue 从被启动开发到实现、验证、审查并结束的完整周期。

**父 Issue（Parent Issue）**：
表达共同目标、范围和跨切片约束，并汇总多个垂直切片进度的 Issue。
_避免使用_：开发任务、实现 Issue

**垂直切片 Issue（Vertical Slice Issue）**：
交付一条可独立验收的端到端产品行为的叶子 Issue。
_避免使用_：水平任务、分层任务

**开发前沿（Development Frontier）**：
需求已经稳定、依赖已经解除、可以开始开发的一组垂直切片 Issue。

**标准 Issue（Standard Issue）**：
内部需求完成澄清和接受后，以统一领域语言正式发布的 Issue。
_避免使用_：内部草稿 Issue

**规范化分诊摘要（Canonical Triage Brief）**：
在外部 Issue 原文之外维护的标准化需求摘要，用来统一范围、术语和验收行为。
_避免使用_：替代 Issue、重写用户报告

## 验收与测试

**产品 BDD（Product BDD）**：
由产品视角定义的业务验收场景，说明用户可观察到的必要行为，而不穷举测试空间。
_避免使用_：测试计划

**测试 BDD（Test BDD）**：
由 QA 从产品 BDD 推导并写在测试代码中的具体场景、边界和反例。
_避免使用_：测试计划文档

**聚焦 RED（Focused RED）**：
DEV 当前选中并准备解决的一个失败验收测试；其他失败测试可以同时存在，但不并行实现。
_避免使用_：整套测试只能有一个失败

**人工决策（Human Decision）**：
超出 DEV 和 QA 职责、必须由有权人员澄清需求或接受风险的阻塞事项。
_避免使用_：测试失败、自动重试

## 执行环境与角色

**Issue Worktree**：
一个 Issue 生命周期独享、由 QA 与 DEV 共享的隔离代码工作区，对应一个实现 PR。
_避免使用_：QA Worktree、DEV Worktree

**执行会话（Execution Session）**：
承担单一工作流角色的独立 AI 对话；它可以恢复或替换，但不承载权威工作流状态。
_避免使用_：子代理

**QA 会话（QA Session）**：
在一个 Issue 生命周期内负责测试设计、验收自动化和无法自动化场景验证的执行会话。

**DEV 会话（DEV Session）**：
在一个 Issue 生命周期内负责实现以及后续代码问题修复的执行会话。

**Reviewer 会话（Reviewer Session）**：
只审查 PR 差异并发布审查结论、不修改实现的执行会话。

**工作台会话（Workbench Session）**：
用户用于澄清需求、作出决策和启动工作的交互式会话。
_避免使用_：Orchestrator 会话

## 调度语言

**工作流状态（Workflow State）**：
描述需求和交付当前阶段的持久状态，是跨执行会话协作的权威记录。

**工作流 Runtime（Workflow Runtime）**：
根据工作流状态确定性地创建或恢复执行会话的本地运行时。
_避免使用_：Orchestrator 代理、轮询对话

**Runtime 定义（Runtime Definition）**：
声明事件如何映射到确定性步骤和角色调度的仓库内工作流定义。
_避免使用_：GitHub Actions 工作流、任意脚本

**会话注册表（Session Registry）**：
Runtime 保存的 Issue、角色、执行会话和工作区之间的本地关联。

**角色结果（Role Outcome）**：
角色完成一次工作后发布的、可被 Runtime 确定性识别的有限结果。
_避免使用_：从自由文本推断结果

**报告摘要（Report Summary）**：
角色完成工作时发布的简短人类可读说明，概括完成内容、验证证据和剩余事项。

**触发引用（Trigger Reference）**：
指向唤醒角色之 GitHub 事件的稳定标识，使会话能够自行读取原始事件和当前状态。
_避免使用_：复制事件正文

## 协作边界

**内部 PR（Internal PR）**：
由本工作流为已接受的 Issue 创建并管理的实现 PR。

**外部 PR（External PR）**：
由外部贡献者提交、其修改责任仍属于提交者的非受信任 PR。

**产品仓库（Product Repository）**：
保存产品代码及所有非敏感 Issue 和 PR 的唯一代码仓库。
_避免使用_：公开代码镜像、私有代码镜像

**控制仓库（Control Repository）**：
保存敏感规划、安全、客户或未公开工作的私有仓库。

**作品集 Project（Portfolio Project）**：
在不改变各仓库访问控制的前提下，汇总产品仓库与控制仓库工作的私有 GitHub Project。
