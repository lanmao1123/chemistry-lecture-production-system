# 10 — 测试与评测策略

## 1. 测试金字塔
- Unit：领域规则、Schema、校验器、状态转换。
- Integration：DB、对象存储、队列、模型 gateway mock、Docling adapter。
- Contract：OpenAPI schema、前后端 DTO、ChemSpec schema。
- E2E：教师完整工作流。
- Visual regression：讲义页面、SVG、PDF 截图。
- LLM evaluation：固定 prompt/模型样例集，评估结构化输出、来源忠实度、教学质量。
- Chemistry fixtures：正确结构、边界结构和错误结构。

## 2. 测试原则
- 测试可重复；LLM 测试固定输入并使用 rubric/人工标注集。
- 单元测试不调用真实付费模型。
- 外部依赖通过 adapter mock 或 test container。
- 每个 bug 修复增加回归测试。
- 不允许为通过测试而删除断言或弱化 fixture。
- 核心业务分支覆盖率建议不低于 80%；覆盖率不是正确性的替代品。

## 3. 后端测试
- Pydantic Schema 正反例。
- 项目资源越权。
- 上传幂等和重复文件。
- Job 重复投递、重试、取消、worker crash recovery。
- DB migration 从空库和上一版本升级。
- API 错误码、分页和过滤。
- 审核门禁阻止未批准 revision 导出。

## 4. 文档解析评测集
建立由教师确认的 PDF corpus，覆盖：
- 数字文本 PDF；
- 扫描 PDF；
- 双栏排版；
- 表格；
- 化学式和上下标；
- 有机结构图；
- 光谱/曲线/实验装置图；
- 页眉页脚和多页题目。
标注页码、题号、关键文本、表格单元格和图像区域。统计页面级完整率、题目切分准确率、引用定位准确率；不能只报“解析成功”。

## 5. 化学 fixture
每个 fixture 包含输入 ChemSpec、预期 parse、预期 rule findings、预期 renderer output 和人工标注。
覆盖：简单共价分子、离子、芳香环、杂环、环状结构、同位素、手性中心、E/Z、形式电荷、共振 contributor、反应箭头、缺失氢、错误价态、错误键连接、重复键、unsupported complex。
视觉测试应比较结构关键点，不仅比较整张图像像素。

## 6. LLM 离线评测
维度：Schema 合法率、事实正确率、引用精确率、答案完整率、教学目标覆盖率、练习难度梯度、重复/幻觉率、语言质量。
- 使用固定测试集和人工 gold labels。
- Prompt/model 变更前后做对比。
- 对关键化学事实做专家抽查。
- 评测报告记录模型版本、prompt version、数据集版本和成本。

## 7. 前端 E2E
核心路径：注册/登录 → 创建项目 → 上传 PDF → 解析完成 → 检查页面 → 建立知识点 → 生成大纲 → 人工确认 → 生成 section → 查看化学图 → 审核 → 编辑 → 导出 PDF → 下载。
还需覆盖断网重连、SSE 断开、任务失败重试、空状态、错误状态、长标题、窄屏和浏览器刷新。

## 8. PDF/视觉回归
- 固定 viewport、字体和 Chromium 版本。
- 检查页面数量、空白页、裁切、公式溢出、字体 fallback、图片 DPI。
- 关键页截图做视觉 diff。
- 对 SVG 检查 viewBox、尺寸、文本边界和危险元素。
- 每次模板变更必须重新生成 golden snapshots。

## 9. 发布阻断
CI 必须通过 lint、typecheck、unit、integration（可用 test services）、migration check、secret scan、dependency scan、build。失败不能合并主分支。
