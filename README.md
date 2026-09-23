# UEmaster

**让需求评审变成「点一下流程图，就看到对应页面」。**

UEmaster 是两个给 AI 编程助手（Claude Code、Codex）用的开源技能：一个把业务流程画成横平竖直的规范流程图，一个把流程图和原型页面连成可以点着走的演示页。

*Two open-source agent skills for product reviews: turn a business flow into a clean orthogonal flowchart, then link it with prototype pages into a clickable walkthrough — click a node, see its page.*

![三栏联动演示页：左边点流程节点，中间显示对应页面，右边是说明和页面状态切换](docs/images/walkthrough.png)

<sub>虚构示例「宠物寄养预约」：点左边的「预约表单页 · 约满弹窗」，中间手机框显示弹窗状态，右边可以一键切到这个页面的其他状态。</sub>

## 评审会上的老问题

- **流程图和原型对不上。** 流程图一张，原型一堆。讲到第六七个判断，就有人问：「等一下，这一步对应的是哪个页面？」大家开始在两份文件之间来回翻。
- **AI 画的流程图太乱。** 线交叉、框重叠、字跑出框，拿去评审，大家的注意力都花在看清楚图上。
- **AI 画的原型混着备注。** 页面里写满解释和待确认事项，评审的人分不清哪句是界面上的字，哪句是备注。

## UEmaster 怎么解决

| 技能 | 解决什么 | 怎么做到 |
| --- | --- | --- |
| **flow-canvas** | 流程图乱、难读 | 用排版脚本按固定规则摆放：主干一条竖线往下走，分支左右展开，回头的线都拐直角。画完自动检查，线交叉、框重叠、字出框三项都必须是 0 才交付。架构图、ER 图、甘特图、时序图用同一套标准。 |
| **flow-walkthrough** | 流程图和页面对不上 | 把流程图和原型页面做成三栏演示页：左边点节点，中间显示对应页面，后台页面会自动换成网页宽窗口；同一页面的几种状态（正常、弹窗、提示等）在右边一键切换。 |

![flow-canvas 画的流程图](docs/images/flowchart.png)

## 用了之后有什么不同

- **评审时大家看的是同一个画面。** 讲到哪一步，就点哪一步，页面和规则说明同时出现，不用在流程图和原型之间来回翻。
- **有原型就用原型，没原型也能讲。** 已有的原型页面可以直接接上；只有需求文档时，按内置的画页面规则画出灰度线框页面。
- **发给谁都能直接打开。** 最后得到一个离线网页文件，发给同事双击就能打开，不需要账号，也不用装任何软件。
- **规则写在技能里，换谁用都一样。** 这几条规则来自真实评审里提过的修改意见：流程图只画页面的各个状态、手机框里只放用户真的会看到的内容、动手之前先列出「节点—页面」对应关系表等你确认。

![flow-canvas 还能画架构图、ER 图、甘特图、时序图](docs/images/diagrams.png)

## 怎么用

装好之后，直接用平常说话的方式告诉 Claude Code 或 Codex：

- 「把这份需求的流程画成规范流程图」 → 调用 flow-canvas
- 「把这张流程图和这几个原型页面做成评审用的联动演示」 → 调用 flow-walkthrough

flow-walkthrough 会先列一张「节点—页面」对应关系表请你确认，确认后才生成演示页。

**适合**：需求评审、交互走查、给开发和测试讲流程。**不适合**：高保真视觉稿，或者需要接真实数据的可交互原型。

## 安装

两个技能放在同一个仓库里，需要一起安装（flow-walkthrough 用 flow-canvas 画的流程图）。

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

### 装好后自检

```bash
python3 skills/flow-canvas/scripts/selftest.py
python3 skills/flow-walkthrough/scripts/selftest.py
```

两条命令都显示「全部通过」就可以用了。每次提交都会在 Ubuntu、macOS、Windows 上自动运行这两组自检。

## 放心用

- 只用 Python 3.8 及以上版本自带的功能，不装第三方库
- 任何时候都不联网，没有使用统计
- 只写入你指定的输出位置
- flow-walkthrough 会把页面文件原样放进演示页，不做过滤（页面里可以有脚本），请只使用可信的页面文件

## 更多说明

- [flow-canvas 详细说明](skills/flow-canvas/README.md)：支持的图类型、输入写法、排版检查
- [flow-walkthrough 详细说明](skills/flow-walkthrough/README.md)：输入格式、页面状态切换、画页面规则

个人项目，有空时维护，欢迎提 issue 反馈。

## License

MIT
