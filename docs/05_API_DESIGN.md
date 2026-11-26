# 05 — API 契约设计

## 1. 通用约定
Base path：/api/v1。JSON 使用 camelCase 或 snake_case 必须统一；建议 API JSON 使用 camelCase，Python 内部 snake_case。所有时间 ISO-8601 UTC。资源 ID UUID。

成功响应建议：
{ "data": ..., "meta": { "requestId": "..." } }

错误响应：
{ "error": { "code": "VALIDATION_ERROR", "message": "Readable message", "details": [], "requestId": "..." } }

分页：page、pageSize；返回 total、hasMore。默认 pageSize 20，最大 100。
写请求支持 Idempotency-Key；上传使用 multipart 或预签名 URL。所有接口校验用户对 project/resource 的归属。

## 2. Authentication
- POST /auth/register（若采用开放注册）
- POST /auth/login
- POST /auth/logout
- GET /auth/me
具体认证方式（Cookie session / JWT）需 ADR。浏览器优先 HttpOnly Secure SameSite cookie，避免 localStorage 长期保存 bearer token。

## 3. Projects
- GET /projects
- POST /projects
- GET /projects/{projectId}
- PATCH /projects/{projectId}
- DELETE /projects/{projectId}
- GET /projects/{projectId}/activity

Project DTO 字段：name、description、curriculum、examSystem、level、targetLanguage、secondaryLanguage、themeId、settings。

## 4. Assets & ingestion
- POST /projects/{projectId}/assets：创建上传记录或返回预签名上传 URL。
- POST /assets/{assetId}/complete-upload：校验对象已上传并创建 ingestion job。
- GET /assets/{assetId}
- GET /assets/{assetId}/pages
- GET /assets/{assetId}/pages/{pageNumber}
- GET /assets/{assetId}/blocks
- POST /assets/{assetId}/reprocess
- DELETE /assets/{assetId}
- GET /jobs/{jobId}
- POST /jobs/{jobId}/cancel
- POST /jobs/{jobId}/retry（仅允许可重试状态）

## 5. Knowledge & questions
- GET/POST /projects/{projectId}/knowledge-items
- GET/PATCH/DELETE /knowledge-items/{knowledgeId}
- POST /knowledge-items/{id}/confirm
- GET/POST /projects/{projectId}/questions
- GET/PATCH /questions/{questionId}
- POST /questions/{questionId}/confirm
- POST /projects/{projectId}/search：请求 query、filters、topK；响应 chunks、source refs、score 和检索策略。
所有 RAG 结果必须包含可点击的文件/页码引用。

## 6. Lesson planning & generation
- POST /projects/{projectId}/lesson-plans
- GET /lesson-plans/{planId}
- PATCH /lesson-plans/{planId}
- POST /lesson-plans/{planId}/generate-outline
- POST /lesson-plans/{planId}/approve-outline
- POST /lesson-plans/{planId}/sections/reorder
- POST /lesson-sections/{sectionId}/generate
- POST /lesson-sections/{sectionId}/revise
- GET /lesson-sections/{sectionId}
- GET /lesson-sections/{sectionId}/revisions
- POST /content-revisions/{revisionId}/restore-as-new-revision
生成请求应返回 jobId；不得同步等待完整 LLM 工作流。

## 7. Chemistry & review
- POST /content-revisions/{revisionId}/validate-chemistry
- GET /chem-specs/{chemSpecId}
- GET /validation-runs/{runId}
- POST /content-revisions/{revisionId}/review
- GET /review-runs/{runId}
- GET /review-runs/{runId}/findings
- POST /review-findings/{findingId}/resolve
- POST /content-revisions/{revisionId}/approve
approve 接口必须重新检查门禁，不能只信任前端状态。

## 8. Editor & export
- PATCH /content-revisions/{revisionId}/draft（保存为新 revision 的草稿或创建 revision）
- GET /artifacts/{artifactId}
- POST /lesson-plans/{planId}/exports
- GET /exports/{exportId}
- GET /exports/{exportId}/download
下载使用短时签名 URL；不要公开对象存储 bucket。

## 9. SSE 事件
GET /jobs/{jobId}/events。
事件类型：job.started、job.progress、job.step_changed、job.waiting_for_human、job.succeeded、job.failed、job.cancelled。
payload 包含 jobId、status、progress、step、timestamp、requestId。
SSE 不作为唯一状态来源；客户端重连先 GET job snapshot。心跳间隔和代理缓冲配置需在部署文档落实。

## 10. API 安全
- 所有资源按 owner/project 授权。
- 防止 IDOR：不能仅凭 UUID 访问资源。
- 限制上传大小、速率、并发和文件类型。
- 对搜索和生成接口做配额保护。
- 统一 requestId 和审计日志。
- 参数错误 422，未认证 401，无权 403，不存在 404，冲突 409，限流 429，服务异常 500。
- 不在错误响应中泄漏路径、密钥、SQL 或原始堆栈。
