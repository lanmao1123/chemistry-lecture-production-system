# S6 — ChemSpec, Validator and Renderer

Status: NOT_STARTED | Priority: P0 | Depends on: S5 (ChemSpec schema design may start earlier)

## Goal
建立可版本化 ChemSpec、确定性结构检查、2D 渲染和 3D 技术验证。

## Tasks
1. 提交 ChemSpec JSON Schema/Pydantic model，版本化并建立兼容测试。
2. 建立 Atom/Bond/Charge/Stereochemistry/ElectronAnnotation/Resonance/Reaction/VisualLayer 类型。
3. 实现 schema、graph、valence、formal charge、hydrogen count、aromaticity、stereo validator。
4. 对不支持结构返回 unsupported/manual_review。
5. 建立 RDKit adapter；Indigo 仅在 PoC 有明确收益时接入。
6. 实现 2D SVG Renderer；SVG sanitize。
7. 实现箭头、电子点、形式电荷、部分电荷、标签、图层和说明标记。
8. 实现 Chromium screenshot visual check。
9. 对共振 contributors 与 resonance hybrid 分别建模。
10. 立体化学：明确 wedge/dash 与 3D 坐标关系。
11. 3D PoC 比较 3Dmol.js/Mol*，测试球棍、氢、标签、截图和 PDF fallback。
12. 建立不少于 50 个 fixture，覆盖合法、非法和 unsupported 案例。
13. 每次 ChemSpec 改动使旧 validation run 失效。
14. UI 提供结构图预览、错误定位和人工修正入口。

## Acceptance
- 结构 schema 可验证。
- 价态/连接/氢/电荷等固定错误可被识别。
- Renderer 输出可重复 SVG。
- 结构关键标签不遮挡、画布不裁切。
- 3D 只在 PoC 验收后宣称支持；计算构象标注清楚。
- Validator 不能把 parse success 当作 chemistry pass。
- unsupported 类型不会静默降级为正确。
