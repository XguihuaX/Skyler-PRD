# INV-29 · 全局复核、DSH 更新与运行时选型证据

日期：2026-11-17。本轮由用户要求重新统合项目，并进一步明确：评估沿用项目还是抽取需求按新框架重搭。推荐与能力清单见 [ARCH-001](product/ARCH-001-runtime-renewal.md)，当前执行状态以 [ROADMAP](../ROADMAP.md) 为准。

## 1. 当前工作区核对

- HEAD 仍为 `00a164da`（2026-10-01）；工作区含此前瘦身、工程验证和调查文档的未提交成果。没有把 HEAD 不变解释为没有进展。
- 扫描 backend 的 Python 与 frontend/src 的 TS/TSX：修改前为 357 个文件、86,440 行（含注释/空行）。chat.py 2,350 行、ws.py 1,580 行；行数只用于定位维护热点，不能直接推导质量或 RAM。
- 前端依赖声明：React 18、Tauri 2、TypeScript、Vite、PixiJS 7。后端项目 venv：Python 3.10.10、FastAPI 0.136.1、LiteLLM 1.83.14、MCP 1.27.0、SQLAlchemy 2.0.49、Sentence Transformers 5.4.1、faster-whisper 1.2.1。
- requirements.txt 多数包没有精确锁定，前端已有 package-lock.json。可重建环境与跨模型兼容值得治理；未进行全量依赖升级或漏洞审计。
- 本机 Node 为 22.15.1；新候选框架未安装到项目 venv。本轮不调用真实模型或账户，不启动完整 Momo。

## 2. 架构判断的代码依据

| 检查项 | 当前证据 | 判断 |
| --- | --- | --- |
| 是否只是单次 LLM 调用 | chat.py 中 `ChatAgent.stream` 包含模型输出、工具结果、追加消息及再次调用，上限 5 轮 | 已是单 Agent 工具循环；“有一个主模型”不是落后证据 |
| Harness 基础 | Provider、工具注册、MCP、分层 Prompt、日志、取消、历史持久化均存在 | 已有部分自研 Harness 职责；缺少统一持久任务执行边界 |
| 职责集中 | chat.py 同时负责记忆工具、消息装配、仲裁、工具调度和句子流；ws.py 同时负责输入、取消、TTS 与持久化协调 | 新能力继续加入这两个入口会扩大耦合；适合分离执行核心 |
| 恢复 | ws.py `_TurnState` 保存本连接的 task、音频、reply_parts；中断保存聊天历史 | 有会话保存与取消；未发现主 Chat 工具循环的通用 checkpoint/resume。proactive 的局部 execution_id 不等于所有任务具备恢复 |
| 记忆读取 | long_term.py 短长度 gate、get_all_memories、逐条 cosine、top_k=5 | 短问句、指代与大库候选定位仍有缺口 |
| 记忆写入 | extractor.py 无效结果返回后外层推进游标；批次使用最后一个 turn ID | 失败重放、逐条证据仍需重建；未被上一轮去重缓存修复 |
| MCP 多模态 | client.py 将非 text block 保存为 type 与 repr；ClientSession 未接 elicitation 回调 | DSH CUA 接入需要真实图片/权限询问适配，不能声称已即插即用 |
| 依赖与资源 | 模型预热、全历史恢复；现有采样主要后端 RSS | 新框架增加 sidecar 也可能增加占用，RAM 收益待本机对照 |
| 计划不一致 | DESIGN_LITE 仍强调先角色状态；ROADMAP 当前顺序仍偏 F-002/QQ；助手 PRD 保留旧串行顺序 | 本轮统一到架构评估＋日常需求发现，保留原社交验收状态 |

可复查命令：

```sh
rg -n 'max_rounds|while True|tool_calls_acc|messages.append' backend/agents/chat.py
rg -n 'class _TurnState|reset_for_new_turn|_save_interrupted_turn' backend/routes/ws.py
rg -n 'get_all_memories|short_input_threshold|for mem in all_memories' backend/memory/long_term.py
rg -n 'update_last_processed_turn_id|last_turn_id|if raw is None' backend/memory/extractor.py
rg -n 'ClientSession|_repr|elicitation' backend/mcp/client.py
```

## 3. 最新上游事实与版本

### DeepSeek Harness

2026-11-17 读取的独立快照：

