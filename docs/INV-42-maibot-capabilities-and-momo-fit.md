# 麦麦能做什么，Momo 应该从哪里开发

2026-11-21 核对 GitHub。最新正式版为 **MaiBot v1.4.8**，北京时间 11 月 20 日 16:07 发布。检查时 `main` 和发布标签指向同一提交 `edf42446f123149c57fc79765dd5a8b004b8524c`，随版 WebUI 为 1.10.8。本报告按产品能力盘点发行版，再核对外部插件和目标设备。示例里标为本机回放的部分有实测，其余按源码、配置和作者说明判断。[正式发布](https://github.com/Mai-with-u/MaiBot/releases/tag/v1.4.8) · [固定源码](https://github.com/Mai-with-u/MaiBot/tree/edf42446f123149c57fc79765dd5a8b004b8524c)

**首轮建议直接以麦麦为主程序开发 Momo。** 私聊、角色、记忆和主人聊天记录都由麦麦维护。Momo 可以保有自己的仓库、角色界面与语音代码，优先通过麦麦插件或客户端接口接入。自拍、视频生成、看主屏已有麦麦侧实现。DSH 用在选中的 DSH 插件、手机通话入口或外部任务上。这个判断是研究建议，最终选型和功能范围仍以双方确认的 PRD 为准。

## 1. 发行版功能盘点

| 能力 | v1.4.8 实际提供的行为 | 用在 Momo 上时需要知道的条件 |
|---|---|---|
| 私聊、群聊与主动接话 | 按会话收拢消息，处理补充、引用、等待、打断当前判断和结束话题。私聊与群聊用不同的参与机制。规划阶段决定做什么，回复阶段组织措辞。可选被动模式及跨会话 Focus。 | ASR 的一句转写可以进入同一主人私聊。聊天调度的等待时间与口语对话体验要在客户端一起测。[聊天场景](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/chat-scenes.md) · [规划循环](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/maisaka/reasoning_engine.py) |
| 人格与表达 | 人设预设可导入、切换和编辑。人格、行为与表达风格分别参与行动判断和正式回复；有群聊和私聊模板、回复模板组合。 | 首轮可整体复用提示词和调用时机。每个 Bot 有自己的配置及资料。[人设预设](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/personality-import.md) · [回复模板](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/replyer-prompt-profiles.md) |
| 表达学习、黑话、表情 | 能从聊天中积累表达、学习圈内词及其含义、按语境选表情；有学习启停、范围与管理入口。可按组决定是否共享表达、黑话或记忆。 | 能沿用角色成长机制。个性化办事习惯还要用真实任务检验。[学习模块](https://github.com/Mai-with-u/MaiBot/tree/v1.4.8/src/learners) · [表情系统](https://github.com/Mai-with-u/MaiBot/tree/v1.4.8/src/emoji_system) · [共享组](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/sharing-groups.md) |
| 近期上下文、长期记忆、人物和图片 | 近期消息压缩，长期写回和检索，人物画像、事件、关系与图片记忆可参与后续回复。可按需查记忆、画像或图片；长工具结果可分段续读。管理端有检查、纠错与回滚。 | 写入、自动召回和反馈纠错都有启用条件。可用“我改主意了”跨会话检验，单轮答对不足以证明长期记录已更新。[工具注册](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/maisaka/builtin_tool/__init__.py) · [记忆诊断](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/memory-diagnostics.md) · [人格记忆专项](INV-40-maibot-memory-and-personality.md) |
| 知识与资料 | A_Memorix 有知识导入、检索及维护，提供原文、OpenIE 和旧知识库导入路径。聊天文件另有按会话授权的服务，可上传、登记、下载、读文本、创建文本和发送附件。 | 文件服务默认关闭。原生能收发 PDF、Office 等二进制；正文解析须有对应读取器。个人资料的版本跟踪、更新提醒另行接入。[知识导入](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/A_memorix/IMPORT_GUIDE.md) · [聊天文件服务](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/file-service.md) |
| 搜索网页和研究 | 原生有网页搜索与读取网页正文的工具，可配置各自的服务提供方。也能用 MCP、查询子代理或桌宠探索扩展搜索和整理。 | 要先配置可用搜索/读取服务。探索记录可保留来源、经验和产物；定期检查资料版本、维护指定仓库仍需相应工具。[联网搜索](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/web_search/README.md) · [探索成果与复用](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/desktop-pet.md#L194-L225) |
| 日程、提醒、主动陪伴 | 可增、查、改、取消和完成日程；到期唤醒原聊天。桌宠支持按应用出现、画面或对话条件触发待办；可开启周期主动思考。 | 时间到了能触发判断，用户是否实际赴约仍取决于后来获得的事实。睡眠、重要程度和紧迫度如何影响 Momo 的播报，要结合事件信号验证。[日程工具](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/maisaka/builtin_tool/schedule.py) · [桌宠触发条件](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/desktop-pet.md#L180-L243) |
| 后台任务、子代理、外部 Agent | 自带子代理可承担有界查询、整理，支持后台执行、检查、停止和回报。独立外部 Agent 可连接 DSH；另有默认关闭的实验选项允许指定聊天使用外部行动规划。 | 查询子代理使用获准工具，由主聊天决定如何向用户说。桌宠探索和跨聊天任务各有开关与回报语义。详见下节。[子代理](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/maisaka/builtin_tool/subagent.py) · [外部连接](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/agent-connections.md) |
| 图片、表情、语音消息 | 能收图片、使用视觉模型、回复时引用消息或工具返回的图片。能转写收到的语音消息并合成语音回复，ASR、TTS 可由插件提供。 | 视觉、语音、嵌入等模型须按任务配置。整段音频转写和生成与终端实时收音、抢话、回声处理分开验证。[回复附图](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/maisaka/builtin_tool/reply.py#L208-L232) · [语音提供方格式](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/plugin-provider-formats.md) · [默认模型配置](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/config/default_model_config.py) |
| 网页聊天室与独立桌宠 | 网页聊天室与原版桌宠共用主人身份和一份历史，支持回看。桌面端可从 Windows、Mac、Linux 连接主机，另有 Android 悬浮窗客户端。原版皮肤支持 PNG/WebP、动画与事件。 | 数据存在 Bot 端，前端可显示同一历史。一个 Bot 同时接一个原版桌宠客户端。共用页面可发送文字和图片。Live2D 另有社区实现，详见下表。[主人会话](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/internal-chat.md) · [桌面客户端](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/pet-clients/desktop/README.md) · [Android 客户端](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/pet-clients/android/README.md) |
| 主屏和应用感知 | 桌宠可在授权后截取客户端主屏，也可持续阅读。默认每 2 秒采样，每 10 秒提交变化，每次最多拼 6 帧并带时间，由角色判断是否值得开口。可选桌宠探索与经验检索。 | 游戏中“看到了，接一句话”已有直接起点。桌面读取本身不操作键鼠。Android 原版未实现自动读屏和应用监测。远端桌宠看到的是运行客户端的电脑。[持续阅读实现说明](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/desktop-pet.md#L231-L243) · [Android 能力范围](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/pet-clients/android/README.md) |
| 接入聊天平台和外部消息 | 随包插件目录有 QQ 官方机器人、Telegram、微信 ClawBot 和中文错字插件。不同适配器处理不同的图片、语音、视频和文件消息。统一 QQ 连接器、邮件、QQ 语音通话、iMessage 等另有适配器或社区项目。 | 微信机器人能参与其接入的会话；个人设备的微信、系统通知及邮箱汇总需要各来源实际提供事件，再决定是否播报。Momo 的桌面主线可独立于 QQ。[随包目录](https://github.com/Mai-with-u/MaiBot/tree/v1.4.8/src/plugins/built_in) · [适配器文档](https://docs.maibot.chat/) · [QQ 通话适配器](https://docs.maibot.chat/manual/adapters/qq-voice-call) |
| 工具、插件与 MCP | 角色可以发现并调用延后加载的工具。插件可提供命令、模型工具、回复扩展、钩子、消息网关、模型服务和管理页面。MCP 可提供外部工具；有插件发现、安装、更新、配置和权限管理。 | 麦麦插件与 DSH 插件各自在对应宿主运行。已有麦麦工具可以直接接收具体任务，已有 DSH 工具可由外部 Agent 使用。[麦麦插件入口](https://docs.maibot.chat/develop/plugin/integration) · [MCP 实现](https://github.com/Mai-with-u/MaiBot/tree/v1.4.8/src/mcp_module) |
| 运维、配置与数据 | Host 可管理多个 Bot 实例及各自的工作进程、配置、插件和数据。WebUI 提供模型供应商和任务分配、提示词、人设、聊天与记忆管理、文件、日志、推理过程、统计、迁移等入口。 | 多 Bot 运行与多角色在同一场景自然协作是不同需求。Momo 可保留麦麦管理台，日常界面只露出常用操作。[多实例](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/multi-bot-host.md) · [WebUI 页面](https://github.com/Mai-with-u/MaiBot/tree/v1.4.8/dashboard/src/routes) · [聊天迁移](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/chat-migration.md) |

## 2. 麦麦收到任务以后怎么做

**麦麦能给 DSH 多大的任务。** 在“独立 Agent”模式下，麦麦可以一次交付整段目标、约束和聊天中的参考图。DSH 在自己的会话中决定步骤、使用该会话可调用的工具；采用 Desktop 桥时，可以复用 Desktop 会话里的模型、凭据和插件。普通麦麦聊天有传话和查询 Agent 状态的工具；单项业务进度仍由执行插件提供。例如 DSH 的 `dsh-media-gen` 注册了参考图生图、图生视频和查视频任务进度，具备接受“照着这张角色图出自拍，再做成短片”这一整项委托所需的工具。普通聊天每次传话收到的是入队回执，DSH 要把结果显式传回来源聊天。图片可随传话返回；MP4 需有对端能打开的链接或另走附件交付。麦麦无需为每个 DSH 工具再开发一个同名插件。这条跨系统媒体实例仅核对了两端实现，尚未完成端到端测试。[委托及状态工具](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/agent/provider.py#L15-L28) · [Desktop 桥](https://github.com/SengokuCola/dsh-maibot-bridge) · [DSH 媒体工具](https://github.com/EmberwingAviation/dsh-media-gen/blob/5a41152fee82060e50c8702192fe438581f9a810/README.md#L60-L70)

| 用户意图 | 首选路径 | 结果怎样回来 |
|---|---|---|
| “记得九点提醒我” | 麦麦日程记录时间和原会话。若是“我打开某个应用时提醒”，可用桌宠条件待办。 | 条件满足后唤醒角色，由角色决定怎么提醒。[日程](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/maisaka/builtin_tool/schedule.py) |
| “查资料，整理好告诉我” | 短查询可直接调用工具，长一些的可交给麦麦自带子代理；需要保留经验时可评估桌宠探索。 | 子代理结果回到主聊天。实验性的跨聊天投递另有规则，提交成功只说明已排入目标会话。[子代理](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/maisaka/builtin_tool/subagent.py) · [桌宠任务](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/desktop-pet.md#L154-L225) |
| “给我一张自拍”“把这张照片做成短片” | 优先调用装在麦麦的生成插件。生成器自己向模型提交请求，有些插件自己维护后台队列。 | 图片可直接发聊天；视频插件按配置回文件或链接。要验证选定客户端能打开成品。具体候选见下节。 |
| “用已经在 DSH 里配置好的插件办事” | 麦麦通过现有独立 Agent 连接把委托发给 DSH；在同机可走命令行连接，DSH Desktop 另有桥接插件。 | DSH 须显式传话给目标聊天，麦麦再向用户表达。`agent_talk` 接受文字与图片；视频可交付可访问的链接，或另经当前聊天文件服务登记。两台电脑不能直接共用一个本地路径。[传话字段](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/agent/provider.py#L15-L28) · [图片校验](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/agent/images.py) · [附件权限](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/file-service.md#L32-L38) |

**2026-11-22 补核授权路径。** DSH 有桌面与手机审批入口。特定 DSH Desktop 委托回合由桥接手审批、向麦麦提供允许和拒绝一次时，麦麦 v1.4.8 原函数的本地回放直接选了允许一次。需用户确认的操作首先在选定的实际路由验收；落在自动答复路径时补转发或选用确实等待用户的原有入口。邮件插件自身的确认仍按其设置。[麦麦答复](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/agent/acp.py#L207-L216) · [桥接手请求的条件](https://github.com/SengokuCola/dsh-maibot-bridge/blob/v0.1.1/desktop-acp.js#L80-L89) · [复核结论](product/SKYLER-FUNCTION-REUSE.md#哪些情况实际需要改代码)

## 3. 自拍、视频和独立角色窗口

| 具体场景 | 已经可以复用到哪一步 | 接入 Momo 时的边界 |
|---|---|---|
| **角色给我发一张自拍** | 麦麦社区插件“麦麦绘卷”4.6.9 提供角色参考图、自拍提示、聊天生图、定时自拍，支持本地 ComfyUI 或配置的在线模型。可让模型工具直接接这件事。 | “角色长得像不像、数次是否一致”取决于参考图、工作流与模型。须配生成后端，并在实际主人窗口看回图。[自拍配置与参考图](https://github.com/1021143806/mais_art_journal/blob/a812689a5f470ea96c3532c772cac79bb6a11500/README.md#L155-L175) |
| **用我的摄像头拍一张真实照片** | DSH 社区的 `dsh-camera` 已有手机扫码、调用系统相机、预览后手动发到指定 DSH 对话的实现。原版麦麦桌宠能接收静态图片。Windows 社区 MaiBot Deskpet 1.0.8 的指南也写有拍照，但该版实际快捷栏只有设置、聊天记录和麦克风，核对源码未找到摄像头采集入口。 | 麦麦侧的 Windows 拍照暂列待验证。手机拍完送到 DSH，与照片进入麦麦原主人会话，仍有一段对接。让角色按口头请求触发设备，也要验证设备授权与操作方式。[手机拍照插件](https://github.com/ilovecnxx/dsh-camera/blob/84ba12ea08b6d9c7f3efed7af38c8d054363dacc/README.md#L15-L24) · [桌宠指南](https://github.com/Maboroshinatsu/maibot-deskpet-plugin/blob/c48b609fa2ed53782218ca1a300d189e54d460a8/docs/GUIDE.md#L73-L83) · [同版实际快捷栏](https://github.com/Maboroshinatsu/maibot-deskpet-plugin/blob/c48b609fa2ed53782218ca1a300d189e54d460a8/deskpet-app/src/renderer/components/DeskpetStage.vue#L35-L83) |
| **把角色图做成短视频** | 麦麦社区视频生成插件 2.2.0 注册 `video_generate`，可对接文生视频、图生视频；文档列有首尾帧控制，实际能力按供应商和型号。后台任务完成后发送自定义文件或文字链接。 | 要配置视频生成服务，并验证文件可被客户端读取，特别是主机生成、Mac 或手机查看。桌宠探索卡也有 HTTP(S) 媒体预览入口，常规聊天对附件的显示方式需单测。[插件生成与交付代码](https://github.com/yueshenqiu/video_generator_plugin/blob/2d71e6308b07ee1412d27f81d4a0334becfadbd7/plugin.py#L199-L248) · [视频插件说明](https://github.com/yueshenqiu/video_generator_plugin) |
| **看我发的一个已有视频** | “视频识眼”1.0.1 会从 QQ 连接器取视频，用 FFmpeg 抽取若干帧，交视觉模型做摘要，再把摘要放回麦麦聊天。还有面向 B 站/抖音分享链接的社区实现。 | QQ 上按消息取媒体的代码不能直接取得 Mac 本地文件或手机相册。需接收文件、交给分析器，再把结果放回原聊天；音轨内容单独处理。抽帧部分已有本机回放。[视频识眼](https://github.com/SaltinAnon/maibot-video-vision/blob/c479e1cc96a183ab9c3922008c25f97efe557443/README.md#L24-L55) · [链接解析与点评](https://github.com/Ling-LA/ling-video-bot) |
| **出门边走边开镜头，与她聊眼前的东西** | DSH 侧 Tobs / DeepSeekBot 的移动通话已有前后镜头、语音、取关键帧随当句发送及播放回复。原版麦麦 Android 客户端用于桌宠和聊天，此标签的清单没有摄像头或录音权限。本次在麦麦侧未核实可直接完成这条手机链路的成品。 | Tobs 通话原本接自己的 Bot 私聊。Momo 要让同一轮画面和语音进入麦麦的主人会话，再把选定角色的声音送回手机。应在真实手机上连续换两次画面并追问。[原版 Android 权限清单](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/pet-clients/android/app/src/main/AndroidManifest.xml) · [Tobs 通话版本说明](https://github.com/BotHarness/DeepSeekBot/blob/v1.7.0/CHANGELOG.md) · [项目已有调查](INV-36-scene-perception-reuse.md#手机摄像头陪伴) |
| **会说话、有口型的独立角色窗口** | 社区 MaiBot Deskpet 1.0.8 已有 Windows x64 成品、Live2D、聊天、口型、按键说话、VAD 免按键和端云语音，支持 SenseVoice、GPT-SoVITS 等。原版桌宠另有可连接主机的 Mac 客户端。 | 社区插件注册自己的 `deskpet` 平台；它与原版网页和桌宠共享的主人身份要实际对齐。Windows 成品不能当作 Mac 或手机已验收。还需验证关键词唤醒、用户插话、原声回听和自己的微调音色。[社区客户端与发布](https://github.com/Maboroshinatsu/maibot-deskpet-plugin/blob/c48b609fa2ed53782218ca1a300d189e54d460a8/README.md) · [平台注册](https://github.com/Maboroshinatsu/maibot-deskpet-plugin/blob/c48b609fa2ed53782218ca1a300d189e54d460a8/plugin.py#L374-L382) |

**本机抽帧回放。** 用“视频识眼”上述提交的原始时长探测、采样规划与抽帧函数，输入一段自行生成的 12 秒动态测试片（640×360、24 fps、H.264、无音轨），FFmpeg 8.1.1，采样范围 2–5 帧、每 6 秒一帧。分别在 0.72 秒与 10.80 秒取得两张不同的 320×180 JPEG，由 ffprobe 核对。此次实际执行止于得到图帧；没有调用视觉模型、生图或视频供应商，没有从手机传入真实画面。[抽帧函数](https://github.com/SaltinAnon/maibot-video-vision/blob/c479e1cc96a183ab9c3922008c25f97efe557443/video_vision_core/ffmpeg.py#L288) · [采样规划](https://github.com/SaltinAnon/maibot-video-vision/blob/c479e1cc96a183ab9c3922008c25f97efe557443/video_vision_core/frames.py#L17)

## 4. 对 Momo 的开发取舍

| 直接用麦麦 | 在麦麦基础上补 Momo 的体验 | 继续用选中的外部能力 |
|---|---|---|
| 聊天与回复、人格、近期与长期记忆、主人聊天历史、日程、查询子代理、主屏持续阅读、工具与插件机制、管理后台。 | 两种窗口形态和自有角色资源；Mac/手机输入进入同一会话；原声随转写可回听、实时打断和防回声；事件结合游戏/睡眠时段决定怎样提醒；Mac/手机可打开生成结果。 | 麦麦侧已有的角色自拍与视频插件；DSH 侧已选的邮箱、游戏操作和上网学习插件；手机摄像头陪伴优先验证 Tobs。账号、模型或硬件依赖仍由各实现承担。 |

建议先以原版麦麦部署角色。原版 Mac 桌宠连 5070 Ti 主机，网页与角色窗口共用主人历史。Live2D 可分别比较已有 Windows 插件和旧 Momo 已实现的角色窗口、口型及角色库。旧版已有的录音、TTS、播放取消也应按实测复用。确认需要改哪一层后，再决定是写麦麦插件、改独立客户端，还是用已有 DSH 接口；新增邮箱这一类工具不会要求另建主人聊天或再维护角色记忆。[旧 Momo 功能与代码入口](PROJECT-MAP.md#3-功能定位表) · [现有接口约定](product/INTERFACE-001-frontend-backend-contract.md)

建议用三条用例验证选型。① 在 Mac 用语音发起自拍，能在同一段历史内回听原话、看到结果，刷新后仍能找到。② 让她把那张图生成短片，等待期间继续聊天，完成后从原会话打开。③ 从手机先后给她看两处路牌，连续追问，检查画面是否更新、她还能否使用同一段对话的记忆。开发侧可用测试文件验证附件和会话归属；内存、响应与真实游戏体验留给用户在目标设备验收。上述三个端到端场景均待实测。

## 5. 插件市场全量索引与统计口径

随报告提供[可筛选的插件目录 CSV](research/maibot-v1.4.8-plugin-catalog.csv)。固定官方市场提交 `a2e5e3a4f255ca733474a41b623f4621c949fd81`，逐条按市场标识对齐 467 条注册、464 份详情和屏蔽版本列表。每一行保留仓库、插件版本、类别、宿主与 SDK 范围，方便以后重新选型。

在这 464 份详情中，283 份的**宿主版本声明**覆盖 v1.4.8，其中 1 份所列版本在屏蔽名单内。其余 181 份声明的宿主范围未覆盖 v1.4.8。另有 3 条注册缺少详情。统计检查了当前详情的宿主上下界和屏蔽名单；SDK、依赖、模型、账号、目标平台及实际运行要按选定插件再验。CSV 中空白类别表示作者未填。正文详细核对发行版和与 Momo 有关的插件，全集索引用于继续筛选。[注册目录](https://github.com/Mai-with-u/plugin-repo/blob/a2e5e3a4f255ca733474a41b623f4621c949fd81/plugins.json) · [插件详情](https://github.com/Mai-with-u/plugin-repo/blob/a2e5e3a4f255ca733474a41b623f4621c949fd81/plugin_details.json) · [屏蔽版本](https://github.com/Mai-with-u/plugin-repo/blob/a2e5e3a4f255ca733474a41b623f4621c949fd81/blocked_release_versions.json)
