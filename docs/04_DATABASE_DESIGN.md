# 04 — 数据库设计

## 1. 设计原则
所有实体使用 UUID；时间统一存 UTC；用户可见时区来自项目/账户设置。核心表有 created_at、updated_at；重要产物使用 immutable version。所有用户资源有 owner_id 或通过 project_id 归属。文件二进制放对象存储，数据库只存元数据和引用。

## 2. 核心实体

### users
id, email, password_hash 或 external_auth_subject, display_name, status, created_at, updated_at。
密码认证与第三方 OAuth 选型需 ADR；不得明文存密码。

### projects
id, owner_id, name, description, curriculum, exam_system, level, target_language, secondary_language, theme_id, settings_json, status, created_at, updated_at, deleted_at。

### assets
id, project_id, owner_id, original_filename, mime_type, size_bytes, sha256, storage_key, upload_status, scan_status, created_at, deleted_at。
唯一约束建议 project_id + sha256（需允许教师明确上传同内容不同版本时采用版本表）。

### ingestion_jobs
id, asset_id, parser_name, parser_version, status, progress, error_code, error_detail, started_at, finished_at, created_at。

### parsed_documents
id, asset_id, ingestion_job_id, title, page_count, language, parser_metadata_json, revision, status, created_at。
每次重新解析创建新 revision，旧版本保留。

### document_pages
id, parsed_document_id, page_number（从 1 开始）, width, height, text_content, page_image_key, parse_status, confidence, created_at。
unique(parsed_document_id, page_number)。

### document_blocks
id, page_id, block_index, block_type, text_content, normalized_text, bbox_json, confidence, reading_order, metadata_json。
block_type: heading、paragraph、table、formula_candidate、image、caption、list、unknown。
bbox 坐标需记录坐标系/归一化方式。

### knowledge_items
id, project_id, title, canonical_key, description, subject_area, level, difficulty, misconceptions_json, learning_objectives_json, status, created_by, created_at, updated_at。
canonical_key 用于同项目知识点去重。

### knowledge_relations
id, project_id, source_knowledge_id, target_knowledge_id, relation_type（prerequisite、related、contrasts_with）, created_at。

### source_chunks
id, parsed_document_id, page_id, block_ids_json, chunk_text, normalized_text, token_count, embedding, embedding_model, metadata_json。
向量列仅在 pgvector ADR 通过后启用；FTS 索引先行。

### questions
id, project_id, source_document_id, source_type, source_year, exam_board, paper_code, question_number, page_start, page_end, stem_json, marks, answer_json, mark_scheme_json, status, confidence, created_at, updated_at。
source_type 枚举：past_paper、adapted、original、teacher_authored、unknown。

### question_assets
id, question_id, asset_id, page_number, bbox_json, role（diagram、table、spectrum、structure、other）。

### question_knowledge
question_id, knowledge_item_id, relevance, created_at；联合唯一键。

### lesson_plans
id, project_id, title, objective, audience, curriculum_context_json, source_selection_json, status（draft、awaiting_approval、approved、generating、completed、archived）, version, created_by, created_at, updated_at。

### lesson_sections
id, lesson_plan_id, parent_section_id, order_index, title, section_type, learning_objectives_json, key_concepts_json, prerequisite_json, misconceptions_json, visual_requirements_json, practice_requirements_json, source_refs_json, generation_status, created_at, updated_at。
section_type: visual_explanation、worked_example、student_practice、answers_pitfalls、summary_reference、mixed。

### content_revisions
id, section_id, revision_number, content_schema_version, content_json, plain_text, created_by_type, created_by_id, prompt_version, model_id, input_snapshot_id, created_at。
immutable；编辑产生新 revision。

### chem_specs
id, content_revision_id, spec_key, schema_version, structure_type, payload_json, validation_status, created_at。
同一 revision 可有多个结构规格。

### validation_runs
id, chem_spec_id, validator_version, status, findings_json, warnings_json, execution_ms, created_at。

### artifacts
id, project_id, section_id, content_revision_id, artifact_type（svg、png、html、pdf、mol、glb 等）, storage_key, sha256, mime_type, dimensions_json, renderer_version, template_version, status, created_at。

### jobs
id, project_id, job_type, status, progress, current_step, idempotency_key, input_snapshot_json, attempt, max_attempts, error_code, error_message, created_at, started_at, finished_at, cancel_requested_at。
unique(project_id, idempotency_key) where key not null。

### review_runs
id, project_id, content_revision_id, review_type, reviewer_role, reviewer_model, rubric_version, input_snapshot_hash, status, score_json, findings_json, raw_result_key, created_at, finished_at。

### review_findings
id, review_run_id, severity, category, location_json, claim, evidence_json, suggested_fix, status, resolution_note, created_at。

### approvals
id, project_id, target_type, target_id, target_version, decision, reviewer_user_id, comment, created_at。

### exports
id, project_id, lesson_plan_id, version, format, artifact_id, status, export_settings_json, created_at, completed_at。

### audit_events
id, owner_id, project_id, actor_type, actor_id, action, target_type, target_id, request_id, metadata_json, created_at。

### model_invocations
id, job_id, node_name, provider, model, prompt_version, schema_version, input_tokens, output_tokens, latency_ms, cost_amount, currency, status, error_code, request_metadata_json, created_at。

## 3. 关系概要
- User 1:N Projects
- Project 1:N Assets / KnowledgeItems / Questions / LessonPlans / Jobs
- Asset 1:N ParsedDocuments（revision）
- ParsedDocument 1:N Pages 1:N Blocks
- Question N:M KnowledgeItem
- LessonPlan 1:N LessonSections
- LessonSection 1:N ContentRevisions
- ContentRevision 1:N ChemSpecs / Artifacts / ReviewRuns
- ChemSpec 1:N ValidationRuns
- ReviewRun 1:N ReviewFindings

## 4. 索引
- 所有 project_id、owner_id 外键列建索引。
- assets(project_id, sha256)。
- document_pages(parsed_document_id, page_number)。
- document_blocks(page_id, reading_order)。
- questions(project_id, source_type, source_year)。
- lesson_sections(lesson_plan_id, order_index)。
- content_revisions(section_id, revision_number desc)。
- jobs(status, created_at)、jobs(project_id, created_at desc)。
- review_runs(content_revision_id, created_at desc)。
- audit_events(project_id, created_at desc)。
- FTS 使用语言配置明确的 tsvector；不能默认把英文 stemmer 用于中文内容。

## 5. 数据约束
- 删除 Project 使用软删除；原文件按保留策略处理。
- Source reference 必须引用有效 parsed_document/page/block。
- Approved revision 必须存在所有 required quality gates 的通过记录。
- 结构校验结果绑定 ChemSpec hash；结构变化后旧校验自动失效。
- 导出记录绑定确切 content revision 和 template version。
- 不能更新已发布 revision 的 content_json；创建新 revision。

## 6. 迁移与数据生命周期
- Alembic 从空数据库可完整建库。
- migration 包含 upgrade/downgrade（不可逆数据变更需明确说明）。
- 生产迁移前自动备份并在 staging 演练。
- 对象存储删除需先检查 DB 引用；使用异步清理任务。
- 记录保留期、用户删除请求和备份过期策略需在上线前由负责人确认。
