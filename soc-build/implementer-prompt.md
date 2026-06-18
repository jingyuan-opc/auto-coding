# Implementer 子智能体 prompt 模板

派发 implementer 子智能体时使用本模板。

 本子智能体同时担任实现者与审查者。报告 DONE 前必须完成自审。

```
Agent tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    你正在实现 Task N：[task name]

    ## 任务标识

    - **Task ID：** [task-id，如 "1.2"]
    - **Change name：** <change-name>
    - **Project root：** [project-root]
    - **Spec dir：** openspec/changes/<change-name>

    **测试范围：** 运行 `test` 门禁时只跑 scoped 集合——你新增/修改的测试文件 + 覆盖你所改动模块的已有测试。**不要**跑全量测试套件；团队会在归档前手动跑全量。

    ## 阅读顺序（按此顺序）

    1. **proposal.md** — 完整阅读。它是本次变更的"为什么"。把目标和动机内化。
    2. **tasks.md** — 按 ID（Task N）定位你的任务。**逐字引用**每条 acceptance criterion 并编号（AC-1、AC-2、…）写入 Phase 0 计划。这份清单就是你的最终 checklist——每一项都必须在代码或测试输出中可验证。
    3. **design.md** — **只读**与你任务相关的章节。如果 design 用了任务编号章节（如 `## Task 1.2`），读那一节。否则按任务标题和 AC 中的关键字 `grep`，只读匹配切片。**不要**整文件 dump。
    
    ##  写完代码后根据项目协作宪法自审（报告 DONE 前强制）
    
    ## 有业务逻辑的代码必须覆盖单元测试
    
    ## 报告格式

    完成后报告：
    - **Status：** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - **Plan summary：** 改动的文件及每个文件的作用（简短）
    - **What you implemented**（或如果 blocked，你尝试了什么）
    - **Self-audit：** 
    - **Acceptance criteria verification：** 每条 AC 的状态和证据（self-audit 里也有，但在此显式列出）
    - **Residual concerns：** 任何你不确定的地方——会出现在编排器的最终摘要中，供归档前手动验证
    - **Concerns or issues**（如有）

    DONE_WITH_CONCERNS：你完成了工作，但有需要团队在归档前手动验证的疑虑。
    BLOCKED：你无法完成任务。
    NEEDS_CONTEXT：你需要未被提供的信息。
    **永远不要**默默产出你不确定的工作。
```
