# flow-canvas

把业务图（mermaid 子集 / 自然语言 / flowspec JSON）渲染成**横平竖直的规范 SVG** 的 Agent Skill：五类图共用一条质量纪律——`--check` 报告 `crossings / overlaps / textOverflow` 全 0 才合格，形态超限明确报错降级、不产烂图。

| 类型 | `--type` | 说明 |
| --- | --- | --- |
| 流程图 | `flowchart`（默认） | 扁菱形判断、"是/否"端口约定、直角回线、零交叉（正交规范） |
| 架构图 | `arch` | 逻辑容器图：一语义一视觉（外部虚线/边界实线/数据圆柱）、拓扑列对齐、三层文字、注释锚点、标签碰撞硬门槛 |
| 架构全景图 | `blueprint` | 能力地图（`layout grid` + `section` 多列流式）与分层架构（`layout stacked` + `band` 整行堆叠）语义分离；浅色分域、命名 rail 贯穿高度、数据存储卡片+小图标 |
| ER 图 | `er` | 实体表格块网格排布（上下左右扩展）+ 乌鸦脚基数记号、零交叉 |
| 甘特图 | `gantt` | nice ticks 时间轴、任务条、里程碑、section 分隔带 |
| 时序图 | `seq` | 生命线 + 消息分层错位，零交叉零溢出 |

排版由零依赖 Python 脚本确定性完成——跨 Claude Code / Codex、跨 Mac / Windows 效果一致，不依赖模型每次发挥。各类型契约见 `contract/*spec-v1.md`（架构图 `archspec-v5.md`、ER 图 `erspec-v2.md`、全景图 `blueprintspec-v4.md`）。

## 信任声明

- **零第三方依赖**：仅 Python 3.8+ 标准库
- **无网络请求**：任何阶段不访问网络
- **文件写入范围**：仅调用方指定的输出路径
- 脚本可审计（共享件 `scripts/flowcommon.py` + 统一入口 `scripts/flowlayout.py` + 四类模块 `scripts/layout_*.py`）

## 安装

本 skill 和 flow-walkthrough 放在同一个仓库 [UEmaster](../../README.md) 里，安装方法见仓库根目录的 README。

安装后自检（流程图 + 各类图的 golden 样例与降级断言，全部 PASS 才算可用；Windows 上把 `python3` 换成 `python`）：

```bash
python3 scripts/selftest.py
```

## 直接当 CLI 用（不经过 agent）

```bash
python3 scripts/flowlayout.py examples/pet-boarding.mmd -o out.html --html --title "宠物寄养预约流程"
python3 scripts/flowlayout.py examples/arch-3tier.mmd  --type arch  -o out.svg
python3 scripts/flowlayout.py examples/er-order.mmd    --type er    -o out.svg
python3 scripts/flowlayout.py examples/gantt-release.mmd --type gantt -o out.svg
python3 scripts/flowlayout.py examples/seq-login-auth.mmd --type seq -o out.svg
python3 scripts/flowlayout.py examples/gantt-release.mmd --type gantt --check   # 布局检查报告
```

## 适用形态与降级

每类有明确的适用形态（见 `SKILL.md` 与各契约文档）：例如流程图适用"单主干 + 左右分支链 + 直角回线"形态（可用 `--spine A,B,C` 强制主干、`--left D1` 分支链放左列）；时序图 v1 仅相邻列消息、不支持 note/alt/激活条。**不适用的形态引擎会明确报错**（「形态超限」），此时降级为 mermaid 原生渲染，不会产出带交叉/重叠/溢出的烂图。HTML 画布带缩放 / 拖拽 / 全屏。

## 契约

中间格式与产物契约见 `contract/` 目录（flowspec/1、archspec/5、erspec/2、ganttspec/1、seqspec/1、blueprintspec/4）。SVG 根元素带对应 `data-*spec` 版本号（arch 为 "5"、er 与 blueprint 为 "4"，其余 "1"）；flowchart 的可交互节点带 `data-node="{id}"` 与内置 `#sel-ring` 选中框（下游如 flow-walkthrough 只依赖契约，不依赖布局器实现）。

## 样式

默认灰度线框、产物禁 emoji（引擎硬校验）。用 `--style style.json` 覆盖任意样式变量（键见各模块 STYLE / 流程图 `DEFAULT_STYLE`）。

## License

MIT
