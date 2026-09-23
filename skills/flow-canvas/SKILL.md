---
name: flow-canvas
description: 把业务图渲染成横平竖直的规范 SVG：流程图（正交规范）、架构图（泳道分层）、ER 图、甘特图、时序图。触发：画流程图、业务流程图、架构图、ER 图、甘特图、时序图、把 mermaid 转规范图、流程图横平竖直、评审用图、render flowchart / orthogonal flowchart / architecture diagram / ER diagram / gantt / sequence diagram / draw a clean diagram from mermaid。不适用：只要 mermaid 源码文本、脑图、表格。
---

# flow-canvas 多图类型排版引擎

把 mermaid 子集（或自然语言描述）确定性排版为规范 SVG / 可缩放拖拽单文件 HTML。五类图共用一条质量纪律：`--check` 报告 crossings / overlaps / textOverflow 全 0 才合格；形态超限明确报错、调用方降级、不产烂图。排版由零依赖 Python 脚本确定性完成，不要手写 SVG 坐标。

## 五类图与输入子集

| 类型 | `--type` | mermaid 子集 | 布局要点 |
| --- | --- | --- | --- |
| 流程图 | `flowchart`（默认） | flowchart TD：`A["文本"]` `A{"判断"}` `A[["后台"]]`、`-->|label|`、`class X external/backend`、`<br>` 换行 | 单主干 + 左右分支链 + 直角回线 + 跳级边（契约 flowspec/1） |
| 架构图 | `arch` | flowchart + `subgraph … end`（嵌套 subgraph = 系统边界）+ `title/view/scope/version/todo` 指令 | 逻辑容器图（一语义一视觉）：外部虚线、边界实线、数据圆柱；拓扑列对齐（父居中于子组）；节点三层文字；注释 [A1] 锚点；同步实线/异步虚线；标签碰撞硬门槛 + 左右线廊/底部总线路由；自动图例 + 待确认项区 |
| 架构全景图 | `blueprint` | `blueprint LR` + `layout stacked|grid` + `band/section` + `domain` + `rail 标题` + `rel` 指令（引擎自有 DSL，非可移植 mermaid） | 能力地图（grid：section 多列流式）与分层架构（stacked：band 整行堆叠、声明序=阅读序）语义分离；浅色分域；数据存储 = 卡片 + 数据库小图标（仅真实存储）；rail 须命名并贯穿主体高度；域标题对比 ≥4.5:1；路由质量指标（折弯/绕行比）硬门槛 |
| ER 图 | `er` | erDiagram 实体 `{ 类型 属性 PK/FK }`、`A \|\|--o{ B : "标签"` | 实体表格块网格排布（上下左右扩展、行列对齐）+ 乌鸦脚基数记号；关系标签 ≤4 个汉字；属性注释可带引号也可不带 |
| 甘特图 | `gantt` | gantt：`dateFormat YYYY-MM-DD`/`HH:mm`、`section`、任务 `:id, 开始, Nd`、`after id`、`:milestone` | 时间轴 nice ticks（1/2/5×10^k，刻度间距 ≥ 标签宽 + 16）、统一行高、条内/条外标签、里程碑支持 after |
| 时序图 | `seq` | sequenceDiagram：`participant A as 名`、`A->>B: 消息`、`A-->>B: 返回`、`A->>A: 自消息` | 生命线 + 消息分层错位防重叠 |

各类型契约（输入语法 / 布局规则 / SVG `data-*` 契约 / 降级规则）：`contract/flowspec-v1.md`、`contract/archspec-v5.md`、`contract/erspec-v2.md`、`contract/ganttspec-v1.md`、`contract/seqspec-v1.md`、`contract/blueprintspec-v4.md`。

## 图种路由（先识别问题，再选图种与后端）

不要一收到「画技术架构图」就默认生成容器关系图。按用户想回答的问题选图种：

