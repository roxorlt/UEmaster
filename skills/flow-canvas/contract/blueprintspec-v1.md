# blueprintspec/1 契约（企业应用架构全景图 / 能力地图）

flow-canvas 多图类型引擎 · 企业应用架构全景图排版契约。这类图以「包含 / 分组」为主、只画少量关键关系箭头：约束式网格布局、浅色分域、横向 16:9 倾向、高密度（30–100 模块）。与 archspec/4 同族（复用节点三层文字、注释锚点、同步/异步边型），但布局从「拓扑重心」改为「约束式网格」。

> **输入 DSL 为引擎自有预处理语法，非可移植 mermaid**：mermaid 标准引擎无法解析 `blueprint` / `band` / `domain` / `rail` / `rel` 关键字与指令行。本契约描述的 DSL 仅由 flow-canvas 蓝图布局器消费。

## 输入 DSL

```
blueprint LR
title 企业应用架构全景图
view blueprint
scope 合成数据演示（模块与关系均为合成）
version 2026-03-02
todo [A1] 部分模块职责待确认

band 用户触达层
  domain 渠道
    M1["门户｜Web 入口"]
    M2["移动端｜App 入口"]
  end
end
band 业务平台层
  domain 交易域
    T1["订单中心"]
    T2["支付中心"]
  end
  domain 用户域
    U1["账号中心"]
  end
end
rail left
  E1[["第三方支付"]]
end
rail right
  G1["安全合规"]
end
rel M1 -->|HTTPS：调用| T1
rel T2 -.->|事件| Q1
```

- **首行** `blueprint LR|TB`：`LR` 默认（横向 16:9 倾向），`TB` 纵向（不强制 16:9）。首行不是 `blueprint` 而出现 `flowchart`/`graph` → 形态超限。
- **元信息指令**（同 arch）：`title …`、`view blueprint`（固定值，唯一合法；其他值报形态超限；可缺省）、`scope …`、`version …`、`todo [A1] …`（编号锚点，未给编号自动 A1..An）。
- **band / domain**：`band 标题` … `end` 为横向层级（声明序自上而下）；band 内 `domain 标题` … `end` 为业务域（左→右）。band 内无 domain 的模块自动归入名为「其他」的 domain；空 band/domain（无模块）允许并丢弃；band 内无任何模块的 band 整体丢弃。
- **rail**：`rail left` … `end` / `rail right` … `end` 为左右侧栏，各最多一个，模块直接平铺；重复声明报形态超限。
- **模块行**（同 arch）：`ID["名称｜次级职责｜状态"]`（≤3 层，超出报形态超限）、`ID[["名称"]]` 外部系统（虚线）、`ID[("名称")]` 数据存储（圆柱体）、`ID("名称")` 圆角。标签尾部 `[A1]` 锚点挂接 todo（孤儿双向警告）。
- **关系**：`rel A -->|标签| B`（同步，实线实心箭头）/ `rel A -.->|标签| B`（异步，虚线空心箭头）。只表达关键关系、数量少。边必须以 `rel` 前缀；`rel` 之外的边或无法识别的行报形态超限。模块 id 全图唯一；`rel` 引用未定义模块报形态超限。

## 内部结构化编译（layout 前先编译成信息模型）

```
spec = {"spec": "blueprintspec/1",
        "meta": {"title","view","scope","version","todos":[{"id","text"}]},
        "direction": "LR|TB",
        "bands": [{"title", "domains": [{"title", "modules": [node_dict...]}]}],
        "rails": {"left": [node...], "right": [node...]},
        "nodes": [...全模块（含 rails），声明序...],
        "edges": [{"from","to","label","refs","async"}],
        "order": [...]  # 渲染顺序：先 bands 各 domain 模块、再 left rail、再 right rail
       }
```

node_dict 复用 arch 节点结构（id/type/shape/segs/refs/label）；`_node_style` 语义一致：external（虚线框）/ datastore（圆柱体）/ container（圆角矩形）。

## 布局算法（约束式网格，确定性）

