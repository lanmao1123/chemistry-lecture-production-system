# S3 — PDF Ingestion and Source Locator

Status: NOT_STARTED | Priority: P0 | Depends on: S2

## Goal
把 PDF 转为可定位、可校对的页面/文本/表格/图像数据，并保留原始来源。

## Tasks
1. 用代表性教师 PDF 建立匿名化/授权解析评测集。
2. Docling adapter：解析文本、页面、标题、表格、图片和公式候选。
3. 扫描页 OCR fallback；OCR 依赖需 PoC 并记录准确率/成本。
4. 保存 ParsedDocument revision、Page、Block、bbox、confidence、parser version。
5. 保留原始 PDF 页码（从 1 开始）和页面图像。
6. 支持多栏、跨页题目、页眉页脚和图文混排的识别策略。
7. 不确定内容标为 low_confidence/needs_review，不得静默补写。
8. 实现 PDF viewer，页码跳转、缩放、旋转和页面缩略图。
9. 解析文本与页面区域联动；点击 block 高亮 bbox。
10. 教师可修正标题、题号、文本、知识点标签；修正记录为新 revision。
11. 提供重新解析和 parser version 对比。
12. 解析任务有页面级进度和失败原因。

## Acceptance
- 文本型 PDF 可提取页面和段落。
- 扫描 PDF 可识别并标记 OCR 置信度。
- 表格、公式候选、图片不被静默丢弃。
- 每个 block 可追溯到 PDF 页码和位置（若 parser 提供）。
- UI 可从抽取文本定位回原页。
- 测试集报告解析指标与已知限制。
- 未确认的解析结果不直接进入正式真题库。
