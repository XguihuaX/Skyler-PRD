# Skyler 功能复用、二开与测试总表

2026-11-22，按 [Skyler PRD v0.96](PRD-momo-next.md) 的十个功能域逐项核对。麦麦作为 Skyler 后端，DSH 承接适合其插件的任务。旧 Momo 提供已有客户端实现、角色与声音资产。表中“直接用”表示有可采用的实现，仍需配置并按目标设备验收。“接入”表示两端已有能力，需要把输入、结果、身份或用户确认接通。“先试”表示有候选，回放发现具体缺口才二开。后续独立测试与产品扩展单列在文末。依据为麦麦 v1.4.8 及项目已核对的 DSH、社区插件版本。[麦麦功能及插件调查](../INV-42-maibot-capabilities-and-momo-fit.md) · [麦麦与 DSH 桌面桥](https://github.com/SengokuCola/dsh-maibot-bridge)

目标机器的验证步骤、通过条件和记录方式见[5070 Ti 装机验收与 Mac 联合开发](SKYLER-5070TI-ACCEPTANCE.md)。

## 哪些情况实际需要改代码

2026-11-22 对照麦麦发布版 v1.4.8（当时与主分支同一提交）、官方目录的 464 份插件详情、Tobs v1.7.0、DSH 桌面桥 v0.1.1，以及下表列出的社区插件和旧 Momo。查到现成实现的先配置。跨插件或跨设备的完整流程仍需联测。以下两条已有源码能指出差异所在。

| 选用路径 | 现有行为和需要补的最小部分 |
| --- | --- |
| **Tobs 电话进入麦麦原私聊，直接说的话能逐条回听** | Tobs 已有手机采音、取帧、通话和播放。v1.7.0 把每句话的转写和画面发到该 Tobs Bot 的私聊；麦克风声音用于转写后释放，用户通话录音默认不存。原版通话要达到 Skyler 的两项要求，须把当句送入麦麦主人会话，并在音频仍可取得时让所选采集端或 ASR 服务保留直接发言的原声、关联回该条消息。麦麦已有的原声播放器可以沿用。先试已有桥能否满足图文往返，音频原件需另有来源。[通话目标](https://github.com/BotHarness/DeepSeekBot/blob/v1.7.0/packages/core/src/voice/calls.ts#L1936-L1950) · [传给对话的内容](https://github.com/BotHarness/DeepSeekBot/blob/v1.7.0/packages/core/src/voice/calls.ts#L1243-L1259) · [录音行为](https://github.com/BotHarness/DeepSeekBot/blob/v1.7.0/CHANGELOG.md#L51-L53) |
| **设为“先问我”的动作经过麦麦 ACP 客户端** | DSH 本身已有桌面和手机审批。桌面桥接手所辖委托回合的审批时，向麦麦发送允许/拒绝一次两个选项。用麦麦 v1.4.8 原函数回放，两项都在时它直接选择允许一次。先用不改变外部状态的测试动作检查所选路由：原入口确实会等待用户就沿用；落到上述自动答复路径才改这一次转发，并验证同意、拒绝都生效。[桥的转发条件](https://github.com/SengokuCola/dsh-maibot-bridge/blob/v0.1.1/desktop-acp.js#L80-L89) · [麦麦的答复](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/agent/acp.py#L207-L216) · [现成手机审批](https://github.com/dsh-remote/plugin/blob/2003ea7fe5b521341599cd4aef3de6eb5394b2a7/README.md#L93-L104) |

另有两个已查明的**候选插件差异**。社区 Live2D 桌宠 v1.0.8 使用自己的 `deskpet/deskpet-user` 身份，选它时要验证与原版 `internal/owner` 对齐。DSH voice-mode 0.7.44 开唤醒词后每句回待机，短应答可以自动续读，硬打断会清掉剩余播报。若选择这条语音路径，须另测一次唤醒后的连续交流及“先回答插问，等我说继续再读”。其他客户端或插件组合仍可先试。[桌宠身份](https://github.com/Maboroshinatsu/maibot-deskpet-plugin/blob/v1.0.8/plugin.py#L327-L331) · [唤醒配置](https://github.com/qishuilalala/dsh-voice-mode/blob/3ce47143ba14bb3ce8e60c60d6ceed0288bd6763/plugin/dsh-voice-mode/README.md#L82-L115) · [硬打断的上游用例](https://github.com/qishuilalala/dsh-voice-mode/blob/3ce47143ba14bb3ce8e60c60d6ceed0288bd6763/plugin/dsh-voice-mode/scripts/smoke-voice-chain.mjs#L163-L172)

Skyler 自己的视觉设计、入口布局和角色外观留给正式前端阶段，由产品取舍决定工作量。以下各功能域保留首轮验收与后续方向。

## 1. 对话与理解

| 功能 | 复用与 Skyler 的工作 |
| --- | --- |
| 文字私聊、上下文与接话节奏 | **直接用麦麦。** 保留连续消息合并、指代与补充、等待、改口、行动规划和回复。用“先别移动文件，先给我看方案”检验执行前的连续更正。 |
| 理解用户是在讨论、询问能力还是委托办事 | **用麦麦私聊与工具选择，按场景回放。** 包括问答、共同梳理需求、只问缺失条件、切换或接续话题。一次回复、持续交流、已经执行的外部任务可分别停止。具体暂停和恢复对话的默认行为待定。 |
| 主动搭话 | **用麦麦的参与判断，Skyler 提供场景。** 当前进展、输赢、已说想安静或正在忙都可影响是否开口。接话频率、分寸要靠实际体验决定。 |
| 主人聊天历史 | **原版直接用。** 网页与官方桌宠（含可用的 Android 入口）有主人聊天历史。只对选择的额外客户端验证身份。多段会话的新建、搜索、归档、删除、跨会话引用及续聊方式待定。[主人身份与同步](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/desktop-pet.md#L7-L11) |

出处见 [PRD §5.11](SPEC-momo-next-details.md#511-n-01-对话与理解二级功能草案) 和[麦麦主人会话说明](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/internal-chat.md)。

## 2. 语音交流

| 功能 | 复用与 Skyler 的工作 |
| --- | --- |
| 普通话夹少量英语、语音记录及原声回听 | **先配现成服务。** 麦麦已有 OpenAI 兼容转写接口，Speaches 和旧 Momo 可提供 faster-whisper。验证 `medium` 与 `large-v3-turbo` 的混合词效果、短句延迟及资源，已有音频消息沿用麦麦转写/播放器。网页的聊天输入与设备实时录音分别验；Tobs 电话的原声差异见文首。[麦麦转写接口](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/llm_models/model_client/openai_client.py#L2085-L2105) · [Speaches](https://github.com/speaches-ai/speaches) |
| 独立唤醒、激活后连续多轮 | **唤醒按独立插件接，持续收听先复用。** 插件提供启停、配置、状态和唤醒事件。旧 Momo 已核实按钮激活和 Silero VAD 自动分段，未核实旧版关键词唤醒入口。激活后自动提交完整发言、继续等下一句。唤醒方式、空闲时长待定。 |
| 半双工、全双工、停顿、短应答、插话 | **先试语音插件。** DSH 的 voice-mode、voice-loop 与麦麦社区语音客户端都有部分能力，所选 ASR 和自有 TTS 要对接。直聊时用户开口先让话，游戏里与队友说话时结合对象判断。插问“谁发的”后说“继续”可接回原段，“不用读了”结束播报，新请求转去做新事。真实双工、续播位置和停止范围要实测。 |
| 特定场景免唤醒记录 | **配置或补场景控制。** 游戏、工作时可独立控制听、记和回应，没喊名字也能记录，角色有合适依据再开口。场景进入退出、可保存哪些环境音、逐句文本和摘要待定。 |
| 角色声音、TTS 库 | **先用已登记的麦麦插件与旧音色库。** GPT-SoVITS 优化插件可配置自训权重、参考音频、文字和风格，多平台 TTS 插件有 PC/手机配置、收藏和试听。两插件分别核实，声明与麦麦 v1.4.8 在 SDK 2.23.0 有交集。用一组已有权重实听，再决定是否补角色绑定、素材编辑或切换。旧 Momo 有 CRUD、标签和角色试听可沿用。句内情绪按引擎实测。[自训权重与多风格](https://github.com/Akenari011/gpt-sovits-voice-plugin/blob/96aa0d755edca5694393028f7be17f49645bd300/README.md#L33-L50) · [音色管理与试听](https://github.com/WhiteCloudOL/maimai-tts-plugin/blob/4d57190e0151e3511dc508a7abf8371f362441fb/README.md#L28-L36) |
| 默认设备、暂停 TTS、防回声 | **先用客户端已有机制。** 跟随系统默认输入输出。暂停 TTS 时文字和收音仍可用，停止当前播报沿用“打断”。验收外放声音不会被当成用户说话，真实插话能被接住。 |
| 多人语音 | **DSH 已有分人插件，后续独立测试。** `dsh-speech` 可流式收音、按句返回说话人标签。麦麦主人会话是否保留标签待联测。基础语音保持可独立运行。 |

出处见 [PRD §5.4](SPEC-momo-next-details.md#54-n-06-语音能力扩展新增方向)、[实时语音插件覆盖](../INV-33-legacy-capabilities-and-technology-refresh.md#14-2026-11-18-语音插件即时复核直接采用二开还是自做)及[语音与窗口接口草案](INTERFACE-001-frontend-backend-contract.md)。

## 3. 记忆与学习

| 功能 | 复用与 Skyler 的工作 |
| --- | --- |
| 短期连续、长期写入、按需回忆 | **直接用麦麦及完整 Prompt。** 包括原文、滚动摘要、未完事项、恢复时的时间间隔，人物/关系/事件/图片记忆，画像、按主题/人物/时间查找、预览和分段续读。随上游沿用身份与共享范围、有效期、衰减归档、来源管理、写回重试和记忆包导入导出。 |
| 记忆纠正 | **先复用麦麦现成的记忆管理页与入口。** 已有查询、编辑、删除及恢复。聊天反馈纠错有开关和查询命中条件，分别回放“刚引用后纠正”和“用户主动纠正”，再开一段聊天核对有效召回。仅在主动纠错无法生效时补这条路径；单独做简洁页面属正式 UI 取舍。[事实编辑](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/dashboard/src/components/memory/MemoryFactEditorDialog.tsx#L193-L217) · [删除界面](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/dashboard/src/components/memory/MemoryDeleteDialog.tsx#L248-L265) |
| 时间连续、短期例外与长期偏好 | **先用麦麦。** “去上海两天”按原先声明的时限理解。“这局让我静静”不覆盖长期偏好。用真实案例检查计划与实际经历、本人和转述的归属。只对回放中出现的缺口抽象旧 Momo 规则。 |
| 表达学习与办事习惯 | **麦麦直接学表达、黑话和明确偏好**，保留场景、来源、审核及开关。多步办事方法先试 DSH 社区的 `dsh-learn`，例如纠正一次“报价要算运费”，看下次相似任务能否用上。与外部知识学习分开验收。 |
| 上网学习并回原私聊反馈 | **已有方案与局部验证，待整链路联测。** 安排在 T-04 委托/回报可用后，与 T-07 资料和持续跟进联测。先组合现成的 Git/网页变更监控、内容读取、`dsh-agent-growth` 记录与去重，以及麦麦文字回传。用真实更新验证角色能带出处说明对 Skyler 的用处。按用户要求整理到知识或实验仓库。学习核心已有局部回放。尚无依据另立学习插件开发项。 |

出处见 [PRD §3.3–3.4](SPEC-momo-next-details.md#34-首轮记忆纠正与管理已确认)、[记忆与人格专项](../INV-40-maibot-memory-and-personality.md)、[办事习惯](../INV-32-skill-mcp-foundations.md#7-新版提醒办事习惯接管与权限)和[上网学习调查](../INV-41-online-learning-and-bilibili-replies.md)。

## 4. 环境与内容感知

| 功能 | 复用与 Skyler 的工作 |
| --- | --- |
| 跟随主屏、识别当前应用、工作或游戏陪看 | **直接用麦麦桌宠的主屏读取、手动启停和变化画面投递。** 取实际使用设备的屏幕。游戏事件插件能补充可信赛况。同步场景、时间与来源给当前私聊和原记忆流程，验证第二天还能聊起共同看到的翻盘。 |
| 手机摄像头陪伴 | **Tobs 的采集、取帧和语音回复可直接试。** 麦麦原主人会话与用户本人原声回听的具体差异见文首。验证两块路牌连续追问、同一角色声音、换设备续聊和每句用户原声。这项列在首轮。 |
| 邮件、微信、QQ、工具和服务事件 | **直接用实际可用的 DSH 插件、MCP 或服务接口。** 可订阅的接到达事件，只能查询的按约检查。收信、查正文、代表用户回复分别按账号授权。机器人和绑定渠道有各自可读范围。 |
| 外源消息筛选和择时提醒 | **先组合插件与设置。** `dsh-mail-digest` 有邮件分级、线程摘要、定时晨报和 Agent 读取报告工具。`dsh-proactive` 有静音时段，`dsh-voice-info` 有 DSH 任务过夜队列和早间合并。用邮件、游戏与睡眠场景验特别紧急例外、醒后回麦麦和来源回查。固定早报时间与用户实际醒来分别验；睡眠状态来源及紧急阈值待定。[邮件晨报工具](https://github.com/skyrocks/dsh-mail-digest/blob/b261a270455c670ca9de8b4010424011af1a184f/README.md#L63-L79) · [过夜队列的适用事件](https://github.com/flyingtimes/dsh-voice-info/blob/d85a4424e961db09d1bfaadff490d6a87b00ae46/README.md#L58-L84) · [静音与定时](https://github.com/john-walks-slow/dsh-proactive/blob/cae01f36b8aa6af2979ef138e9f8b2b62e6af0f9/README.md#L39-L68) |
| 环境声音、已有视频、平台评论 | **环境音类别识别留独立评估。** 用户确认的视频主场景是手机现场通话。已有视频文件/分享链接解析、手机手动拍照、B 站公开回评均有社区候选。设为各自的场景，公开回评还需指定账号和发布权限。直播与弹幕本轮先不展开。 |

出处见 [场景、镜头和游戏插件](../INV-36-scene-perception-reuse.md)、[PRD §5.8.2](SPEC-momo-next-details.md#582-外源事件消息筛选与择时提醒2026-11-18)及[B 站回评调查](../INV-41-online-learning-and-bilibili-replies.md#b-站评论区的大肥鱼怎么回复)。

## 5. 信息、文件与个人资料库

| 功能 | 复用与 Skyler 的工作 |
| --- | --- |
| 读网页、收发聊天附件、个人知识库 | **直接用麦麦。** 聊天附件归发起会话，长期材料显式导入，可查询和回查文件名、来源及可核对的原文。PDF/Office 原件能收发，提取正文要选读取器。Excel 等成果由相应工具生成。 |
| 交付文件、角色自拍和生成视频 | **已有生成与交付方式。** 麦麦有媒体插件和会话附件，DSH 有媒体与分享插件。`dsh-lan-share` 可把工作目录设为只读分享，URL 经现有文字桥回原聊天。本机用其 HTTP 处理分别下载真实 MP4 和 XLSX，均返回 200 且字节与原件一致。Mac/手机能否访问、外出网络、原聊天链接打开、必须以内嵌附件交付时的登记，以及角色生成质量分别联测。[DSH 只读分享](https://github.com/WODE25500/dsh-lan-share/blob/db3da5d4cd19784c0579f2b069178358c6317dc3/README.md#L13-L38) · [麦麦跨机文件传输](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/file-service.md#L22-L38) |
| 资料与已装插件有新版 | **按来源配置已有监控。** DSH 可检查部分已装插件的新版，`dsh-upstream-watch` 查指定 GitHub 分支，`dsh-daily-watcher` 查 RSS/网页变化，`dsh-folderwatch` 可发现本地文件修改。网页候选由作者标为开发预览。联测发现新版、回原聊天、用户暂缓则保留旧版、同意后才替换。只有一次上传而缺少来源时，须有新副本才能比较。[GitHub 变化](https://github.com/QuinnTang74/dsh-upstream-watch/blob/9d7d116d23bab6bc75f84f8d97a7a00cd7380227/README.md#L19-L46) · [RSS 与网页](https://github.com/zhouwei713/dsh-daily-kit/blob/ab3165acb3d0843ebc6bff32330aa5625f396efc/packages/watcher/README.md#L3-L19) · [目录修改](https://github.com/11111353535/dsh-folderwatch/blob/c11125d12738945d291b6067fb5fa03ea48c1c08/README.md#L3-L16) |

出处见 [PRD §5.9](SPEC-momo-next-details.md#59-长期资料操作模式与在线持续工作)、[资料和插件的版本检查](../INV-32-skill-mcp-foundations.md#7-新版提醒办事习惯接管与权限)和[麦麦与 DSH 的媒体交付差异](../INV-42-maibot-capabilities-and-momo-fit.md#2-麦麦收到任务以后怎么做)。

## 6. 任务与工作流

| 功能 | 复用与 Skyler 的工作 |
| --- | --- |
| 单次提醒、条件提醒、到点办事 | **直接用麦麦日程的增查改撤和条件待办。** “九点提醒我看邮件”直接提醒，“九点帮我查邮件”到点调用邮箱，“打开剪辑软件时提醒我”由应用条件触发。到点执行外部工具需联测。 |
| 周期、离线恢复、过时记录 | **用 DSH Schedule 做周期，用麦麦保留待发事项。** 麦麦单次提醒到期后会领取，所查版本的该入口只有到期时间。已明确结束的事项如何在恢复时静默记为过时、一次临时改期后下周是否照旧，需结合 DSH 到期条件和记录联测。核对已修改或取消的事项；未送达不能推断人是否赴约。[麦麦领取条件](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/services/schedules/store.py#L202-L226) |
| 按事件持续跟进 | **配置现成触发器，业务结果联测。** `dsh-sentinel` 已支持条件订阅、最大触发次数、截止时间、取消和跨重启恢复；`dsh-email` 有来信事件，正文可另读。测“对方说稍后发时继续等、收到最终报价才报一次并撤销检查”。持续工作以主机在线为基础，聊天窗口可以关闭。[DSH 条件与超时](https://github.com/fuhefei/dsh-sentinel/blob/05c28d8761fa9cfdfa61194ed86b5004fb2f9806/README.zh.md#L26-L54) |
| 普通后台任务与多步委托 | **用麦麦子代理做适合的查询和整理。** 后台工作时仍可聊，可查状态、停指定任务，结果回到原聊天。DSH 专用插件可通过已有桥接受整单委托。入队、某个工具进度、完成和回传分别核对。并非每个 DSH 工具都提供进度或取消。运行中追加要求并复用已做工作留下一轮。 |

出处见 [PRD §5.9](SPEC-momo-next-details.md#59-长期资料操作模式与在线持续工作)及[委托与状态回传](../INV-42-maibot-capabilities-and-momo-fit.md#2-麦麦收到任务以后怎么做)。

## 7. 网页、电脑与游戏操作

| 功能 | 复用与 Skyler 的工作 |
| --- | --- |
| 一次网页委托、扫码续办、上传与收尾 | **直接用 DSH Playwright MCP。** 让用户在当前页扫码，交还后重读页面、上传目标文件、核对真实回执。上传与提交按委托分别执行。“传完退出”默认关哪一层仍待定。 |
| 桌面持续操作、用户接手再交还 | **按操作设备选 DSH 驱动或插件。** Windows 有 Esc 停止输入，macOS 驱动有取消能力。从手机或 Mac 说“我来”，目标设备实际停手，交还后按最新状态继续，需跨端联测。 |
| 游戏临时接手用户角色、以自己角色当队友 | **两种玩法和陪看均列首轮，按游戏使用已有工具。** Minecraft 在 Windows/PCL2 和匹配的 Java 服务器、离线认证条件下可优先用一个 `dsh-mc-client`。麦麦插件作其他环境备选。《英雄联盟》已查到的是赛况陪看。《钢铁雄心 IV》有外部 MCP 操作当前国家。《杀戮尖塔 2》两种玩法均有条件支持，《星露谷》《泰拉瑞亚》查到当前角色操作。新游戏按所需动作再查插件或模组接口。 |

出处见 [PRD §5.9](SPEC-momo-next-details.md#59-长期资料操作模式与在线持续工作)及[各游戏的实际动作及条件](../INV-36-scene-perception-reuse.md#按游戏核对陪伴与实际操作)。

## 8. Skills、MCP 与扩展

| 功能 | 复用与 Skyler 的工作 |
| --- | --- |
| 聊天选工具、读取技能、执行并交付 | **麦麦有工具发现、MCP、市场和子代理，DSH 有按需读取技能及 Office 工作流。** 先测一份真实技能与 MCP，让它交付表格或图片。核对依赖、账号、启停、失败、取消和真实文件/结构化结果。WorkBuddy 是目标生态，具体样本兼容性待测。Codex Skills 不在本轮范围。 |
| 缺工具时主动找扩展、下载、更新 | **沿用麦麦市场提议、各宿主的管理和 DSH 权限。** 默认下载先问，预授权可在相应范围内直接下载。升级由用户决定。每个 DSH 插件继续在其宿主运行，已有 MCP 可直接接，无需另装一份麦麦插件。 |

出处见 [PRD §5.3](SPEC-momo-next-details.md#53-n-031-skills--mcp-接入与使用)和[宿主管理、依赖与兼容性调查](../INV-32-skill-mcp-foundations.md)。

## 9. 角色与交互呈现

| 功能 | 复用与 Skyler 的工作 |
| --- | --- |
| 单角色人设与完整 Prompt | **直接用麦麦。** 人格、身份、行为/表达风格、预设的导入编辑切换、规划与回复模板一并沿用。当前先保留稳定个性和可学习的表达。角色的具体背景、称呼、措辞待链路通后调整，后续实验见下表。 |
| 窗口、仅角色、立绘馆和 Live2D | **基础形态已有。** 原版桌宠能收起/展开聊天面板，网页、桌宠共用主人聊天；旧 Momo 有窗口/仅角色、Live2D 与立绘馆。社区 Live2D 桌宠采用单独的平台身份，选用时检验历史能否对齐。角色资产、音色、真实收听/播放/任务状态按所选客户端测试。Skyler 的正式视觉和入口按偏好后定，Live2D 与立绘馆保留，批次待定。[官方桌宠面板](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/desktop-pet.md#L156-L162) |

出处见 [单角色人格](SPEC-momo-next-details.md#51-n-071-单角色人设首轮范围已定)、[窗口与仅角色形态](SPEC-momo-next-details.md#551-窗口与仅角色两种形态2026-11-18)、[客户端平台对照](../INV-42-maibot-capabilities-and-momo-fit.md#3-自拍视频和独立角色窗口)。

## 10. 设置与用户控制

| 功能 | 复用与 Skyler 的工作 |
| --- | --- |
| 统一管理入口、账号与游戏插件 | **复用宿主和服务提供的管理、登录、重连。** Skyler 负责必要的简洁入口。角色 Prompt、声音和素材跟角色。模型/API、扩展、感知、语音启停及模式可管理。区分“已启用”和“实际就绪”。游戏专页先留入口。统一查看账号状态为建议，具体界面后定。 |
| 授权和单次确认 | **先选已有桌面或手机审批，联测实际请求走哪条路径。** 若请求经过麦麦自动选择 `allow_once` 的 ACP 回调，需要在该路径补用户决定，见文首回放。邮箱插件按自身规则处理可信收件人与抄送人。 |
| 数据和运行状态 | **复用麦麦管理后台。** 聊天、长期记忆、语音原件和感知记录分别按各自规则查看处理。历史与直接发言原声的保留期限、导出、重启后的展示和中间转写更正尚待定。感知用于当前交流或进入长期记忆的范围也待定。 |

出处见 [PRD §5.10](SPEC-momo-next-details.md#510-统一管理入口与角色-prompt-归属)、[部署与账号讨论](ARCH-001-runtime-renewal.md#模块边界与账号连接)及[桥接授权处理](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/agent/acp.py#L207-L216)。

**主机与 Mac 计划交接（2026-11-22 已确认先验证）。** 出差前主动将服务交给 Mac，返程带回新增数据并交回主机。先复用麦麦现有导出/恢复与换机机制验证基础往返；DSH 接入后补验其独立任务和插件状态。每次由一个运行端处理该角色的消息和提醒，模型位置与设备专属能力按实际环境核对。自动故障接替未选定，尚无双机交接实测结果。[需求 §5.12](SPEC-momo-next-details.md#512-主机与-mac-的计划交接2026-11-22-已确认验证方向) · [验收清单](SKYLER-5070TI-ACCEPTANCE.md#主机与-mac-计划交接验证)

## 后续独立测试与延期事项

下面的阶段不表示技术缺失。各项的现成来源、待测内容和接入窗口集中见 [PRD §2.2](PRD-momo-next.md#22-已有方案待测内容与接入时机)及[专项任务](SKYLER-TASKS.md#专项与扩展接入窗口)。本节保留版本和源码证据。

| 项目 | 已定安排与要回答的问题 |
| --- | --- |
| 多人语音识别 | **先试 DSH 的 `dsh-speech` 0.4.0。** 本地路径需配能返回声纹向量的识别服务，插件把各句归到会话内 A/B，返回文字、标签和时间。本次用模拟向量调用插件原代码，连续三句得到 0、1、0；真人分人效果、实时延迟、内存与显存未测。用同一段普通话夹少量英文的双人录音比较这条路径和保留 faster-whisper 的 WhisperX 路径，再验麦麦原聊天是否保留每句归属。已登记身份识别、重叠语音、是否回应他人仍另定。[DSH 本地识别服务要求](https://github.com/agent-mobile/dsh-speech/blob/008b06fa0e181a9de01e69af38988fe8c7733c9c/README.md#L179-L189) · [实时输出](https://github.com/agent-mobile/dsh-speech/blob/008b06fa0e181a9de01e69af38988fe8c7733c9c/README.md#L253-L267) · [WhisperX](https://github.com/m-bain/whisperX) |
| 环境音识别 | **已有模型，随 T-05/T-06 做专项测试。** SenseVoice 已提供部分声音事件分类，可作为现成对照。旧版已核到 Silero 人声检测和 faster-whisper 转写，用户提及的旧组件身份待定位。所需类别、标签能否透过插件传出、准确性和资源需测试。[SenseVoice](https://github.com/FunAudioLLM/SenseVoice) |
| 角色建模与驱动 | **已有工具，独立试做后在 T-09 接入成熟结果。** 图片分层、绑定、制作可运行的 Live2D 或 3D 模型，验证可控关节/图层、批量动作表情、口型、自然语言驱动表情、实时动画。Tripo/Astra/Blender、LumenAct、ADRY 等仅作候选。按成品质量、成本、运行占用、可修改性反定 Skyler 目标。 |
| 人格 Prompt 与推理前缀 | **链路好后做对照。** 先调具体角色文案。固定角色、输入和模型比较多套 Prompt，另测有无角色视角前缀，对闲聊接梗、安慰与操作求助的正式回复做盲评。记录自然度、任务结果、延迟、Token 与费用。 |
| Life、DailyAgent、多角色 | **已有基础，首轮后分项接入评估。** Life 可先试已调研的 `dsh-dafeiyu-life`；DailyAgent 有旧 Momo 的日常计划、运行时和生活线代码；多角色有麦麦多 Bot、DSH 多 Agent 和旧角色管理基础，完整共同会话插件尚未选定。Life 下一轮评估共同经历与关系表达。DailyAgent 的角色私人作息、用户不在场时的活动与跨天推进继续后置。多个角色同场并能彼此回应是后续目标，发言顺序、共享/私有记忆、关系与协作待定。用户自己的待办提醒、共同经历记忆仍在首轮。 |
| 后台任务执行中追加要求 | **保持首轮后验证。** DSH 的 `Agent.steer()` 已能向运行中的 Agent 补充输入。麦麦聊天中追加运费要求能否找到正确执行中的任务，并沿用已完成报价，留待联测。[DSH 运行中补充](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.2-rc.5/docs/subsystems/core.md#L113-L140) |

依据见 [多人语音安排](SPEC-momo-next-details.md#549-n-065-说话人区分的阶段安排2026-11-18)、[环境音与摄像头](SPEC-momo-next-details.md#581-场景感知复用调研2026-11-18)、[建模研究交接](../research/CHARACTER-LAB-HANDOFF.md)、[Prompt 对照](SPEC-momo-next-details.md#61-后续多-prompt-对照测试)及[多角色范围](SPEC-momo-next-details.md#57-n-073-后续多角色共同会话与交互)。

## 如何判断首轮已跑通

- **开发侧**用录制的语音/画面、两个版本的资料、测试网页、所选工具或测试服核对结果。记录成功、失败、取消、断线恢复和实际产物。检查闲置/执行时资源、上下文按需加载、Token、费用及稳定性，具体门槛未定。保持组件可替换，实施时记录配置、依赖、诊断、复现与旧数据迁移依据。
- **用户侧**在 5070 Ti 主机与实际 Mac/手机上体验内存、响应速度、音色与交互，在平时玩的游戏检验陪看、交还控制或队友效果。开发可用 Minecraft 测试服，用户无需专门玩 Minecraft。
- **范围未定的旧功能**按旧能力盘点逐项处理。例如助手卡/社交卡双入口、独立问候语音、剪贴板与长期活动采集、音乐播放、内容平台、旧日历适配与 ComfyUI 连接器。已有资产或代码可复核，不据此扩大新版范围或删除旧数据。
- 现有十一个[重构对话案例](../../.agents/skills/momo-refactor-cases/references/cases.md)覆盖上海行程、游戏临时偏好、夜间邮件、隔天聊翻盘、两块路牌、报价途中追加、扫码续传、游戏两种操作、代回邮件和学习仓库反馈。案例 06 用于后续增强，案例 10 在 T-04 后随 T-07 联测；案例 11 已确认先验证计划交接。更多实施验收按上表逐条补。

测试分工出自 [PRD §4.2](HISTORY-momo-prd-v0.94.md#42-已表达的质量方向与验收准备)。本表记录覆盖与安排，目标设备的完整链路仍待联测。已做过的局部回放可查[学习与评论插件](../INV-41-online-learning-and-bilibili-replies.md)和[视频抽帧](../INV-42-maibot-capabilities-and-momo-fit.md#3-自拍视频和独立角色窗口)。
