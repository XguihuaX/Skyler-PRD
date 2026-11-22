# MaiBot v1.4.6 源码学习汇总

调查日期为 2026-11-19（北京时间）。面向正在定义 Momo 需求的读者。本文追踪官方发布版的收信、参与判断、行动、发信和后台写回，并对照当天主分支。证据来自源码、配置、发布说明和客户端文档。实际模型的表现、设备占用和端到端耗时仍待运行验证。

## 版本与本次更新

| 入口 | 核对结果 |
| --- | --- |
| 最新正式版 | **MaiBot v1.4.6**，2026-11-17 22:54 发布，标签提交 `d531b7eac5686f42bd63ea08fcb462a33c85981c`。随版 WebUI 为 1.10.6。[官方发布记录](https://github.com/Mai-with-u/MaiBot/releases/tag/v1.4.6) |
| 调查时的 main | `59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1`，比标签多两次提交，差异限于 CNB 镜像同步脚本及测试、桌宠发布工作流。本文使用发布标签对应的固定源码链接。[官方差异](https://github.com/Mai-with-u/MaiBot/compare/v1.4.6...59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1) |
| 这次具体改了什么 | 聊天侧调整主循环提示与工具调用、消息等待时间和群聊查看频率，让快速模型承担更多任务，增加按 Token 限制聊天上下文的选项。桌宠新增历史浏览、未读指示与提示音，更新日程唤醒及远端连接器。管理侧改进提示词对照编辑与插件更新。以下各节介绍整版机制，以上才是本次发布的变化。[发布说明](https://github.com/Mai-with-u/MaiBot/releases/tag/v1.4.6) |

## 从收到消息到发出回复

以主人私聊为例。用户先发“明天要汇报”，紧接着补一句“用上次那版提纲，九点提醒我”。这个例子用于说明代码能走哪些路径。真正会选哪个工具取决于模型、会话权限和配置。

| 步骤 | 麦麦怎样处理 | 源码入口 |
| --- | --- | --- |
| 1. 接收并记录 | 平台驱动把消息交给接入层。存在稳定的上游消息标识时可以去重。普通消息经预处理、命令检查后进入会话，写聊天记录，允许回复时才交给对应会话的运行实例。暂停发言的聊天仍能保存消息。 | [平台收发](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/platform_io/manager.py#L461)、[接收入口](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/chat/message_receive/bot.py#L878)、[落库与会话注册](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/chat/heart_flow/heartflow_message_processor.py) |
| 2. 等用户说完 | 同一人短时间续发时，按当前档位调整截止时间，并保留整批上限。示例里的两句可以进入同一次判断。群聊和私聊各自配置，普通私聊在这一批准备好后直接进入查看。 | [等待规则和示例](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/chat/debounce/README.md)、[私聊调度](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/scenes/private/scheduling.py) |
| 3. 决定做什么 | Planner 读取当前会话上下文及可用工具。它可以查旧事、处理日程、选择回复、结束本轮，或等待更多信息。等待可被新消息唤醒。Planner 请求还有处理中途来信的逻辑。 | [行动循环](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/reasoning_engine.py#L404)、[等待工具](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/builtin_tool/wait.py) |
| 4. 完成动作 | “上次”可以触发长期记忆查询。如果记忆只记得讨论过提纲，实际附件仍要经会话文件工具取得。“九点提醒”可调用日程服务，创建、修改及取消都由日程服务处理。工具结果进入后续判断。 | [回忆查询](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/builtin_tool/query_memory.py)、[会话文件](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/files/provider.py)、[日程工具](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/builtin_tool/schedule.py) |
| 5. 组织并发送 | 常规路径由 Planner 选择 reply，再由 Replyer 结合回复目标、人物设定、场景和表达习惯生成可见回复。发信路径处理引用、消息片段及插件后处理，再交给发送服务。另有默认关闭的实验模式，可将 Planner 最终正文用作 Replyer 的指导。 | [回复提示组装](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/chat/replyer/reply_prompt.py)、[生成与发送](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/builtin_tool/reply.py#L430)、[实验分支](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/reasoning_engine.py#L481) |
| 6. 为下次交流留下信息 | 收到消息和发出回复会触发不同的记忆入口。收到时观察发送者身份并触发图片写回队列，发送后可触发人物事实及聊天摘要队列。最终写入还受长期记忆开关、来源、阈值和提取结果影响。 | [记忆写回入口](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/services/memory_flow_service.py)、[配置默认值](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/config/official_configs.py) |

## 哪些机制影响聊天体验

**私聊里的连贯感来自按会话收拢消息。** v1.4.6 仓库规则说明中，普通私聊“自然”档在安静后开批等待 2 秒，同一人补充后再计 2 秒，整批最多等 8 秒。这些数值只约束等待用户续发的阶段。模型和工具的处理耗时另算。移到语音场景时，值得借鉴的是识别用户还在补充的时机，文本聊天的等待秒数需要重新验证。[群聊与私聊等待规则](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/chat/debounce/README.md)

**群聊先决定值不值得看，再决定要不要说。** 一批消息到齐后，按配置的查看频率决定是否进入 Planner。进入一段活跃对话后，系统会连续关注，再逐步降低查看机会。沉默超过预测的来信间隔也能触发一次查看，之后由 Planner 决定行动。默认私聊走直接查看路径。另有可选 Focus 模式，让指定聊天共享思考与等待，工具按会话选择目标。[群聊频率和静默查看](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/turn_trigger/README.md)、[Focus 分组管理](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/focus/manager.py)

**人设、表达和人物认识各有用途。** 人设预设保存身份和回复风格，切换时先持久化，再检查运行时是否已加载。表达学习从允许的聊天中积累情境化说法，黑话学习单独处理。回复阶段再按当前会话和设置挑选表达。这样可以让麦麦记得某人谈过什么，也可以让她熟悉对方的用词，两种变化分别控制。[人设切换](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/config/personality_service.py)、[表达学习](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/learners/expression_learner.py)、[黑话学习](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/learners/jargon_learner.py)、[回复时选择表达](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/chat/replyer/maisaka_expression_selector.py)

## 上下文、长期记忆和日程

| 需要回答的问题 | 使用的机制 | 具体含义 |
| --- | --- | --- |
| 刚才谈到哪一步了？ | 近期原文和滚动压缩 | 较早对话可压成话题、背景、进展、未完事项、约定五类状态，近期原文继续保留。整理工具历史时还要保持调用与结果配对。[滚动状态](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/context/compaction.py)、[历史整理](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/context/history.py) |
| 上次说过什么？我对这个人知道什么？ | 可选 A_Memorix 长期记忆 | 总开关默认关闭。启用后，系统能整理新增事实与聊天摘要、生成情景记忆和人物画像。Planner 可查询往事，画像注入与按话题自动联想可分别配置。提取规则保留计划、否定和说话人归属。管理页支持修正预览及回滚，聊天反馈自动纠错默认关闭。[增量提取规则](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/prompts/zh-CN/main/memory/a_memorix_chat_summary.prompt)、[记忆配置](https://docs.maibot.chat/manual/configuration/amemorix-config)、[与旧 Momo 的逐项对照](INV-38-maibot-memory-and-learning.md) |
| 到点要做什么？ | 持久化日程与投递 | 服务领取到期事项，检查版本和状态。用户提醒可显示在主人桌面，要求麦麦到时办事的日程则唤醒所属聊天。后续编辑或取消会改变投递所依据的状态。[日程执行与投递](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/services/schedules/service.py) |

## 模型、工具与外部扩展

模型按任务配置。发布版模板分别为 Planner、Replyer、快速判断、记忆、学习、视觉、语音和嵌入等任务留出配置位置。不同岗位可以指向同一模型，部分岗位的默认模型列表为空。每轮实际有哪些调用取决于启用路径。以媒体消息为例，Planner 的候选模型都具备视觉能力时才启用其视觉输入。语音消息有音频字节转写入口，桌面陪伴所需的持续麦克风收音和语音打断需要单独验证。[模型任务配置](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/config/default_model_config.py)、[视觉模式判定](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/visual/mode_utils.py)、[语音消息转写](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/common/utils/utils_voice.py)

能力从内置工具、插件及 MCP 等入口进入。Planner 可以用 `tool_search` 按名称或关键词查找延后加载的工具，再在后续轮次调用。MCP 提供器负责列出服务能力并转发调用。子代理可以承担有范围的查询和工具工作，具体可授予的工具有限制，发言、等待和日程由主 Planner 掌握。[按需发现工具](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/builtin_tool/tool_search.py)、[MCP 提供器](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/mcp_module/provider.py)、[子代理权限和轮次](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/builtin_tool/subagent.py)

插件可在已有流程上注册组件和处理钩子，例如回复生成后的内容转换。宿主侧的监督器管理 Runner、组件注册和权限，Runner 侧负责发现插件、读取清单并加载。评估某个插件时，要同时核实它需要的宿主接口、模型任务和会话权限。[插件监督器](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/plugin_runtime/host/supervisor.py)、[发现与加载](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/plugin_runtime/runner/plugin_loader.py)、[回复扩展接入点](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/builtin_tool/reply.py#L717)

## 宿主、管理台与桌宠

- **部署和数据。** 启动入口打开宿主工作区，宿主管理 Bot 实例和 Worker 生命周期，实例内初始化消息接收、配置、插件运行时、会话、记忆自动化、日程等服务。主业务数据使用 SQLite。分开看这些职责，有助于判断“客户端在线”“机器人在线”“模型可用”各自指哪一层。[启动入口](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/bot.py)、[宿主与实例](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/host/supervisor.py)、[实例初始化](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/main.py)、[数据库](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/common/database/database.py)
- **配置与诊断。** WebUI 能管理会话参与频率、暂停和学习规则。本版支持提示词对照编辑。后端按会话记录可回放的运行事件，并能读取子任务事件，方便追查一轮是停在等待、模型还是工具阶段。[聊天管理接口](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/webui/routers/chat/routes.py)、[事件记录与回放](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/maisaka/monitor/event_store.py)、[WebUI 发布说明](https://github.com/Mai-with-u/MaiBot/releases/tag/v1.4.6)
- **角色客户端。** 独立桌面客户端以 Tauri 2 和系统 WebView 连接指定 Bot，聊天、待办与探索记录保存在服务器，窗口位置在客户端。Android 版用原生悬浮窗和共用角色页面，支持查看聊天历史。桌面读屏工具要求主人会话和读屏开关，返回主屏截图及采集时间。Android 版文档明确其当前读屏与应用监测能力缺席。[桌面客户端](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/pet-clients/desktop/README.md)、[Android 客户端](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/pet-clients/android/README.md)、[读屏调用条件](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/desktop_pet/screen_tools.py)

## 回到 Momo，先用哪些场景评估

以下是基于源码的产品判断，供后续讨论。这次调查只形成研究文档。

| Momo 场景 | 可学习的做法 | 能区分方案好坏的观察 |
| --- | --- | --- |
| 一对一连续说话 | 把补充和更正合进同一个交流意图，保留等待和中断能力。 | 连说“明早提醒我”“等等，改下午”后，最终是否只留下一个正确提醒，语音等待是否自然。 |
| 游戏或工作中的主动陪伴 | 麦麦将“有新情况值得看”和“此刻值得开口”分成两个判断。Momo 可把场景事件作为判断依据。 | 同一局胜负是否重复评论，用户还在讲话时是否给对方留出空间。 |
| 回忆旧事、落实提醒 | 分别保存短期进展、长期事实、需要执行的事项。 | 新会话里的“上次那版”能否找回正确来源，修改日期后原提醒是否还会触发。 |
| 窗口和仅角色两种形态 | 两个呈现入口可以消费同一会话和服务端历史，客户端声明自身设备能力。 | 从角色视图切回窗口能否看到同一条语音记录，读屏请求能否得到真实可用状态。 |

先读代码时，可从“接收入口 → 群／私聊调度 → 行动循环 → 回复发送”四处跟完一轮，再按具体问题进入记忆、日程或插件。技术取舍需要用同一批场景比较回复时机、记忆归属、任务结果、Token、延迟和内存。当前版本的静态源码提供了这些验证入口。
