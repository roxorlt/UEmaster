# seqspec/1 契约（时序图）

flow-canvas 多图类型引擎 · 时序图排版契约。破坏性变更时主版本 +1。

## 输入：mermaid sequenceDiagram 子集

```
sequenceDiagram
  participant U as 用户
  participant A as 应用
  participant D as 数据库
  U->>A: 登录请求
  A->>D: 查询用户
  D-->>A: 用户记录
  A-->>U: 登录成功
  A->>A: 记录日志
```

- `participant ID as 显示名`（v1 只支持 participant；actor 报错）。
- 消息：`A->>B: 文本`（实线）、`A-->>B: 文本`（虚线返回）、`A->>A: 文本`（自消息）。
- 消息两端必须为相邻列（列距 ≤ 1）或自消息；跨列消息 → 报「形态超限」错误降级。
- **不支持**：note、alt/loop/opt、activate/deactivate → 报「形态超限」错误降级。

## 布局规则

1. 参与者顶部一排：矩形头 + 竖直虚线生命线；列间距 = 相邻两列间消息文本最宽者 + 边距。
2. 消息箭头水平分层：每条消息 y 递增（行高 26px），永不重叠；自消息为右侧小回环（宽 40px、高 20px）。
3. 标签在箭头上方居中；虚线返回用空心箭头。

## 输出 SVG 契约

- 根元素：`<svg data-seqspec="1" viewBox="0 0 W H">`
- 参与者：`<g class="participant" id="seq-{id}">`
- 消息：`<g class="message" id="seq-msg-{i}">`（polyline + 箭头 + 标签）

## 质检

统一 `--check` 报告字段；消息交叉 0（消息-消息）、重叠 0（参与者头/标签盒）、文字溢出 0，任一非 0 拒绝产出并降级。
参与者 > 6 → 报「形态超限」错误降级。
