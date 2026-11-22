# INV-25 资料与证据登记

访问与复核日期：2026-10-13。对应 [主报告](../INV-25-commercial-assistant-research.md)。定位为“先自用，按商业产品质量建设”。

本轮使用 39 个官方资料页面或上游仓库入口。下表登记它们支持的范围；实现建议、候选阈值和费用演算由本报告提出。没有引用搜索摘要作为竞品性能证据，没有安装运行 MaiBot/Hermes，也没有调用私人账户做端到端测量。动态文档可能继续更新。

## 官方来源

| 编号 | 来源 | 本轮用途与边界 |
| --- | --- | --- |
| S01 | [工作流与 Agent](https://www.anthropic.com/engineering/building-effective-agents) | 工作流与自主 Agent 的适用条件；用于架构取舍，不证明 Momo 的任务表现。 |
| S02 | [Raycast 工具设计](https://developers.raycast.com/ai/learn-core-concepts-of-ai-extensions) | AI 扩展工具的暴露方式；用于自然语言与直接操作共享能力的设计。 |
| S03 | [Apple 自动化](https://support.apple.com/en-qa/guide/shortcuts-mac/apdfbdbd7123/mac) | 应用动作及自动化触发；具体系统版本和动作可用性仍需本机验证。 |
| S04 | [Notion 权限与隐私](https://www.notion.com/help/enterprise-search-security-and-privacy-practices) | 检索权限和同步边界；企业产品仅作机制参照。 |
| S05 | [Computer use](https://developers.openai.com/api/docs/guides/tools-computer-use) | 动作与截图反馈循环；权限充分不等于桌面任务必然流畅。 |
| S06 | [Gmail scopes](https://developers.google.com/workspace/gmail/api/auth/scopes) | 邮箱 OAuth scope 分类和相关要求；是否适用取决于实际数据流与发布方式。 |
| S07 | [Gmail push](https://developers.google.com/workspace/gmail/api/guides/push) | Gmail watch/PubSub 与续期；尚未选择 Gmail 作为首批功能。 |
| S08 | [Graph notifications](https://learn.microsoft.com/en-us/graph/change-notifications-overview) | Graph 变更通知机制；实际资源订阅生命周期需按所选场景查明。 |
| S09 | [LiteLLM Router](https://docs.litellm.ai/docs/routing) | 模型路由与回退；项目配置、模型能力仍需回归。 |
| S10 | [OpenAI 计费说明](https://help.openai.com/en/articles/9039756-managing-billing-for-chatgpt-and-the-api-platform) | ChatGPT 与 API 分别计费；不推断订阅额度或记忆可供 Momo 使用。 |
| S11 | [Apple 内存测量](https://developer.apple.com/documentation/xcode/gathering-information-about-memory-use) | Xcode/Instruments 的内存观察方法；HTML 为 JS 页面，实际读取其官方 Markdown 版本。 |
| S12 | [官方基准](https://github.com/SYSTRAN/faster-whisper) | 指定硬件下的推理基准；不能外推为本项目 Apple Silicon 性能。 |
| S13 | [CTranslate2 内存管理](https://opennmt.net/CTranslate2/memory.html) | 模型对象资源管理；Translator 接口不能直接假定适用于 Whisper 包装层。 |
| S14 | [Anthropic 上下文工程](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | 相关上下文、压缩和长任务状态；各层预算为本报告建议。 |
| S15 | [SQLite FTS5](https://www.sqlite.org/fts5.html) | FTS5 与 trigram 限制；中文检索需要自己的语料评测。 |
| S16 | [OpenAI Prompt caching](https://developers.openai.com/api/docs/guides/prompt-caching) | 提示缓存机制；配置和计费依供应商与模型而异。 |
| S17 | [Function calling](https://developers.openai.com/api/docs/guides/function-calling) | 函数工具协议与参数设计；不等于工具已经执行或获授权。 |
| S18 | [工具设计](https://www.anthropic.com/engineering/writing-tools-for-agents) | 工具描述、输出与评测；工具数量应由 Momo 实验决定。 |
| S19 | [MCP 安全实践](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices) | MCP 安全实践；本项目暴露面尚未作专项测试。 |
| S20 | [MDN 约束说明](https://developer.mozilla.org/en-US/docs/Web/API/MediaTrackConstraints/echoCancellation) | 音频回声消除约束；申请约束不证明设备的实际处理效果。 |
| S21 | [LiveKit turn tuning](https://docs.livekit.io/agents/logic/turns/tuning/) | 回合判断、打断等语音配置；尚未引入或实测 SDK。 |
| S22 | [OpenAI VAD](https://developers.openai.com/api/docs/guides/realtime-vad) | 实时 VAD 机制；单独启用不构成全双工验收。 |
| S23 | [MaiBot 仓库](https://github.com/Mai-with-u/MaiBot) | MaiBot 仓库与许可证元数据；未安装运行，固定快照见下。 |
| S24 | [Hermes 仓库](https://github.com/NousResearch/hermes-agent) | Hermes 仓库与许可证元数据；未安装运行，固定快照见下。 |
| S25 | [MaiBot 配置](https://docs.mai-mai.org/manual/configuration/bot-config) | 人格、参与及表达配置；自然度提升需 Momo 盲评。 |
| S26 | [A_Memorix 配置](https://docs.mai-mai.org/manual/configuration/amemorix-config) | A_Memorix 的范围、检索、生命周期及模型指纹；未复现效果。 |
| S27 | [记忆与学习](https://docs.mai-mai.org/faq/memory-and-learning) | 记忆学习与共享范围；群聊边界不能直接搬到个人助手。 |
| S28 | [Memory](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/) | 有界常驻记忆、历史检索和后台审查；不表示权重训练。 |
| S29 | [Skills](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills/) | 技能按需加载与写入管理；流程正确性仍需回归。 |
| S30 | [Tool Search](https://hermes-agent.nousresearch.com/docs/user-guide/features/tool-search/) | 工具搜索与真实工具调用控制；目录筛选不能替代执行授权。 |
| S31 | [Cron](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron/) | 定时会话和脚本模式；Momo 的本地通知仍需自有可靠性设计。 |
| S32 | [SDK 发布许可](https://www.live2d.com/en/sdk/license/) | Live2D SDK 发行许可入口；仅未来发行时按实际方式核验。 |
| S33 | [Expandable Applications](https://www.live2d.com/en/sdk/license/expandable/) | Expandable Applications 类别；不推断当前项目已获或必需某项许可。 |
| S34 | [Tauri capabilities](https://v2.tauri.app/security/capabilities/) | Tauri IPC capability 边界；CSP、API/WS 鉴权还需分别审计。 |
| S35 | [Tauri updater](https://v2.tauri.app/plugin/updater/) | Tauri 更新签名；自用先建立可恢复版本，发行能力后续验证。 |
| S36 | [分发指南](https://v2.tauri.app/distribute/) | Tauri 分发要求；本轮未进行打包与干净安装测试。 |
| S37 | [LangGraph persistence](https://docs.langchain.com/oss/python/langgraph/persistence) | LangGraph checkpoint/store；选择持久 saver 才有相应重启恢复。 |
| S38 | [Temporal](https://docs.temporal.io/evaluate) | Temporal 持久执行选型；外部副作用仍需幂等与补偿。 |
| S39 | [Agent evals](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) | Agent 评测、状态验证和重复试验；建议样本量不是统计保证。 |

## 上游仓库快照

通过 GitHub API 读取仓库元数据及默认分支提交；时间均为 UTC。以下仅固定引用基点，不代表逐文件审核了该 commit。

| 项目 | 默认分支快照 | 提交时间 | 仓库标示许可证 |
| --- | --- | --- | --- |
| Mai-with-u/MaiBot | [0f94cfa83fee87a93275951c9ba2207a90df9070](https://github.com/Mai-with-u/MaiBot/tree/0f94cfa83fee87a93275951c9ba2207a90df9070) | 2026-10-11 16:32:49 | GPL-3.0 |
| NousResearch/hermes-agent | [63d0b94120027d487c9a5235b9ec64b696af1b26](https://github.com/NousResearch/hermes-agent/tree/63d0b94120027d487c9a5235b9ec64b696af1b26) | 2026-10-13 03:25:50 | MIT |

文档站与仓库发布节奏可能不同。若实施时复制或依赖外部代码，应重新固定版本，检查该版本 LICENSE、NOTICE、资产和依赖。

## 本地证据

本地分支 codex/code-slimming-baseline-20261001，HEAD 00a164da；检查包含未提交修改的工作区。以下锚点可能随任务 4 继续工作而移动，函数名与表达式用于复核。

| 结论 | 路径 / 检索锚点 |
| --- | --- |
| 提取后推进游标，None 时提前返回 | backend/memory/extractor.py:366、397；`_process_user_turns` / `update_last_processed_turn_id` |
| 短输入门槛、向量检索 | backend/memory/long_term.py:289、305；`search_relevant_memories` |
| 近期与摘要预算 | backend/memory/short_term.py:1；backend/memory/summary.py:67 |
| 常规全工具目录 | backend/agents/chat.py:593；`_get_all_tools` |
| 静态屏幕工具说明 | backend/agents/prompt/tool_addendum.py:126；`屏幕写` |
| 模型预热 | backend/main.py:358；embedding / Whisper preload |
| 四路就绪 | frontend/src/hooks/useLoadingSequence.ts:97；`appReady` |
| 音频输入与播放 | frontend/src/hooks/useAudio.ts:210；frontend/src/hooks/useWebSocket.ts:295 |
| MCP 凭证直接写 value | backend/mcp/credentials.py:57；`upsert` / `INSERT INTO mcp_credentials` |
| AI provider 加密路径 | backend/database/ai_providers.py；`Fernet` / encrypt / decrypt |
| 本地接口和 WebView 配置 | backend/main.py:838；frontend/src-tauri/tauri.conf.json:29 |
| 配置定位 | frontend/src-tauri/src/main.rs:14；`locate_config` |
| 确认门禁 | backend/mcp/confirm_gate.py:111；确认超时与无连接处理 |

历史测试、构建体积和 MCP 初始化对照来自 INV-21/23 等已有记录；本轮没有将它们重跑或当成 RAM 改善证据。没有读取私人邮件、凭证值或当前用户数据库内容。

## 访问限制及排除项

- Apple 正文实际读取地址：[官方 Markdown](https://developer.apple.com/documentation/xcode/gathering-information-about-memory-use.md)。网页正文需要 JavaScript，浏览工具不支持该 Markdown MIME，随后通过普通 HTTP 获取并阅读。
- MaiBot 某旧流程页重定向到手册首页，未用于支持特定执行流程结论。
- OpenTelemetry GenAI 的部分候选地址未取得可核对的规范正文，未用来断言具体字段或规范稳定性；本报告 trace_id 是项目设计建议。
- 发生访问错误的旧 LiveKit 地址已改为表中现行官方页面。LangGraph 候选旧地址跳转后采用表中实际持久化文档。
- 第三方转述、营销排名、社区声誉和未经复现的速度数字没有作为选型证据。商业发行与许可结论仍需针对实际复用/分发方式单独审查。
