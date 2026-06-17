# Architect Reviewer Prompt 模板

用于派发 OpenSpec proposal 的**架构质量 review**。
这是 `soc-spec` Step 7 中的**第二个** review——它在 spec-completeness review
（`spec-reviewer-prompt.md`）已派发并通过**之后**运行。

**目的：** 验证设计**架构上合理**——不只是完整和一致。
spec review 问"我们把所有东西写下来了吗？"，architect review 问
"我们写下来的东西真的是个好设计吗？"

**派发为：** `subagent_type: "architect"`。

```
Agent tool (subagent_type: "architect"):
  description: "OpenSpec change <change-name> 的 architect review"
  prompt: |
    你在执行一个 OpenSpec change 的架构质量 review。

    **Change：** <change-name>
    **工件位置：** openspec/changes/<change-name>/

    Spec-completeness review 已通过。你的工作是第二个视角：
    架构合理性、模式对齐、代码味道检测。

    ## 阅读什么

    读所有生成的工件（`proposal.md`、`specs/*.md`、`design.md`、`tasks.md`）。
    也读项目本身告诉你其约定的内容：
    `CLAUDE.md`、`README.md`、顶层 `docs/`，以及设计触及的目录中的相关源文件。
    你有文件读取工具——用它建立对这个代码库如何做事的真实图景，而不是泛泛的。

    ## 检查什么

    ### 1. 架构合理性

    - **边界：** 模块/服务/分层边界清晰吗？每个单元都有单一明确目的吗？
      依赖单向吗？无循环依赖，无伪装成"简单通信"的共享可变状态。
    - **内聚/耦合：** 你能改一个单元的内部而不破坏消费者吗？不相关的关注点
      混在同一个单元里吗？
    - **数据流：** 数据流端到端可追踪吗？状态在哪里、谁拥有、谁读、谁改？
    - **失败模式：** 设计考虑了如何失败吗——部分失败、重试、幂等、超时、
      级联失败？还是只描述了 happy path？
    - **Trade-off 已陈述：** 设计解释了为什么选这个方案而非替代方案吗？
      如果没有，实现者会猜，并猜错。

    ### 2. 模式对齐

    - **已有模式：** 设计用了代码库已有的模式吗，还是为代码库已经解决的问题
      发明新抽象？（如项目有 `Repository` 模式，设计应该用它，而不是引入新的
      data-access 层。）
    - **项目教义：** 设计尊重 `CLAUDE.md`（或等价物）对分层、错误处理、
      事务纪律、安全、测试的论述吗？
    - **命名与文件组织：** 设计遵循项目的命名约定和文件组织吗，还是强加了一个
      外来的结构？

    ### 3. YAGNI / 过度设计

    - **投机性灵活性：** 为不在 proposal 中的需求做的 hook、抽象或配置。
      "我们以后可能需要这个"不是理由。
    - **未要求的特性：** 用户没要求的任何东西。Proposal 是范围的真理源。
    - **过早优化：** 没有实际测量或明确需求支撑的缓存层、索引、并行 pipeline。
    - **Cargo cult：** 从其他系统复制但不适合这个代码库的规模、团队规模或约束的
      模式。

    ### 4. 代码味道风险

    标记如果设计**正在走向**（还没到——那是实现者要抓的）以下任一：
    - God service / god object（一个单元做很多不相关的事）
    - 贫血领域模型（数据容器，所有逻辑在 controller/handler）
    - 霰弹手术（一个逻辑改动迫使跨多文件编辑）
    - 漏的抽象（通过公共接口暴露内部细节）
    - 长事务在持有 DB 连接期间做外部调用
    - 业务逻辑在 route handler / UI 组件 / 中间件
    - 已有工具的影子实现

    ### 5. 运维关注（如适用）

    - **Migration / rollout：** 如果这改了现有行为，有 migration 路径吗？
      向后兼容？feature flag 或分阶段 rollout？
    - **Rollback：** 如果在生产中表现异常，能 revert 吗？
    - **Observability：** 新行为的日志、指标、告警？
    - **性能/规模：** 有设计需要处理的负载特征吗，设计是否为处理它们而塑形？

    ## 校准

    **只标记会在实现、review 或生产中导致真实问题的 issue。** 标准比 spec review 高：
    - spec review 抓不到的微妙耦合 → 标记
    - "我会做得稍微不同" → 不标记
    - 为假设的 100x 流量峰值缺失的优化 → 不标记
      （除非 proposal 阐述了设计错过的负载需求）
    - 两种都行的模式偏好 → 不标记

    除非有实现者会撞上的具体架构问题，否则批准。这里通过的 review 意味着：
    一个资深工程师读这个设计不会对整体方法 push back。

    ## 输出格式

    ## Architect Review

    **Status：** Approved | Issues Found

    **Issues（如有）：**
    - [工件、章节]：[具体问题] —— [为什么会导致真实问题]
      —— [改什么，如果明显]

    **Recommendations（建议性，不阻塞批准）：**
    - [改进建议]

    **Pattern alignment 注释（信息性）：**
    - [关于设计如何与 CLAUDE.md / 已有模式对齐的一行注释
      —— 只标记不匹配，不要枚举所有匹配]
```

**Reviewer 返回：** Status、Issues（如有）、Recommendations、Pattern alignment 注释。
