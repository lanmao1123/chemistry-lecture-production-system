# S0 — Project Bootstrap

Status: NOT_STARTED | Priority: P0 | Depends on: none

## Goal
从空仓库建立可重复启动、可测试、可持续集成的工程骨架，不实现业务功能。

## Required structure
建议 monorepo：
- apps/web：Next.js + TypeScript。
- apps/api：FastAPI 应用入口。
- apps/worker：异步 Worker 入口（初期可与 api 共用 Python package）。
- packages/contracts：OpenAPI/共享契约或生成客户端的存放位置。
- infra：Docker Compose、容器配置。
- scripts：开发、检查、seed 辅助脚本。
- docs、tasks、prompts：项目文档。
目录名称可微调，但必须保持边界清楚并在报告说明。

## Tasks
1. 建立 Node 与 Python 版本约束和 lockfiles。
2. 建立 Next.js App Router、TypeScript strict、基础 layout、错误页、loading 状态。
3. 建立 FastAPI app、/health/live、/health/ready、统一错误响应和 request_id middleware。
4. 建立 Python lint/format/typecheck/test 工具链（例如 Ruff、mypy/pyright、pytest；最终选型写明）。
5. 建立前端 lint、format、typecheck、test、build。
6. 建立 Docker Compose：PostgreSQL、Redis；健康检查和持久化 volume。
7. 建立 .env.example、.gitignore、README 本地启动指南。
8. 建立 GitHub Actions CI：前后端 lint/typecheck/test/build；依赖与 secret scan。
9. 建立 pre-commit 或等效本地质量检查。
10. 建立基础日志格式和配置加载模块。
11. 添加最小 smoke test，证明 web/API 启动。
12. 生成开发任务报告目录。

## Acceptance
- 新克隆仓库按 README 可启动。
- API health endpoint 返回可识别状态。
- Web 首页能访问，展示产品名称和空项目状态，不伪造数据。
- Postgres/Redis healthcheck healthy。
- CI 在 push/PR 自动运行并通过。
- 无真实密钥、用户资料和临时文件入库。
- 从空库 migration 命令路径已预留，但业务表在 S1 实现。
- README 说明启动、停止、测试和常见问题。

## Out of scope
登录、项目 CRUD、上传、LLM、化学结构和业务 UI 均不得在 S0 实现。
