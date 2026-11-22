# INV-17 · 日常实用助手、资源与交互架构调研

日期：2026-10-01。性质：调查与建议，不代表功能已经实现或通过真机验收。

## 结论

Skyler 下一阶段建议以「能替用户完成可核验的日常小任务」为产品主线。邮件摘要、提醒备忘、回复草稿是调研时采用的**示例场景**，不是已选定的第一批任务。任务 1 应先与用户访谈，整理真实需求，再共同确定功能和优先级。现有角色卡、Live2D、主动交互可作为交付体验，但任务结果需要有明确来源、状态和失败提示。

工程上先做三项基线：实际 RAM 分段测量、每轮 Token 组成与任务质量、语音打断与误打断。之后逐步调整加载策略、工具暴露、上下文层级与模块边界。未经测量，不能承诺任何 RAM 降幅。

## 1. 项目现状与可复用基础

- 日历：`backend/capabilities/calendar.py` 已路由 Apple / Google 日历读能力；`backend/capabilities/apple_calendar.py` 支持事件及提前提醒参数。`backend/scheduler/briefing.py` 有晨间简报和唤醒简报入口。
- 能力与模型：`backend/capabilities/registry.py` 有能力注册和调度分类；`backend/llm/client.py` 通过 LiteLLM 与数据库中启用的 provider 路由模型。这足以作为连接 OpenAI 等模型 API 的接入层，不等于接入 ChatGPT 网页会话或其记忆。
- 邮件与备忘录：在当前 `backend/capabilities/`、`backend/integrations/` 中未发现对应的个人邮箱或独立备忘录接入模块。不能把日历事件、角色记忆当成用户任务数据库。
- 内存：`backend/main.py:795-822` 启动时并行预热 Whisper 与 embedding；`backend/asr/whisper.py` 默认 small / CPU int8；`backend/memory/long_term.py` 持有 sentence-transformers 模型。它们是**待测的常驻 RAM 候选项**，目前没有运行中的 Skyler 进程作为实际占用证据。
- 上下文：`backend/memory/short_term.py` 对每个 `(user, character, conversation)` 有 4,000 token 近期历史上限；`backend/memory/summary.py` 有 2,000 token 滚动摘要。旧审计中「短期历史无上限」的说法已经过时。
- 工具：`backend/agents/chat.py:592-598` 与 `:2182-2200` 显示每轮把 memory tools 加全部已注册工具 schema 交给模型。`backend/memory/long_term.py:317-360` 每次检索会加载该范围全部记忆后逐条计算相似度；量增大时需要索引或分层筛选，但当前瓶颈尚未实测。
- 语音：`frontend/src/hooks/useAudio.ts:203-205` 以 `audio: true` 获取麦克风；`:457-474` 在 AI 讲话时用音量持续阈值触发打断。`frontend/src/hooks/useWebSocket.ts:263-278` 以完整 WAV/base64 段进入播放队列，可配置播放时抑制麦克风。当前是轮流说话加初步打断能力，不是经过回声消除与重叠语音识别验证的全双工。

### 补充：桌面操控能力（2026-10-01 本机状态）

用户明确希望 Momo 在授权后直接操作电脑。技术路径是「读取 AX/截图 → 选目标 → 点击/输入/滚动 → 重新读取并核验结果」，复杂任务还需要处理弹窗、失败与恢复。当前 `screen.read_current_screen` 只读，但 `mcp.config.yaml` 配置了 macos-use 的 8 个写工具；本机 `momoos.db` 的 `mcp_client_state` 显示 macos-use 已启用，8 个 `mcp_tool_state` 行均为 `enabled=1, require_confirmation=1`，本机可执行文件存在。`backend/mcp/client.py:327-343` 会把已启用的 MCP 工具注册到能力列表，`:488-504` 在调用前检查确认门，`backend/mcp/confirm_gate.py:111-197` 在拒绝、超时或前端连接缺失时取消调用。因此代码层面已有执行单步动作的路径。

