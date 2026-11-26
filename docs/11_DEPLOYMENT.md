# 11 — 部署、配置与运维

## 1. 环境
- local：开发者本地。
- test：CI 自动测试。
- staging：与生产拓扑接近，使用测试数据和测试密钥。
- production：真实教师数据。
各环境数据库、对象存储、Redis、模型密钥严格隔离。

## 2. Docker Compose 服务
建议服务：web、api、worker-ingestion、worker-agent、worker-render、postgres、redis、object-storage（本地可 MinIO）。
可先合并 Worker 进程，但逻辑队列仍应分离。健康检查、依赖启动顺序、持久化 volume、日志输出必须配置。

## 3. 配置
提供 .env.example，至少包括：
APP_ENV、APP_SECRET、DATABASE_URL、REDIS_URL、OBJECT_STORAGE_ENDPOINT、OBJECT_STORAGE_BUCKET、OBJECT_STORAGE_ACCESS_KEY、OBJECT_STORAGE_SECRET_KEY、MODEL_PROVIDER、MODEL_API_KEY、MODEL_BASE_URL、FRONTEND_URL、CORS_ORIGINS、UPLOAD_MAX_BYTES、JOB_TIMEOUT_SECONDS。
真实值不得提交 Git。

## 4. 本地开发
README 提供一条清晰启动路径：复制 env → docker compose up dependencies → 安装前后端依赖 → alembic upgrade head → 启动 API/Worker/Web → health check。
提供 seed demo project 和 synthetic sample PDF；不得提交用户真实真题资料。

## 5. 监控
日志结构化输出 JSON，包含 timestamp、level、service、request_id、trace_id、job_id、project_id（避免敏感信息）、node_name。
指标：API latency/error rate、queue depth、job duration、retry count、worker memory、parse pages/min、LLM latency/tokens/cost、render duration、export failure rate。
告警：队列积压、连续任务失败、磁盘空间不足、数据库连接耗尽、对象存储错误、模型限流。

## 6. 备份恢复
- PostgreSQL 定期备份并测试恢复。
- 对象存储启用版本控制或备份策略。
- Redis 仅作为缓存/队列，不作为唯一业务数据存储。
- 备份加密、权限最小化。
- 设定 RPO/RTO 后再确认生产配置；当前数值为 Open decision。
- 至少每季度做一次恢复演练。

## 7. 发布流程
PR → CI → staging deploy → migration dry-run → smoke tests → 人工批准 → production deploy → health check → 观察指标 → 可回滚。
数据库迁移采用 expand/contract，避免新旧版本不兼容。
模型、prompt、ChemSpec、renderer、template 都独立记录版本。

## 8. 资源与成本
PDF 解析、Chromium 渲染、LLM 调用资源隔离。设置单文件页数/大小、项目并发、每次生成最大 section 数和模型 token 上限。记录模型调用成本，但不在 MVP 内实现用户计费。
