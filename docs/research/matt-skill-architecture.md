# Matt Pocock Skill 架构研究

> 状态：研究笔记，不是本项目规范。  
> 范围：`/Users/sony/.agents/skills` 下 21 个 Skill 及其直接引用的说明文件。  
> 目的：记录 Matt 的设计原则，供后续设计和 `to-spec` 综合时参考。

## 核心结论

Matt 追求的是**过程可预测**，而不是每次输出完全相同。一个 Skill 只有在以下至少一个条件成立时才值得独立存在：

1. 有独立而清晰的触发语义；
2. 必须被其他 Skill 组合调用；
3. 需要形成新的上下文边界，避免代理看到后续步骤后过早宣告完成；
4. 拥有会被多个流程复用的纪律或语言。

角色、状态、命令、文件、工作流节点和自动化动作本身都不等于 Skill。拆分会产生代价：模型可调用 Skill 持续占用上下文，用户可调用 Skill 则增加用户需要记忆的入口。

每种含义只保留一个事实来源。通用纪律由可复用 Skill 单独维护，编排 Skill 尽量薄，只负责组合，不复制规则。

来源：[`writing-great-skills/SKILL.md`](/Users/sony/.agents/skills/writing-great-skills/SKILL.md)、[`writing-great-skills/GLOSSARY.md`](/Users/sony/.agents/skills/writing-great-skills/GLOSSARY.md)。

## 21 个 Skill 的类型和语义

`disable-model-invocation: true` 表示只能由用户显式调用；未设置时，模型和其他 Skill 可以按语义调用。

| Skill | 调用类型 | 语义 |
|---|---|---|
| `ask-matt` | 用户显式 | 整套 Skill 的路由入口，只告诉用户下一步使用哪个 Skill |
| `grill-me` | 用户显式 | 对 `grilling` 的无文档访谈包装 |
| `grill-with-docs` | 用户显式 | 组合 `grilling` 与 `domain-modeling`，边澄清边维护领域语言 |
| `handoff` | 用户显式 | 把当前上下文压缩到操作系统临时文件，交给新会话 |
| `implement` | 用户显式 | 根据 Spec 或 Ticket 完成一次开发，组合 TDD、验证、审查和提交 |
| `improve-codebase-architecture` | 用户显式 | 调查代码库、选择改进点并讨论模块边界 |
| `setup-matt-pocock-skills` | 用户显式 | 一次性初始化 Matt Skills 所需的仓库能力和说明 |
| `teach` | 用户显式 | 组织多会话教学工作区和学习记录 |
| `to-spec` | 用户显式 | 把已经完成的讨论综合成 Spec，不重新访谈 |
| `to-tickets` | 用户显式 | 把 Spec 拆成可独立交付的垂直 Ticket 及阻塞关系 |
| `triage` | 用户显式 | 维护者驱动的外部输入分诊状态机 |
| `wayfinder` | 用户显式 | 用多会话调查复杂决策，产出决定而非交付物 |
| `writing-great-skills` | 用户显式 | 编写 Skill 的设计参考 |
| `grilling` | 模型可调用 | 一次只问一个问题的深度需求访谈纪律 |
| `domain-modeling` | 模型可调用 | 维护统一语言，并极克制地记录真正必要的 ADR |
| `codebase-design` | 模型可调用 | 深模块、接口和边界设计的共享语言 |
| `tdd` | 模型可调用 | 一次推进一个行为切片的测试驱动开发纪律 |
| `code-review` | 模型可调用 | 分别从 Standards 与 Spec 两条轴审查变更 |
| `diagnosing-bugs` | 模型可调用 | 面向困难故障和性能回退的诊断闭环 |
| `prototype` | 模型可调用 | 用可丢弃原型回答一个具体设计问题 |
| `research` | 模型可调用 | 委托后台代理依据一手资料形成带引用的研究笔记 |

来源：各 Skill 的 `SKILL.md` frontmatter 与正文。

## 主流程及上下文边界

Matt 的主路径可以概括为：

