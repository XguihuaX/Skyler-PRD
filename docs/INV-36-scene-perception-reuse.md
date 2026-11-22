# INV-36 · 场景感知与主动陪伴复用调查

核对日期：2026-11-18。用户要求调查大肥鱼/DSH 与 MaiBot 的可复用能力；通过一个子任务并行调查 DSH，主任务核对 MaiBot 与衍生项目。仅上游文档与定向源码核对，未安装、启动或测量目标设备效果。

## 1. 与现有需求的关系

环境与内容感知已经是 PRD 一级能力 4，包括屏幕、应用、视频等来源；本轮补充其如何支撑 §5.4.5 的游戏/工作主动陪伴。链路可拆为采集线索、理解场景、判断是否接话、按角色表达。截图变化和“发生了值得评论的事件”分别处理，不把像素差异直接视为游戏胜负。

本轮候选研究不改变自主设计 Momo、DSH 插件加外置模块方向，不选择社区母项目，也不把候选已有功能直接扩为 Momo 必做项。

## 2. MaiBot 主仓：持续读屏与应用检测

源码快照：`Mai-with-u/MaiBot@59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1`。

- [官方桌宠文档](https://docs.maibot.chat/manual/desktop-pet)描述单次查看、持续阅读、应用打开条件、定期主动思考；桌宠与网页使用同一个主人会话。主程序内置桌宠和第三方 Live2D 桌宠插件是不同实现。
- [screen.py](https://github.com/Mai-with-u/MaiBot/blob/59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1/src/desktop_pet/screen.py#L38)：缩略图亮度差检测；ScreenTimeline 保留最多六个变化关键帧，批次完成保留末帧；拼图带捕获时间，提供前后变化。采样间隔与投递间隔可分别配置。
- [service.py](https://github.com/Mai-with-u/MaiBot/blob/59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1/src/desktop_pet/service.py#L468)：持续采样、远端设备侧筛选未变化帧、变化时间线投递；串行采样和投递避免排队积压，屏幕结果进入主人会话。520 行起是图文事件组装与投递接口。
- [屏幕阅读提示词](https://github.com/Mai-with-u/MaiBot/blob/59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1/prompts/zh-CN/instructions/desktop_pet/desktop_pet_screen_read.prompt)：持续模式结合此前交流，仅在值得关注的变化时参与，无需反复播报；截图带时间，不推断未观察过程。这是提示策略，实际效果本轮未测。
- [app_conditions.py](https://github.com/Mai-with-u/MaiBot/blob/59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1/src/desktop_pet/app_conditions.py#L10)：Windows 获取进程名/窗口标题，标准化后包含匹配。它识别应用打开，不能单凭此知道游戏胜负或工作是否完成。

**复用判断：**差分筛选、时间线、采集与投递分频、应用检测很贴合需求，可优先评估抽取；宿主会话投递要接 Momo/DSH 的实际接口。原地截图函数和应用枚举明确限制 Windows；其他设备需适配采集实现，不能把主仓远端客户端支持平台直接套到这两个函数上。

**代码许可核对：**[src/desktop_pet/LICENSE](https://github.com/Mai-with-u/MaiBot/blob/59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1/src/desktop_pet/LICENSE)为 MIT；其 service.py 仍引用其他目录，抽取时列清依赖，不把子目录许可扩大到整个仓库或外部素材。

## 3. Amaidesu：独立视觉模块与画面流

源码快照：`Mai-with-u/Amaidesu@fdb13867ccecc525fa3a4f7ab0034311b7a2f44d`，根 LICENSE 为 MIT。

- [视觉架构](https://github.com/Mai-with-u/Amaidesu/blob/fdb13867ccecc525fa3a4f7ab0034311b7a2f44d/docs/architecture/vision.md)：屏幕工具、多显示器/区域、图像短期缓存、最新帧槽位；调用方能直接看图时省略额外视觉模型转述。
- [look_at_screen.py](https://github.com/Mai-with-u/Amaidesu/blob/fdb13867ccecc525fa3a4f7ab0034311b7a2f44d/src/modules/vision/look_at_screen.py#L281)：输入区域、显示器、缩放参数；349 行起依据调用方视觉能力决定是否使用独立 VLM。适合作为 Momo 感知接口的参考，而非要求整个主播系统一并接入。
- [live_frame_collector.py](https://github.com/Mai-with-u/Amaidesu/blob/fdb13867ccecc525fa3a4f7ab0034311b7a2f44d/src/modules/collectors/frame/live_frame_collector.py#L294)：按配置从 OBS 或屏幕取图，每次成功采样覆盖最新帧；变化阈值影响基线刷新，未通过变化阈值也仍更新最新帧。决策侧按需取用，采集不逐帧触发一次回复。当前 `_grab` 失败路径仅重试/记录，未见自动切换到另一来源，故不沿用文档“自动回退”措辞。

**复用判断：**优先学习感知模块边界、图像时效、按需看图，以及已用视觉模型时省去重复转述。其内部 ToolSpec/图像仓/LLM 能力声明需适配 Momo，不是直接加载到 DSH 的现成插件。

## 4. 游戏事件路线：Minecraft 桥接样例

源码快照：`DHShikka/minecraft-maibot-bridge@292a0b2156fc3c03afdb379820aa8dc6261eeb4b`，根 LICENSE 为 MIT。

[协议](https://github.com/DHShikka/minecraft-maibot-bridge/blob/292a0b2156fc3c03afdb379820aa8dc6261eeb4b/docs/PROTOCOL.md)定义 Forge 1.20.1 客户端模组通过 WebSocket 上报状态和事件：受伤、死亡、击杀、升级、背包变化等。[bridge.py](https://github.com/DHShikka/minecraft-maibot-bridge/blob/292a0b2156fc3c03afdb379820aa8dc6261eeb4b/minecraft-bridge/mcai_bridge/bridge.py#L374)实际接收 state/event，更新状态及近期事件并调用处理函数。

**复用判断：**它证明存在不依赖截图猜测的结构化游戏信息入口，可借鉴“游戏适配器→场景事件”的契约。仅覆盖其目标游戏/版本，不推断通用胜负识别。项目还包含动作执行，本轮只研究感知部分，不把自动玩游戏纳入此次陪伴实现。

### 4.1 同服队友与临时接手（2026-11-20 补查）

用户希望两种玩法都能选。用《我的世界》Java 版核对谁在接收输入，以及用户和队友能否在同一世界各自操作。本次阅读了插件清单、工具实现及上游测试记录，Momo 尚未接入联机测试。

| 社区插件 | 可以复用什么 | 适用条件与验证情况 |
| --- | --- | --- |
| DSH `dsh-mc-client` 1.2.0 | 窗口工具能接手当前玩家。无窗口工具能用独立玩家名连接用户已在玩的服务器，移动、采集、读取自己的背包。 | 窗口接手使用 Windows 的 PCL2 客户端和真实键鼠。队友连接使用离线认证，需匹配游戏协议版本。与用户同服时，填写用户正在玩的服务器地址。导入单人存档会生成一份副本。作者记录了机器人加入已有的 26.1.2 测试服并退出。[窗口操作](https://github.com/DRheEheAMGary/dsh-mc-client/blob/69cdc3d9c5af144e398cf8b7b60e3cf9af08a72c/README.md#L168-L176) · [现有服务器连接实现](https://github.com/DRheEheAMGary/dsh-mc-client/blob/69cdc3d9c5af144e398cf8b7b60e3cf9af08a72c/lib/headless/service.js#L74-L89) · [上游验证](https://github.com/DRheEheAMGary/dsh-mc-client/blob/69cdc3d9c5af144e398cf8b7b60e3cf9af08a72c/docs/headless-verification.md#L159-L167) |
| 麦麦「Minecraft 陪玩」2.0.5 | 用独立玩家名入服，走到指定玩家身边，寻找并采集指定数量的方块。社区插件可以由麦麦直接调用。 | 清单声明 MaiBot 1.3.15–1.4.99、SDK 2.16.0–2.23.99，覆盖项目参考的 MaiBot v1.4.7 与 2.23.x SDK。主目标为 Java 26.2，另保留 1.20.1 后端。26.2 的上游验证包括本地协议测试，完整生存联机效果待测。[插件清单](https://github.com/Daniel-0702/maibot-minecraft-26.2/blob/102bb03845d51531de1eee3b537acd532a153cc9/_manifest.json#L17-L24) · [入服、找玩家、采集工具](https://github.com/Daniel-0702/maibot-minecraft-26.2/blob/102bb03845d51531de1eee3b537acd532a153cc9/plugin.py#L391-L445) · [26.2 验证范围](https://github.com/Daniel-0702/maibot-minecraft-26.2/blob/102bb03845d51531de1eee3b537acd532a153cc9/docs/NATIVE_26_2.md#L85-L96) |
| DSH `banbixieyang/dsh-mc-companion` 0.5.0 | 队友以普通玩家身份上线，可跟随用户、聊天，按指令伐木或采集指定数量的材料。 | 作者列出 Java 1.20.4、26.3，使用离线认证。适合优先验证“我继续盖房，她去采集”。[插件说明](https://github.com/banbixieyang/dsh-mc-companion/blob/4c6a0dce28a73cfd4f5b0002617825b241e831e7/README.md#L3-L7) · [跟随与采集工具](https://github.com/banbixieyang/dsh-mc-companion/blob/4c6a0dce28a73cfd4f5b0002617825b241e831e7/README.md#L235-L251) |

上节的麦麦 Forge 1.20.1 桥也是接手客户端的候选。麦麦官方将历史项目 MaiCraft 列为暂停维护，上表的「Minecraft 陪玩」有独立的仓库和插件清单。AIRI 的独立角色入服仍可作玩法参照，其现有 Mineflayer 实现计划迁移到 Fabric。[MaiBot 官方项目介绍](https://docs.maibot.chat/about/) · [AIRI 游戏集成说明](https://airi.moeru.ai/docs/en/docs/integrations/minecraft)

## 5. DSH 社区候选

### 5.1 MerZlin/dsh-pet-indesktop：最接近工作/游戏场景入口

快照 `34d810b1eadd98d00e7cb9b3b51428a45b5bf03e`，根代码 LICENSE 为 MIT，角色素材另有声明。

- [pet/proactive/__init__.py:126](https://github.com/MerZlin/dsh-pet-indesktop/blob/34d810b1eadd98d00e7cb9b3b51428a45b5bf03e/pet/proactive/__init__.py#L126)：依据前台进程/标题关键词分类工作、游戏等活动。
- [344 行起](https://github.com/MerZlin/dsh-pet-indesktop/blob/34d810b1eadd98d00e7cb9b3b51428a45b5bf03e/pet/proactive/__init__.py#L344)：白名单、停留、忙碌与频控检查后截图；488 行起处理迟到帧、dHash 差异和频控；560 行起将活动变化与短期记忆送给视觉模型并显示回复。
- [proactive_limiter.py](https://github.com/MerZlin/dsh-pet-indesktop/blob/34d810b1eadd98d00e7cb9b3b51428a45b5bf03e/pet/proactive_limiter.py)：冷却、日上限与失败限制。

**复用判断：**优先参考前台应用场景入口、短期上下文与发言调度。主动识屏目前是 Windows + Chat 版能力，Python/PySide6 外壳整体跨平台不代表该子功能跨平台。需接 Momo 的感知/会话接口；活动分类不等于游戏胜负检测。

### 5.2 dafeiyu-screensense：现成 DSH 插件骨架

仓库 `marx-otto/dafeiyu-dsh-plugins`，快照 `5f0d05669fac9b7ae46ead540e6cd89fd6e4bb76`，根代码 LICENSE 为 MIT。

[dafeiyu-screensense/index.js](https://github.com/marx-otto/dafeiyu-dsh-plugins/blob/5f0d05669fac9b7ae46ead540e6cd89fd6e4bb76/dafeiyu-screensense/index.js#L263)：Windows PowerShell 截主屏，8×8 均值哈希比较，保存 DSH 图片附件，297 行起通过 ctx.llm.stream 请求模型，367 行起调用 dsh-pet 气泡路由。提供带当前屏幕的 /fish 交流入口。

**复用判断：**适合学习最小插件注册、截图和模型调用链。耦合 ctx.llm、ctx.attachments、ctx.webServer 与 dsh-pet；Momo 需要将结果送统一记录和主动陪伴流程。截图写临时文件并存为附件，保留规则应按 Momo 定义。

**文档与实现差异：**[408 行起](https://github.com/marx-otto/dafeiyu-dsh-plugins/blob/5f0d05669fac9b7ae46ead540e6cd89fd6e4bb76/dafeiyu-screensense/index.js#L408)条件为 changed 或 idleDue，默认 idleRepeatSec=120，静止超时仍会请求；不能按 README 将其认定为静止时绝无模型调用。

### 5.3 mizuki5101/dsh-dafyfish-pet：感知到发言的分层

快照 `c22bf4bd7263ee0d9a08405296bcef3d56c52b99`，代码 MIT，许可证明确不覆盖模型素材。

- [capture.js](https://github.com/mizuki5101/dsh-dafyfish-pet/blob/c22bf4bd7263ee0d9a08405296bcef3d56c52b99/shell/src/heart/capture.js)：Electron 截图，遮掉自身和 UI。
- [index.js](https://github.com/mizuki5101/dsh-dafyfish-pet/blob/c22bf4bd7263ee0d9a08405296bcef3d56c52b99/shell/src/heart/index.js)：条件检查→截图→视觉→仲裁→动作，包含忙碌/睡眠跳过、重复检测与上下文。
- [decide.js:282](https://github.com/mizuki5101/dsh-dafyfish-pet/blob/c22bf4bd7263ee0d9a08405296bcef3d56c52b99/shell/src/heart/decide.js#L282)：优先级、发言间隔/小时上限、动作冷却。

**复用判断：**参考感知、表达与动作仲裁分层。Electron 壳加本地服务，可脱离 DSH，当前推荐 Windows；[prompt.js:19](https://github.com/mizuki5101/dsh-dafyfish-pet/blob/c22bf4bd7263ee0d9a08405296bcef3d56c52b99/shell/src/heart/prompt.js#L19)要求每轮生成台词，需调整为 Momo 允许安静的策略。仍未选择其作为母项目。

### 5.4 不混同的邻近能力

子任务核对 QCYTSN/dsh-dafeiyu 自述不读取屏幕，主要呈现 Agent 状态；dsh-speak-up 是闲置主动搭话机制，不提供场景感知。本轮三个相关项目均有通用屏幕/活动处理，但尚未定位跨游戏通用且可靠的胜负检测器；该结论只限定本次核对范围。


## 6. 早先的组件组合建议（已被插件优先顺序覆盖）

**建议，尚未选定：**以可替换感知模块承接屏幕、应用、语音和可用游戏事件；先复用采集/变化筛选/时间线机制，将观察结果送入 Momo 场景上下文，再由参与策略决定是否发言。具体采用哪个项目、哪些文件、插件与外置进程分工后定。

可取的优化：应用状态或游戏事件能直接回答的问题先用该来源；读屏按场景选窗口/区域；采样与模型分析分频，保留最新帧或短时间线；已有视觉主模型时可省一次视觉转述。游戏画面高频变化下，仅做像素差分仍可能频繁触发，需在所选游戏验证分析节奏和有效事件识别，不先承诺低成本或准确率。

后续验证应覆盖：工作进展、游戏结果、动态但无关键事件的画面、重复结果画面、短暂出现的结果提示；检查是否识别有依据、是否重复评论、主动发言时机、分析延迟和实际调用量。Mac/Windows 按目标运行设备验证。当前所有效果仍是待测项。


## 7. 用户纠偏：现成 DSH 插件优先，以及多渠道消息

**2026-11-18 最新决定：**用户认为无需预先从多个项目拆取、融合。先研究现成 DSH 插件，后续接起来看效果，再根据实际缺口二开。§2–5 的源码调查保留为参考，§6 的多来源抽取组合不再作为当前推荐实施路线；MaiBot 等作为候选插件不足时的参考。当前仍是需求阶段，未授权本轮安装、账户接入或采集。

用户新增目标：在场景陪伴同时感知直播评论、微信、QQ、邮件等消息，重要消息由 Momo 主动告知。重要性判断、提醒时机和渠道消息范围继续定义。

### 本轮现成插件初筛（上游文档级，未实测）

| 候选 | 核对到的现成能力 | 下一步研究重点 |
| --- | --- | --- |
| [STARDUSTLC666/dsh-email](https://github.com/STARDUSTLC666/dsh-email) | 多邮箱收信、搜索和新邮件弹窗；[emailTasks](https://github.com/STARDUSTLC666/dsh-email/blob/main/docs/OUTGOING-AUTOMATION.md#本地新邮件订阅emailtasks) 可订阅到达元数据，不必转成任务 | 先验证现成收信/通知。智能判断重要性并让 Momo 开口尚未证实已有，需要试用后确认差距。订阅只含账号、UID、发件人、主题、日期等，不含正文；需要读正文时使用实际邮件读取能力。文档标明通知为 best-effort，未保证无遗漏 |
| [rentaoyan0628-collab/dsh-bili-danmaku-lite](https://github.com/rentaoyan0628-collab/dsh-bili-danmaku-lite) | 原名 dsh-bili-live；实时弹幕、礼物/SC/进场事件，可选直播语音转写与自动陪聊；当前 README 文首还标有 0.5.1-vision.1 视觉改造版，含直播抽帧、变化筛选、时间拼图 | 优先核对该插件自身是否覆盖所需直播场景；README 后文保留旧版“不看画面”说明，应按确切安装版本与视觉文档验证，当前不能把源码分支能力等同 npm 已发布能力 |
| [AbcdefgXW/dsh-msg-hub](https://github.com/AbcdefgXW/dsh-msg-hub) | 已有微信、QQ、飞书渠道，可同时启用，消息进入 DSH 会话并支持向 IM 推送 | 确认能收到哪些会话消息、是否支持只收集并转为本地提醒；QQ 为开放平台机器人，微信为绑定通道，不能据此宣称能读取用户全部个人聊天 |

MaiBot 适配器文档也有 [QQ/微信接入](https://docs.maibot.chat/manual/adapters/) 和 [邮件](https://docs.maibot.chat/manual/adapters/email)，但本轮用户优先 DSH 现成插件。MaiBot 微信文档的范围为 ClawBot 私聊；邮件适配器面向专用机器人邮箱，不将这两者直接视为个人全部消息收件箱方案。

**后续插件验证顺序：**按具体能力选择一个合适候选，研究配置及兼容→接入验证目标体验→保留可用部分→只对实际缺口二开。不预设统一重写采集层或先组装多个项目；不同渠道按需要接各自插件。


### 接入概念纠正（2026-11-18）

用户指出邮件也可能通过 MCP 接入。此前“不同渠道各接插件”的写法过窄：来源（邮件/聊天/评论）与接入方式（MCP/DSH 插件/服务接口）应分开。§7 的 DSH 插件保留为候选，已有 MCP 能满足时直接复用；不增设插件包装作为前置条件。读取、主动发现新消息、重要性判断与提醒分别核对，具体 MCP 具有何种能力需看实际服务，当前没有核验或连接用户邮箱 MCP。


## 8. 现成插件能力清单（2026-11-18 续查）

用户最新要求：具体接入方式到时候再看，当前先看插件已经能做什么。本节只盘点现成功能，不指定融合、重写或二开方案。已有 MCP 可直接使用的决定保留。

| 插件 | 当前核对到的功能 | 当前需明确的范围 |
| --- | --- | --- |
| dafeiyu-screensense + 配套菜单 | 定时截图、变化检测、主动短评、结合当前屏幕聊天、手动查看与间隔设置；浏览器和桌宠气泡同步 | Windows 截屏，依赖 dsh-pet；文档只明确验证 dsh-pet 0.3.7。idleRepeatSec 允许静止画面超时再说，不采用其“静止零调用”的笼统宣传 |
| dsh-bili-danmaku-lite | 房间信息、实时弹幕/礼物/SC/进场、后台监听、可选语音转写、自动决定接话或沉默；当前源码另含直播抽帧、变化筛选、短时间线拼图；可离线分析弹幕高光候选 | 源码包 0.5.1-vision.1；视觉说明标题仍为 0.4.10-vision.1 且声明未完成实播验证。源码存在不等于目标设备可用或 npm 同版本已发布；其自动陪聊输出主要为直播间弹幕 |
| dsh-email | 多邮箱收发、搜索、附件、草稿、定时发送、新邮件弹窗；emailTasks 提供到达事件订阅，可使用发件人/主题等配置 | 已核对订阅实现，但当前未证实自带“理解重要性并通过 Momo 语音提醒”的完整功能。订阅元数据不含正文，通知为 best-effort |
| dsh-msg-hub | 微信/QQ/飞书渠道同时接入，文字/图片/语音处理，DSH 会话关联、任务状态推送与主动消息推送 | 本轮为 README 级核对；个人日常全部聊天覆盖未证实，QQ使用开放平台机器人。向 IM 推送与在 Momo 本地提醒用户是不同输出位置 |

补充证据：

- [screensense 仓库 README](https://github.com/marx-otto/dafeiyu-dsh-plugins)：两个配套包、功能、Windows/dsh-pet 依赖、idleRepeatSec 与菜单补丁机制；不是首次源码调查时遗漏的子目录 README（该路径本轮返回 404，使用根 README）。
- [直播当前快照](https://github.com/rentaoyan0628-collab/dsh-bili-danmaku-lite/tree/3aabf0208ec95b4de7ce9f2594f36cec8055d090)：package.json 为 0.5.1-vision.1；[视觉说明](https://github.com/rentaoyan0628-collab/dsh-bili-danmaku-lite/blob/3aabf0208ec95b4de7ce9f2594f36cec8055d090/docs/live-visual.md)说明图像与弹幕/转写共同进入模型，允许 SKIP；lib/visual.mjs 有帧元数据、过期与图像检查。说明中的作者本机安装/重启状态不代表 Momo 环境状态。
- [邮件订阅源码](https://github.com/STARDUSTLC666/dsh-email/blob/e4889029799c6d0b00a95636cc2c466433a79405/src/mail-consumers.ts)：EmailArrivalEvent 与消费者配置、扫描及进度管理已有实现。本轮未运行其测试或连接邮箱。

当前结论：有现成插件可以覆盖看屏评论、直播信息理解、邮件接入与 IM 通道等多项能力；先记录现成覆盖范围与验证状态。具体选哪一个、以插件还是 MCP 接入、是否需要二开，待后续讨论和试用。


2026-11-18 需求粒度纠正（按用户随后澄清修正）：用户指的是邮件到达作为事件消息，不单设邮件聊天。此前写成“不建立评论聊天”属于助手误读，撤回该归因。上面的“自动陪聊”等仅描述上游插件能力，不直接决定 Momo 的模块边界。具体接入仍后定。

<a id="手机摄像头陪伴2026-11-20首轮待定"></a>

## 手机摄像头陪伴

用户场景是出门时把手机镜头对准现场，在同一通话里问“这个呢”，听角色就当前画面作答。已核对上游发行记录、手机端说明和源码入口，未安装插件、连接手机或测量户外通话。

| 候选 | 可复用部分 | 接入 Momo 时要查明的差异 |
| --- | --- | --- |
| Tobs / DeepSeekBot v1.7.0 | DSH 插件配套 iPhone、Android 客户端。通话支持切换前后镜头、提取关键帧，将说话期间的画面与该句一起发送，并播放语音回复；已有配对与通话网络路径。移动 App 独立发布。 | 上游通话进入自己的 Bot 私聊。需验证能否让同一份画面与话语进入 Momo 会话，由麦麦的人格与记忆作答，并回传所选角色声音。原版将共享画面记入私聊。户外需在线宿主和可用通话网络，发行说明列出同一 Wi-Fi 或 Tailscale。 [发布说明](https://github.com/BotHarness/DeepSeekBot/blob/v1.7.0/CHANGELOG.md#L51-L91) · [发布与客户端说明](https://github.com/BotHarness/DeepSeekBot/blob/v1.7.0/README.md#L115) · [通话画面保存](https://tobs.app/en/privacy/) |
| 鱼眼 See on Phone v2.5.8 | Android 应用可独立访问模型，支持前后摄像头、连续取帧、自动挑帧拼图与朗读。配套 DSH 插件支持从手机按需取图。该版同时发布 APK 与插件包。 | 通话页的麦克风走系统单句识别，电脑插件取图与手机自己的聊天是两条路径。适合复用采集和按需看图；接麦麦会话及连续语音需单独处理。 [发布附件](https://github.com/SAOS-3624/see-on-phone/releases/tag/v2.5.8) · [通话与语音](https://github.com/SAOS-3624/see-on-phone/blob/v2.5.8/app/README.md) · [插件入口](https://github.com/SAOS-3624/see-on-phone/blob/v2.5.8/README.md#L140-L159) |
| 大肥鱼 dsh-dafeiyu-dot | 可复用沿用当前 DSH 会话的半双工语音交互、静音、打断、挂断。 | 当前公开功能与使用说明里未找到向该通话投递手机实时镜头的入口。[作者说明](https://github.com/wz20/dsh-dafeiyu-dot#readme) |

本项已纳入首轮，优先用 Tobs 自带通话复现[两块路牌场景](../.agents/skills/momo-refactor-cases/references/cases.md)，观察第二句能否引用新画面、语音是否自然接续。接入 Momo 时再按同一例区分手机链路问题和角色会话问题。真实手机和 Momo 后端的往返仍待验证。

## 按游戏核对陪伴与实际操作

2026-11-20 核对。Minecraft Java 的窗口接手与独立玩家进服都由 §4.1 的 DSH `dsh-mc-client` 提供。窗口操作使用 Windows/PCL2，独立玩家连接要求服务器支持离线认证并匹配协议版本。这套组合只需一个游戏插件。麦麦的游戏插件留作其他环境的候选。[插件依赖](https://github.com/DRheEheAMGary/dsh-mc-client/blob/69cdc3d9c5af144e398cf8b7b60e3cf9af08a72c/package.json) · [连接已有服务器](https://github.com/DRheEheAMGary/dsh-mc-client/blob/69cdc3d9c5af144e398cf8b7b60e3cf9af08a72c/README.md#L124-L134)

| 游戏 | 能交给角色做什么 | 接入条件与核对结果 |
| --- | --- | --- |
| 英雄联盟（端游） | 用户打比赛时，DSH 的 LOL 陪玩数据插件把当前对局的英雄、击杀/死亡/助攻、新事件和赛后结果送入原会话，角色可结合赛况说话。 | Windows。作者核对的是本机只读接口，插件输出为桌宠气泡。本轮查阅 DSH 社区仓库和麦麦插件目录，尚未找到能接手操作英雄或以独立队友打实际对局的插件。[插件说明](https://github.com/jinghuashuiyue2004/dsh-plugin-lol-companion/blob/c26c300bbad04736cf847dad285cdd3de165c729/README.md#L21-L46) · [对局数据项](https://github.com/jinghuashuiyue2004/dsh-plugin-lol-companion/blob/c26c300bbad04736cf847dad285cdd3de165c729/docs/USER_GUIDE.md#L37-L45) |
| 钢铁雄心 IV | 外部 MCP 服务 HOI4 Live Tools v1.4.2 能操作当前国家。作者在 Linux 的真实游戏中验证了开始科研、选择国策、新增建设、添加和调整生产线。 | 可由 DSH 的 MCP 客户端接入。Windows 发布包有游戏窗口输入，战役层面留有尝试记录。macOS 与 Linux 原生版为实验性，支持的游戏构建见版本表。上游尚未验收一场完整战役，本轮也未核实联机时独立控制盟国。[实机动作记录](https://github.com/klimPaskov/HOI4-Live-Tools/blob/v1.4.2/docs/LIVE_VERIFICATION_LINUX_1_19_4.md#L38-L57) · [平台与战役状态](https://github.com/klimPaskov/HOI4-Live-Tools/blob/v1.4.2/README.md#L35-L66) · [DSH 外部 MCP 入口](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.2-rc.4/packages/mcp/mcp-client/README.md) |
| 杀戮尖塔 2 | DSH 的 `dsh-sts2` 能在单人局替当前角色出牌、选路和拿奖励。开启带 AI 队友的多人局后，DSH 可为队友做决定，用户打自己的牌。 | 需要游戏端 AiTeammate 模组并开启外部操作。插件文档分别给出单人和多人入口。[两种玩法](https://github.com/YuWan886/dsh-sts2/blob/3a2954127ddf7e49cdbb24952c7b6d41b468ece6/README.md#L143-L157) · [游戏端模组](https://github.com/YuWan886/STS2-AiTeammate) |
| 星露谷物语 | DSH 插件「大肥鱼种白饭」配合 SMAPI 模组，能操作当前农夫播种、浇水、购物、与村民对话。用户按 F7 可以接回控制。 | 已核到当前农夫的操作与交还。独立农场队友未在这个插件上核实。[玩法](https://github.com/Darianliu23/dsh-stardew-blind-whale-farmer/blob/833ffc83d01e26336e3c61f0675116fc503d3dd7/README.md#L3-L10) · [接回控制](https://github.com/Darianliu23/dsh-stardew-blind-whale-farmer/blob/833ffc83d01e26336e3c61f0675116fc503d3dd7/README.md#L70-L75) |
| 泰拉瑞亚 | DSH 的 Terraria Agent 通过本机 tModLoader 模组操控当前角色移动、挖方块、放置材料和合成物品。 | 例如放置后会核对地块是否变化、背包数量是否减少。本轮核到的是对当前角色的操作。[模组与工具](https://github.com/Cogin0606/dsh-terraria-agent/blob/177b5ec982c04139415047c4c44a753112d8d6b5/README.md#L1-L5) · [挖掘、放置与合成](https://github.com/Cogin0606/dsh-terraria-agent/blob/177b5ec982c04139415047c4c44a753112d8d6b5/README.md#L290-L344) |

以上为插件作者的文档、仓库与测试记录。游戏陪看和游戏操作都在首轮内，开发侧负责可自动完成的连接、事件和动作回执检查。用户平时打什么就用什么验实战。《我的世界》留作可选的开发测试样例，作者已有无窗口进服与动作记录。[上游无窗口记录](https://github.com/DRheEheAMGary/dsh-mc-client/blob/69cdc3d9c5af144e398cf8b7b60e3cf9af08a72c/docs/headless-verification.md#结论)
