# S5 — Teaching Planning and Section Generation

Status: NOT_STARTED | Priority: P0 | Depends on: S4

## Goal
实现教师可控的教学规划、大纲审批和按 section 生成内容的 LangGraph 工作流。

## Tasks
1. 定义 Content Schema 和 LessonPlan/Section Pydantic models。
2. 建立 ModelGateway interface、provider adapter、timeout、retry、usage/cost logging。
3. Prompt registry：prompt 文件版本化，调用记录 prompt/model/schema version。
4. PlanningGraph：需求解析、来源检索、学习目标、考点、先修知识、误区、页面功能和大纲。
5. 大纲生成必须解释教学顺序与页面功能选择，不固定页数。
6. 实现教师编辑、排序、批准大纲的 HITL。
7. SectionGenerationGraph 按 docs/06 的节点契约实施。
8. 生成 Visual Explanation、Worked Example、Student Practice、Answers & Pitfalls、Summary/Reference 等按需内容。
9. 每道题保留 source_type 和 source_refs。
10. 生成内容必须包含可测量目标、解释、示例推导、练习、答案/评分点及图示规格（按需）。
11. 内容保存为 immutable revision，单节可重试。
12. Graph Checkpointer 持久化；任务重启后可恢复。
13. 建立固定 LLM evaluation dataset 和人工评分 rubric。

## Acceptance
- 教师可创建讲义计划并指定范围。
- 生成的大纲可编辑、可批准。
- 未批准大纲不能启动正式批量生成。
- section 单独生成/重试，不影响其他 section。
- 生成题目来源类型准确且有引用。
- Schema 不通过不能进入化学渲染。
- 每个节点耗时、重试、token、模型版本可追踪。
