# S2 — Asset Upload, Storage and Jobs

Status: NOT_STARTED | Priority: P0 | Depends on: S1

## Goal
实现安全文件上传、私有对象存储、异步任务基础设施和 SSE 进度。

## Tasks
1. 对象存储 adapter：本地开发 MinIO/S3-compatible，生产配置可切换。
2. 上传前校验文件大小、扩展名、MIME、magic bytes。
3. 计算 SHA-256，记录资产元数据。
4. 私有 bucket；上传/下载使用短时签名 URL。
5. 实现 assets、jobs、audit_events 基础表和迁移。
6. 建立任务队列 PoC：投递、持久状态、有限重试、超时、取消、worker restart recovery。
7. Celery/ARQ 必须按 docs/03 做对比并通过 ADR 批准后确定。
8. 实现幂等键、重复请求处理和重复投递保护。
9. 实现 GET job、cancel、retry API。
10. 实现 SSE 事件流；断线重连后可拉取快照。
11. 前端文件上传进度、任务状态和错误重试 UI。
12. 文件删除先检查引用并异步清理对象。

## Acceptance
- 上传文件 hash 与本地原文件一致。
- 非法 MIME、超限文件、路径穿越被拒绝。
- 重复 complete-upload 不产生重复解析 job。
- Worker 被杀后任务可恢复或明确失败，不会永久 stuck。
- SSE 事件顺序合理，断线后状态一致。
- 资源访问受项目权限控制。
- 文件不会出现在 public URL。
