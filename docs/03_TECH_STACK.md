# 03 — 技术栈与选型决策

## 1. 推荐基线

| 层 | 技术 | 决策状态 | 理由 |
|---|---|---|---|
| Web | Next.js + React + TypeScript | Proposed | 页面路由、组件生态、类型安全 |
| UI | Tailwind CSS + shadcn/ui | Proposed | 可控组件和统一视觉 token |
| Editor | TipTap / Lexical 评估后选一 | Open | 需验证数学公式、块编辑、图片和自定义化学块 |
| API | Python 3.12 + FastAPI | Proposed | 与解析、化学和 Agent 生态一致 |
| Schema | Pydantic v2 | Proposed | 请求/响应与 ChemSpec 数据验证 |
| Workflow | LangGraph | Confirmed direction | 持久状态、条件路由、人工中断 |
| ORM | SQLAlchemy 2 + Alembic | Proposed | 显式 ORM 与迁移 |
| DB | PostgreSQL 16+ | Proposed | 事务、全文检索、JSONB、可扩展向量能力 |
| Queue | Redis + Celery 或 ARQ | Open | 通过 ADR 和 PoC 决定 |
| PDF parse | Docling | Proposed | 文档结构化解析 |
| OCR | Docling OCR pipeline / Tesseract 或托管 OCR | Open | 需用真实扫描卷评估准确率和成本 |
| Chemistry | RDKit | Proposed | 分子结构解析、规范化、描述符、部分结构检查 |
| Chemistry format | ChemSpec 内部 schema | Proposed | 让 LLM、验证器、渲染器使用统一协议 |
| 2D drawing | RDKit SVG + 自定义 SVG overlay | Proposed | 可控结构绘图 |
| 3D | RDKit conformer + 3Dmol.js 或 Mol* 评估 | Open | 需评估立体教学图与导出能力 |
| Math | MathJax + mhchem | Proposed | 化学公式与数学表达 |
| PDF output | Playwright Chromium | Proposed | HTML/CSS 到 PDF，统一预览与导出 |
| Storage | S3-compatible object storage | Proposed | 原文件与产物分离 |
| Test | pytest + Playwright + Vitest | Proposed | 后端、E2E、前端单测 |
| Deploy | Docker Compose | Proposed | MVP 简化部署 |

## 2. 技术选型规则
- 先验证关键能力，再锁定版本；所有依赖必须 pin 版本并提交 lockfile。
- 生产依赖不得使用浮动 latest。
- 新增第三方 SaaS 必须记录数据处理位置、费用、隐私、可替代性。
- 模型供应商通过 ModelGateway 接入。
- 避免为了“先进”引入不必要的向量数据库、微服务、Kubernetes。
- 对化学结构图、PDF 解析和编辑器进行 PoC，结果写入 ADR。

## 3. 前端建议
- Next.js App Router。
- TypeScript strict true。
- React Query/TanStack Query 管理服务端状态。
- Zod 校验前端表单和 API 边界。
- Zustand 仅用于确有必要的跨组件临时 UI 状态。
- SSE 处理任务进度；重连后先拉取任务快照。
- 组件按 feature/domain 组织，不做超大通用组件。

## 4. 后端建议
- FastAPI async endpoints；CPU 密集任务进入 Worker。
- SQLAlchemy async session；事务范围短且明确。
- Pydantic DTO 与 ORM entity 分离。
- 领域逻辑放 application/domain service，不放路由函数。
- Alembic 所有数据库变更迁移化。
- 统一错误响应、request_id、trace_id。
- 对外 API 使用 /api/v1 前缀。

## 5. Worker 选型
Celery 适合成熟任务路由、重试和生态；ARQ 更轻量且与 asyncio 结合紧密。必须先实现 PoC：任务重启恢复、重试、超时、取消、任务状态持久化、重复投递幂等。将结果记录在 docs/adr/ADR-004-QUEUE.md 后再正式选定。

## 6. 向量检索
第一版先建立 Postgres FTS + metadata filters，评估资料规模和语义召回需求。若引入 pgvector，需评估 embedding 模型、索引、更新策略和召回评测。禁止为了“RAG 标配”而默认引入独立 Milvus/Weaviate。

## 7. 化学能力边界
RDKit 能提供结构解析和多种计算，但不能自动证明任意教学机理、反应条件、命名或图示解释完全正确。必须按具体规则实现 Validator；不支持的化学类型明确返回 unsupported/manual_review，不允许返回虚假的 pass。

## 8. 版本与兼容
- Python、Node、PostgreSQL、Redis、浏览器镜像版本写入 .tool-versions / Dockerfile / lockfile。
- API schema 变更须更新契约测试和文档。
- ChemSpec schema 使用独立 semver。
- Prompt、模板、校验规则均有版本号。
- 数据库 migration 必须支持从空库创建完整 schema。
