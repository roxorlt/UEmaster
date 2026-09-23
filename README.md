# UEmaster

给产品评审用的两个 Agent Skill：**flow-canvas** 把业务流程（以及架构图、ER 图、甘特图、时序图）画成横平竖直的规范图；**flow-walkthrough** 把流程图和原型页面组装成三栏联动演示页，点流程图上的节点，右侧就显示对应的页面。

Two agent skills for product reviews: **flow-canvas** renders business flows (plus architecture, ER, Gantt and sequence diagrams) as clean orthogonal SVG; **flow-walkthrough** links a flowchart with prototype pages into a clickable three-pane walkthrough. Works with Claude Code and Codex. Pure Python standard library, no network access.

## 包含的 skill

| skill | 做什么 | 详细说明 |
| --- | --- | --- |
| `flow-canvas` | 把 mermaid 子集或自然语言描述排版成规范 SVG / 可缩放拖拽的 HTML；交叉、重叠、文字溢出全部为 0 才算合格 | [skills/flow-canvas](skills/flow-canvas/README.md) |
| `flow-walkthrough` | 左侧流程图可点击，中间设备框自动在手机框和后台宽窗口之间切换，右侧显示节点标注和页面状态切换；产出单个离线 HTML 文件 | [skills/flow-walkthrough](skills/flow-walkthrough/README.md) |

flow-walkthrough 的流程图由 flow-canvas 生成，两个 skill 需要一起安装。

## 安装

### macOS / Linux

```bash
git clone https://github.com/roxorlt/UEmaster.git
cd UEmaster

# Claude Code
mkdir -p ~/.claude/skills
ln -s "$(pwd)/skills/flow-canvas" ~/.claude/skills/flow-canvas
ln -s "$(pwd)/skills/flow-walkthrough" ~/.claude/skills/flow-walkthrough

# Codex
mkdir -p ~/.codex/skills
ln -s "$(pwd)/skills/flow-canvas" ~/.codex/skills/flow-canvas
ln -s "$(pwd)/skills/flow-walkthrough" ~/.codex/skills/flow-walkthrough
```

### Windows（PowerShell）

Windows 默认不能用 `ln -s`，改用目录联接（不需要管理员权限）：

```powershell
git clone https://github.com/roxorlt/UEmaster.git
cd UEmaster

# Claude Code（Codex 把下面的 .claude 换成 .codex）
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
New-Item -ItemType Junction -Path "$HOME\.claude\skills\flow-canvas" -Target "$PWD\skills\flow-canvas"
New-Item -ItemType Junction -Path "$HOME\.claude\skills\flow-walkthrough" -Target "$PWD\skills\flow-walkthrough"
```

Windows 上的 Python 命令通常是 `python` 或 `py -3`，不是 `python3`。

## 安装后自检

```bash
python3 skills/flow-canvas/scripts/selftest.py
python3 skills/flow-walkthrough/scripts/selftest.py
```

两条命令都输出「全部通过」才算可用。

## 信任声明

- 只用 Python 3.8 及以上版本自带的标准库，没有第三方依赖
- 任何阶段都不访问网络，没有遥测
- 只写入调用方指定的输出路径
- 产物是离线可用的单文件 SVG / HTML
- flow-walkthrough 会把页面片段原样放进产物 HTML，不做任何过滤（片段里可以有脚本），请只使用可信的页面文件

## 维护说明

个人项目，有空时维护。

## License

MIT