但 `backend/agents/prompt/tool_addendum.py:125-127` 仍告诉模型「写工具看不到、只能读」，与本机数据库状态不一致；旧 `docs/design/desktop-control.md` 也以只读 MVP 描述现状。当前没有运行中的后端进程，尚未验证 macOS 辅助功能授权、MCP 连接、模型实际选工具、确认弹窗和动作结果。截图 helper 本机未安装，因此 AX 盲区仍在。结论：**能够作为桌面操控架构的基础，不能据此宣称 Momo 目前已可靠地自动完成任意电脑任务。** 下一步应先对一个低风险任务做端到端真机验收，再统一配置、提示与权限状态；多步任务需要单独的执行状态和逐步核验。

## 2. 外部项目值得借鉴的机制

| 项目 | 已确认机制 | Skyler 可借鉴之处 | 不宜直接照搬 |
| --- | --- | --- | --- |
| MaiBot | 将人格设定、参与时机与回复风格分开配置；按聊天氛围决定说、等或保持安静；支持有审核与范围控制的表达学习；A_Memorix 管理记忆来源和范围，并提供可视化与 Token 统计 | 学习其「何时说、说什么、怎么说」的交互判断、情境化表达与记忆管理，同时给助手任务保留清楚的完成状态 | 它以社交聊天流为主要对象；群聊发言策略和多层向量/图结构不应直接搬到单人桌面助手 |
| Hermes Agent | 小型常驻 `MEMORY.md` / `USER.md`；历史会话存 SQLite FTS5 并按需检索；工具按 toolset 启用；定时任务可用独立新会话或无模型脚本 | 常驻上下文预算、按需历史搜索、按任务加载工具、确定性提醒不用每次调用 LLM | 不需要迁移到文件制记忆，也不需要一步引入全部代理工具 |

