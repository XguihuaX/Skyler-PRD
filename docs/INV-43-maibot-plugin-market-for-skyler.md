# INV-43 麦麦插件市场与 Skyler 选型

2026-11-22，5070 Ti 主机。MaiBot 1.4.8、Dashboard 1.10.8、插件 SDK 2.23.0。本机 WebUI 的插件市场共显示 **478 款**，打开「仅显示兼容」后有 **424 款**。随表给每一款写了功能摘要，按用途归了 14 类，并为 20 款与 Skyler 特别相关或容易混淆的插件核对了作者说明。[可筛选的完整目录](research/maibot-plugin-market-2026-11-22.xlsx) · [官方市场快照](https://github.com/Mai-with-u/plugin-repo/tree/1007daae87fa580a9e6afcdc906785c5e2b6a7d7)

**11 月 22 日调研时，T-01 正在进行 Mac 同会话联测。** 主机原版聊天和重启回看已有记录。插件里最值得留给后续任务的是本地角色语音、获取资料更新和文档问答。三者都有现成实现，也都能用一条简短的测试验证能否接到主人私聊。[主机进度](product/SKYLER-HOST-PROGRESS.md) · [任务及验收条件](product/SKYLER-TASKS.md)

## 市场里能做什么

下表按主要用途归类。每款只计一次，例如社区 Live2D 桌宠在 WebUI 中属于「适配器」，在本表按用户会拿它做什么，计入「桌宠与界面」。工作簿同时保留两套分类，可从任一方向查找。

| 主要用途 | 全部 | 本机兼容筛选显示 | 例子 |
| --- | ---: | ---: | --- |
| 社群互动与运营 | 53 | 49 | 群日报、表情回应、跨群传话、QQ 空间及 B 站互动 |
| 游戏与娱乐 | 50 | 48 | Wordle、海龟汤、围棋复盘、绝地潜兵 2 配装 |
| 管理与运行统计 | 50 | 47 | 模型开销、错误日志、掉线通知、审批与群管理 |
| 图像、视频与创作 | 51 | 46 | 画图、角色自拍、读动图、从视频抽帧做摘要 |
| 搜索与资料 | 40 | 39 | 网页检索、B 站字幕、聊天附件问答、外部知识库 |
| 接话与人设 | 51 | 38 | 合并连续短句、判断要不要接话、切换人格和表达 |
| 语音与音乐 | 38 | 32 | 语音转写、角色音色、QQ 或 Discord 通话、点歌翻唱 |
| 订阅与提醒 | 35 | 30 | RSS 新文章、GitHub 新版本、开播推送、定时提醒 |
| 记忆与角色生活 | 33 | 28 | 回查原话、保存约定、角色作息、日记与阅读进度 |
| 消息渠道 | 26 | 20 | QQ、邮件、Telegram、iMessage、飞书、Matrix |
| 模型接入与调度 | 18 | 16 | 提供方适配、按群或时段选模型、调用限速 |
| 任务与系统工具 | 16 | 15 | DSH 委托、办公文件、桌面和设备工具 |
| 日常信息与服务 | 11 | 10 | 时区时间、F1 赛历、菜品推荐、机厅排队人数 |
| 桌宠与界面 | 6 | 6 | Live2D 窗口、Coopet、3D 房间和 WebUI 主题 |
| **合计** | **478** | **424** | |

在 WebUI 中，兼容结果里有 **202 款无类别**，其中 157 款未填类型，45 款使用旧值 `extension`。其余 222 款分在页面列出的 13 类，数量最多的是工具 53、媒体 30、外部集成 27。联网搜索、RSS 阅读器、麦麦×dsh 桥都能在「全部类型」或名称搜索里找到。工作簿可直接按「功能摘要」检索，也可叠加「本机兼容筛选」与「用途」。[WebUI 分类规则](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/dashboard/src/routes/plugins/types.ts#L6-L42)

## 与当前任务最相关的插件

### T-05 可以在主机上做语音识别和角色发声

- **说话转文字。** PRD 约定先比较自托管 faster-whisper 的 `medium`、`large-v3-turbo`。Speaches 能把本地推理包装成麦麦可调用的转写接口。市场还收录了 **SenseVoice STT 0.3.0**，它接收 QQ 语音消息段，交给独立的 SenseVoiceSmall 服务，再把文字送给模型。在 Mac 对着麦克风说话属于另一个入口，验收时还要核对原声能否和同一条转写对应。[现有语音方案](product/SKYLER-FUNCTION-REUSE.md) · [Speaches](https://github.com/speaches-ai/speaches) · [SenseVoice 的输入和模型服务](https://github.com/DMJXMJ/maibot-sensevoice-stt/blob/e7f8d397d321a8b789e5e978d1fcaa5b8136a7da/README.md#L12-L26)
- **文字转角色声音。** **GPT-SoVITS 语音插件（优化版）1.7.2** 可以连接 5070 Ti 上自建的推理服务，按音色配置已有权重、参考录音和对应文字。**麦麦多平台 TTS 2.2.6** 提供网页试听、音色收藏和引擎管理，也列有本地 BreezeTTS。等到 T-05，用同一句中英混合短句比较两种识别模型，用同一套角色音色在 Mac 上听回复，再测说到一半打断和继续。[GPT-SoVITS](https://github.com/Akenari011/gpt-sovits-voice-plugin/blob/96aa0d755edca5694393028f7be17f49645bd300/README.md#L29-L50) · [多平台 TTS](https://github.com/WhiteCloudOL/maimai-tts-plugin/blob/4d57190e0151e3511dc508a7abf8371f362441fb/README.md#L28-L48)

### T-07 能分别找到发现更新、读资料、交付文件的工具

| 需求 | 作者已经提供的功能 | Skyler 最小验证 |
| --- | --- | --- |
| 盯一个仓库的新版 | **MaiBot GitHub Subscriber 1.3.0** 可按仓库、分支和事件订阅，首次默认建立基线。作者文档中的通知目标是 QQ 群或 QQ 私聊。 | 用测试仓库发一个新版本，核对是否只收到一次通知，以及通知最终进入哪个会话。[作者说明](https://github.com/HK560/Maibot-GitHub-Subscriber/blob/b15f164f9bedb0668d2659387b21efbcccb1c562/README.md#L13-L32) |
| 持续跟踪一项资料 | **麦麦 RSS 阅读器 0.4.3** 可按聊天流订阅，把新条目交给麦麦思考，也能询问刚收到哪些更新。 | 用一条测试源发布新条目，核对麦麦能否在原私聊说明改了什么，并提供原文链接。[作者说明](https://github.com/yufei-pan/MaiBot-rss-reader/blob/009c98c09a0db41a8f0dadc11c7fcb83d27f7180/README.md#L28-L36) |
| 问附件里写了什么 | **文件读取 1.9.1** 面向 QQ 群及私聊附件，为每个会话单独建索引，用关键词和向量两种方式找相关段落。**B 站字幕提取解读 1.1.0** 可取字幕或交给模型通读，完整 AI 字幕要有对应登录态。 | 在两版测试资料中放入不同编号，问“第二版改了哪项”，核对文件名和原段。视频另测一条带字幕的链接。[文件读取](https://github.com/orge-8/file-reader/blob/b136eb4cc1abd59e2215a5ff36767aeb06c8bb5a/README.md#L3-L23) · [B 站字幕](https://github.com/OMSociety/maibot_plugin_bilicaption/blob/a8992153651f9dc643cce5d1b21bb5493422a2f5/README.md#L23-L32) |
| 交回一个能打开的成品 | **办公文件工具箱 1.2.7** 可以从 WebUI 或 QQ 处理办公文件，支持生成文档和表格。聊天通道发送不了附件时，作者说明会改发文件路径。 | 用一张小表生成新文件，由 Mac 从收到的消息打开，核对一个已知单元格。这个结果决定是否还需要项目里已有的跨端文件交付方案。[办公文件工具箱](https://github.com/qiaozhixiang/office-toolkit-dist/blob/b07b40d2c19d9dfb1d98c47be31a992949eff6b7/README.md#L3-L27) |

### T-04 三个名字相近的 DSH 桥，各自接到不同地方

| 入口 | 用途和验收重点 |
| --- | --- |
| PRD 已列的 **DSH Desktop 桥** | `SengokuCola/dsh-maibot-bridge` 属于 DSH Desktop 侧，通过 ACP 连接已经打开的桌面会话。T-04 延用这条路径，先做一次只读委托，核对聊天能继续、结果回到原会话、用户可以等待或拒绝。[桌面桥说明](https://github.com/SengokuCola/dsh-maibot-bridge/blob/v0.1.1/README.md#L3-L32) |
| 麦麦市场的 **麦麦×dsh 桥 0.6.0** | 将一次任务交给服务器上的 DSH，能把摘要写回麦麦上下文。文件上传用 NapCat；其他适配器可能只收到转发卡片。适合比较 QQ 入口的任务交付。[作者说明](https://github.com/ji-or-ji/maibot-dsh-bridge/blob/e9932fcd895fd62bcc16169bc55d7be36ce16a36/README.md#L19-L29) |
| 麦麦市场的 **DSH 远程桥接 1.6.4** | 在麦麦 WebUI 嵌入 DSH 控制台，能跟随会话输出并向 QQ 投递文件。需要从麦麦网页观察一个持续任务时可以试它。[作者说明](https://github.com/qiaozhixiang/dsh-remote-bridge/blob/7c2a48152988ca774bf9b77569128e94c359377a/README.md#L3-L19) |

## 还有哪些值得留到后面

- **私聊节奏与运行开销。** 想让 Skyler 在一句“收到”后自然收住，可以在原版基线之后对比 **JEV 接话判断**。它依赖兼容的决策服务或能返回 token 概率的模型。「礼貌性沉默」则是针对某个 QQ 用户按时长忽略消息，适用于骚扰处理。**开销统计**能按模型看用量和缓存命中，其一天窗口是滚动 24 小时，适合比较调整模型分工前后的消耗。[JEV 接话判断](https://github.com/Pawnnwap/maibot-jev-reply-gate/blob/0695801c92b5a09c312e894a41b808f5b5417fa5/README.md#L3-L14) · [礼貌性沉默](https://github.com/GYyxjsj/maibot-plugin_polite_silence/blob/ec4fbeba8cf9a84fd7ece7a7a2a0ab77129c2be1/README.md#L3-L13) · [开销统计](https://github.com/Akenari011/maibot-cost-report/blob/8148f413613ad4fb3f6d717accac2f8b2c5b8ba3/README.md#L46-L63)
- **T-06 多入口与视频。** 邮件适配器能把专用邮箱接进麦麦，按发件人建立私聊，回复沿邮件线程返回。「视频识眼」会从 QQ 收到的视频抽帧，送给视觉模型生成摘要。一个适合测试额外入口，一个适合测试已有视频。手机现场镜头是否连续更新、原声是否能逐条回听，仍按 T-06 用实机记录。[邮件适配器](https://github.com/yufei-pan/maibot-email-adapter/blob/186a470c9f663970e1e6189abe2125751a7f86ac/README.md#L1-L19) · [视频识眼](https://github.com/SaltinAnon/maibot-video-vision/blob/c479e1cc96a183ab9c3922008c25f97efe557443/README.md#L44-L56)
- **T-08 游戏与 T-09 桌宠。** 《绝地潜兵 2》已有配装卡插件，也有战况情报插件；配装卡可以离线渲染。它们能回答游戏资料问题。陪看、接手角色和独立队友分别需要游戏内证据。社区 **MaiBot Deskpet** 有 Windows Live2D 客户端和语音，插件使用 `deskpet-user`；后续试外观时应核对它能否接回原来的主人历史。[游戏配装](https://github.com/TouristH/Helldivers2DemocracyChooses/blob/3cae72d17eb49c994ce2d6cb64cc34ded81f353f/README.md#L20-L30) · [桌宠客户端](https://github.com/Maboroshinatsu/maibot-deskpet-plugin/blob/c48b609fa2ed53782218ca1a300d189e54d460a8/README.md#L11-L23) · [桌宠发送身份](https://github.com/Maboroshinatsu/maibot-deskpet-plugin/blob/c48b609fa2ed53782218ca1a300d189e54d460a8/plugin.py#L327-L331)
- **角色长期陪伴。** **MaiTrace** 会将作息和当前活动带入聊天，**麦麦绘卷** 可接 ComfyUI，结合日程生成角色自拍。**MaiWeave** 更侧重关系、约定和读书进度。等 T-02 原生记忆更正、提醒改期和取消各有记录后，再用同一段对话单独试一个插件，就能看出它实际增加了什么。[MaiTrace](https://github.com/Yae-Miko-Rabbit/MaiTrace/blob/d1ea75d822557378aa274c767a766dca56b5c976/README.md) · [麦麦绘卷](https://github.com/1021143806/mais_art_journal/blob/a812689a5f470ea96c3532c772cac79bb6a11500/README.md#L26-L33) · [MaiWeave](https://github.com/ilworkcn/MaiWeave/blob/f56b954edbb6090a50df27d58e5ab4d05c20a55c/README.md#L7-L14)

## 数字怎样核对

1. 在主机 WebUI 保持全部类型和空搜索词，分别读取开关两种状态下的卡片数。固定官方注册库提交 `1007daae87fa580a9e6afcdc906785c5e2b6a7d7`，核对 CNB 镜像和 GitHub 的插件详情、版本索引，两份文件分别逐字节相同。注册表有 480 条，分支详情 472 条，版本索引 479 条。用已安装的 MaiBot 版本在临时目录离线合并后得到 478 张卡片，兼容数和页面各分类计数也一致。[注册表](https://github.com/Mai-with-u/plugin-repo/blob/1007daae87fa580a9e6afcdc906785c5e2b6a7d7/plugins.json) · [版本索引](https://github.com/Mai-with-u/plugin-repo/blob/1007daae87fa580a9e6afcdc906785c5e2b6a7d7/plugin_versions.json) · [目录合并逻辑](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/plugin_market/service.py)
2. 424 款里有 285 款有推荐的稳定发行版，139 款通过分支清单筛选；另有 54 张卡片被筛出。宿主允许部分旧声明进入兼容模式，SDK 超出声明范围也可能只提示。就本次 424 款所用清单而言，135 款超出宿主声明范围，80 款超出 SDK 声明范围，其中 60 款两者重叠。旧调查的 283 是另一快照里按宿主上下界数出的条目，不能据此计算新增兼容数。工作簿保留两个声明范围供核对。[宿主校验](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/plugin_runtime/runner/manifest_validator.py#L1063-L1108) · [发行版选择](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/plugin_market/releases.py#L49-L75) · [上一版目录](INV-42-maibot-capabilities-and-momo-fit.md)
3. 20 款按作者说明核对输入、依赖或结果去向，另核了两款名称难以判断用途的插件源码。其余功能摘要依据市场清单整理。页面访问下载、评分等统计时显示超时，表内顺序不代表热度。本次记录的是市场与文档调查，插件运行效果留给各任务实测。[回声命令源码](https://github.com/DeepSeek-V4-Pro/semantic_loopback_engine/blob/621e644338d14c66e5869d42f7a2419f5799a99e/plugin.py#L58-L70) · [时间查询源码](https://github.com/DeepSeek-V4-Pro/chrono_temporal_sync/blob/7d5698965f482dd39d5a2a8c57769d871e04622f/plugin.py#L50-L65)