```text
grill-with-docs
  ├─ grilling
  └─ domain-modeling
        ↓
可选：handoff → prototype/research → handoff 返回
        ↓
to-spec → to-tickets → 每个 Ticket 在新上下文中显式调用 implement
                                      ├─ tdd
                                      └─ code-review
```

关键边界如下：

- 澄清到拆 Ticket 尽量保持连续上下文；需要换会话时使用 `handoff`。
- `to-spec` 只综合已讨论内容，不再开展需求访谈；它只会在正式产出前确认测试 seam。
- `to-tickets` 负责垂直切片和阻塞边，不负责实现。
- 每个 `implement` 从 Ticket 恢复所需上下文并完成一个开发单元。
- `implement` 是用户显式启动的开发编排，不是后台 Runtime 的别名。
- `tdd` 是被 `implement` 组合的开发纪律，一次只解决一个行为 RED。
- `code-review` 在实现完成后做 Standards 与 Spec 双轴审查。
- `triage` 处理未经整理的外部输入；由 `to-tickets` 生成的工作已经可执行，不应再次分诊。

来源：[`ask-matt/SKILL.md`](/Users/sony/.agents/skills/ask-matt/SKILL.md)、[`to-spec/SKILL.md`](/Users/sony/.agents/skills/to-spec/SKILL.md)、[`to-tickets/SKILL.md`](/Users/sony/.agents/skills/to-tickets/SKILL.md)、[`implement/SKILL.md`](/Users/sony/.agents/skills/implement/SKILL.md)。

## 编排与复用纪律

`grill-with-docs` 本身只有一条组合指令；访谈规则单一来源于 `grilling`，持久化规则单一来源于 `domain-modeling`。`implement` 同样不重复 TDD 和审查细节，只要求在合适的 seam 使用 `tdd`，结束时使用 `code-review`。

这说明“薄编排”不是能力不足，而是刻意避免规则漂移。若本项目需要复制 Matt Skills 以保证稳定，应把所需 Skill 作为受控版本一并复制，并保留这种单一来源和组合关系，而不是把它们展开复制进每个自有 Skill。

## 持久化边界

- `CONTEXT.md` 只保存项目特有的术语，每个定义一到两句话，不能保存实现细节、状态机或配置。
- ADR 只有在决定同时满足“难以逆转、缺少背景会令人意外、存在真实权衡”时才创建。
- `research` 保存引用一手来源的研究事实；研究不代替产品决策。
- `prototype` 的实现可以丢弃，只保留影响决定的结果。
- `handoff` 写入操作系统临时目录，引用已有 Spec、ADR、Issue、提交和差异，不重复它们。
- `to-spec` 才负责把已澄清内容综合为正式 Spec；讨论过程不是 Spec。
- Tracker 保存跨会话的工作状态，代码和测试保存实现与验证事实。

来源：[`domain-modeling/SKILL.md`](/Users/sony/.agents/skills/domain-modeling/SKILL.md)、[`CONTEXT-FORMAT.md`](/Users/sony/.agents/skills/domain-modeling/CONTEXT-FORMAT.md)、[`ADR-FORMAT.md`](/Users/sony/.agents/skills/domain-modeling/ADR-FORMAT.md)、[`handoff/SKILL.md`](/Users/sony/.agents/skills/handoff/SKILL.md)、[`research/SKILL.md`](/Users/sony/.agents/skills/research/SKILL.md)。

## 明确不应独立成 Skill 的内容

从 Matt 的现有拆分可以推导出以下负面边界：

- QA、DEV、Reviewer 等角色不因为是角色就自动成为 Skill；
- CI、Merge Queue、标签、状态和评论标记属于平台状态或配置；
- 单个 Runtime 步骤、命令和报告格式属于运行协议；
- Standards 与 Spec 是 `code-review` 的两个隔离审查轴，不必仅因双轴而创建两个用户入口；
- 只有某个概念具备独立触发语义、需要被组合调用或需要新的上下文边界时，才考虑把它提升为 Skill。

这些是对 Matt 结构的语义分析，不是本项目最终 Skill 列表。