1. **统一模块尺寸**：全图同一 cell_w/cell_h。文本用 `wrap_label` 按 ≤10 字/行换行（segs 各层分别换行）；`cell_w = clamp(max(各行文字宽)+24, 96, 220)`；`cell_h = 16 + 4×13（行数上限 4）+（任一节点有 [A1] 锚点则 +14）`。换行后仍超 cell_w−12 → textOverflow。
2. **业务域网格**：domain 内模块按列排布（行优先填充）：`cols_d = ceil(n/4)`（每列最多 4 个）、`rows_d = ceil(n/cols_d)`；`domain 宽 = cols_d×cell_w + (cols_d−1)×8 + 16`；`domain 高 = 标题行(22) + rows_d×cell_h + (rows_d−1)×row_gap(8) + 8`（row_gap 为行间隙，供正交路由穿行）。
3. **band 排布**：`band 高 = max(domain 高) + 标题行(26) + 12`；`band 宽 = 各 domain 宽和 + 间隙 16`；每个 band 在中央区水平居中；band 背景不加底色（留白 + 左侧 band 标题列区分层级，标题列宽 = 最长 band 标题 + 16）。
4. **侧栏**：left/right rail 宽 = cell_w + 32，模块从上往下单列排（顶部留标题行）；rail 底色极浅灰 `#f7f7f7` 无边框。
5. **浅色分域**：domain 背景 = 10 色浅色盘循环取色（全图 domain 出现序）：`#eef3fb,#e8f5ee,#fdf3e3,#f5eefb,#e9f4f5,#fbeef0,#f2f6e8,#efe9e1,#e6f0f7,#f7efe6`；边框 = 每通道 ×0.85 加深版；domain 标题文字用加深色。颜色不作唯一载体：domain 同时有标题 + 边框；图例必含「业务域」样例块说明颜色含义。
6. **关键关系路由**：正交折线。相邻 band 边走「A 出域 → 域间隙垂直 → band 间隙水平 → 域间隙垂直 → B 入域」；长跨 band 边走「空区脊柱」（域间隙垂直只在单带内，长垂直放空区避免交叉）；同 band 边走上下 band 间隙；侧栏边经侧栏间隙垂直走廊。候选走廊贪心取零交叉、零穿盒者，同 gap 内 lane 去重错开；关系标签在间隙中贪心摆放，避开连线/模块盒/预留区。交叉无法消除 → 形态超限降级。
7. **预留区域参与布局**：页眉（title + 图种/scope/version）在顶部；图例右上（只列实际用到：业务域/模块/数据存储/外部系统/同步调用/异步事件）；待确认项区（「A1」编号列表）底部左侧；这些区域先定尺寸，主体网格在其内排布，任何重叠由 `label_collisions` 的 `reserved` 检出（reserved 必含图例区、待确认区、页眉行）。
8. **画布**：`W = max(中央区宽 + 左栏 + 右栏 + margins, 900)`，`H = 主体高 + 页眉 + 底部区`；LR 时若 W < H×1.4 则 W = ceil(H×1.4) 并让 band 按新宽重新居中（居中随 W 自适应）。模块总数 >120、band >8、domain >12 → 形态超限降级。
9. **质检**：layout 产物必含 `crossings`（ortho_crossings）、`overlaps`（box_overlaps：模块盒 + domain 标题盒）、`textOverflow`（模块文字 vs cell、domain/band 标题 vs 列宽、关系标签边界）、`labelCollisions`（`label_collisions(routes, labels, boxes, reserved)`：labels 含所有关系标签 + 锚点徽章，boxes 为模块盒，reserved 为图例/待确认/页眉矩形）、`warnings`（无标签 rel、孤儿锚点、字号 <10，照抄 arch）。任一非 0 拒绝产出。

## 输出 SVG 契约

- 根元素 `<svg data-blueprintspec="1" viewBox="0 0 W H">`
- band：`<g class="band" id="bp-band-{i}">` + `<text class="band-title">`
- domain：`<g class="domain" data-domain="{title}" data-color="{hex}">` + `<text class="domain-title">`
- rail：`<g class="rail" id="bp-rail-left|right">`（`#f7f7f7` 底，无边框）
- 模块沿用 arch 节点渲染：`<g class="node" data-node data-style="{external|datastore|container}">`、三层文字（状态 pill）、锚点徽章 `class="note-badge"`、数据存储圆柱体（含 `<ellipse>`）
- 图例 `<g class="legend">`；待确认 `<g class="todo">`；箭头 marker `#arw`（同步实心）/ `#arw-open`（异步空心）

## 质检与自动验收

统一 `--check` 报告字段（flowcommon.build_report 自动并入 `labelCollisions`）。`crossings` / `overlaps` / `textOverflow` / `labelCollisions` 任一非 0 → 拒绝产出。可读性硬阈值：所有字号 ≥10px（`--style` 覆盖低于阈值出 warning）。warnings 覆盖：无标签边、孤儿/悬空锚点、字号阈值。