| 用户想回答的问题 | 图种 | 引擎入口 | 布局后端 |
|---|---|---|---|
| 哪些运行单元互相调用、请求怎么流转 | 逻辑容器图（关键链路） | `--type arch`（`view container`） | 拓扑对齐 + 正交路由 |
| 系统整体有什么、如何分层分域（汇报/评审全景） | 企业应用架构全景图/能力地图 | `--type blueprint` | 约束式网格 + 侧栏 |
| 业务流程/数据流转步骤 | 流程图 | `--type flowchart` | 主干 + 分支链 |
| 实例、网络区、高可用拓扑 | 部署图 | 引擎暂不支持 → mermaid 原生或说明 | — |
| 用户角色、页面层级、导航权限 | 产品信息架构图 | 不是本引擎图种 → 改问信息架构问题 | — |

**默认产出两张互补的图**：① 全景图（有什么、如何分层分域；能力视图默认无连线，关键关系视图只写 3–8 条跨域高层语义关系，如「交易调用/数据供给/事件通知」，协议级标签留给容器图）；② 关键链路容器图（重要请求如何流转，协议与调用方向）。两者信息模型与验收标准不同，不要靠加模块把链路图扩成全景图。全景图测试用合成数据时必须在 `scope` 里显式标注。

## 架构图（逻辑容器图）工作规则

对齐「结构化图形渲染」规范（C4 容器级）；不要和部署图、产品信息架构图混用一套结构（部署视角→另画部署图；产品 IA→改问角色/页面层级/导航）。

1. **先定图种再画图**：`view context|container|deployment|data-flow|information-architecture` 声明主视角，一张图一个视角。先列事实清单（已确认/待确认/不可推断），不得自行添加负载均衡、微服务、鉴权、消息队列、主从复制、CDN、容灾等未提供的组件。
2. **视觉语法交给引擎**：外部实体 `[["名称"]]`（虚线）、组件 `["名称"]`、数据存储 `[("名称")]`（圆柱）、系统边界 = 最外层 subgraph、逻辑层 = 内层 subgraph；同步 `-->` / 异步 `-.->`；不要再手工用虚线表达别的语义。
3. **节点三层文字**：`"名称｜次级职责｜状态"`（≤3 层）：名称主标题、职责次级小字、状态渲染为徽标；不用引号和过多括号。
4. **每条边带语义标签**：方向 + 协议/行为（`|HTTPS：请求/响应|`，不加引号）；无标签边引擎出 warnings。
5. **未知项用编号锚点挂接**：`todo [A1] 职责待确认` + 节点/边标签尾部 `[A1]`，引擎渲染编号徽章并双向检查孤儿；不要伪造结论。
6. **范围声明**：`scope 不描述部署、高可用与安全边界`；`version` 标版本/日期；`title 系统名｜逻辑容器图（视角）`。
7. 阅读方向统一自上而下；分支从明确锚点分出；不要为装饰造折线（引擎只做避障正交折线，对齐后直落 0 折弯）。

## 执行流程

1. **归一输入**。用户给了 mermaid：直接使用；给了自然语言/会议纪要：你负责梳理成对应类型的 mermaid 子集——节点用短名词句，流程图判断节点必须用"是/否"类标签区分出边（主干选择依赖它），外部系统标 `class … external`、后台/非前端节点标 `class … backend`，架构图用 subgraph 分泳道，ER 图写清 PK/FK 与基数，甘特图写清日期与依赖，时序图写清参与者与消息方向。
2. **渲染**（`scripts/flowlayout.py` 统一入口，`--type` 分发）：
   ```bash
   python3 scripts/flowlayout.py input.mmd -o out.svg                          # 纯 SVG（默认 flowchart）
   python3 scripts/flowlayout.py input.mmd -o out.svg --type arch              # 架构图
   python3 scripts/flowlayout.py input.mmd -o out.svg --type er                # ER 图
   python3 scripts/flowlayout.py input.mmd -o out.svg --type gantt             # 甘特图
   python3 scripts/flowlayout.py input.mmd -o out.svg --type seq               # 时序图
   python3 scripts/flowlayout.py input.mmd --type gantt --check                # 仅布局检查报告
   python3 scripts/flowlayout.py input.mmd -o out.html --html --title "标题"    # 可缩放拖拽画布
   ```
   flowchart 专属：`--spine A,B,C` 强制主干顺序、`--left D1` 分支链放左列；对其他类型无效。
