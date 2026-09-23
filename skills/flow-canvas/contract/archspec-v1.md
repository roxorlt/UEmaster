# archspec/1 契约（架构图）

flow-canvas 多图类型引擎 · 架构图（分层/泳道）排版契约。破坏性变更时主版本 +1。

## 输入：mermaid 子集

flowchart TD + subgraph 泳道：

```
flowchart TD
subgraph 接入层
  A["负载均衡"]
  B["网关"]
end
subgraph 服务层
  C["用户服务"]
  D["订单服务"]
end
A --> C
B --> D
C --> D
```

- 节点：`A["文本"]` 矩形、`A[["文本"]]` 外部系统（虚线）、`A("文本")` 圆角矩形；`<br>` 换行；`class X,Y external|backend` 标类型（backend 灰底双线）。
- subgraph：`subgraph 标题` … `end`；节点按 subgraph 归组；无 subgraph 的节点归入默认泳道「其他」。
- 边：`-->` / `-.->`，带 `|label|` 标签。
- **不支持**：decision 菱形 `{}` → 报「形态超限」错误降级，不产图。

## 布局规则

1. 泳道 = subgraph，横向排列（左→右按声明序）；泳道内节点纵向排列（上→下按声明序）；泳道顶对齐。
2. 跨泳道边走泳道间共享垂直总线（两泳道间隙中点）：源右端口水平出 → 总线 → 垂直 → 目标左端口水平入；端口取边中点，同一端口多边错位 ±8px。
3. 跳级边（泳道内非相邻节点、跨多个泳道）走**顶部走廊**：源侧总线 x 上探 → 图顶走廊（每条边一条，按泳道跨度升序分配）→ 目标侧总线 x 下探；嵌套跨度零交叉，交错跨度产生交叉时报「形态超限」错误降级。
4. 泳道内相邻边垂直直线、端口取边中点；边标签放第一水平段上方居中。

## 输出 SVG 契约

- 根元素：`<svg data-archspec="1" viewBox="0 0 W H">`
- 节点：`<g class="node" id="arch-{id}" data-node="{id}">`
- 泳道：`<g class="lane" id="arch-lane-{i}">` 含背景带与 `<text class="lane-title">`

## 质检

统一 `--check` 报告字段（nodes/edges/crossings/overlaps/textOverflow/warnings/canvas）。
`crossings` / `overlaps` / `textOverflow` 任一非 0 即不合格：拒绝产出并降级。
泳道数 > 6 或交叉无法消除 → 报「形态超限」错误降级。
