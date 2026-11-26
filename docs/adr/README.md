# Architecture Decision Records

ADR 用于记录影响系统结构、技术栈、数据协议、兼容性和成本的决策。

## 状态
- Proposed：提出中，不能当作已批准实现依据。
- Accepted：负责人批准。
- Rejected：明确不采用。
- Superseded：被后续 ADR 替代。

## 规则
1. 一个 ADR 只讨论一个决策。
2. 必须包含背景、选项、决策、理由、后果、验证方式和日期。
3. 已 Accepted 的 ADR 不可直接覆盖；变更需新增 ADR 并标注 supersedes。
4. Agent 可以提出 ADR，但不得自行把 Proposed 改为 Accepted。
5. 影响 API、DB、ChemSpec、任务队列、身份认证、模型供应商的变更必须有 ADR。

## 计划中的 ADR
- ADR-001：模块化单体与 Worker 部署边界。
- ADR-002：身份认证方案。
- ADR-003：ChemSpec 与结构数据边界。
- ADR-004：Celery vs ARQ。
- ADR-005：TipTap vs Lexical。
- ADR-006：3D Renderer 选型。
- ADR-007：模型供应商与 ModelGateway。
