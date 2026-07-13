# GitHub 交付工作流

用于描述从需求澄清到代码合并的统一语言。这里只定义本项目特有的概念，不记录流程协议、配置字段或实现步骤。

## 需求与塑形

**规范 Spec（Canonical Spec）**：
一次变更经过澄清并获批的需求事实来源，定义目标、边界、约束和产品验收行为。
_避免使用_：测试计划、实现方案

**父 Issue（Parent Issue）**：
表达共同目标、范围和跨切片约束，并汇总多个垂直切片进度的 Issue。
_避免使用_：开发任务、实现 Issue

**垂直切片 Issue（Vertical Slice Issue）**：
交付一条可独立验收的端到端产品行为的叶子 Issue。
_避免使用_：水平任务、分层任务

**Issue 图（Issue Graph）**：
由 Parent Issue、真实 Sub-issues 及其依赖关系组成的唯一交付计划。
_避免使用_：独立 Issue Plan、用 Project 表达交付语义

**开发前沿（Development Frontier）**：
需求已经稳定、依赖已经解除、可以开始开发的一组垂直切片 Issue。

**规范 Issue（Canonical Issue）**：
内部需求完成澄清和接受后，以统一领域语言正式发布的 Issue。
_避免使用_：标准 Issue、内部草稿 Issue

**规范化分诊摘要（Canonical Triage Brief）**：
在外部 Issue 原文之外维护的标准化需求摘要，用来统一范围、术语和验收行为。
_避免使用_：替代 Issue、重写用户报告

**塑形审查（Shaping Review）**：
在开发前对规范 Spec、Issue 图及相关仓库标准变更进行的语义与交付审查。

**审查修订版（Review Revision）**：
标识一次塑形审查所对应语义版本的内容指纹。
_避免使用_：把执行状态或普通元数据版本当作需求版本

**Reference 影响检查（Reference Impact Check）**：
发布规范 Spec 前，对项目级事实来源是否受到影响进行的完整预检。
_避免使用_：发布过程中临时猜测缺失决定

## 项目级事实

**仓库标准（Repository Standards）**：
对后续变更持续生效的产品、设计、架构与开发约束的总称。
_避免使用_：只把编码规范称为全部 Standards

**视觉设计 Reference（Visual Design Reference）**：
有用户界面的项目按需维护的视觉语言与布局基建，包括颜色、字体、间距、栅格、断点、形状和组件视觉规则。
_避免使用_：页面地图、导航结构、功能交互说明

**UX Reference**：
有用户界面的项目按需维护的信息架构、全局导航、页面地图、页面骨架、内容层级及响应式结构变化。
_避免使用_：视觉 Token、产品行为验收标准

**UI 原型（UI Prototype）**：
为回答尚未确定的 UI 或 UX 设计问题而创建的可抛弃实现。
_避免使用_：长期维护的 UX 文档、生产实现

**符合性缺口（Conformance Gap）**：
当前代码或 Active Contract 与已批准仓库标准之间、由开放 Issue 图覆盖的差距。
_避免使用_：另建一份 Current Architecture 文档

## 契约、验收与测试

**目标契约（Target Contract）**：
在塑形中获批、可以领先实现的目标接口语义与兼容约束。
_避免使用_：Active Contract、生成代码

**生效契约（Active Contract）**：
默认分支当前实际用于生成、编译、发布或下游消费的机器可验证契约。
_避免使用_：未来契约草案、Target Contract

**契约物化（Contract Materialization）**：
把当前切片获批的 Target Contract 转化为可供验收测试和实现使用的候选 Active Contract 与接口骨架。
_避免使用_：Interface Scaffold、产品行为实现

**契约影响（Contract Effect）**：
一个垂直切片对契约演进方式的显式分类，用来表达兼容扩展、迁移或契约实现责任。

**产品 BDD（Product BDD）**：
由产品视角定义的业务验收场景，说明用户可观察到的必要行为，而不穷举测试空间。
_避免使用_：测试计划

**测试 BDD（Test BDD）**：
由验收测试流程从产品 BDD 推导并写在测试代码中的具体场景、边界和反例。
_避免使用_：测试计划文档

**聚焦 RED（Focused RED）**：
垂直 TDD 当前选中并准备解决的一个失败验收测试。
_避免使用_：整套测试只能有一个失败

**人工决策（Human Decision）**：
超出执行流程职责、必须由有权人员澄清需求或接受风险的阻塞事项。
_避免使用_：测试失败、自动重试

## 执行与调度

**Issue 生命周期（Issue Lifecycle）**：
一个 Issue 从被启动开发到实现、验证、审查并结束的完整周期。

**Issue Worktree**：
一个 Issue 生命周期独享、由验收测试与垂直 TDD 流程共享的隔离代码工作区，对应一个实现 PR。
_避免使用_：QA Worktree、DEV Worktree

**执行会话（Execution Session）**：
执行一个工作流流程的独立 AI 对话；它可以恢复或替换，但不承载权威工作流状态。
_避免使用_：子代理

**验收测试会话（Acceptance Testing Session）**：
在一个 Issue 生命周期内持续执行验收测试流程的会话。
_避免使用_：QA Session

**垂直 TDD 会话（Vertical TDD Session）**：
在一个 Issue 生命周期内持续执行垂直 TDD 流程的会话。
_避免使用_：DEV Session

**审查会话（Review Session）**：
执行一个独立 PR 审查流程、发布结论但不修改实现的会话。
_避免使用_：Reviewer 角色

**工作台会话（Workbench Session）**：
用户用于澄清需求、作出决策和启动工作的交互式会话。
_避免使用_：Orchestrator 会话

**工作流状态（Workflow State）**：
描述需求和交付当前阶段的持久状态，是跨执行会话协作的权威记录。

**工作流 Runtime（Workflow Runtime）**：
根据工作流状态确定性地创建或恢复执行会话的本地运行时。
_避免使用_：Orchestrator 代理、轮询对话

**Runtime 定义（Runtime Definition）**：
声明事件如何映射到确定性步骤和流程调度的仓库内工作流定义。
_避免使用_：GitHub Actions 工作流、任意脚本

**会话注册表（Session Registry）**：
Runtime 保存的 Issue、流程、执行会话和工作区之间的本地关联。

**流程结果（Process Outcome）**：
一个流程完成本次工作后发布的、可被 Runtime 确定性识别的有限结果。
_避免使用_：从自由文本推断结果

**报告摘要（Report Summary）**：
流程完成本次工作时发布的简短人类可读说明，概括完成内容、验证证据和剩余事项。

**触发引用（Trigger Reference）**：
指向唤醒流程会话之 GitHub 事件的稳定标识。
_避免使用_：复制事件正文

## 仓库协作边界

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
