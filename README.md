<h1 align="center">Hi there 👋, I'm</h1>

<p align="center">
  <a href="https://github.com/canyueY">
    <img src="https://readme-typing-svg.demolab.com?font=IBM+Plex+Mono&weight=600&size=34&duration=2600&pause=1100&color=4F8EF7&center=true&vCenter=true&width=520&height=52&lines=canyueY" alt="canyueY" />
  </a>
</p>

<p align="center">
  <strong>Python 桌面应用 · Live2D 集成 · AI 角色交互</strong><br>
  <sub>把桌宠里能复用的部分，一个个拆成独立开源库</sub>
</p>

---

## 📦 开源组件

从桌宠项目里剥出来的通用件。每个都能单独用，不依赖桌宠本体。

| 组件 | 解决什么问题 | 许可 |
| :--- | :--- | :--- |
| **[agent-gate](https://github.com/canyueY/agent-gate)** | 给 LLM Agent 的工具调用装一道 fail-closed 权限闸门 | AGPL-3.0 |
| **[window-locator](https://github.com/canyueY/window-locator)** | Windows 显示器几何与前台窗口查询，全部用物理像素 | AGPL-3.0 |
| **[netease-cdp](https://github.com/canyueY/netease-cdp)** | 用 CDP 驱动网易云桌面客户端（不碰 API、不需登录） | AGPL-3.0 |
| **[context-budget](https://github.com/canyueY/context-budget)** | 把对话历史裁到 token 预算内，保留人设丢掉最旧的轮次 | AGPL-3.0 |
| **[pyqt-live2d-bridge](https://github.com/canyueY/pyqt-live2d-bridge)** | 在 PyQt5 里渲染 Live2D Cubism，只给引擎不给窗口部件 | MIT |

### 🛡️ [agent-gate](https://github.com/canyueY/agent-gate) · 零依赖 · 73 项测试

多数 Agent 框架的工具注册表里 `risk` 字段早就定义了，却**没有任何一处读它**——
工具一旦注册就会被无条件执行。对一个能读你屏幕、还要碰你文件的常驻进程来说，
"无条件执行"是最危险的默认值。

这个包把那道缺失的闸门补上，默认值全部保守：未知风险归为 `confirm`、
`allowed_roots` 为空即全部拒绝、无确认通道时 `confirm` 一律不执行、
被拒的调用不占配额。附带路径沙箱（拦 `..` 穿越与符号链接逃逸）与 JSONL 审计。

### 🖥️ [window-locator](https://github.com/canyueY/window-locator) · 零依赖 · 90 项测试

显示器几何改用 `GetMonitorInfoW` 的**原生物理矩形**，不再用
`QScreen.geometry()` 乘本屏 dpr——后者在混合 DPI 多屏环境下位置会偏，
而尺寸恰好是对的，所以特别难查。取窗口矩形时默认包一层 per-monitor-v2
DPI 上下文。

顺手修掉一个真实 bug：原来那段"临时切 DPI 感知"的代码写成
`try: return fn() finally: restore() except: return fn()`，回调抛异常时会被
**执行两次**（对截图就是失败时抓两次屏）。

### 🎵 [netease-cdp](https://github.com/canyueY/netease-cdp) · 1 个依赖 · 124 项测试

不碰 HTTP API、不需要登录、不读 cookie——驱动的是你**已经登录好**的那个客户端
窗口，所以 VIP 与私人推荐天然可用。打开调试端口后即可读当前曲目、点播、
播放每日推荐与私人雷达。

测试不需要真的开客户端：`tests/conftest.py` 里是一个真的 HTTP + WebSocket
假 CDP 服务器，所以 JSON 组包、`id` 匹配、事件插播跳过、超时都真的被执行到。

### 🧠 [context-budget](https://github.com/canyueY/context-budget) · 零依赖 · 52 项测试

对话历史只增不减，而上下文窗口有限。朴素的 `history[-20:]` 会把一问一答拆散，
让模型看到"助手回答了但用户没问过"。这个包按**轮**裁、按**预算**裁，并明确保证：
常驻 system 永不丢弃、临时时间播报只留最新一条、对话成对丢弃、最后一轮永远保留。

不绑定 tokenizer——它的职责是"决定丢哪些消息"，不是"精确算 token"；
需要精确时用 `counter=` 注入。

### 🌉 [pyqt-live2d-bridge](https://github.com/canyueY/pyqt-live2d-bridge) · MIT · 53 项自检

把 Live2D Cubism 在 PyQt5 里的渲染、动作与表情绑定，从桌宠主程序里剥出来。

做法很朴素：**只提供渲染引擎，不提供窗口部件**——宿主可以是任何
`QOpenGLWidget`，这样它能嵌进任意 Qt 应用，而不是绑死在某个项目上。模型路径
也不写死（显式参数 > 环境变量 > 约定布局兜底），动作目录 / 表情合成 / 情绪映射
这些纯数据部分不依赖 GL 上下文，可以单独使用。

这是五个包里唯一以 **MIT** 发布的：上游
[AIpet-Murasame](https://github.com/kuxiaowo/AIpet-Murasame) 用的是 2D 立绘，
不含任何 Live2D 相关代码，这部分属于完全原创。其余四个既然来自 AGPL 项目，
就继续沿用 AGPL，不做改许可。

---

## 🚀 主要项目

### 🎭 MurasamePet（丛雨桌宠）

一个基于 **Live2D** 与 **GPT-SoVITS** 的 AI 桌面伴侣。PyQt5 承载无边框透明窗口与托盘，
Live2D Cubism 负责立绘、动作与表情，Python 侧用 FastAPI 提供对话 / 语音 / 识屏能力，
另有一套 **Vue 3 + Pinia + naive-ui** 的 Web 管理后台（70+ 路由）用于调试与配置。

覆盖的能力包括：多动作组与表情绑定、情绪驱动表情、点击部位互动、语音合成与口型同步、
屏幕感知主动发言、长期记忆，以及番剧 / 天气 / 美食 / 音乐等本地路由。

上面那五个开源组件就是从这个项目里陆续拆出来的。

> **来源说明**：本项目以 [kuxiaowo/AIpet-Murasame](https://github.com/kuxiaowo/AIpet-Murasame)（AGPL-3.0）为起点继续开发，
> 上游原始著作权归其作者所有；相对上游的主要修改见项目 `CHANGELOG.md`。
> 依据 AGPL-3.0，本项目同样以 AGPL-3.0 发布。

<sub>仓库暂为私有，整理完成后再考虑开源。</sub>

### ⛏️ [LittleMaidReengagedFirisPatch](https://github.com/canyueY/LittleMaidReengagedFirisPatch)

Minecraft 1.12.2 的女仆模组补丁。

---

## 👤 关于我

我是 **canyueY**（胃疾癣）。主要做 Python 桌面应用。

这个桌宠的起点很普通：我在网上刷到上游项目
[AIpet-Murasame](https://github.com/kuxiaowo/AIpet-Murasame) 的演示视频，
觉得挺有意思，但用起来不少地方不合自己的习惯——于是干脆在它的基础上做衍生，
让它更贴合我自己的需求和使用方式。改着改着，就成了现在这个样子。

我的开发方式是**公开借助 AI 开发**：以 AI Agent 为主要实现工具，我负责定方向、拆问题、
判断交互手感、写测试验收，以及把不满意的地方打回去重做；具体的代码实现大量交给 Agent。
这样做的代价是要把验收标准讲清楚，好处是能把精力放在「到底要什么」上。

拆包这件事也是同一套打法：我把"哪些模块能独立出去"交给 Agent 做依赖分析，
自己判断该不该拆、许可能不能用、边界划在哪，再用测试把行为钉死——
所以每个包都带着一份能独立跑的测试。

## 🧰 技术栈

![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![PyQt5](https://img.shields.io/badge/PyQt5-Desktop-41CD52?logo=qt&logoColor=white)
![Live2D](https://img.shields.io/badge/Live2D-Cubism-FF7BAC)
![Vue](https://img.shields.io/badge/Vue-3-4FC08D?logo=vuedotjs&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?logo=powershell&logoColor=white)

## 📫 联系

- GitHub: [@canyueY](https://github.com/canyueY)
