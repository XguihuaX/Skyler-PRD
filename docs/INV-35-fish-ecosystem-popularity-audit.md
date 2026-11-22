# INV-35 · 养大肥鱼生态热度复核与候选纠偏

核对日期：2026-11-18（北京时间）。只读调研，未安装、未实施。仓库指标直接读取 GitHub 仓库 API；数字是本次读取快照。最近 push 日期不等于发布、实机验收或持续维护承诺。

## 结论与撤回

此前围绕 Identity、AuraKaliya、Life、mizuki Pet 的讨论属于功能相关候选研究，没有先核对热度和生态覆盖。将它们作为热门代表，或优先建议 Identity/AuraKaliya 二开，证据不足，撤回该候选优先顺序。原源码分析可以保留，但不代表社区采用度或质量排名。

已找到数千 Star 的桌宠、呈现和社交项目，以及数百到上千 Star 的生活和记忆插件。本轮未核实到超过一万 Star 且同时覆盖 Momo 人格、记忆、生活、任务、桌宠的完整养鱼系统；这不是不存在的断言。DSH 宿主、移动客户端、模型接入等高 Star 项目不可充当完整养鱼系统的热度证据。

DSH 底座既有决定不变；具体主项目与桌宠实现重新开放比较。一个主要后端加生活机制和表现层仍可作为待评估方向，但不锁定此前四个仓库。

## 原始指标快照

