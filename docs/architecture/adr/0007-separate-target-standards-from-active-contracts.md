# 分离目标标准与生效契约

仓库标准和 Target Contract 可以在塑形阶段领先实现，代码、测试与 Active Contract 则只表达默认分支当前生效的现实；两者之间的 Conformance Gap 必须由开放 Issue 图覆盖，Active Contract 只能与生成代码、测试和实现原子交付。这个选择保留设计先行的能力，同时避免未来接口过早进入构建与下游消费，但要求塑形审查验证缺口覆盖和安全迁移顺序。
