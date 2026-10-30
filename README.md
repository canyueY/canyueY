<h1 align="center">各位大牛好 👋, 我是</h1>

<p align="center">
  <a href="https://github.com/canyueY">
    <img src="https://readme-typing-svg.demolab.com?font=IBM+Plex+Mono&weight=600&size=34&duration=2600&pause=1100&color=4F8EF7&center=true&vCenter=true&width=520&height=52&lines=Weijixuan" alt="Weijixuan" />
  </a>
</p>

<p align="center">
  <strong>Python 桌面应用 · Live2D 集成 · AI 角色交互 · DeepSeek Harness 插件 · 演化记忆</strong><br>
  <sub>把桌宠里能复用的部分拆成独立开源库，再做成可安装的 dsh 插件</sub>
</p>

---

## 🆕 最近更新

| 日期 | 更新 |
| --- | --- |
| 2026-10-31 | **Mnemosyne 2.0** 成为个人认知基础设施的权威 Node；`canyueY/mnemosyne` 公开；Memory Bank 拆为独立数据目录并发布说明仓 |
| 2026-10-31 | **肥鱼 Live2D 模型渲染插件** 发布：复用原 MurasamePet Lite 的 PyQt + Cubism + 文字框 + 大肥鱼模型，绑定 DSH |
| 2026-10-31 | `dsh-chat-memory-ingest`：微信/QQ 已导出聊天 → 记忆管线 |
| 2026-10-30 | Mnemosyne 经验闭环：图式演化、反馈学习、可选语义空间 |

---

## 📦 开源组件（Python）

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

## 🔌 DeepSeek Harness 插件

