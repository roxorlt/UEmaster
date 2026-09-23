# flow-walkthrough

三栏流程联动原型的 Agent Skill：左侧是 [flow-canvas](../flow-canvas/README.md) 产出的正交流程图画布（可点击、可缩放拖拽），点节点右侧同步展示对应原型页面，中间设备框自动在手机框 / 后台宽窗口间切换，右栏展示节点标注；页面有多个状态（弹窗 / toast / 只读态等）时右栏给出「页面状态切换」按钮，同一页面的状态之间可来回切，单状态页面不显示。产出单文件 HTML，离线可用，适合需求评审时"过流程"演示。

## 信任声明

- **零第三方依赖**：仅 Python 3.8+ 标准库
- **无网络请求**：任何阶段不访问网络
- **文件写入范围**：仅调用方指定的输出路径
- 产物为静态单文件 HTML，零运行时依赖

## 安装

本 skill 和 flow-canvas 放在同一个仓库 [UEmaster](../../README.md) 里，需要一起安装，方法见仓库根目录的 README。

安装后自检（Windows 上把 `python3` 换成 `python`）：

```bash
python3 scripts/selftest.py
```

## 直接当 CLI 用

```bash
python3 scripts/build.py examples/walkmap.json -o walkthrough.html
python3 scripts/build.py examples/walkmap.json --check
```

打开 `walkthrough.html` 即可看到内置示例：虚构的「宠物寄养预约」完整走查（登录 → 选门店预约 → 门店确认 → 支付定金 → 预约成功），包含两组多状态页面和两个后台宽窗口页面。

## 输入契约

见 [contract/walkmap-v1.md](contract/walkmap-v1.md)。上游依赖 flow-canvas 的 flowspec/1 SVG 契约；页面片段可使用模板内置 `wt-*` 组件类与 `goNode(id)` 联动 API；`state_groups` 定义页面状态组，驱动右栏的「页面状态切换」。

## 工作流（agent 执行时）

1. 拿到流程 SVG（没有则先调 flow-canvas）与页面材料（原型 HTML / PRD / 无）
2. 自动生成节点-页面关联，**先输出关联确认表给用户过目**（含匹配依据与置信度）
3. 确认后构建单文件走查页；构建器校验契约版本、映射完整性、无 emoji

## License

MIT
