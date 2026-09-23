# blueprintspec/4 契约（企业应用架构全景图 / 能力地图）

flow-canvas 多图类型引擎 · 全景图排版契约。v4 在 v3 语义分离（layout stacked/grid、band/section、命名 rail）基础上修复版式与输出忠实度：**布局不再改变架构语义**——`band`（层）与 `section`（板块）显式分离，`layout stacked|grid` 显式声明排版模式；rail 语义补齐（命名 + 贯穿高度）；节点图形严格由节点类型决定；域标题对比度达标。

## 输入 DSL（引擎自有预处理语法，非可移植 mermaid）

- 首行 `blueprint LR|TB`（LR 只控制板块内部模块网格方向，**不改变层级语义**）。
- **`layout stacked|grid`**（缺省 stacked）：
  - `stacked`：`band 标题` … `end` 声明层级，**始终按声明顺序整行上下堆叠**（分层架构语义，绝不折叠成多列）；`section` 在此模式报「形态超限」。
  - `grid`：`section 标题` … `end` 声明能力板块，可多列流式排列（能力地图）；`band` 在此模式报「形态超限」。
- `domain 标题` … `end` 在 band/section 内分组；空 band/section/domain 丢弃。
- `rail left 标题` / `rail right 标题`：**必须带栏标题**（如「外部生态」「治理保障」）；有标题 → 贯穿主体内容高度的命名侧栏；无标题 → 警告并降级为普通外部域块。
- 模块行：`ID["名称｜次级职责｜状态"]`（≤3 层）、`ID[["外部系统"]]`、`ID[("数据存储")]`。**图形由节点类型决定**：capability/module = 圆角矩形卡片；datastore = 卡片 + 左上角数据库小图标；数据域内的报表/标签/引擎等非存储能力仍用普通卡片。
- `rel A -->|标签| B` / `-.->`：关键关系（能力视图默认不写 rel = 0 条关系线）。
- 上限：模块 ≤120、band/section ≤8、domain ≤12。

## 布局规则（v3）

1. **stacked**：band 全宽纵向堆叠（声明序 = 阅读序），band 内 domain 左→右、模块网格 LR 两行优先；画布 = 内容紧包围盒（层级宽度差异是自然留白，不触发紧凑度门槛）。
2. **grid**：section 按 `bi % ncols` 落列（行优先阅读 = 声明序），列数枚举 1–3 取最接近目标宽高比；多列时 section 标题画在各自顶部右侧；LR 域网格 = 两行优先（4→2×2、5/6→3×2、7/8→4×2）。
3. **rail**：命名 rail 背景贯穿 `[y_band_top, content_bottom]`，栏标题顶部居中（fs 12 加粗）；模块列于标题下方。
4. **视觉层级与对比度**：band/section 标题 fs 13 加粗 #333（layer 级）；domain 标题 fs 12 加粗、颜色 = 域底色加深 0.45（对底色对比 ≥4.5:1）；模块名 fs 13 常规。层级：layer > domain > module。
5. **路由**：同 band 左右端口直连/L 形、相邻 band 直落两折、长跨/跨列顶底共享总线 + 脊柱兜底；质量评分（硬碰撞 > 折弯 > 路径 > 绕行 > 拥挤）；routeQuality 逐边指标入报告。
6. **紧凑度指标**：centralFillX（grid 模式硬门槛 ≥0.60）、contentFill（grid 模式软门槛 ≥50%）、maxHorizontalGap（列内最大留白）、maxVerticalGap（实测）、railDistance（实测 ≤2× 配置 + 8）、canvasAspect。stacked 模式不设紧凑度门槛（层级宽度差异属自然留白），横屏比例不达标出诚实 warning。

## 输出 SVG 契约

- 根元素：`<svg data-blueprintspec="4" viewBox="0 0 W H">`
- band/section `<g class="band">` + `<text class="band-title">`；domain `<g class="domain" data-domain data-color>` + `<text class="domain-title">`；命名 rail `<g class="rail">` 贯穿背景 + `<text class="rail-title">`；模块 `data-style="container|datastore|external"`（datastore = 卡片 + 左上角小图标）；图例/待确认同前。

## 质检

统一 `--check`：crossings / overlaps / textOverflow / labelCollisions 任一非 0 拒绝产出；metrics + routeQuality 附报告；warnings 覆盖：无标题 rail、路由质量门槛、孤儿锚点、grid 紧凑度、横屏比例等。诚实降级：交叉无法消除 → 「形态超限」拒绝产图并建议拆图。
