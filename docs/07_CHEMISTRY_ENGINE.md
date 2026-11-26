# 07 — 化学领域引擎与 ChemSpec

## 1. 目标
把化学内容从自由文本中分离，建立统一的结构化协议，使生成模型、化学验证器、2D/3D 渲染器、审核器和编辑器共享同一事实来源。

ChemSpec 是内部协议，不等同于 SMILES，也不意味着 RDKit 能表达所有教学图示。它必须同时容纳分子结构数据、教学标注和渲染意图。

## 2. ChemSpec 顶层概念
建议顶层字段：
- schemaVersion
- specId
- title
- representationType：molecule2d、molecule3d、reaction、resonance、mechanism、orbital、symmetry、conceptual_diagram
- chemicalEntities
- bonds
- conformer / stereochemistry
- electronAnnotations
- formalCharges / partialCharges
- resonanceContributors
- reactionSteps
- visualLayers
- labels
- sourceRefs
- validationRequirements
- renderingHints
- provenance

具体 JSON Schema/Pydantic 模型由 S5 任务落地，schema version 必须独立管理。

## 3. 原子与键
Atom：稳定 atomId、元素、同位素、形式电荷、显式/隐式氢策略、芳香性、立体标记、坐标（如有）、标签。
Bond：稳定 bondId、atomIdA、atomIdB、bondOrder、aromatic、stereo、displayStyle。
必须检查：
- atom 引用存在；
- 不允许重复冲突键；
- 键端点不同；
- 元素、键级、价态与形式电荷组合合理；
- 氢计数策略明确；
- 芳香性表示一致；
- 手性中心/双键立体标记可被解析。
对工具不支持的元素/价态/配位体系返回 unsupported，不得自动忽略。

## 4. 电子与机理标注
Electron annotation 不能混同于原子/键本身。弯箭头需要 start、end、electronCount（2 或 1）、arrowType、stepIndex、semanticIntent。
规则检查箭头起点/终点合法性、电子来源和去向是否符合该类机理的基本规则；复杂机理仍需专家审核。
形式电荷属于离散整数属性；部分电荷属于模型/计算或教学近似数据，必须带来源/方法/说明，不能混为一谈。
共振 contributors 是多个合法 Lewis 表示；resonance hybrid 是单独的教学表示，不能把单一 contributor 当作真实结构。
键级/键长的平均或部分特征必须注明为模型/理论或教学近似。

## 5. 立体化学与 3D
- 2D 图只表达约定的投影和立体键标记，不能声称显示完整空间结构。
- 3D 数据需有明确 conformer 来源：RDKit 生成、导入文件、人工定义或外部计算。
- 生成 conformer 不等于真实实验几何；必须标注为计算/示意构象。
- 支持原子球、键棒、元素配色、标签、选中原子、旋转/缩放、视角重置。
- 需要显示隐含氢时必须显式生成或显示；不显示时要根据教学目标说明。
- 立体中心的 wedge/dash、楔线方向和空间坐标必须一致。
- 对用户特别强调的结构完整性，需展示所有关键原子、取代基、氢和环连接；禁止只给简化骨架式。
- 视频/动画属于后续能力：先实现可交互 3D，再评估导出旋转动画。

## 6. 2D Renderer
输入 ChemSpec，输出 SVG（主格式）和 PNG（派生）。
- 原子坐标布局由确定性布局算法生成，必要时允许教师拖拽。
- SVG 使用安全白名单；禁止注入任意 script、foreignObject 或外部资源。
- 结构图中的标签必须可读，不得遮挡键线、箭头或电荷。
- 字体、线宽、颜色、间距和画布尺寸由 theme tokens 控制。
- 渲染结果记录 renderer version、ChemSpec hash、theme version。
- SVG 输出后用 Chromium 截图，执行视觉检查和尺寸检查。

## 7. 3D Renderer
Proposed：评估 3Dmol.js 或 Mol*，封装统一接口。需要验证：
- 原子/键显示；
- wedge/dash 或立体信息；
- 隐含氢显示；
- 标签和电荷；
- 相机控制；
- 截图导出；
- 在 PDF/静态图中降级呈现。
3D 组件必须有静态 fallback，PDF 导出采用明确的预设视角快照，并注明图为构象示意（如适用）。

## 8. Validator 分层
A. Schema Validator：字段、类型、引用和版本。
B. Graph Validator：分子图连通性、原子/键引用、重复边。
C. Chemistry Rule Validator：价态、形式电荷、键级、氢计数、芳香性、立体约束。
D. Mechanism Validator：电子箭头语义、步骤连续性、形式电荷变化（按已实现规则）。
E. Rendering Validator：标签遮挡、超界、空 SVG、缺失原子、图片分辨率。
F. Teaching Validator：图示是否与文字解释和学习目标一致（主要依赖规则+审核，不可完全自动化）。
每条规则有 ruleId、ruleVersion、severity、message、location、evidence、suggestedFix。

## 9. RDKit / Indigo 边界
RDKit 用于分子解析、标准化、部分价态/立体/指纹等；Indigo 可作为交叉解析/绘图能力补充。二者结果冲突时记录差异并人工复核。不得将“RDKit parse 成功”误当作“化学完全正确”。
对有机机理、复杂配位、超价体系、聚合物、共振杂化体、轨道图、对称面示意等，按能力边界返回 specialized/manual_review。

## 10. 来源与可追溯
每个 ChemSpec 必须带 provenance：创建方式、关联内容 revision、来源题目/资料、人工修订记录、生成模型和 prompt version。结构修改使原 validation run 失效，必须重新校验。

## 11. 交付验收
至少准备 50 个固定化学 fixture（基础分子、芳香体系、离子、手性、环、共振、反应箭头、错误价态、缺氢、错误连接、复杂 unsupported 案例）。每个 fixture 有预期校验结果和人工审核依据。具体数量可随 S5 扩展，但不得只用 happy path。
