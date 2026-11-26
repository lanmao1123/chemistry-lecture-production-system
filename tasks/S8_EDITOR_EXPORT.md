# S8 — Editor, Versioning and Export

Status: NOT_STARTED | Priority: P0 | Depends on: S7

## Goal
交付教师可编辑、预览、比较版本并导出 PDF 的完整讲义体验。

## Tasks
1. 基于 PoC 选定 TipTap 或 Lexical，记录 ADR。
2. 定义 block schema：heading、paragraph、formula、chemistry_figure、worked_example、practice、answer、pitfall、source_citation。
3. 编辑器按 section 组织；支持拖动排序、增删块、文字编辑、图示替换。
4. 选区修改请求必须绑定 section/revision 和变更范围。
5. 编辑保存创建新 revision；支持版本 diff 和恢复为新 revision。
6. 来源引用可点击回原 PDF 页。
7. 建立设计 token 和讲义 HTML renderer。
8. PDF 使用 Playwright；固定 Chromium/字体版本。
9. 导出前冻结 revision/template/renderer version。
10. 检查页数、空白页、字体嵌入、公式溢出、图像分辨率、裁切。
11. 支持整份和单 section 导出。
12. 下载链接短时有效，导出产物私有。
13. UI 支持审核状态、待处理 findings 和批准状态展示。

## Acceptance
- 教师能编辑内容并保存新版本。
- 历史版本可比较和恢复。
- HTML preview 与 PDF 主要版式一致。
- PDF 无关键公式/结构裁切，字体正常。
- 未批准版本不能标记为正式发布。
- 导出可追溯到准确 revision、模板和 renderer 版本。
