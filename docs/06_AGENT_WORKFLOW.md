# 06 — Agent 与 LangGraph Workflow 设计

## 1. 设计目标
Workflow 是可观察、可恢复、可测试的确定性业务编排，不是一个自由循环的超级 Agent。LLM 只在明确节点中承担分类、抽取、规划、生成和审核任务；业务状态、权限、重试、持久化和质量门禁由程序控制。

## 2. 图拆分
建议拆成独立图：
A. IngestionGraph：解析完成后的结构化整理和知识抽取。
B. PlanningGraph：需求澄清、资料检索、教学目标和大纲生成。
C. SectionGenerationGraph：逐节生成教学内容和 ChemSpec。
D. ChemistryValidationGraph：结构解析、规则检查、渲染。
E. ReviewGraph：独立审查、差异分析、修复任务。
F. ExportGraph：版本快照、HTML/PDF 渲染和导出检查。

图之间通过 job_id、artifact/version ID 传递，不在 graph state 中保存大文件二进制。

## 3. SectionGenerationGraph 节点
1. load_project_context：加载项目设置、目标学生和 curriculum。
2. retrieve_sources：按知识点/页码范围检索资料，返回有定位的 source refs。
3. build_learning_objectives：生成可测量的学习目标。
4. analyze_exam_focus：分析考点、评分要求、难度和高频误区；结论需关联来源或标注为推断。
5. choose_page_functions：决定该知识点需要哪些页面功能和图示。
6. draft_section_outline：生成 section 级结构。
7. validate_outline_schema：检查必填字段、顺序、目标可测量性。
8. generate_explanation：生成教学解释和机制链。
9. generate_worked_examples：选择有来源的真题/改编/原创例题，分步解答。
10. generate_practice：生成迁移练习及答案/评分点。
11. generate_visual_specs：生成图示意图规格，而不是直接输出最终 SVG。
12. generate_chemspec：将化学结构意图转为 ChemSpec。
13. deterministic_validation：运行规则校验。
14. assemble_content：组装符合 Content Schema 的 section。
15. quality_precheck：检查来源完整、内容字段、结构验证状态。
16. save_revision：创建 immutable revision。
遇到缺来源、结构歧义或必填字段失败时，进入 waiting_for_human 或 failed，不得伪造补齐。

## 4. PlanningGraph
- parse_teacher_request：抽取课程、知识点、目标、输出语言和约束。
- resolve_knowledge_scope：将自由文本映射到知识点/资料。
- retrieve_curriculum_evidence：找考纲、真题和教材来源。
- analyze_prerequisites：构建先修依赖。
- propose_learning_sequence：提出教学顺序。
- allocate_page_functions：按复杂度决定教学内容结构。
- draft_outline：输出结构化大纲。
- human_approval_interrupt：教师确认/修改。
- persist_approved_plan：保存批准版本。
规划不得固定每个知识点页数。应说明为何某个主题需要 1、2 或多页。

## 5. ReviewGraph
1. freeze_review_input：固定 content revision、source refs、ChemSpec、render artifacts 的 hash。
2. prepare_reviewer_a_packet：只提供 Reviewer A 所需内容。
3. prepare_reviewer_b_packet：只提供 Reviewer B 所需内容。
4. run_reviewers_independently：并行执行但隔离上下文。
5. normalize_findings：统一严重级别、分类和位置。
6. compare_findings：识别一致、冲突、单方发现。
7. hard_gate_check：程序规则判断是否阻断。
8. human_dispute_review：争议进入教师/专家确认。
9. create_fix_tasks：将已确认问题转为具体修复项。
10. apply_bounded_fixes：限定修复轮次，只修改问题涉及内容。
11. regression_review：对新增 diff 和受影响规则复测。
12. finalize_review：记录结果与证据。
Reviewer B 不得在独立审查阶段读取 Reviewer A 的意见；比较节点之后才允许合并。

## 6. State 设计
使用 typed state，建议字段：
job_id、project_id、plan_id、section_id、input_snapshot_id、source_refs、project_context_ref、outline_json、content_draft_json、chemspec_refs、validation_refs、review_refs、current_step、retry_counters、warnings、errors、approval_status、artifact_refs。
大字段存数据库/对象存储，只传引用和摘要。

## 7. 节点契约
每个节点必须声明：
- 输入字段及 schema；
- 输出字段及 schema；
- 允许副作用；
- timeout；
- retry policy；
- idempotency strategy；
- 可观测指标；
- 失败分类；
- 是否允许人工恢复。
禁止节点隐式修改共享全局状态或直接跨模块写数据库。

## 8. 重试和收敛
- 网络瞬时错误：指数退避 + jitter，最大次数有限。
- 结构化输出不合法：最多 1-2 次修复请求，之后失败/人工介入。
- 化学结构验证失败：不能只让 LLM 自我宣称修正成功；修正后重新解析并运行全部相关 Validator。
- 审核修复最多建议 2 轮自动修复；超过上限进入人工决策。
- 每轮只针对新增/未解决 findings；禁止重复报告已解决问题。
- 同一 finding 需要 fingerprint（category + location + normalized claim + artifact hash）。
- 禁止 Agent 与 Reviewer 无限循环。
- 重试不得重复创建相同 revision/export；写入操作使用幂等键。

## 9. Human-in-the-loop
必须人工确认的节点：
- 讲义大纲批准；
- 真题/改编题来源确认；
- 化学结构歧义或 Validator 不支持的类型；
- 重大审核分歧；
- 发布/导出正式版本。
人工中断必须持久化，恢复时使用同一 thread/checkpoint，并校验 revision 未被更新。

## 10. Prompt 管理
Prompt 文件按任务和版本管理，禁止散落在代码字符串。每个 model invocation 记录 prompt_version、model、schema version、temperature、token usage、latency 和 outcome。Prompt 变更必须跑离线评测集。

## 11. 评估与观测
为每个节点定义成功率、平均耗时、重试率、结构化输出校验失败率、token 成本。保存失败样例（脱敏）用于回归。模型输出不得作为唯一 ground truth。
