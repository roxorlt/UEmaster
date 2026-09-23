# blueprintspec/2 契约（企业应用架构全景图 / 能力地图）

flow-canvas 多图类型引擎 · 全景图排版契约。v2 修复 v1 的画布膨胀缺陷（系统化复现：53 模块被排成 3778×2698，中央 band 仅 216–472px 宽，检查器却报全绿）。v1 → v2 破坏性变更：横屏网格改为两行优先、模块高度按实际文本行数、锚点徽章改角标不增高、候选网格按目标宽高比择优重排（禁止纯空白扩宽）、rail 紧邻中央区、图例横向、画布由内容紧包围盒生成、新增紧凑度指标。

## 输入 DSL（引擎自有预处理语法，非可移植 mermaid）

- 首行 `blueprint LR|TB`（LR 默认 = landscape 倾向横屏；TB = 纵向）。
- 指令：`title/view/scope/version/todo [A1]`（view 仅接受 blueprint）。
- `band 标题` … `end`：横向层级（声明序自上而下）；band 内 `domain 标题` … `end`：业务域分组（左→右）；band 内无 domain 的模块归「其他」；空 band/domain 丢弃。
- `rail left` / `rail right` … `end`：左右侧栏（各最多一个，重复报形态超限）。
- 模块行同 archspec：`ID["名称｜次级职责｜状态"]`（≤3 层）、`ID[["外部"]]`、`ID[("数据存储")]`；标签尾部 `[A1]` 挂接 todo（孤儿双向警告）。
- `rel A -->|标签| B` / `rel A -.->|标签| B`：关键关系（少量；同步实线/异步虚线）。
- 上限：模块 ≤120、band ≤8、domain ≤12；其他形态超限输入明确报错。

## 布局规则（v2）

1. **统一模块尺寸**：cell_w = clamp(最宽文本行 + 24, 96, 220)；cell_h = 16 + 全图最多实际文本行数 × 13（≤4 行）。**注释锚点 [A1] 渲染为右上角标，不增高节点**（正文左移避让，角标宽度计入溢出检查）。
2. **域网格（方向感知）**：LR 横屏优先两行——cols = ceil(n/2)（4→2×2、5/6→3×2、7/8→4×2）；TB 纵屏按目标列数 cols = cap。**候选网格搜索**：枚举行/列上限方案，取最接近目标宽高比（LR 1.6 / TB 0.625）者；禁止通过纯空白扩宽画布伪造横屏（内容偏高时出 warning 说明原因）。
3. **画布 = 内容紧包围盒 + margin**：W = 左栏 + 标题列 + 左右脊柱线廊 + 中央最宽 band + 右栏 + margins；H = 页眉 + band 栈 + 横向图例 + 待确认区 + margin。
4. **rail 紧邻中央内容**：与中央区固定 24px 间距（railDistance ≤ 2× 配置间距），不锚画布边缘。
5. **图例**：底部横向一至两行（按画布宽度自动换行）。
6. **band 标题**：渲染在预留标题列 x（不在固定 x=20，避免与左 rail 冲突）；band 标题/图例/待确认/页眉全部纳入碰撞检测。
7. **关系路由**：间隙走廊通道登记（每 gap 4 通道）+ 长跨 band 专用脊柱线廊（每边一个专属槽位，位于 band 两侧预留区）+ rail 水平直连 + 碰撞感知贪心（交叉×1000 + 穿盒×10000 + 通道拥挤×3）；标签沿自身路由各段多档搜索避碰。
8. **紧凑度指标**（报告 metrics，防膨胀回归）：`centralFillX`（中央 band 宽/中央可用宽，硬门槛 ≥0.60）、`contentFill`（内容面积/画布面积）、`maxHorizontalGap`（最宽与最窄 band 差）、`maxVerticalGap`（0：band 紧密堆叠）、`railDistance`、`canvasAspect`。

## 输出 SVG 契约

- 根元素：`<svg data-blueprintspec="2" viewBox="0 0 W H">`
- band `<g class="band">` + `<text class="band-title">`；domain `<g class="domain" data-domain data-color>` + `<text class="domain-title">`；rail `<g class="rail" id="bp-rail-left|right">`；图例 `<g class="legend">`；待确认 `<g class="todo">`；模块沿用 arch 节点渲染（data-style、三层文字、状态 pill、角标徽章、圆柱体）。

## 质检

统一 `--check` 报告：`crossings / overlaps / textOverflow / labelCollisions` 任一非 0 拒绝产出；`metrics` 附紧凑度指标；warnings 覆盖无标签 rel、孤儿锚点、字号 <10、横屏比例 <1.0、紧凑度硬门槛触发。
