# 开发任务清单

## 执行规则
- 严格按 docs/12_DEVELOPMENT_ROADMAP.md 的依赖执行。
- 每次只执行一个任务文件；完成并验收后再进入下一个。
- 任务状态：NOT_STARTED、IN_PROGRESS、BLOCKED、READY_FOR_REVIEW、ACCEPTED。
- 不得跳过前置依赖，不得自行修改任务验收标准。
- 每个任务完成后提交独立 commit，并更新任务状态和报告。

## 任务索引
| ID | 文件 | 目标 |
|---|---|---|
| S0 | S0_PROJECT_BOOTSTRAP.md | 从空仓库建立工程骨架、规范和 CI |
| S1 | S1_PLATFORM_FOUNDATION.md | 认证、项目、数据库和基础 API |
| S2 | S2_ASSET_PIPELINE.md | 上传、对象存储、Job、SSE |
| S3 | S3_PDF_INGESTION.md | PDF 解析、页面预览、来源定位 |
| S4 | S4_KNOWLEDGE_QUESTION_BANK.md | 知识点、题目、检索和引用 |
| S5 | S5_PLANNING_GENERATION.md | 教学规划、LangGraph、逐节生成 |
| S6 | S6_CHEMISTRY_ENGINE.md | ChemSpec、Validator、Renderer、3D PoC |
| S7 | S7_REVIEW_QUALITY_GATE.md | 独立盲审、门禁、有限修复 |
| S8 | S8_EDITOR_EXPORT.md | 编辑器、版本、HTML/PDF |
| S9 | S9_RELEASE_HARDENING.md | E2E、安全、性能、部署、教师试用 |

## 任务报告位置
每个任务在 tasks/reports/ 下新增 Sx_REPORT.md，报告包含 commit、修改文件、测试命令、测试结果、截图/日志证据、未完成项和风险。
