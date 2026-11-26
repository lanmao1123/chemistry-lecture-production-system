# S9 — Integration, Security and Release

Status: NOT_STARTED | Priority: P0 | Depends on: S8

## Goal
完成端到端闭环、真实教师试用、部署和上线前风险处理。

## Tasks
1. 完成从新建项目到 PDF 导出的 E2E。
2. 使用经授权/脱敏的代表性资料跑完整流程。
3. 检查解析、题目定位、知识点、生成、ChemSpec、审核和 PDF 的链路追踪。
4. 执行 IDOR、CSRF、XSS、SVG injection、恶意 PDF、SSRF、rate limit 测试。
5. 依赖漏洞和 secret scan。
6. 做并发上传、并发生成、worker 重启和模型限流测试。
7. 测量关键 API latency、job duration、LLM cost、render time。
8. 验证 DB 和对象存储备份恢复。
9. staging 部署和 smoke tests。
10. 教师试用并收集错误、编辑量、耗时和学生反馈。
11. 建立已知限制和发布回滚方案。
12. 完成 docs 与 README 更新。

## Acceptance
- 核心 E2E 全通过。
- 无未处理 BLOCKER 安全/化学问题。
- 备份恢复演练成功。
- 监控和告警可用。
- 教师能独立完成至少一份讲义生产。
- 已知限制公开记录；不得声称未经验证的准确率。
- 负责人签署发布验收。
