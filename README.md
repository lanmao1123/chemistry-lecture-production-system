# Chemistry Lecture Production System

面向国际化学教师的 AI 化学讲义生产平台。项目从空仓库开始建设，目标不是通用 PDF 总结器，而是一个以**学生能否独立解题并获得分数**为质量目标的、可追溯、可审核、可迭代的化学教学内容生产系统。

> 当前状态：方案与任务规划阶段；仓库尚无业务代码。本文档集是第一版工程基线。所有未验证的产品假设均须标注，不得被实现 Agent 擅自视为已确认事实。

## 从哪里开始

1. 阅读 [项目上下文与执行规则](PROJECT_CONTEXT.md)。
2. 阅读 [产品需求与范围](docs/01_PRODUCT_REQUIREMENTS.md)。
3. 阅读 [总体架构](docs/02_SYSTEM_ARCHITECTURE.md) 和 [技术栈决策](docs/03_TECH_STACK.md)。
4. 按 [开发路线图](docs/12_DEVELOPMENT_ROADMAP.md) 的依赖顺序执行。
5. 每个阶段只执行对应 tasks/ 文件，并按 [验收标准](docs/13_ACCEPTANCE_CRITERIA.md) 验收。

## 文档导航

| 文档 | 内容 |
|---|---|
| [PROJECT_CONTEXT.md](PROJECT_CONTEXT.md) | Agent 必读项目背景、原则、执行约束 |
| [01 产品需求](docs/01_PRODUCT_REQUIREMENTS.md) | 用户、场景、功能范围、非目标、验收 |
| [02 系统架构](docs/02_SYSTEM_ARCHITECTURE.md) | 分层架构、模块边界、数据流、部署拓扑 |
| [03 技术栈](docs/03_TECH_STACK.md) | 技术选型、版本策略、替代方案与限制 |
| [04 数据库设计](docs/04_DATABASE_DESIGN.md) | 核心实体、关系、索引、状态与迁移 |
| [05 API 设计](docs/05_API_DESIGN.md) | REST API、SSE、错误码、幂等与权限 |
| [06 Agent Workflow](docs/06_AGENT_WORKFLOW.md) | LangGraph 状态机、节点、重试、人工审核 |
| [07 化学引擎](docs/07_CHEMISTRY_ENGINE.md) | ChemSpec、结构校验、图形渲染、3D 规范 |
| [08 质量门禁](docs/08_QUALITY_GATE.md) | 教学、化学、图形、来源与盲审质量体系 |
| [09 安全与隐私](docs/09_SECURITY.md) | 文件安全、认证、授权、密钥、审计 |
| [10 测试策略](docs/10_TEST_STRATEGY.md) | 单测、集成、视觉回归、E2E、评测集 |
| [11 部署运维](docs/11_DEPLOYMENT.md) | 本地开发、Docker、备份、监控、发布 |
| [12 开发路线图](docs/12_DEVELOPMENT_ROADMAP.md) | 阶段依赖、优先级、里程碑 |
| [13 验收标准](docs/13_ACCEPTANCE_CRITERIA.md) | 全局 Definition of Done 与阶段验收 |
| [14 UI 与交互规范](docs/14_UI_UX_SPEC.md) | 信息架构、页面、组件、视觉和响应式要求 |
| [15 风险与待决策项](docs/15_RISKS_AND_OPEN_QUESTIONS.md) | 风险登记、假设、需人工决策事项 |
| [开发任务目录](tasks/README.md) | 逐阶段可执行任务 |
| [ADR 决策记录](docs/adr/README.md) | 架构决策记录与变更规则 |
| [Zcode 总控提示词](prompts/ZCODE_MASTER_PROMPT.md) | 可直接交给 Zcode 的执行指令 |

## 核心产品原则

1. **教学结果优先**：以学生能否理解、迁移并独立答题为判断标准，不按固定页数或固定模板生产。
2. **教学链条完整**：按需建立“识别电子结构 → 展示电子移动/结构变化 → 解释微观机制 → 推导可观察性质 → 迁移到陌生题 → 按评分点检验”的教学链；不是要求每页机械包含全部步骤。
3. **多层化学表征**：根据知识点需要组合 Lewis contributors、弯箭头、共振杂化体/电子云、形式电荷/部分电荷、键级/键长、3D 球棍模型等表征。
4. **确定性校验优先**：LLM 负责生成与解释，程序化 Validator 负责可规则化的结构约束；模型评分不能替代化学事实验证。
5. **真题可追溯**：所有题目、图表、答案和评分点尽可能关联来源文件、页码、题号和原始裁剪区域；不伪造真题来源。
6. **教师拥有最终决定权**：大纲、关键化学结构、审核争议和发布均支持人工确认。
7. **版本与证据完整**：输入快照、生成产物、校验结果、审核记录和发布版本可追踪。
8. **失败可恢复**：任务状态持久化、节点可重试、幂等写入、错误可定位。

## MVP 建议

先交付单教师闭环：创建课程项目 → 上传 PDF → 解析和校对 → 选择知识点/真题 → 生成大纲 → 分节生成讲义 → 化学与教学审核 → 人工修改 → HTML 预览 → PDF 导出。多租户、付费、公开模板市场、复杂协作等暂不作为 MVP 阻塞项。

## 状态说明

- **Confirmed**：项目已有明确要求或已批准的决策。
- **Proposed**：本方案推荐的实现方案，实施前可通过 ADR 批准。
- **Assumption**：为了让工程可推进而暂定的假设，需在真实使用中验证。
- **Open**：尚缺信息，不允许 Agent 擅自定案。

详见 [待决策项](docs/15_RISKS_AND_OPEN_QUESTIONS.md)。
