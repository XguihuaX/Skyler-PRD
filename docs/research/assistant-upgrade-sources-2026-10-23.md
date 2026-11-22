# Momo 四方向调研 · 来源登记

访问/核对日期：2026-10-23。对应 [INV-28 修改大纲](../INV-28-assistant-upgrade-research-outline.md)。以项目官方文档、仓库与原作者研究为依据；以下“用途”说明它支持的设计判断，采用何种实现仍需本机对照。

## 1. 日常任务与应用接入

| 来源 | 用途与适用范围 |
| --- | --- |
| [Raycast Tool](https://developers.raycast.com/api-reference/tool) | 工具输入、执行及按参数描述确认动作，参考应用能力接入体验。 |
| [Apple Shortcuts CLI](https://support.apple.com/guide/shortcuts-mac/run-shortcuts-from-the-command-line-apd455c82f02/mac) | 快捷指令的命令行运行、文件输入输出；交互型指令可能等待用户输入。 |
| [Playwright MCP](https://playwright.dev/docs/getting-started-mcp) | 结构化页面快照与浏览器会话方式；复用登录和浏览器生命周期设计。 |
| [Gmail sync](https://developers.google.com/workspace/gmail/api/guides/sync) | 全量/增量同步与历史游标失效后的恢复。 |
| [Gmail push](https://developers.google.com/workspace/gmail/api/guides/push) | 明确安装在用户设备上的客户端推荐同步拉取；Pub/Sub 推送路径按需要另选。 |
| [Microsoft Graph delta](https://learn.microsoft.com/en-us/graph/delta-query-overview) | 增量变化、分页与游标；邮箱场景应继续核对对应资源接口。 |

## 2. RAM、模型与前端资源

| 来源 | 用途与适用范围 |
| --- | --- |
| [Apple TN2434](https://developer.apple.com/library/archive/technotes/tn2434/_index.html) | 内存分配和 footprint 分析方法；属于归档方法资料，不提供当前 Momo 或 M2 的性能结论。 |
| [faster-whisper](https://github.com/SYSTRAN/faster-whisper) | 量化、batch 对资源的影响；其公开 Intel 基准不作为 M2 实测。 |
| [whisper.cpp](https://github.com/ggml-org/whisper.cpp) | Apple Silicon、Core ML encoder 与本地 ASR 原型；首轮编译和中文质量需本机验证。 |
| [Sentence Transformers efficiency](https://www.sbert.net/docs/sentence_transformer/usage/efficiency.html) | PyTorch/ONNX/OpenVINO 候选后端及任务相关的性能取舍。 |
| [CTranslate2 memory](https://opennmt.net/CTranslate2/memory.html) | 对象释放与模型卸载方法；Translator API 不直接等同于 Whisper API。 |
| [PixiJS 7 performance](https://pixijs.com/7.x/guides/production/performance-tips) | 与项目主版本一致的纹理、GC 和显示资源管理参考。 |

## 3. 记忆、上下文、工具与用量

| 来源 | 用途与适用范围 |
| --- | --- |
| [SQLite FTS5](https://www.sqlite.org/fts5.html) | 全文索引、tokenizer、trigram 的短查询限制；配合下方合成实验。 |
| [LiteLLM token usage](https://docs.litellm.ai/docs/completion/token_usage) | 统一 usage 与成本估算；记录实际模型与供应商返回，缺失值保持可辨识。 |
| [Hermes Tool Search](https://hermes-agent.nousresearch.com/docs/user-guide/features/tool-search/) | 工具目录按需搜索与执行桥接，避免全部 schema 常驻。 |
| [Claude Tool Search](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool) | 原生延迟工具加载的 API 契约；延迟进入模型上下文不等于 HTTP 请求省略工具定义。 |
| [OpenAI Prompt caching](https://developers.openai.com/api/docs/guides/prompt-caching) | 稳定前缀和缓存命中；工具集合变化应一起比较实际总成本。 |
| [LongMemEval](https://github.com/xiaowu0162/LongMemEval) | 长期记忆评测的提取、跨会话、时间、更新和无答案维度；不是 Momo 已取得的分数。 |
| [Anthropic Agent evals](https://www.anthropic.com/research/demystifying-evals-for-ai-agents) | 任务结果、轨迹和重复试验；适配为小规模固定验收集。 |

## 4. 语音与回合控制

| 来源 | 用途与适用范围 |
| --- | --- |
| [LiveKit turn handling](https://docs.livekit.io/reference/agents/turn-handling-options/) | 回合结束、打断判定和误打断恢复的控制点。 |
| [OpenAI Realtime VAD](https://developers.openai.com/api/docs/guides/realtime-vad) | 静音检测与语义回合判定；当前 Momo 尚未因此接入该服务。 |
| [Moshi](https://github.com/kyutai-labs/moshi) | 全双工语音模型及推理实现，作为独立原型候选。 |

## 5. MaiBot 与 Hermes

| 来源 | 用途与适用范围 |
| --- | --- |
| [MaiBot bot config](https://docs.mai-mai.org/manual/configuration/bot-config) | 人格、行为、回复风格与表达学习范围；文档标示更新于 2026-10-20。 |
| [MaiBot A_Memorix](https://docs.mai-mai.org/manual/configuration/amemorix-config) | 中文稀疏/混合检索、事件分段与增量作业；不默认复制全部模块。 |
| [Hermes Memory](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/) | 有界常驻快照与历史搜索；Momo 当前更正需要独立的即时更新路径。 |
| [Hermes Skills](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills/) | 渐进加载和可复用技能；项目采用策略与上游默认写入行为分别定义。 |
| [Hermes Cron](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron/) | 定时任务及无需模型的脚本方式，减少固定事务的后台开销。 |

仓库元数据通过 GitHub API 查询。以下用于明确调查时点，未声明完整审计这些提交的所有代码：

| 项目 | 默认分支提交 | 提交时间 UTC | 仓库声明许可 |
| --- | --- | --- | --- |
| Mai-with-u/MaiBot | [c602a2780f4f4dda4a6be46d67abc1cfb1ad420c](https://github.com/Mai-with-u/MaiBot/tree/c602a2780f4f4dda4a6be46d67abc1cfb1ad420c) | 2026-10-21 18:31:57 | GPL-3.0 |
| NousResearch/hermes-agent | [a01edde680a73fdcabc2aa20b94d0581338e499b](https://github.com/NousResearch/hermes-agent/tree/a01edde680a73fdcabc2aa20b94d0581338e499b) | 2026-10-22 19:58:45 | MIT |

## 6. 本机证据与实验

- `sysctl -n machdep.cpu.brand_string`：Apple M2；`sysctl -n hw.memsize`：8589934592，等于 8 GiB。
- 本轮本地代码 HEAD：`00a164da`；工作区包含未提交修改，以 INV-28 的文件锚点对应本轮读取内容。
- 已检查源码：启动与资源采样、记忆提取/向量检索、工具装配、usage 与 Token 探针、语音打断及队列。逐项位置见 INV-28 §2。
- [FTS5 实验脚本](../../references/SOURCE-REFERENCES.md#source-923c9d39d0) 与 [结果 JSON](../../references/SOURCE-REFERENCES.md#source-68883498a1)：7 条合成中文文本，6 个查询，对照 unicode61、trigram 和预分词双字索引。执行环境 SQLite 3.53.0。脚本运行方式：`python3 docs/research/fts5-probe-2026-10-23.py`。
- 实验只检验这些输入的词法匹配；没有测 LLM、embedding、大库性能或用户数据。没有启动完整 Momo，因此 RAM 基线、语音体感和端到端费用仍是下一批实测交付物。