托管在组织 [DSH-Raising-Plan](https://github.com/DSH-Raising-Plan)。

对应上面这些组件的 **独立 npm 插件包**（分支 `plugin/dsh`），可直接：

```bash
dsh plugin --profile web add github:canyueY/<包名>#plugin/dsh
```

### 🧠 [Mnemosyne](https://github.com/canyueY/mnemosyne) · 个人认知基础设施

Mnemosyne 不是“多存一点事实”的记忆库，而是个人长期认知的**权威层**：

- **证据**：区分直接观察、当事人表达、外部来源与模型推断；
- **时间**：事件 / 记录 / 有效 / 确认四种时间；
- **状态**：偏好与关系会变化，旧状态不必被新状态覆盖；
- **情绪**：情绪是事件上下文，不是固定标签；
- **纠错**：新证据可以修正旧判断；
- **图式演化**：同化 / 顺应 / 冲突 / 平衡；
- **反馈学习**：用户明确反馈、实际结果、Agent 自评分开。

**2.0 权威仓：** [canyueY/mnemosyne](https://github.com/canyueY/mnemosyne)  
**数据层说明：** [canyueY/mnemosyne-memory-bank](https://github.com/canyueY/mnemosyne-memory-bank)（没有独立数据库）  
**DSH 旧壳：** [DSH-Raising-Plan/dsh-mnemosyne](https://github.com/DSH-Raising-Plan/dsh-mnemosyne)（Legacy，不作 2.0 标尺）

```bash
pip install -e ".[dev]"
mnemosyne doctor
mnemosyne start --data-dir "%LOCALAPPDATA%\Mnemosyne Memory Bank"
```

### 🖥️ [肥鱼 Live2D 模型渲染插件](https://github.com/canyueY/live2d-renderer-plugin) · 现有桌宠绑定 DSH

不重写渲染器，复用原 MurasamePet Lite 的 PyQt + Live2D Cubism + 文字框 + 大肥鱼基础模型；插件负责启动/停止、人物/文字显示模式、文字比例，以及副屏 DPI 修复。

```bash
dsh plugin --profile desktop add 'dsh-murasame-pet@github:canyueY/live2d-renderer-plugin#v0.1.0'
```

发布包：`MurasamePetLite-0.1.0-win64.zip`；控制面板在 DSH「插件 → dsh-murasame-pet」。

### 💬 [dsh-chat-memory-ingest](https://github.com/DSH-Raising-Plan/dsh-chat-memory-ingest) · 微信/QQ 聊天进记忆

把**已经导出**的微信 / QQ 聊天记录同步到 inbox，按主人/他人分流后交给记忆管线；不解密私有库，不内置导出工具。

### 🔌 DSH 插件总表

| 插件 | 对应能力 | 主要工具 |
| :--- | :--- | :--- |
| **[dsh-agent-gate](https://github.com/DSH-Raising-Plan/dsh-agent-gate)** | 权限闸门 | `gate_check` / `gate_check_path` / `gate_policy` |
| **[dsh-context-budget](https://github.com/DSH-Raising-Plan/dsh-context-budget)** | 上下文裁剪 | `trim_context` / `estimate_context_tokens` |
| **[dsh-mnemosyne](https://github.com/DSH-Raising-Plan/dsh-mnemosyne)** | 演化记忆（Legacy DSH 壳） | `memory_search` / `memory_context` / `memory_observe` / `memory_feedback` |
| **[dsh-window-locator](https://github.com/DSH-Raising-Plan/dsh-window-locator)** | 显示器 / 前台窗 | `list_monitors` / `foreground_window` / `desktop_snapshot` |
| **[dsh-netease-cdp](https://github.com/DSH-Raising-Plan/dsh-netease-cdp)** | 网易云控制 | `netease_probe` / `netease_now_playing` / `netease_play` / `next` / `prev` |
| **[dsh-video-storyboard-hp](https://github.com/DSH-Raising-Plan/dsh-video-storyboard-hp)** | 视频分镜高性能版 | `video_storyboard` / `video_zoom` / `video_probe` |
| **[dsh-chat-memory-ingest](https://github.com/DSH-Raising-Plan/dsh-chat-memory-ingest)** | 微信/QQ 导入 | `im_status` / `im_sync_*` / `im_preview` / `im_import` |
| **[live2d-renderer-plugin](https://github.com/canyueY/live2d-renderer-plugin)** | 桌面角色绑定 | `pet_start` / `pet_set_visibility` / `pet_set_scale` |

记忆权威入口是 `canyueY/mnemosyne`，不是旧 `dsh-mnemosyne` 插件壳。许可与运行边界见各仓库 README。

## 🚀 主要项目

### 🎭 MurasamePet（丛雨桌宠）

一个基于 **Live2D** 与 **GPT-SoVITS** 的 AI 桌面伴侣。PyQt5 承载无边框透明窗口与托盘，
Live2D Cubism 负责立绘、动作与表情，Python 侧用 FastAPI 提供对话 / 语音 / 识屏能力，
另有一套 **Vue 3 + Pinia + naive-ui** 的 Web 管理后台（70+ 路由）用于调试与配置。

覆盖的能力包括：多动作组与表情绑定、情绪驱动表情、点击部位互动、语音合成与口型同步、
屏幕感知主动发言、长期记忆，以及番剧 / 天气 / 美食 / 音乐等本地路由；
近期又补了 Minecraft / B 站视频观察层，以及对接本机 DeepSeek Harness 的能力面。

上面那些开源组件与 dsh 插件，就是从这个项目里陆续拆出来的。

> **来源说明**：本项目以 [kuxiaowo/AIpet-Murasame](https://github.com/kuxiaowo/AIpet-Murasame)（AGPL-3.0）为起点继续开发，
> 上游原始著作权归其作者所有；相对上游的主要修改见项目 `CHANGELOG.md`。
> 依据 AGPL-3.0，本项目同样以 AGPL-3.0 发布。

<sub>仓库暂为私有，整理完成后再考虑开源。</sub>

### ⛏️ [LittleMaidReengagedFirisPatch](https://github.com/canyueY/LittleMaidReengagedFirisPatch)

Minecraft 1.12.2 的女仆模组 Firis 补丁（fork）。

---

## 📓 开发日志

### 2026-10-31 · Mnemosyne 2.0 / Memory Bank 分界

- 权威仓改为 [`canyueY/mnemosyne`](https://github.com/canyueY/mnemosyne)；
- 代码仓与 `Mnemosyne Memory Bank` 数据目录分离，旧的 `%LOCALAPPDATA%\DSH\cognitive-memory` 迁出并归档；
- README / Matrix / Readiness 增加权威仓声明；`dsh-mnemosyne` 明确为 Legacy 壳。

### 2026-10-31 · 肥鱼 Live2D 模型渲染插件

- 复用 MurasamePet Lite 的现有渲染器，不再重写 Live2D；
- DSH 插件负责启动、显示模式与文字比例；
- 修复副屏缩放时模型被裁：PerMonitorV2 + 物理像素视口。

> 桌宠最近在做的事。仓库暂为私有，这里只写能落地验证的部分。

### 2026-10-29 · dsh-cognitive-memory 正式改名 Mnemosyne

项目从 `dsh-cognitive-memory` 更名为 **Mnemosyne / `dsh-mnemosyne`**：仓库、npm 包、DSH 插件 ID、Python 服务模块（`mnemosyne`）与文档全部同步；旧配置键、旧环境变量和旧数据目录保留迁移兼容。

同时把定位从“认知记忆插件”抬到“**DSH 演化记忆系统**”：不只记录事实，还要处理证据来源、有效时间、状态变化、情绪事件与后续纠错。组织仓与个人镜像均已同步。

### 2026-10-28 · 四个组件做成 DeepSeek Harness 插件

把 `agent-gate` / `context-budget` / `window-locator` / `netease-cdp` 各自做成独立的
`dsh-*` npm 仓库，默认分支 `plugin/dsh`，可按官方方式 `dsh plugin add`。
同时维护视频分镜 fork [`dsh-video-storyboard`](https://github.com/DSH-Raising-Plan/dsh-video-storyboard-hp)。
各仓库 GitHub 简介统一改为中文。

### 2026-10-22 · 她开始看得见你在玩什么、在看什么

这一轮的主线是**观察层**：让她知道主人此刻在游戏里做什么、在 B 站看什么，并且能顺着话头接下去。

- **P6 · Minecraft 观察**：客户端 Mod 经 `ws://127.0.0.1:28565/minecraft/ws` 把事件推给本地服务，
  服务侧维护实时状态、物品 registry、容器交互图、行为与 Episode 采样，再接进对话注入与主动搭话；
  管理后台多了「Minecraft 观察」面板。
- **P7 · 视频观察**：读 B 站页面判断「正在看什么」，维护观看会话 → 按关注度阈值写临时记忆 →
  推理前注入对话。最早靠 CDP 调试端口扫标签页，要手动开端口、权限弹窗，窗口标题还常对不上。
- **10-22 · 把 CDP 换成浏览器插件**：MV3 扩展 `extensions/murasame_bilibili`，content script 抓 BV、
  标题、播放器字幕、播放/暂停，每 4 秒上报 `/video/extension/heartbeat`，后台标签页也能报状态；
  插件数据新鲜时优先于 CDP（`prefer_over_cdp`）。另外加了 28566 端口的唤醒服务，插件安装或浏览器
  启动时请求 `/ensure-api`，由它把 `api.py` 拉起来。

**三处会「假死」的地方**

- `api.py` 是单进程单事件循环，async 端点里任何同步网络调用都会冻住所有接口（包括 `/health`），
  现在一律 `asyncio.to_thread`。其中一处是运维助手探测**本进程自己**的 `/health`，构成自调用死锁：
  实测 12.48s 且探测失败，改后 4.97s 且成功。
- `/video/status` 原本在 async 端点里内联跑完整 `observe_tick`（CDP + 截图 + 视觉推理），而
  `video_admin_snapshot()` 自己也调一次，等于一次请求跑两遍；改成后台线程 + 防抖。
- `SessionTracker.tick` 全程持锁做网络 I/O（最长 600s），改为锁内只取快照；临时记忆不再全量读文件，
  改为 `seek` 只读尾部 384KB。

**MC 侧两个真 bug**

- Mod 从来没构建成功过：`connectBlocking` 没有单参重载、Fabric 没有 `ScreenEvents.BEFORE_CLOSE`、
  javac 在中文 Windows 上按 GBK 解中文注释、NeoForge 没有 `reobfJar`。逐层修完才产出
  `murasame-mc-observer-0.1.0.jar`。
- registry 是 **100% 被拒**：Python 侧按字符数限 256K，Java 侧按条目数发 12000 条（≈984KB），
  超限后客户端置位就不再重试。改成 800 条一片、带 `uploadId` 合并上报，单帧上限放宽到 512KB。

**零散几处**

- 天气 / 美食新增 `weather_home_city`；IP 定位城市与目标不一致时，不再拿本地坐标去搜外地。
- 记忆编排接入 `minecraft_game` 层：游戏记忆可召回，回合结束后落盘。
- 任务队列加上限（32）并回收僵尸 `running`（`timeout + 180s`）。
- 运维助手能识别被安全软件掏空的 `bitsandbytes`，改走本地 wheel 离线修复。
- 工具沙箱从「整个桌面」收紧到 `data/agent/notes`——此前 `..` 被解析成项目根的上一级。
- 回归跑器：单个脚本卡死不再中断整轮，12 个从未被跑过的 live 验证脚本纳入 `--live`。
- 同批拆出 `netease-cdp` / `window-locator` / `context-budget` 三个零依赖包，加上 `agent-gate`，见上面的组件表。

<sub>对应 17 条变更记录（2026-09-20 → 10-15）；插件与唤醒服务是之后补的，还没归档进日志。</sub>

---

## 👤 关于我

我是 **胃疾癣**（也是canyueY）。主要做 Python 桌面应用，最近也在写 DeepSeek Harness 的 Node 插件。

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

## 📊 GitHub 统计

<p align="center">
  <img src="cards/stats.svg" alt="统计概览" width="860">
</p>

<p align="center">
  <img src="https://img.shields.io/github/stars/canyueY?label=Stars&color=4F8EF7&labelColor=0d1117" alt="Stars">
  <img src="https://img.shields.io/github/followers/canyueY?label=Followers&color=4F8EF7&labelColor=0d1117" alt="Followers">
  <img src="https://img.shields.io/badge/Python-JS-3776AB?logo=python&logoColor=white" alt="Python / JS">
  <img src="https://img.shields.io/github/last-commit/canyueY/mnemosyne?label=最近提交&color=4F8EF7&labelColor=0d1117" alt="最近提交">
</p>


## 🧰 技术栈

![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-20+-339933?logo=nodedotjs&logoColor=white)
![PyQt5](https://img.shields.io/badge/PyQt5-Desktop-41CD52?logo=qt&logoColor=white)
![Live2D](https://img.shields.io/badge/Live2D-Cubism-FF7BAC)
![Vue](https://img.shields.io/badge/Vue-3-4FC08D?logo=vuedotjs&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![DeepSeek](https://img.shields.io/badge/DeepSeek-Harness-4F8EF7)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?logo=powershell&logoColor=white)

## 📫 联系

- GitHub: [@canyueY](https://github.com/canyueY)
