# ganttspec/1 契约（甘特图）

flow-canvas 多图类型引擎 · 甘特图排版契约。破坏性变更时主版本 +1。

## 输入：mermaid gantt 子集

```
gantt
  dateFormat YYYY-MM-DD
  title 发布计划
  section 开发
  需求分析 :done, a1, 2026-01-05, 3d
  编码     :active, a2, after a1, 5d
  里程碑   :milestone, m1, 2026-01-15, 0d
  section 测试
  回归测试 :a3, after a2, 2d
```

- `dateFormat`：v1 支持 `YYYY-MM-DD` 与 `HH:mm`，其他报错。
- 任务行：`名称 :[crit,] [done|active,] id, 开始, 时长`；时长 `Nd`（天）/ `Nw`（周）/ `Nm`（月 ≈ 30d）；开始为日期/时间或 `after 任务id`。
- 里程碑：`:milestone, id, 开始, 0d`；开始可为绝对日期/时间，也可为 `after 任务id`（依赖前序任务结束）；时长必须为 `0d`。
- **不支持**：`until`、百分比、双日期区间依赖等 → 报「形态超限」错误降级。

## 布局规则

1. 时间轴 nice ticks：1/2/5×10^k 步进（天单位；`HH:mm` 用分钟），**刻度间距 ≥ 刻度标签宽度 + 16px**（刻度日期文案互不重叠，间距按当前 dateFormat 的标签宽度计算，不再用固定 40px 下限）。
2. 行高统一 28px；左标签列宽 = 最宽任务名 + 边距；section 为灰底分隔带 + 标题。
3. 任务条：矩形，起于开始日期、宽 = 时长；条内文本（宽-8）放得下则居中，放不下右置条外。
4. 里程碑：日期处菱形。
5. 条外标签不能超画布右缘，超了 → 报「形态超限」错误降级。

## 输出 SVG 契约

- 根元素：`<svg data-ganttspec="1" viewBox="0 0 W H">`
- 任务：`<g class="task" id="gantt-{id}" data-task="{id}">`（含条 rect 与标签 text）

## 质检

统一 `--check` 报告字段（edges 恒为 0）；同行条重叠 0、标签溢出 0、交叉 0，任一非 0 拒绝产出并降级。
任务数 > 40 或日期解析失败 → 报「形态超限」错误降级。
