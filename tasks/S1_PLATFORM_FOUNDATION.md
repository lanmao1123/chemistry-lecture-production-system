# S1 — Platform Foundation

Status: NOT_STARTED | Priority: P0 | Depends on: S0

## Goal
完成基础身份、数据库迁移、项目管理 API 和最小工作台。

## Backend
1. 配置 SQLAlchemy 2 async engine/session。
2. 配置 Alembic；验证从空库 upgrade head。
3. 实现 users、projects 表，字段按 docs/04。
4. 认证方案必须先查 docs/15 Open questions；若未获负责人确认，不得自行选型上线。可先实现 AuthProvider interface 和 mock dev auth。
5. 实现 /auth/me、logout/login 的已批准流程。
6. 实现 Project CRUD、软删除、分页、字段校验。
7. 所有资源按 owner_id 授权，增加 IDOR 测试。
8. 统一异常处理、requestId、审计事件。
9. 提供 OpenAPI 文档和契约测试。

## Frontend
1. 建立登录页（若认证方式已确认）。
2. 建立主导航和工作台。
3. 建立项目列表、创建项目、项目设置页。
4. 提供 loading/empty/error/success 状态。
5. 表单校验与后端错误映射。
6. 页面不得放入假统计数据；demo 数据须明确标记。

## Acceptance
- 用户能按已批准认证方式进入系统。
- 创建/查看/编辑/归档项目成功。
- 用户 A 不能读取用户 B 项目。
- migration 从空库成功。
- API contract 和前端调用一致。
- 单元、集成和 E2E 测试通过。
- 项目设置保存 curriculum、exam system、level、语言和 theme。