3. **读报告 JSON**。统一字段 `{nodes, edges, crossings, overlaps, textOverflow, warnings, canvas}`：`crossings`/`overlaps`/`textOverflow` **必须全 0 才合格**。非 0 或引擎报「形态超限」时，明确告知用户并降级为 mermaid 原生渲染，**不要交付带缺陷的图**。
4. 产物默认灰度线框、禁 emoji（引擎硬校验）。调用方项目有自己的样式规范时，写覆盖 JSON 经 `--style` 传入（可覆盖键见各模块 STYLE / 流程图 `DEFAULT_STYLE`）；未声明则使用默认。

## 适用形态与降级（红线）

| 类型 | 适用形态 | 超限即报错降级（引擎明确报错，不产烂图） |
| --- | --- | --- |
| flowchart | 单主干 + 左右分支链 + 直角回线 + 左侧旁路源 + 主干跳级边 | 不符合形态（孤立链/孤岛）报错；交叉 > 3 建议 mermaid 原生 |
| arch | 泳道数 ≤ 6；矩形/外部/后台节点；subgraph 嵌套 ≤ 2 层（边界 + 层）；路由族：直落/总线/间隙走廊/左右线廊/底部总线；标签碰撞硬门槛 | decision 菱形、多系统边界、嵌套过深、穷举路由族 + 自动扩画布后仍有交叉或标签碰撞（诚实降级并建议拆图） |
| blueprint | band ≤ 8；domain ≤ 12；模块 ≤ 120；`band/domain/rail/rel` DSL；合成数据须在 scope 标注 | 重复 rail、非法 view、模块超限、关系交叉无法消除 |
| er | 实体 ≤ 8；合法基数记号；属性行齐全；绕行边 ≤ 6 | 候选排布交叉无法消除、绕行边超限 |
| gantt | `YYYY-MM-DD` / `HH:mm`；任务 ≤ 40；`Nd`/`Nw`/`Nm` 时长；`after` 依赖 | 其他 dateFormat、`until`、解析失败 |
| seq | participant ≤ 6；消息仅相邻列或自消息 | note/alt/loop/opt/激活条、跨列消息、actor |

引擎报错后告知用户该图更适合 mermaid 原生渲染；四类新类型在 `-o` 模式下检查不过会拒绝落盘。

## 联动契约（供下游消费）

- SVG 根元素带类型契约版本：`data-flowspec="1"` / `data-archspec="5"` / `data-erspec="2"` / `data-ganttspec="1"` / `data-seqspec="1"` / `data-blueprintspec="4"`。
- flowchart 可交互节点 `<g class="node" id="flowchart-{id}-{n}" data-node="{id}">`（判断节点不带 `data-node`）+ 内置 `#sel-ring` 选中框。

## 环境降级

- Windows：Python 命令通常是 `python` 或 `py -3`，不是 `python3`，按实际可用的命令替换即可。
- 无 python3：告知用户质量将降级，按对应 `contract/*spec-v1.md` 的布局规则手工生成 SVG（规则完备可手算，但无自动交叉/重叠检测）。
- 无浏览器验证工具：跳过截图目检，`--check` 报告的断言仍然有效。

## 自检

安装后运行 `python3 scripts/selftest.py`（流程图 21 项 + 四类新类型 golden 样例与形态降级断言），全部 PASS 才算可用。