来源：[MaiBot 记忆配置](https://docs.mai-mai.org/manual/configuration/amemorix-config)、[MaiBot 1.3.x 更新](https://github.com/Mai-with-u/MaiBot/releases)、[Hermes 记忆](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/)、[Hermes 工具集](https://hermes-agent.nousresearch.com/docs/user-guide/features/tools/)、[Hermes 定时任务](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron/)。

MaiBot 的人设交互尤其值得单列：官方[Bot 配置](https://docs.mai-mai.org/manual/configuration/bot-config)把 `personality`（身份与性格）、`behavior_style`（何时参与）、`reply_style`（语言风格）分开；[消息处理流程](https://docs.mai-mai.org/en/manual/features/message-pipeline)描述了根据聊天气氛决定立即回复、等待或安静；表达学习还有审核、聊天范围和情境选择。Skyler 目前已有结构化 Persona、状态与角色记忆（`ROADMAP.md:66-96`），适合研究的是交互时机与情境表达如何稳定落地，而不是再堆一份更长的人设提示词。可以用「同一条提醒如何在用户专注、闲聊、深夜时表达」这类用例验证自然度与打扰率。

MaiBot 也有社区维护的[邮件适配器](https://docs.mai-mai.org/manual/adapters/email)，但它的主要用途是让别人发邮件给**机器人专用邮箱**并由机器人回信；Skyler 的目标是处理用户自己的邮箱，需要单独设计账户授权、同步与审核流程。

## 3. 日常实用性：以邮件和提醒为例的任务梯度

以下是技术路径示例，用于评估候选场景的能力与代价；不等于产品排期。最终场景需由任务 1 的用户需求访谈决定。

1. **信息读入**：用户授权的 Gmail / Outlook 或其他明确服务；列出来源、时间、发件人及链接；去重、过滤垃圾与敏感范围。Gmail 的 `watch` 需要 Cloud Pub/Sub 与定期续订，桌面客户端也可先用增量轮询；Outlook 由 Microsoft Graph 提供邮件变更订阅。见 [Gmail push 文档](https://developers.google.com/workspace/gmail/api/guides/push) 与 [Microsoft Graph 邮件通知](https://learn.microsoft.com/en-us/graph/outlook-change-notifications-overview)。
2. **可核验的整理**：每日邮件摘要，抽取需回复、截止时间与待办，并回链原邮件。每个结论标注来源与置信程度；不能把模型猜测写成确定提醒。
3. **提醒与备忘**：建立独立的 `reminder / note` 任务实体，含到期时间、时区、重复规则、完成/稍后提醒、来源与投递状态。简单定时由本地调度器执行；自然语言只负责解析和措辞。关机后的补发与重复提醒需明确定义。
4. **行动草稿**：邮件回复在编辑界面展示收件人、原邮件、草稿、附件与发送账户；由用户确认后发送并记录结果。先做好草稿成功率，再决定更高自动化。

可用一条完整任务检验产品价值：「今天哪些邮件需要我处理？」→ 展示有原文入口的摘要 → 生成回复草稿或提醒 → 用户编辑/确认 → 记录完成状态。指标包括正确归类率、漏报率、从打开到完成的耗时、提醒准时率、错误发送次数。

## 4. RAM 压力：先测，再减

区分 Python 后端、Tauri/WebView、Live2D 图形资源以及可能独立运行的 TTS 服务。最少记录冷启动、空闲 10 分钟、开启语音、连续 20 轮聊天、切换角色、停止语音后的各进程 RSS / 峰值、系统内存压力与首轮延迟。用 Instruments 的 Allocations / VM Tracker 追踪前端与图形资源，Python 侧同时观察 native 模型内存；仅用 `tracemalloc` 无法解释所有 native 分配。见 [Apple 内存分析指南](https://developer.apple.com/documentation/Xcode/gathering-information-about-memory-use)。

候选实验按单变量进行：

1. 对比启动预热 Whisper + embedding 与按需加载；同时测首轮语音/记忆检索延迟。
2. 为不使用语音的会话设定可卸载/休眠策略，检查模型对象、音频上下文与播放队列能否释放。
3. 评估更小 ASR 模型或外部 ASR 的准确率、延迟和 RAM；`faster-whisper` 官方 CPU small/int8 基准说明模型和推理设置会显著影响占用，但不是本机结果：[项目基准](https://github.com/SYSTRAN/faster-whisper)。
4. 检查未关闭的 conversation bucket、Live2D 纹理、音频元素及 WebSocket 资源是否随生命周期增长。

## 5. 上下文、Token 与记忆层

历史 `logs/token_probe.jsonl` 的 1,405 条请求记录（2026-05-19 至 08-11）中，估算输入 Token 中位数为 19,361；其中工具 schema 9,423、系统提示合计 9,076、近期历史 203、滚动摘要 0。P90 总数 24,361，工具 schema 16,020。该探针是本地估算，不等于供应商账单；异常超大请求需要另查附件/输入来源。由此推断当前优化优先级应是工具 schema 与常驻系统提示，而不是先压缩普通近期聊天。

建议的上下文结构：

- **始终可见**：任务安全边界、当前角色最小设定、用户稳定偏好。给硬预算与版本号；静态前缀尽量稳定，以便使用供应商缓存。
- **按任务加载**：邮件、日历、文件、社交等工具族；只暴露可用且当前任务相关的 schema。路由选择可以先用界面入口、显式用户意图及低成本规则，回退时扩大集合。测工具调用成功率，避免省 Token 却导致任务失败。
- **近期对话**：现有 4,000-token 窗口保留；滚动摘要限额并定期查事实失真。
- **长期记忆**：分 `用户稳定偏好 / 待办与提醒 / 会话事实 / 角色关系设定`。提醒是可操作任务，必须有状态，不应只写自然语言记忆；长期记忆标来源、时间、范围、可撤销性，按需检索少量结果。用户个人资料可跨助手卡共享，角色私有经历应隔离。
- **历史检索**：先评估 SQLite FTS5 对明确词、人物、邮件主题和日期的召回，再决定是否需要向量模型常驻与更复杂索引。对结果设条数、字数和过期规则。

需要增加每轮分层 Token、provider 实报 usage、缓存命中、工具选择与任务成功率的同屏统计；避免只凭总 Token 做优化。

## 6. 半双工、打断与全双工

建议把语音能力分为三个明确等级：

1. **轮流说话**：AI 播放时暂停有效收音；最稳定、最低实现成本。
2. **可打断**：播放时持续监听用户开口，清理播放队列并取消后端当前生成；需测试耳机/扬声器、环境噪音、短应和误触发。目前 Skyler 已有音量阈值版。
3. **全双工**：在重叠讲话时仍能分离用户与系统音、连续识别、判断「嗯/好的」是附和还是真打断，并保持轮次一致性。需要确认设备实际回声消除效果、流式音频、分段时间戳和中断后文本/音频同步。浏览器可请求并检查 `echoCancellation`，但请求成功本身不证明端到端效果。见 [MDN](https://developer.mozilla.org/en-US/docs/Web/API/MediaTrackConstraints/echoCancellation)、[LiveKit 自适应打断](https://docs.livekit.io/agents/logic/turns/adaptive-interruption-handling/) 与 [OpenAI 实时 VAD](https://developers.openai.com/api/docs/guides/realtime-vad)。

当前建议先完善等级 2：真实回声场景、误打断率、打断响应时间、播放队列和被打断回复的持久化一致性。若高频语音日常任务确需双方重叠说话，再做等级 3 原型；不要仅凭连续开麦声称全双工。

## 7. 代码与模块整理

体积较大的 `backend/agents/chat.py`、`backend/routes/ws.py`、`backend/main.py`、`frontend/src/hooks/useAudio.ts` 是可读性和边界审查的起点。建议按契约切分：聊天消息装配/工具选择/工具执行，WebSocket 运输/单轮状态/音频推送，启动编排/迁移/预热，前端捕获/VAD/中断/播放。先锁定已有可观察行为和最小回归案例，再小步抽取模块。不要以文件行数本身当完成指标；以改动隔离、循环依赖减少、延迟与内存无退化为验收。

## 8. 模型接入边界

Skyler 的 LiteLLM + DB provider 路由已有多供应商基础。接 OpenAI、Gemini 等应先验证所需模型、流式输出、工具调用、视觉/语音能力、错误处理与实际计费；无需为了「连 ChatGPT」重做整个聊天层。

「调用 OpenAI 模型 API」与「让自建应用使用 ChatGPT 订阅额度」是两个不同通道。OpenAI 官方说明 API 单独计费；2026 年已有部分**参与计划的**应用可用 Sign in with ChatGPT 共享 Plus/Pro 额度，但它不自动提供 ChatGPT 对话/记忆访问，且参与资格和功能有约束。目前未确认 Skyler 是参与应用，因此可执行的常规路线是 OpenAI API。见 [计费说明](https://help.openai.com/en/articles/9039756-managing-billing-for-chatgpt-and-the-api-platform) 与 [ChatGPT 计划用于其他应用](https://help.openai.com/en/articles/20001542-using-your-chatgpt-plan-in-other-apps-and-sites)。

## 9. 建议顺序与待验证问题

| 顺序 | 工作包 | 可验收产物 |
| --- | --- | --- |
| P0 | 与用户访谈并整理日常场景、功能状态与授权边界；建立 RAM/Token/语音基线 | 候选场景清单、用户确认的优先级、基线数据、缺口清单 |
| P1 | 实现用户选定的首个场景，保留来源、状态及失败恢复 | 一条完整闭环任务及失败恢复 |
| P1 | 按任务筛选工具 schema，精简重复常驻提示 | 同任务质量下更低的实报输入 Token |
| P1 | 基于测量调整模型预热和资源释放 | 空闲/语音 RAM 与首轮延迟对照 |
| P2 | 用户选定场景的后续动作、记忆范围/来源管理、模块抽取 | 对应功能的用户验收与回归记录 |
| P3 | 经需求验证的全双工、多来源数据接入、复杂向量/图记忆 | 在实际用例中证明优于较简单方案的效果 |

待决定：用户最希望解决的真实日常任务、可用应用与数据来源、主动程度和操作权限。若用户选择邮件或提醒，再决定邮箱服务、投递方式以及应用退出后的行为。RAM 目标设备与可接受的首轮语音延迟也需确认；这些决策不妨碍先完成 P0 测量与需求发现。
