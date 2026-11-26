# 02 — 系统总体架构

## 1. 架构原则
采用模块化单体 + 独立 Worker 的起步架构。API、Agent、解析、化学引擎和导出按模块分层，部署时可在一个代码仓库中运行多个进程。MVP 不拆成大量微服务，避免接口治理和部署成本。

## 2. 逻辑分层
1. Presentation：Next.js 页面、编辑器、预览、SSE 客户端。
2. API：FastAPI 路由、认证、授权、请求校验、响应模型。
3. Application：项目、文件、知识库、讲义、版本、审核等用例服务。
4. Workflow：LangGraph 状态图、节点编排、Checkpoint、人工确认。
5. Domain：项目/文档/题目/教学单元/ChemSpec/审核规则等领域模型。
6. Infrastructure：PostgreSQL、Redis、对象存储、模型供应商、Docling、RDKit、浏览器渲染。
7. Worker：解析、生成、化学校验、审核、导出异步任务。

## 3. 模块边界
- identity：用户、会话、角色、权限。
- workspace：课程项目、项目设置。
- asset：上传文件、对象存储、哈希、生命周期。
- ingestion：PDF 解析、OCR、页面/块/图像抽取。
- knowledge：知识点、先修关系、检索索引。
- question_bank：题目、答案、评分点、来源证据。
- planning：教学目标、大纲、section 规划。
- generation：LLM provider、prompt registry、结构化内容生成。
- chemistry：ChemSpec schema、结构适配器、Validator、Renderer。
- review：审核任务、评分维度、盲审、争议处理。
- editor：内容块、修订、版本比较。
- export：HTML/PDF 渲染和文件产物。
- observability：日志、指标、追踪、成本。

模块只能通过应用服务或定义清晰的接口交互，禁止跨模块随意读写 ORM 内部对象。

## 4. 主要数据流
### 4.1 文件入库
Browser -> Upload API -> Asset metadata + object storage -> ingestion job -> parser -> normalized document/page/block/image records -> review status -> knowledge extraction -> teacher confirmation.

### 4.2 讲义生成
Teacher request -> planning service -> retrieve source evidence -> outline draft -> teacher approval -> create generation job -> LangGraph section generation -> ChemSpec extraction -> deterministic validation -> rendering -> review graph -> quality gate -> editor draft.

### 4.3 导出
Teacher selects approved version -> export job -> resolve immutable content snapshot -> render HTML -> Playwright PDF -> PDF checks -> artifact storage -> export record -> download URL.

## 5. 异步任务模型
任务类型：ingestion、knowledge_extraction、outline_generation、section_generation、chem_validation、review、export。
统一 Job 字段：id、project_id、job_type、status、progress、current_step、input_snapshot_id、attempt、max_attempts、idempotency_key、error_code、error_message、created_at、started_at、finished_at、cancel_requested_at。
状态：queued、running、waiting_for_human、succeeded、failed、cancel_requested、cancelled。
所有 Worker 使用至少一次投递语义，业务处理必须幂等；不能假设队列 exactly-once。
长任务进度通过 DB 状态 + SSE 通知，断线后前端可重新 GET 最新状态。

## 6. 任务队列与并发
Proposed：Redis + Celery 或 ARQ。选定前用一个 ADR 确认。
- 每个项目可配置并发上限，避免多个大任务耗尽资源。
- PDF 解析、模型调用、浏览器渲染使用不同队列或并发池。
- 设置任务超时、软超时、重试退避和死信/失败记录。
- 对同一 section 的重复生成请求，使用 idempotency_key 防止重复创建。
- 取消任务需要协作式检查；正在进行的外部模型请求不一定能被即时取消。

## 7. 模型调用层
建立统一 ModelGateway，不允许业务节点直接依赖某个厂商 SDK。
输入：provider、model、messages、schema、timeout、max_tokens、temperature、trace_id。
输出：structured_result、raw_response_ref、usage、latency、provider_request_id。
要求：超时、限流、重试策略、JSON Schema/Pydantic 校验、模型版本记录、成本记录、敏感数据脱敏。
禁止无限重试；格式错误可进行有限修复，超过上限进入失败或人工处理。

## 8. 可靠性与恢复
- LangGraph Checkpointer 使用持久化存储；开发环境可用 SQLite，集成/生产建议 PostgreSQL。
- 每个节点输出结构化状态，避免把所有上下文塞进无限增长的 messages。
- 节点输入输出保留版本和摘要，敏感原文按最小化原则保存。
- 失败从安全 checkpoint 恢复；对外部副作用（写文件、创建版本）使用幂等键。
- 人工修改后创建新 revision，不能让旧 checkpoint 覆盖人工内容。

## 9. 部署拓扑
MVP：一个 Docker Compose 环境，包含 web、api、worker、scheduler（如需要）、postgres、redis、object storage（可使用 S3 兼容服务）。生产时可将对象存储切到托管 S3，Worker 横向扩展。
不建议第一版 Kubernetes。

## 10. 关键架构质量属性
- 任何讲义产物可回溯到输入文件版本、来源片段、prompt 版本、模型版本、ChemSpec 版本、模板版本和审核结果。
- Worker 崩溃不导致任务永久丢失。
- API 服务重启不影响任务状态。
- 生成内容与渲染内容分离。
- 化学验证失败可定位到具体结构/原子/键/规则。
- 不通过质量门禁的版本不能被标记为 approved。