| 来源 | 结果 | 适用范围 |
| --- | --- | --- |
| 官方网站 | 提供 macOS Apple 芯片与 Windows 桌面下载 | “官方只有 Web”已不适用 |
| GitHub master | `39163dd30f703a79079e2d2446545cff3c453738`；提交 2026-11-13 UTC；MIT | 本轮源码文档固定于此提交 |
| npm latest | `@deepseek-ai/dsh 0.2.2-rc.1`，Node >=24.9.0 | 本机 Node 22 不满足这条 CLI 安装路线 |
| PyPI SDK | `deepseek-harness-sdk 0.2.2rc2`，Python >=3.10 | 依赖同版本 runtime-bin；发布列表含 macOS 14 arm64 wheel |
| Deep Agents / LangGraph | `deepagents 0.7.39`，Python >=3.11；`langgraph 1.2.22`，Python >=3.10 | 新 Deep Agents 原型需独立兼容环境，不能直接装进现有 Python 3.10 |

包元数据查询：[npm](https://registry.npmjs.org/@deepseek-ai/dsh/latest)、[DSH SDK PyPI](https://pypi.org/pypi/deepseek-harness-sdk/json)、[runtime PyPI](https://pypi.org/pypi/deepseek-harness-runtime-bin/json)、[Deep Agents PyPI](https://pypi.org/pypi/deepagents/json)、[LangGraph PyPI](https://pypi.org/pypi/langgraph/json)。不同分发渠道版本不相同，选型时应锁实际安装版本并再次核对能力。

DSH 对 Momo 的三种价值：

1. **单项能力复用。** 官方源码提供实验性 `cua-mcp` bundle，可向其他 Harness 提供电脑操作工具，不需要运行第二套模型循环。Momo 可先验证这个范围。
2. **任务执行候选。** Python SDK 使用 stdio JSON-RPC 子进程，支持会话结果和恢复接口；恢复历史不恢复外部 shell/浏览器进程。需要验证 Momo 的取消、流式事件和任务结果语义。
3. **产品直接替代候选。** 如果未来只需要现成通用助手，可以直接使用 DSH 桌面端；若仍需 Momo 的 Live2D、角色音色和交互，它更适合作为可选底层或能力来源。

SDK 运行自带 runtime，无须系统 Node；Node 22 不是所有 DSH 路线的阻塞。另一方面，官方 minimal 教程说明该 profile 的执行权限和日志上传默认值，初次对照应用合成材料与独立 home，按实际需求修改 profile，而不是直接挂接用户聊天库。此处依据具体默认行为，不扩展为额外审批体系。

来源：[官方网站](https://deepseek.com/harness/)、[仓库 README](https://github.com/deepseek-ai/deepseek-harness)、[架构](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md)、[Python SDK](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/python/sdk/README.md)、[minimal 教程](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/user/guide/python-sdk.md)、[Computer Use](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/subsystems/computer-use.md)、[CUA MCP](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/experimental/cua-mcp/README.md)。

### Deep Agents / LangGraph

Deep Agents 将规划、技能、文件上下文与可选子 Agent 放在 Harness 层，使用 LangGraph 的运行能力。LangGraph 将线程 checkpoint 与跨线程 store 分开，并提供中断/恢复。恢复会重跑节点，外部写操作仍需幂等或结果查询；换框架不自动得到恰好一次执行。

来源：[Deep Agents overview](https://docs.langchain.com/oss/python/deepagents/overview)、[Persistence](https://docs.langchain.com/oss/python/langgraph/persistence)、[Interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts)、[Memory](https://docs.langchain.com/oss/python/deepagents/memory)。

### MaiBot / Hermes 与上下文

- MaiBot 当前官方文档域名为 docs.maibot.chat，Bot/A_Memorix 页标记 2026-11-11 更新。表达快速档可直接注入样例而不额外调用模型；中文记忆仍使用 FTS5 与语义等通道。Momo 借鉴语义与模块边界，按已有失败选择复杂度。
- Hermes 继续采用有界常驻记忆与按需历史搜索，工具搜索是渐进暴露机制。这支持保留轻量索引/存储、改善读写策略，不构成“必须换图数据库”的理由。
- DSH 的 compaction 为可替换能力，Prompt 区分静态与动态贡献；这些可以作为运行接口的参照。

来源：[MaiBot Bot](https://docs.maibot.chat/manual/configuration/bot-config)、[A_Memorix](https://docs.maibot.chat/manual/configuration/amemorix-config)、[Hermes Memory](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/)、[Tool Search](https://hermes-agent.nousresearch.com/docs/user-guide/features/tool-search/)、[DSH compaction](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/subsystems/compaction.md)、[DSH Prompt](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/subsystems/system-prompt.md)。

本轮为文档、包元数据与 Momo 源码核对；未安装或运行候选框架，未获得它们在本机的性能/费用排名。

## 4. 本轮已落实的窄修正

### F-006 测试契约

实际复现 `test_state_event_prompt_parser_and_strip_contract` 失败：旧测试调用默认关闭社交事件的 `_render_layer_a([])`，却期望包含 `<state_event>`。当前 renderer 会依据 Persona.card_type 开启，output_adapter 测试也验证了该边界。

修正测试为显式 social enabled，并增加 assistant disabled 断言。未改变产品输出协议或弱化社交事件要求。F-006 仍待真机验收；这项自动契约不再作为悬而未决问题。

### Token 观测

- 每次主聊天模型调用生成独立 call_id，请求与响应按此配对，修正多个 stream 都从 turn_n=1 开始导致的歧义。
- `total` 改为完整 messages＋tools 估数，包含工具参数/结果与额外 system；失败或计数器禁用时为 null。
- 显式记录 estimator_model、estimate_status、找到的层标记；usage 缺失为 missing；响应返回的 model 单独记录。
- 图片估数使用 LiteLLM 的默认图片计数参数，避免观测代码为尺寸探测额外访问图片 URL；日志不记录消息正文。
- 新增 schema_version=2。历史固定估算模型仍用于可比估数，**尚未完成 dispatcher 实际模型计数、全部后台调用账本或费用计算**；响应模型名也不替代账户计价配置。

相关文件：[_token_probe.py](../references/SOURCE-REFERENCES.md#source-07475b223f)、[chat.py](../references/SOURCE-REFERENCES.md#source-6a784c600a)、[新回归](../references/SOURCE-REFERENCES.md#source-b565689fa3)、[工具循环接线验证](../references/SOURCE-REFERENCES.md#source-53ef7c768a)、[F-006 契约](../references/SOURCE-REFERENCES.md#source-4cebaa26ce)。

## 5. 验证证据

- 改动前重跑原定向入口：16/16 文件通过，结果见 [baseline JSON](../references/SOURCE-REFERENCES.md#source-37b5af7063)。
- F-006 原失败＋output_adapter：修正前 1 failed / 8 passed；修正后完整 F-006 文件与 output_adapter 共 26 passed。
- Token probe 新增 5 项通过；改动后 chat 分组 6/6 文件通过，见 [chat JSON](../references/SOURCE-REFERENCES.md#source-5246ea66cd)。其他分组运行文件未受本次运行时代码修改影响。定向入口现增加为 17 个文件。
- 本机已安装 LiteLLM 的实际计数器以合成纯文本离线验证：基础请求估数 20，加入工具调用/结果后为 233；未调用模型。确认完整请求路径可执行，不仅由 mock 返回固定值。
- 前端 `tsc --noEmit`、`tsc -b` 均通过；麦克风与 TTS 生命周期共 4 项通过。前端本轮没有新增运行时代码。
- 本轮改动 Python AST、文档相对链接与 `git diff --check` 检查通过。
- 未测真实应用进程树、模型费用、外部账户与真机语音/视觉；没有更新依赖、修改 config.yaml、commit 或 push。

原始命令与范围已记录在各结果及脚本中。没有把定向测试当作全仓回归，也没有将新框架文档能力当成 Momo 已实现能力。

## 6. 同日后续：先定义新版功能，再做技术选择

用户进一步明确按“功能 → 功能需求 → 技术栈与详细要求”推进，再交给 Astra 实施。当前主入口改为 [新版 PRD](product/PRD-momo-next.md)，只完成一级功能讨论稿；此前局部重建与组件复用建议作为候选，不能限制新版产品定义。

语音例子的补充核对：GPT-SoVITS 官方 README 当前列出 V5 与推理加速选项，API 源码支持多种流式方式和 gpt_streaming。Momo 的 gsv.py 仍主要读取完整 WAV response.content，TTSBase 主要返回完整 bytes。由此确定的调查项是“端到端流式接口与回合取消是否需要重建”；尚未验证新引擎在 M2 上的 RAM 或延迟收益，CUDA 路线不能直接等同于 Apple Silicon 收益。CosyVoice 官方也提供双流式能力，可在语音需求确定后作为对照。

来源（2026-11-17）：[GPT-SoVITS README](https://github.com/RVC-Boss/GPT-SoVITS)、[API 源码](https://github.com/RVC-Boss/GPT-SoVITS/blob/main/api_v2.py)、[CosyVoice 官方仓库](https://github.com/QwenAudio/CosyVoice)。这是选型线索，没有据此升级模型、依赖或现有语音服务。
