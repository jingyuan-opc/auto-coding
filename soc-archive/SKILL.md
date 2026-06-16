---
name: soc-archive
description: “当 OpenSpec 变更完成并准备归档时使用。通过 OpenSpec CLI 自动同步规范和归档。由 /soc-build 完成后直接触发。”
---

# 自动归档变更

归档已完成的变更。完全自动化 — 无需用户交互。

**输入**：可选择在 `/soc-archive` 后指定变更名称（例如 `/soc-archive add-auth`）。如果省略，则从对话上下文中自动检测。如果有歧义，选择最近活跃的变更。

## 步骤

1. **解析变更名称**

   如果未提供，则从对话上下文中自动检测。如果在 `/soc-build` 之后调用，使用相同的变更名称。如果确实存在歧义，使用 `openspec list --json` 并选择唯一活跃的（未归档的）变更。如果存在多个活跃变更，选择最近被引用的那个。

2. 根据项目执行基本的单元测试和编译/构建等步骤。

3. **检查产物完成状态**

   运行 `openspec status --change “<name>” --json` 检查产物完成情况。

   解析 JSON 以了解：
   - `schemaName`：正在使用的工作流
   - `planningHome`、`changeRoot`、`artifactPaths` 和 `actionContext`：路径和作用域上下文
   - `artifacts`：产物列表及其状态（`done` 或其他）

   如果状态报告 `actionContext.mode: “workspace-planning”`，报告错误并停止。

   如果有任何产物不是 `done`：报告错误并停止。

4. **检查任务完成状态**

   读取任务文件（通常是 `tasks.md`）以检查未完成的任务。

   统计标记为 `- [ ]`（未完成）与 `- [x]`（已完成）的任务数量。

   如果发现未完成的任务：报告错误并停止。

   如果不存在任务文件：报告错误并停止。

5. **自动同步增量规范**

   使用状态 JSON 中的 `artifactPaths.specs.existingOutputPaths` 检查增量规范。

   如果存在增量规范，通过 Skill 工具调用 `openspec-sync-specs` 进行同步。无论同步结果如何，都继续归档。

   如果不存在增量规范，跳过同步。

6. **执行归档**

   如果 `planningHome.changesDir` 下的 `archive` 目录不存在，则创建它：
   ```bash
   mkdir -p “<planningHome.changesDir>/archive”
   ```

   使用当前日期生成目标名称：`YYYY-MM-DD-<change-name>`

   如果目标已存在：报告错误并提供选项（重命名现有 / 删除重复项 / 稍后重试）并停止。

   否则：
   ```bash
   mv “<changeRoot>” “<planningHome.changesDir>/archive/YYYY-MM-DD-<name>”
   ```

7. **显示摘要**

   报告最终结果：
   ```
   ## 归档完成

   **变更：** <变更名称>
   **Schema：** <schema名称>
   **归档至：** <归档路径>
   **规范：** ✓ 已同步（或“无增量规范”或“同步已跳过”）
   **警告：** <任何警告，或“无”>
   ```

## 护栏

- 从上下文中自动解析变更名称，不提示用户
- 如果存在增量规范，始终同步，无需确认
- 移动到归档时保留 .openspec.yaml
- 显示清晰的最终摘要，说明发生了什么