| 仓库 | Star | Fork | 最近 push（UTC） | 已归档 | 仓库描述（发现线索，非验收） |
| --- | ---: | ---: | --- | --- | --- |
| [MeteorNOX/DeepSeek-Balance-Whale-Widget](https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget) | 7305 | 241 | 2026-10-11T12:25:55Z | 否 | DeepSeek Harness（DSH）一只住在 DSH 界面右下角的小鲸鱼娘，帮你盯着DeepSeek账户余额。QQ弹弹，支持拖拽吸附、左吸附翻转、数字滚动动画，随界面自动启用，建议直接喊来你的dsh安装 |
| [PC2005-cloud/dsh-pet](https://github.com/PC2005-cloud/dsh-pet) | 6448 | 570 | 2026-11-17T05:06:16Z | 否 | DSH 桌面宠物：一行命令装好即用的透明动画小桌宠，支持多开、大小位置随心配置；还内置 DIY 素材链，能用 AI 视频自造专属宠物 |
| [whiteguo233/OpenBiliClaw](https://github.com/whiteguo233/OpenBiliClaw) | 3777 | 205 | 2026-11-17T17:17:07Z | 否 | 本地私有、开源的自进化跨平台 AI 内容发现 Agent：先理解你，再主动从 B站、小红书、抖音、YouTube、X、知乎、Reddit、微博等平台与开放 Web 寻找内容。（支持 deepseek harness 插件） ／ Local-first open-source cross-platform AI content discovery agent: understands you, then proactively finds content across Bilibili, Xiaohongshu, Douyin, YouTube, X, Zhihu, Reddit, Weibo and the open web.（support deepseek harness plugin） |
| [Derpyu520/qq-bridge](https://github.com/Derpyu520/qq-bridge) | 3682 | 353 | 2026-11-18T04:12:53Z | 是 | Bridge between QQ (SnowLuma OneBot v11) and DeepSeek Harness agents: social simulation, safe MCP tools, slang learning and more. |
| [MerZlin/dsh-pet-indesktop](https://github.com/MerZlin/dsh-pet-indesktop) | 3303 | 236 | 2026-11-15T06:05:07Z | 否 | 基于项目PC2005-cloud/dsh-pet.git，将桌宠移植到Windows、Linux和MacOS上，现在可以随时看到蓝色大肥鱼了：），本项目还额外实现了其他交互功能，欢迎体验 |
| [Small-tailqwq/dsh-deep-whale](https://github.com/Small-tailqwq/dsh-deep-whale) | 3126 | 103 | 2026-11-15T12:42:34Z | 否 | Whale Girl skin series for DeepSeek Harness. 适用于 DeepSeek Harness 的，鲸鱼娘系列皮肤。 |
| [c1overzz/dsh-bili-scroll](https://github.com/c1overzz/dsh-bili-scroll) | 1437 | 132 | 2026-11-12T16:21:04Z | 否 | DSH 插件：每天替你刷一轮 B站，读字幕挑出值得的、写一份日报，值得的 @ 你 |
| [Alterssss/dsh-bili-whale](https://github.com/Alterssss/dsh-bili-whale) | 1071 | 92 | 2026-10-25T10:17:11Z | 否 | 给 DSH 装一条有日子的鱼：自己按作息刷 B 站、真看懂视频、写两本日记，把值得的 @ 你。 |
| [omdsh-dev/dsh-mnemon](https://github.com/omdsh-dev/dsh-mnemon) | 1037 | 73 | 2026-11-17T20:53:42Z | 否 | Composable, view-based context for DeepSeek Harness. Pluggable sources and strategies, with layered memory and self-evolving skills out of the box. |
| [whyever77/dafeiyu-astrbot](https://github.com/whyever77/dafeiyu-astrbot) | 896 | 68 | 2026-11-10T10:40:11Z | 否 | 🐋 大肥鱼 · AstrBot QQ 机器人全家桶：会看会听会说话、有记忆、会主动找你的鲸鱼娘（AstrBot + NapCat + DeepSeek +   阿里云百炼） |
| [aureth-labs/dsh-aurabot-qq](https://github.com/aureth-labs/dsh-aurabot-qq) | 681 | 129 | 2026-11-18T05:12:00Z | 否 | 让 DeepSeek Harness 里的 AI 以「大肥鱼」住进 QQ：会聊天、发表情、发语音、记事、发空间、刷 B 站、唱歌。功能都是模块，能开关、能自己写，还能从模块市场装更多（SnowLuma / OneBot v11）。 |
| [QCYTSN/dsh-dafeiyu](https://github.com/QCYTSN/dsh-dafeiyu) | 627 | 57 | 2026-10-29T07:19:09Z | 否 | Desktop-native BigFish companion for DeepSeek Harness — real Agent status, always on top on Windows. |
| [Yuuyuko-uu/dsh-fish-memory](https://github.com/Yuuyuko-uu/dsh-fish-memory) | 528 | 16 | 2026-11-17T08:39:56Z | 否 | 🐋 DSH 鱼的记忆：给会话一份会淡忘的长期记忆，每条都带时间。名字里的「鱼」来自 DeepSeek 的鲸鱼。 |
| [Yuuyuko-uu/dsh-feiyu-mood](https://github.com/Yuuyuko-uu/dsh-feiyu-mood) | 268 | 6 | 2026-11-17T06:40:07Z | 否 | 给 AI 伙伴一个会自己判断、你看得见的心情 —— 每轮聊完自动判断涨跌，右下角挂饰看数值、情绪和时间线 |
| [Yuuyuko-uu/dsh-speak-up](https://github.com/Yuuyuko-uu/dsh-speak-up) | 233 | 6 | 2026-11-17T06:06:38Z | 否 | DSH 插件：让她在没人跟她说话的时候主动搭话（本地掷骰子，掷中就走 DSH 自己的一轮） |
| [ssamurias/dsh-fish](https://github.com/ssamurias/dsh-fish) | 158 | 18 | 2026-11-17T19:32:02Z | 否 | DSH插件：养一个会自己刷b站会成长的agent：不通过面板配置，通过学习诞生人格。 |
| [Andersen216/dsh-whale-girl-live2d](https://github.com/Andersen216/dsh-whale-girl-live2d) | 95 | 11 | 2026-10-20T16:41:15Z | 否 | 🐋 鲸鱼娘桌宠 · Whale Girl Live2D —— DSH（DeepSeek Harness）Web 界面里的 Live2D 桌宠：跟着 agent 的真实状态换表情与动作，点一下就能直接跟 agent 说话，右键是钱包。A Live2D desktop pet for the DSH Web UI. 安装 Install: dsh plugin --profile web add github:Andersen216/dsh-whale-girl-live2d |
| [minedalang/dsh-dafeiyu-life](https://github.com/minedalang/dsh-dafeiyu-life) | 39 | 7 | 2026-11-18T04:09:45Z | 否 | 让大肥鱼在 DSH 中长期生活：在B站社交（多平台规划中）、生活，具有长期记忆、可塑造人格、自然好友关系、可视化桌宠（规划中） |
| [Anarrogantcat/dsh-identity](https://github.com/Anarrogantcat/dsh-identity) | 7 | 1 | 2026-11-07T12:24:39Z | 否 | DSH 身份系统插件（大肥鱼）：可编辑内核 + 证据制 traits + 记忆/日记/复盘 + 心情 v2 + 自主提议 + 治理与能力运行时。身份段与面板在任意 DSH 会话生效，不接 QQ 也能用；QQ 只是第一个通道，额外提供接话判断、主动开口与群治理。 |
| [mizuki5101/dsh-dafyfish-pet](https://github.com/mizuki5101/dsh-dafyfish-pet) | 3 | 0 | 2026-11-01T05:15:31Z | 否 | 大肥鱼桌宠：DSH Live2D 桌面宠物（DSH 插件 + 独立 Electron 外壳），可脱离 DSH 运行 |
| [AuraKaliya/dsh-dafeiyu](https://github.com/AuraKaliya/dsh-dafeiyu) | 0 | 0 | 2026-11-08T16:16:55Z | 否 | 大肥鱼：基于 DeepSeek Harness 的可成长智能体，统一记忆与成长、QQ、B站与自主活动；配置后独立启动。 |

## 关键身份与能力核对

1. **AuraKaliya/dsh-dafeiyu（0 Star）和 aureth-labs/dsh-aurabot-qq（681 Star）是不同项目。**后者自述提供人格、记忆、主动聊天、语音、B站及可替换模块与模块市场，应加入主要候选比较。最新抓到的正式版 v1.2.0（11-05），预发布 v1.3.0-beta.2（11-17）。[README](https://github.com/aureth-labs/dsh-aurabot-qq/blob/main/README.md) · [Releases](https://github.com/aureth-labs/dsh-aurabot-qq/releases)
2. **PC2005-cloud/dsh-pet（6448 Star）与 mizuki5101/dsh-dafyfish-pet（3 Star）是不同项目。**前者提供透明动画、角色素材包、桌面/网页/独立运行、聊天语音、待办及 DSH 状态联动，并有 HTTP、Cordis 调用接口。主要呈现由视频动画驱动，不应标成 Live2D。此前忽略这个上游候选不合理。[README](https://github.com/PC2005-cloud/dsh-pet/blob/main/README.md)
3. **MerZlin/dsh-pet-indesktop（3303 Star）**来自 PC2005-cloud 项目，现为 Python/PySide6 独立桌宠，包含角色切换及可选聊天，文档覆盖 Windows/macOS/Linux；是否适合 Momo 的 DSH 连接仍需检查，不凭名字认定为可直接加载的 DSH 插件。[README](https://github.com/MerZlin/dsh-pet-indesktop/blob/main/README.md)
4. **Derpyu520/qq-bridge（3682 Star）于 11-18 归档**，当前 README 明示不再提供新装支持和维护。可研究历史群友仿真等设计，不作为持续维护中的首选；当前许可与较早 MIT 版本不同。[归档说明](https://github.com/Derpyu520/qq-bridge/blob/main/README.md)
5. **MeteorNOX 余额挂件（7305 Star）和 Small-tailqwq 皮肤（3126 Star）**说明外观生态有明显热度，但各自功能范围不能被改写为完整陪伴系统。
6. **生活与探索**优先补查 c1overzz/dsh-bili-scroll（1437）和 Alterssss/dsh-bili-whale（1071）；Life（39）仍可学习其增量设计，不能代替上游生态比较。OpenBiliClaw（3777）为邻近的内容发现系统，需单列，不混为人格养鱼成品。
7. **记忆**补查 omdsh-dev/dsh-mnemon（1037）和 Yuuyuko-uu/dsh-fish-memory（528）。后者 README 描述按相关证据召回、有限条数注入、事实键替换及旧档案导入；当前 README 的自定义许可应按选定版本核对，不沿用旧 MIT 印象。[鱼的记忆](https://github.com/Yuuyuko-uu/dsh-fish-memory/blob/main/README.md)
8. **养大肥鱼不全部使用 DSH。**whyever77/dafeiyu-astrbot（896）采用 AstrBot；本轮保留为生态对照，未改变 Momo 的 DSH 决定。

## 调研范围与限制

使用 Web 搜索、社区目录发现仓库，再用 GitHub 仓库元数据核对。检索词包括“养大肥鱼”、大肥鱼、dafeiyu、dsh fish/whale/qq/memory/bili；GitHub 搜索按 Star 降序查看结果，并排除书单、素材集合、屏蔽脚本和其他误命中。部分公开搜索请求遇限流，未完成全量翻页，故不声称全网排名或完整覆盖。第三方目录的安装验证评级、下载数、搜索热度不作为本轮独立验证结论。

下一轮比较应按类别：人格与交流（Aurabot 等）、记忆（Mnemon/Fish Memory 等）、生活探索（Bili Whale/Scroll 等）、桌面呈现（PC2005/MerZlin/QCYTSN/Live2D 候选），再检查接口与共同运行。社区指标作为筛选证据，实际选型仍需核对接口、维护、依赖和 Momo 场景。当前不安装、不选定二开仓库。

## 桌面呈现续查（2026-11-18）

用户新增需求是窗口与仅角色两种前端形态，并希望学习大肥鱼。此处先对照已有候选 PC2005-cloud/dsh-pet 和 MerZlin/dsh-pet-indesktop，未将任一项目认定为用户唯一指定版本。本轮重新读取 main 的 README 与 dsh-pet 的 FEATURES；以下是上游文档描述，未启动软件或完成视觉、交互验收。

| 参考项目 | 本轮核对到的设计 | Momo 可借鉴点（建议） |
| --- | --- | --- |
| [PC2005-cloud/dsh-pet README](https://github.com/PC2005-cloud/dsh-pet/blob/main/README.md) | 浏览器覆盖层、桌面透明置顶小窗、独立模式；菜单可开启对话，角色随 DSH 事件显示状态 | 将角色呈现与详细会话入口分开；角色状态来自实际运行事件 |
| [dsh-pet FEATURES](https://github.com/PC2005-cloud/dsh-pet/blob/main/FEATURES.md) | DSH 侧边栏内有完整历史聊天窗口；聊天记录完整保存，传给模型的历史轮数另配；回复可单句重播，气泡配合语音播放结束收起；语音输入转文字填入输入框 | 聊天记录与上下文装配分别管理；研究语音重播、字幕与播放时序。其输入框确认路径不自动成为 Momo 的发送规则 |
| [MerZlin/dsh-pet-indesktop README](https://github.com/MerZlin/dsh-pet-indesktop/blob/main/README.md) | 透明无边框角色窗；独立聊天窗，有双栏工作台与经典窗口；聊天窗可选跟随角色移动；托盘可恢复显示或关闭鼠标穿透；Agent 事件驱动状态 | 窗口/角色分工、找回入口、位置与显示控制、实际状态反馈 |

具体差异：MerZlin 文档描述“看看屏幕”的回复只在聊天窗已打开时写入当前会话，未打开时仅气泡显示。因此 Momo 若采用“仅角色期间的语音也持续记入会话、开窗可回看”，需自行明确记录与视图分离，不能照搬该行为。两个参考项目以视频动画呈现角色；Momo 保留立绘与 Live2D 要求，渲染选型仍待后续讨论。以上参考未改变自主建设、后端优先和正式前端后置的安排。
