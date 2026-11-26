# S7 — Independent Review and Quality Gate

Status: NOT_STARTED | Priority: P0 | Depends on: S6

## Goal
将化学、答案、教学、视觉和来源审核落地为可追溯门禁。

## Tasks
1. 定义 rubric version、severity、finding schema 和 category taxonomy。
2. 实现 review input snapshot freeze，保存 hash。
3. Reviewer A/B 独立输入组装与权限隔离。
4. 并行执行独立审核，保存 raw outputs 和 normalized findings。
5. 比较器识别共识、分歧和单方发现。
6. BLOCKER/MAJOR/MINOR/SUGGESTION 分级。
7. 硬门禁由程序执行，不能只依赖模型总分。
8. 人工争议处理 UI 和 approvals。
9. finding fingerprint 去重，修复最多 2 轮。
10. 修复后对受影响内容重新执行 schema、化学、渲染和来源检查。
11. 发布审批绑定 revision hash。
12. 审核报告展示证据、来源定位、规则 ID、模型和 rubric 版本。

## Acceptance
- Reviewer B 在独立输出完成前无法读取 Reviewer A 结果，反之亦然。
- BLOCKER 存在时 approve/export 被 API 层拒绝。
- 所有 findings 可定位、处理和审计。
- 自动修复轮数有限，不会无限循环。
- 人工 override 必须填写理由并留痕。
- 审核报告可重现输入版本。
