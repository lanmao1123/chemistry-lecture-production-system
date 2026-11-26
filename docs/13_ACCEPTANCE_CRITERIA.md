# 13 — 全局验收标准与 Definition of Done

## 1. 通用 DoD
一个任务只有同时满足以下条件才算完成：
1. 代码符合现有架构和命名规范。
2. 所有验收条款逐条映射到实现和测试。
3. 单元/集成/契约测试通过。
4. 失败路径、边界值和权限场景已覆盖。
5. 日志可定位，不泄露敏感数据。
6. 文档、OpenAPI、Schema 和配置示例同步更新。
7. 无新增 secret、明显高危依赖或临时调试代码。
8. Git diff 只包含任务范围；无无关格式化。
9. 任务报告记录命令、测试结果、commit 和剩余风险。
10. 人工验收要求已完成；未完成则状态标记 awaiting_acceptance。

## 2. S0 验收
- 仓库有明确 monorepo 结构。
- Web/API 可分别启动。
- Docker Compose 可启动 Postgres/Redis。
- 健康检查成功。
- lint/typecheck/test/build 在 CI 运行。
- .env.example 有完整占位配置，真实密钥不入库。
- README 能让新开发者从零启动。

## 3. S1 验收
- 登录/登出/当前用户 API。
- 项目创建、查询、更新、软删除。
- 资源越权测试通过。
- DB migration 从空库成功。
- 前端登录和项目工作台可操作。

## 4. S2/S3 验收
- 上传后对象存储存在且 hash 一致。
- 重复请求幂等。
- Worker 重启后任务可恢复/可重试。
- PDF 页面数与预览可见。
- 页面文本/图片块有页码和定位。
- 解析低置信内容显式标记。
- 失败任务可解释、重试和取消。

## 5. S4/S5 验收
- 知识点和题目可人工确认。
- 真题来源字段不可缺省为 past_paper。
- 检索结果显示来源文档、页码和片段。
- 大纲包含学习目标、考点、先修知识、误区、页面功能。
- 教师可修改和批准大纲。
- 分节生成支持单节重试和版本记录。
- 所有结构化输出通过 schema。

## 6. S6 验收
- ChemSpec 有版本化 schema。
- 固定 fixture 覆盖正确、错误和 unsupported。
- Validator 可给出 ruleId、severity、location、evidence。
- 修改结构后旧验证自动失效。
- 2D SVG 渲染可复现，Chromium 截图检查通过。
- 3D PoC 明确记录支持范围和限制；未完成不得宣传为完整 3D 化学编辑器。

## 7. S7 验收
- Reviewer 输入独立，不能读取另一 reviewer 结果。
- 原始审核结果和 hash 持久化。
- BLOCKER 阻断发布。
- 自动修复有轮数上限和 finding 去重。
- 争议可人工处理。
- 每个 finding 有状态、证据和解决记录。

## 8. S8/S9 验收
- section 级编辑和版本回滚。
- HTML 预览与 PDF 视觉一致。
- 导出检查页数、字体、公式、图像和空白页。
- 一份完整讲义可从资料上传走到导出。
- E2E、权限、安全、备份恢复和部署 smoke tests 通过。
- 教师完成真实试用并记录反馈。

## 9. 验收报告模板
- Stage / Task ID：
- Commit：
- Scope：
- Acceptance item：
- Evidence（测试命令、截图、日志、fixture）：
- Passed / Failed / Blocked：
- Known limitations：
- Manual approval：
