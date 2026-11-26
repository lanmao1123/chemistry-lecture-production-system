# S4 — Knowledge Base and Question Bank

Status: NOT_STARTED | Priority: P0 | Depends on: S3

## Goal
将解析资料组织成有来源的知识点和题目，并支持检索。

## Tasks
1. 实现 knowledge_items、knowledge_relations、source_chunks、questions、question_assets、question_knowledge。
2. 知识点手工创建/编辑/合并/归档。
3. 自动抽取知识点进入 pending confirmation，不直接成为 confirmed。
4. 题目抽取支持跨页题干、图表关联、分值、选项、答案和 mark scheme。
5. source_type 强制枚举：past_paper、adapted、original、teacher_authored、unknown。
6. 来源信息：文件、年份、考试局、试卷编号、题号、页码、bbox；缺失字段保持 null/unknown。
7. 题目人工确认、编辑和来源校验。
8. 初期检索实现 PostgreSQL FTS + metadata filter；pgvector 必须先评测并走 ADR。
9. Search API 返回 source snippet、asset/page/block refs 和 relevance。
10. 前端知识点树、题目列表、筛选、详情和 PDF 来源跳转。
11. 去重建议可用 canonical key + embedding（若已批准），最终由教师确认。

## Acceptance
- 可建立知识点并关联来源。
- 可建立真题/改编/原创/教师自编题。
- 未知来源不能被系统标为真题。
- 检索结果可点击回到原 PDF 页。
- 过滤条件和分页稳定。
- 对抽取低置信题目有人工校对状态。
