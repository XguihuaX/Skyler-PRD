# INV-33 · 旧能力盘点与重构技术更新

日期：2026-11-17。依据：本地当前工作区源码（含未提交内容）、既有功能台账及本日官方/作者资料。状态：只读核对与建议；未运行产品、安装模型、迁移数据或删除代码。全量功能验收状态仍见 [FEATURE-INDEX](product/FEATURE-INDEX.md)。

## 1. 新增决定与盘点方法

用户希望先识别上一代有哪些能力，再决定保留哪些、哪些重做或已被新版取代。已明确保留立绘馆和 Live2D，支持语音全/半双工与不同说话人区分的扩展方向，补全 Skills/MCP，前端重新设计，代码模块化并带技术文档。MaiBot/DSH 的可学习内容继续扩展。

以下区分产品能力、实现代码、用户数据/资产：保留一个功能不代表保留旧实现；新版技术存在不证明已经兼容或比旧版更适合。除用户已明确的方向外，处置意见均为建议，不能直接当删除清单。研究服务 [PRD v0.10](product/PRD-momo-next.md)，不在本文安排实施排期。

## 2. 上一代能力清单与建议

| # | 旧能力及当前证据入口 | 本轮核对到的情况 | 新版处置建议 / 确认状态 |
| --- | --- | --- | --- |
| 1 | [聊天与流式输出](../references/SOURCE-REFERENCES.md#source-6a784c600a)、[WebSocket](../references/SOURCE-REFERENCES.md#source-5220959ddd) | ChatAgent.stream 已有工具循环与回复；ws 同时处理附件、语音、持久化和中断 | 保留功能，拆清上下文、任务执行、输出与语音职责；具体实现待选 |
| 2 | [会话与历史](../references/SOURCE-REFERENCES.md#source-215f649c43) | 列表、读取、删除等接口与对应 UI 存在 | 建议保留能力与有用历史，重做呈现；迁移覆盖须另验 |
| 3 | [附件输入](../references/SOURCE-REFERENCES.md#source-5220959ddd) `_select_attachments`、chat `_user_content` | 已有图片/文件输入链路；不是完整个人知识库 | 建议保留输入、补齐成果交付；资料长期管理另定义 |
| 4 | [助手卡/社交卡与人设](../references/SOURCE-REFERENCES.md#source-d0ecf2a481)、[Prompt renderer](../references/SOURCE-REFERENCES.md#source-cc507e6cb0) | 卡片和模板已存在，legacy fallback 仍有调用 | 保留单角色设定需求；双卡入口是否需要待用户取舍；不原样迁移旧 Prompt |
| 5 | [记忆](../references/SOURCE-REFERENCES.md#source-521d60a11a)、[抽取](../references/SOURCE-REFERENCES.md#source-9abb47cb4b)、[画像](../references/SOURCE-REFERENCES.md#source-e9bcb1c142) | 近期、摘要、长期记忆及管理入口存在；有已记录召回/来源问题 | 分层按需回忆方向已确认；保留有来源数据，重做组织与读写边界；embedding 选择用质量/RAM 决定 |
| 6 | [MCP](../references/SOURCE-REFERENCES.md#source-9aa1d0cadd)、[工具注册](../references/SOURCE-REFERENCES.md#source-3a15c19267) | stdio/HTTP/SSE、工具注册、启停和确认存在；非文本结果被转 repr | 保留可用适配，补完整类型结果、按需工具与技能机制；不等于整生态已兼容 |
| 7 | [ASR](../references/SOURCE-REFERENCES.md#source-0bb602bd68)、[录音/VAD](../references/SOURCE-REFERENCES.md#source-1d1428b12c) | 当前音频片段转文本，segments 拼成字符串 | 保留能力、引擎可替换；说话人输出作为新增能力 |
| 8 | [语音打断](../references/SOURCE-REFERENCES.md#source-1d1428b12c) `micMuted` 分支 | Momo 说话时不录入正常语音段，独立能量阈值触发 interrupt | 保留打断目标，重做全/半双工回合策略；不能写成现有全双工 |
| 9 | [TTS registry](../references/SOURCE-REFERENCES.md#source-17ce0edace)、[GSV](../references/SOURCE-REFERENCES.md#source-96fccff601)、Fish/CosyVoice/Edge 等 | 多 provider 与音色接口存在，GSV 调用旧权重切换及 /tts | 保留接口思路和音色资产；逐 provider 验证保留/升级，v5 未完成兼容测试 |
| 10 | [立绘馆](../references/SOURCE-REFERENCES.md#source-038f12c77d)、[问候语音](../references/SOURCE-REFERENCES.md#source-afe2710097) | 角色资产、背景和语音资源有独立管理入口 | 立绘馆用户明确保留；问候语音是否继续作为独立功能待选择 |
| 11 | [Live2D 运行层](../references/SOURCE-REFERENCES.md#source-b2313a1a08)、[导入](../references/SOURCE-REFERENCES.md#source-dcfa1092d5) | 已有口型/动作与模型上传，运行链使用 Cubism 4；单图生成链未找到 | Live2D 用户明确保留；SDK、导入器和新生成产物的兼容需重新评估 |
| 12 | [App](../references/SOURCE-REFERENCES.md#source-e56cb91573)、[Panel](../references/SOURCE-REFERENCES.md#source-64bf1bc9fa)、[Widget](../references/SOURCE-REFERENCES.md#source-f8a57752c4)、[设置](../references/SOURCE-REFERENCES.md#source-faa2165cfd) | 主窗口、桌宠、设置和加载状态已存在 | 用户明确重设计前端；具体入口/视觉共同设计，继续使用真实运行状态 |
| 13 | [读屏](../references/SOURCE-REFERENCES.md#source-41ed4e9edc)、[剪贴板](../references/SOURCE-REFERENCES.md#source-ad254d3ef1)、[活动观察](../references/SOURCE-REFERENCES.md#source-4d7b270477) | 已有应用活动与读取能力，不证明通用鼠标键盘操作完成 | 按需读取与持续活动采集分别讨论；可用 MCP 适配替换具体实现 |
| 14 | [Cron](../references/SOURCE-REFERENCES.md#source-6fa6787fdc)、[早报](../references/SOURCE-REFERENCES.md#source-8bc8132372)、[日历](../references/SOURCE-REFERENCES.md#source-ba32f42b3c) | 有定时/主动触发与日历适配；休眠后行为未在本轮验证 | 归入任务与工作流；复用合适的日历适配器，重新定义调度/通知契约 |
| 15 | [模型/API 配置](../references/SOURCE-REFERENCES.md#source-65c004b576)、[LLM client](../references/SOURCE-REFERENCES.md#source-aa520db9ae) | 有 provider 配置和模型调用入口 | 建议保留配置能力，重做设置体验；旧默认模型不锁定新版 |
| 16 | [时间一致性 F-001](../references/SOURCE-REFERENCES.md#source-e035356118) | 台账待验收，有时间上下文实现 | 建议保留“时间理解正确”的要求，具体实现随新上下文层调整 |
| 17 | [DailyAgent 与角色世界](../references/SOURCE-REFERENCES.md#source-e3a9aa0b66)、生活线/分享/成长/FSM | F-002–006 多项待验收，主要服务社交角色 | 按既有决定后置；用户偏好学习不与虚构角色生活成长混为一体 |
| 18 | [上下文仲裁 F-007](../references/SOURCE-REFERENCES.md#source-8ec172e8c4) | 台账部分完成，有已知意图误判 | 保留一致性/自然交流目标，重新评估旧策略，不直接迁移为完成模块 |
| 19 | [网易云](../references/SOURCE-REFERENCES.md#source-b312a67321)、[播放](../references/SOURCE-REFERENCES.md#source-a93e5c9123)、[mpv](../references/SOURCE-REFERENCES.md#source-19d35e3c80) | 音乐与系统媒体能力有代码入口，真实服务未复测 | 可选插件候选；保留需求时再比较现有适配器与 Skill/MCP 覆盖 |
| 20 | [Bilibili](../references/SOURCE-REFERENCES.md#source-7a85e0a73b)、[小红书](../references/SOURCE-REFERENCES.md#source-b33fad2d1b)、[DOCX](../references/SOURCE-REFERENCES.md#source-ae1c3184d6) | 各有独立适配，当前网络/账号有效性未验证 | 候选按需能力，适合插件化或由成熟技能替换；有替代验证再退役旧实现 |
| 21 | [ComfyUI F-008](../references/SOURCE-REFERENCES.md#source-8c982d6e6f) | 独立连接器待验收，尚未暴露给角色工具 | 可选插件；图片生成路线确实需要时再评估，不自动列为核心 |
| 22 | [资源观测](../references/SOURCE-REFERENCES.md#source-e596941acb)、[Token](../references/SOURCE-REFERENCES.md#source-07475b223f)、[迁移](../references/SOURCE-REFERENCES.md#source-8a5be5df7b) | 已有观测、启动/关闭与数据迁移机制；backend RSS 不代表整机 | 保留工程要求并整合实现；不能因代码多删除历史迁移或关停清理 |

需要用户以后取舍的集中在：双卡入口、问候语音、持续活动采集、音乐和外部内容插件、ComfyUI，以及角色自主生活恢复的时机。立绘馆/Live2D 的保留方向不再重复询问。

## 3. 有依据的重构与退役检查点

- 本轮行数快照：`chat.py` 2359、`ws.py` 1580、`main.py` 959、旧设置组件 `SettingsPanelLegacy.tsx` 1636。问题依据是职责集中：聊天同时负责格式处理、上下文、工具和社交扩展；WS 负责连接、持久化、附件、音频队列和中断。行数仅用于定位，不作为删除阈值。
- [useAudio.ts](../references/SOURCE-REFERENCES.md#source-1d1428b12c) 的 `micMuted` 分支明确是播放时监听打断而非正常持续录入。ASR 返回 `str`；定向查询没有发现说话人分离/声纹识别实现。
- Live2D 当前导入器要求 `.moc3` 与 `.model3.json`，并按本地支持值拒绝更新的 moc3 版本；运行层导入 `pixi-live2d-display/cubism4`。这是旧运行链的约束，不是图片生成 Live2D 不可行的依据。
- GSV `synthesize` 在权重加载失败等路径可能返回固定参考音频。**建议重构时重新定义失败反馈**，避免与请求文本无关的音频被当作正常回复；本轮未改行为。
- 旧 `AlarmScheduler`、旧 todo 调度链、旧 `profile_summary` 接口已有退役记录，迁移时不要重新带回；实际调用仍存在的 Prompt legacy fallback 不因名称旧而删除。依据见 [INV-20](INV-20-legacy-path-evidence.md)，历史测试状态不能直接当作当前测试结论。
- 没有证据支持“React 18、FastAPI 或 SQLite 因年龄而必须淘汰”。选型应对应页面体验、模块边界、数据和资源需求。

## 4. 新语音路线核对

### GPT-SoVITS v5

官方仓库本轮核对主分支提交 `6d51ffceafbf240145619af4c92408a6e54b7924`（2026-11-11）。[README 的 V5 Release Notes](https://github.com/RVC-Boss/GPT-SoVITS/blob/6d51ffceafbf240145619af4c92408a6e54b7924/README.md#v5-release-notes)明确 V5 与 CUDA Graph/Flash Attention 加速；[官方变更记录](https://github.com/RVC-Boss/GPT-SoVITS/blob/6d51ffceafbf240145619af4c92408a6e54b7924/docs/cn/Changelog_CN.md)记载 2026-10-05 的 V5 更新。

判断：应纳入候选，不能继续只按旧版能力做设计。但 README 中的 V2 ProPlus RTF 测量不能当作 V5 首音频延迟，更不能直接套到本机 Apple Silicon。需区分音频生成吞吐、首音频、句间停顿、冷启动及 RAM/显存；检查旧 GPT/SoVITS 权重与参考音频的迁移方式。当前项目 GSV 适配仍以完整响应字节为主，升级模型之外还要验证流式交付。

### 全/半双工

[LiveKit turn handling](https://docs.livekit.io/agents/logic/turns/)提供 VAD、STT 端点、实时模型和手动回合等路径及打断控制，可作为行为与接口参考，不代表选定 LiveKit。产品支持全双工，不要求本机同时跑全部大模型。轻量采集/回声处理/播放与远端推理的组合可列候选，全本机路线另测；不能仅依据“支持 API”就承诺低延迟。

验证应覆盖耳机/外放、自身播放回声、短应答和真正打断、迟到的音频/识别结果、取消与下一轮衔接。本轮不为其设未测阈值，也不自动认定本机不可运行。

### 不同说话人

[sherpa-onnx 说话人识别](https://k2-fsa.github.io/sherpa/onnx/speaker-identification/index.html)提供本地识别候选；[pyannote.audio](https://github.com/pyannote/pyannote-audio)提供说话人分段/嵌入等路线。说话人分段回答“哪段是谁说的”，已登记身份匹配回答“是否是某个已知人”；同声重叠分离又是额外能力。用户当前只提出区分不同人的声音，具体范围和是否实时尚未确定。不能把身份识别自动升级为账户认证功能。

## 5. 图片到 Live2D

确有无需 Blender 的图像到可运行 Live2D 制作路线。以下为作者资料核对；用户没有指定具体项目，不能把某个候选当作用户已经选定。

| 路线 | 交付物与当前判断 | 依赖及边界 |
| --- | --- | --- |
| [image-to-live2d](https://github.com/lvhaojie456/image-to-live2d/blob/3c32db19ad28e60d5da84399792f960c4a524043/README.md) | 单图经过分层与绑定，生成 moc3/cmo3、纹理、物理和动作等；是可继续精修的初版路线 | 固定版本组合要求 macOS/Windows 制作端、Python/JDK、Cubism Core，分层另用 Linux + 24GB NVIDIA GPU 和相关 API；不能概括成轻量本机一键生成 |
| [See-through](https://github.com/shitagaki-lab/see-through/blob/fc4440db7c076518e313f378e28b1504e9886a95/README.md) + [PSD2Live](https://github.com/tsunehimatoi/psd2live/blob/d51bb5b730fc80f72ba5995707b9c443a65b488b/README.md) | 前者生成分层 PSD，不做 rigging；后者提供绑定/编辑与 moc3/model3.json/cmo3 导出，可组成完整生产链 | 分层阶段仍需评估 GPU；PSD2Live 当前支持多平台和不同 Cubism 目标版本，并提供 MCP，是技能工具集成候选。两项目组合未在本机验证 |
| [Live2D Agent Kit](https://github.com/Ariakage/live2d-agent-kit/blob/354d95800b7d498246dfb658e0e2ee379c29caa4/README.md) | 提供参考图/PSD 到模型、预览和打包的 Agent 制作工作流 | 作者验证环境含 Apple M4；仍需素材处理与配套依赖，不表示任意单图无需修整 |
| [Live2D 官方素材分离](https://www.live2d.com/en/cubism/download/material-separation-ps-plugin/)与[脸部半自动运动](https://docs.live2d.com/en/cubism-editor-manual/face-auto-edit/) | 官方辅助制作路线，可作产物与制作质量对照 | 分离工具依赖 Photoshop/Cubism 许可；非官方承诺的任意单图自动完成全模型 |
| [Anime2.5DRig](https://github.com/852wa/Anime2.5DRig) | 自有 WebGL 角色与动画路线，作为非 Cubism 对照 | 本轮未发现 moc3 导出，不可直接记为现有 Live2D 模型兼容方案 |

具体评估还需区分图片生成、素材分层、补画、绑定、导出与播放器运行。制作端较高算力不代表播放端必须承担同样成本；可在别的机器制作后只导入产物，本机播放 RAM 仍需实测。

PSD2Live 的新版本与 image-to-live2d 固定的旧版本依赖条件不同，不能混为一套环境。复用时逐项核对代码、模型权重与 Cubism 运行库许可；各项目许可证不同，不因外层仓库许可就推定整条链许可一致。此处是选型信息，本轮不执行安装或分发。

建议纳入候选能力：“从图片/分层素材生成、编辑及导入可运行角色模型”，制作工具通过 Skill/MCP 接入，展示运行时独立。是否在 Momo 内直接提供制作入口、目标美术质量和首版范围待用户决定。下一步技术验证应使用同一张角色图，检查生成文件、动作/口型、遮挡质量与选定渲染器兼容性。

## 6. MaiBot / DSH 更广的学习范围

| 参考来源 | 已有证据与可继续研究内容 | 对 Momo 的评估问题 |
| --- | --- | --- |
| MaiBot 聊天、人设和表达 | [Bot 配置](https://docs.maibot.chat/manual/configuration/bot-config)、[聊天场景](https://docs.maibot.chat/manual/chat-scenes)；既有自然感/表达学习研究保留 | 一对一连续消息、参与频率、表达学习如何落到可感知行为，避免额外模型调用无收益 |
| MaiBot 记忆与纠正 | [A_Memorix](https://docs.maibot.chat/manual/configuration/amemorix-config)、[记忆管理](https://docs.maibot.chat/manual/webui/memory-management) | 来源、跨会话范围、纠正与按需检索如何满足 Momo，不照搬所有组件 |
| MaiBot 插件与配置 | [插件开发](https://docs.maibot.chat/develop/plugin/)描述生命周期、注册、配置更新和能力接口 | 借鉴契约、健康状态与扩展方式；其独立 Runner 进程策略须按本机 RAM 需求评估 |
| MaiBot 观察与管理 | [统计与观察](https://docs.maibot.chat/manual/webui/statistics) | 如何让用户定位模型、工具和费用问题；是否适合新前端信息组织 |
| DSH 上下文压缩 | [Compaction](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/subsystems/compaction.md)将压缩作为可替换能力，记录范围与用量 | 借鉴压缩边界和可追查事件，评估是否保留任务继续所需信息 |
| DSH Prompt 与工具范围 | [System Prompt assembly](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/subsystems/system-prompt.md)定义贡献者、范围、工具 schema | 如何按当前任务组装、减少不必要说明，同时保留实际可用能力 |
| DSH Skills/MCP/电脑操作 | 已核对证据见 [INV-32](INV-32-skill-mcp-foundations.md) | 按需加载、类型结果、依赖和取消如何适配 Momo；完整主 Agent 不接入 |
| DSH 会话、模型与生命周期 | 列为进一步定向核对项；本轮未全量审查 | 研究事件、恢复、预算、资源释放与模型适配，按 Momo 需求选择机制 |

研究产出应记录“参考机制→对应需求→采用/调整/不采用→验证结果”。不把配置项数量或插件数量当成成熟度，也不把研究方向直接变成首版范围。

## 7. 技术文档与下一步取舍

用户已要求清爽模块化及技术文档。建议以一个功能切片交付代码、接口/事件与配置说明、依赖版本、启动/诊断、验证记录和必要的迁移说明；模块划分需避免重复状态和无需求的抽象。

当前先由用户查看 §2 的能力清单，表达仍需或不需的使用需求。已确认保留的功能继续进入 PRD；可替代能力先核对覆盖与产物，再形成退役决定。前端具体问题与新交互设计随后共同讨论，本轮未选择设计风格、替换 UI 或删除旧组件。

## 8. 2026-11-18 语音接口与 AnifLive-TTS 续查

应用户要求读取 [规划多平台新知学习汇总](codex://threads/01a1c835-cb58-7d21-904c-98e15ff43a0a) 及其本地 `lessons/speech.md`，定位项目为 [AnifLive-TTS](https://github.com/Hiruynk/AnifLive-TTS)。本次再经 GitHub 连接器读取主分支官方 API、基准、Mac 安装文档；未安装、未跑本机基准，主分支后续仍可能变化。

- [API](https://github.com/Hiruynk/AnifLive-TTS/blob/main/docs/API_EN.md)：支持模型/音色/表达能力发现，`POST /v1/audio/speech` 可返回完整 WAV 或原始 PCM 流，另有取消接口。采样率与支持参数应读返回元数据及能力目录，不由客户端硬编码所有模型相同。
- [基准](https://github.com/Hiruynk/AnifLive-TTS/blob/main/docs/BENCHMARKS.md)：视频的约 74.6 ms 对应历史 v1.3、RTX 5070 Ti、预热与持久连接条件下的可听首音 P50，不含设备播放延迟。当前 v1.5 v5Turbo 对应 Windows/RTX 5070 Ti 为 80.93 ms，M5 Max 为 404.69 ms（均为文档所列 keep-alive 条件）；是项目方基准，不能当作 Momo 从用户说完到回答的耗时。
- [Mac 条件](https://github.com/Hiruynk/AnifLive-TTS/blob/main/docs/MACOS_INSTALLATION_EN.md)：官方验证 M5 Max/128 GB，其他 Apple Silicon 与最低内存尚未验证，打包库声明 macOS 26.2+。Mac 流式在自回归 EOS 与完整 CFM 后由声码器分窗输出，不能套用 Windows 首音收益。ASR、模型回答、网络和播放缓冲仍需另外测量。
- 产品意义：可作为 GPT-SoVITS 语音输出的优化运行时候选，优先验证首音、长句稳定、取消后恢复、音色兼容和资源占用。它本身不提供整套 ASR/回合控制/全双工体验。

全双工与打断参考 [LiveKit 官方回合说明](https://docs.livekit.io/agents/logic/turns/)：其流水线模式支持回合检测、自适应区分短应答与打断，并提供误打断恢复；原生实时模型的打断设置受供应商行为约束，不能假定一套参数对所有引擎生效。Momo 的两条音频方向应独立管理，执行层与播放器需共享回复标识和取消状态。

MaiBot 界面依据本轮官方文档而非实机视觉验收：[配置管理](https://docs.maibot.chat/manual/webui/config-management) 是分节表单、模型与服务商管理、配置保存/生效状态等 WebUI；[QQ 语音通话](https://docs.maibot.chat/manual/adapters/qq-voice-call) 是社区适配器加独立媒体桥，复用 Bot 的人格和记忆。可以借鉴模块配置与状态反馈，不据此推定其页面就是适合 Momo 的桌面语音界面。

本轮交互草案见 [INTERFACE-001](product/INTERFACE-001-frontend-backend-contract.md)；字段和事件名未锁定，后续继续随二级需求补全。


## 9. 2026-11-18 Whisper / faster-whisper 语音输入评估

用户已确认普通话为主、少量英文；在比较后确认后续使用 faster-whisper 的 medium 与 large-v3-turbo 两个选项，默认值和运行参数后定。本轮为当前源码和上游文档核对，未安装、下载模型或运行识别测试。

- **当前源码：**`backend/asr/whisper.py:21` 导入 faster_whisper.WhisperModel，`requirements.txt:8` 声明 faster-whisper；现有 transcribe 处理一段音频，使用 VAD 过滤、拼接所有识别段、经 OpenCC 转为简体后返回字符串。本轮未读取或验证实际运行的模型大小和设备。
- **能力关系：**[Whisper 官方仓库](https://github.com/openai/whisper)提供多语言模型；[faster-whisper](https://github.com/SYSTRAN/faster-whisper)用 CTranslate2 实现 Whisper 推理并提供量化选项。它是同一模型家族的另一种推理实现；上游速度基准不直接代表 Momo 的短句端到端延迟。
- **已确认选项与验证建议：**保留 faster-whisper 路线，后续使用 medium 与 large-v3-turbo。用户此前反馈 small 有识别错误，medium 可能稍好但未确定够用；原先建议纳入完整 large-v3 的对照范围已按用户最新选择收敛。验证普通话识别、少量英文术语保留、数字和短句、静音误识别、从说完到最终文本的延迟，以及与其他模块共存时的资源占用。
- **连续识别：**Whisper 原始 transcribe 面向音频窗口；边说边显示需要额外的分块、上下文与文本稳定策略。研究入口 [WhisperStreaming](https://github.com/ufal/whisper_streaming)已注明其后续方向为 SimulStreaming。现有字符串返回不代表已有增量事件；全双工回合、播放取消和说话人区分仍由对应模块定义，不因采用 Whisper 而放弃这些目标。
- **Apple Silicon 注意点：**本轮 [CTranslate2 4.9 硬件文档](https://opennmt.net/CTranslate2/hardware_support.html)已列出实验性 MPS 后端，需显式 `device="mps"`，`auto` 仍选 CPU。旧 Momo 的计算类型选择仅区分 cuda 与其他设备，不能直接假定既有封装已适配 MPS；具体 faster-whisper/CTranslate2 安装版本与组合尚未验证。

采用状态：语言需求及 medium / large-v3-turbo 两个选项已确认；默认模型和运行参数后定，目标设备体验尚未验证。


同日准确率澄清：同一 Whisper 权重、音频和相近解码设置下，原版与 faster-whisper 预期准确率接近，不给两种运行实现排固定高低。上游要求比较时对齐 beam size 等选项，见 [比较条件](https://github.com/SYSTRAN/faster-whisper#comparing-performance-against-other-implementations)。[Turbo 模型卡](https://huggingface.co/openai/whisper-large-v3-turbo)说明其由 large-v3 剪枝和微调而来，以轻微质量损失换速度；这是模型变化，与 faster-whisper 的推理实现优化分开。用户随后将使用范围确定为 medium 与 large-v3-turbo，当前不扩展其他 ASR 候选。


## 10. 2026-11-18 旧 Momo 激活与持续收听核对

用户回忆旧方案为“唤醒＋持续监控语音”，要求承接。本轮仅核对当前源码，未启动语音或检索全部 Git 历史。

- 激活入口：[ChatInput](../references/SOURCE-REFERENCES.md#source-5d00f10db5) handleMic（约 285 行）与 [ControlBar](../references/SOURCE-REFERENCES.md#source-3b30e68df7) handleMic（约 35 行）在 VAD 模式调用 toggleVad。当前源码定向检索未定位唤醒词检测，不将用户“唤醒”直接改写为关键词唤醒已实现。
- 持续收听：[useAudio](../references/SOURCE-REFERENCES.md#source-1d1428b12c) toggleVad（523 行起）从 sleep 激活；onSpeechStart/onSpeechEnd（651/662 行起）自动分段，结束后回 active 并 sendVoice，无每句发送确认。实际监听为 Silero VAD；暂停时有麦克风/音频资源清理。
- 空闲策略：useAudio 503 行按 vadIdleTimeoutMs 暂停；[store](../references/SOURCE-REFERENCES.md#source-b8386abad6) 985 行默认 60000 毫秒。该数值仅为旧实现参考，不自动锁成新版要求。
- 记录与回复：[ws](../references/SOURCE-REFERENCES.md#source-5220959ddd) 841 行起接收 voice，855 行调用 ASR，861 行起持久化用户识别文本、885 行起回推 asr_result，随后进入 ChatAgent/TTS。语音文本已有持久化路径，新版可参考其消息关联。
- 旧版播放期间的收听另受 micMuted 等状态影响（useAudio 468 行附近）；该核对不覆盖新版已提出的全双工目标，也不把 VAD 视为唤醒词检测。

需求同步：PRD §5.4.4 与 INTERFACE-001 §2.3 承接激活后持续收听及自动分段处理；具体唤醒来源和新版参数后定。


## 11. 2026-11-18 唤醒优化与 DSH 插件调查

用户要求继续需求文档阶段，调查优化方式及 DSH 插件，随后确定唤醒可独立写成插件。本轮核对上游文档及部分源码；未安装插件、编写重构代码或运行麦克风测试。此前“当前源码”指旧 Momo 参考实现。

| 插件与证据 | 已有机制 | 对 Momo 的意义 |
| --- | --- | --- |
| [qishuilalala/dsh-voice-mode](https://github.com/qishuilalala/dsh-voice-mode)，树 eace2a1116d4dc3dce68d647455125e55cd1f5b3 | 持续聆听、停顿发送、朗读与打断；zipformer2 + SenseVoice。子包 README 明确启用唤醒词后每句结束或打断后需重新唤醒 | 优先参考会话控制与取消。src/wakeword.ts 是转写文本匹配；src/asr.ts 540 行附近保留唤醒词与正文连说的音频。需要适配持续交流策略、已选 faster-whisper 和自有前端 |
| [supersyh-sss/dsh-voice-assistant](https://github.com/supersyh-sss/dsh-voice-assistant)，树 079efc64e2cc437f481629acf4a7123a387abedb | 浏览器 sherpa-onnx WASM / Web Speech；唤醒后持续听写到草稿，默认 30 秒无语音回待命，支持“执行发送” | 可参考唤醒/空闲状态，lib/client.js 226/408 行附近可见定义。README 将多轮自动分段提交列为未完成，不能当作现成自动接话方案 |
| [tangzheng202202/dsh-voice-live](https://github.com/tangzheng202202/dsh-voice-live)，当前 README | 宿主 WebSocket 连接火山流式 ASR/TTS；事件携带 voiceSessionId、utteranceId、sequence，取消使旧消息失效 | 可参考协议。语音服务不同于已选本地 ASR；限制节明确 TTS 期间门控麦克风、采用半双工加打断，唤醒使用 Web Speech |

这些插件主要接入 DSH Web UI，移入 Momo 两种前端仍需适配，当前未选完整语音插件。目录另收录 Miyamiz39/dsh-koein，但本轮 GitHub 仓库及 README 请求均返回 404，仅保留线索，不列为已核验候选。

**优化建议，未锁定实现：**

- 激活状态与单句识别分开，激活后持续交流；退出与空闲策略单独定义。
- 若用喊名字唤醒，评估 [sherpa-onnx 官方 KWS](https://k2-fsa.github.io/sherpa/onnx/kws/index.html)：提供自定义关键词、中文及中英模型，无需为每个词重新训练。待机检测关键词，激活后 VAD 分段并调用 medium / large-v3-turbo，减少完整 ASR 推理次数。实际资源收益待测，停止推理不等同释放模型内存。
- 统一断句与发送等待，避免机械叠加静音窗口；保留短音频前缓冲，承接“唤醒词＋正文”连说，时长后定。
- 结果用统一发言标识关联聊天记录；取消或切会话后，迟到结果不串入新会话。双工及打断继续单独细化。

**用户最新决定：**唤醒按独立插件定义。内部算法、部署位置、是否直接采用 DSH 插件加载机制尚未锁定；先留启停、配置、输入、状态与唤醒事件契约，见 INTERFACE-001 §2.3。模块化有利于插件化，成为可接入插件还需符合宿主的注册和生命周期约定，不要求所有内部模块都独立分发。


## 12. 2026-11-18 GPT-SoVITS v5 与句内情绪核对

**触发：**用户已有自行微调的 GPT-SoVITS 模型，指出说话方式应取决于 TTS，询问 v5 是否支持句内级别情绪。当前仅查官方资料和源码，未加载用户模型或试听。

**官方依据：**

- [官方版本 Wiki](https://github.com/RVC-Boss/GPT-SoVITS/wiki/GPT%E2%80%90SoVITS%E2%80%90features(Latest%E2%80%90Including%E2%80%90v5))：v5 包含 dev/turbo；侧重音色相似度、声码器和推理加速；v5 的少样本微调尚未研究验证。官方区分整合包版本与实际 GPT/SoVITS 模型版本，更新整合包不等于旧权重变成 v5。
- [api_v2.py](https://github.com/RVC-Boss/GPT-SoVITS/blob/main/api_v2.py)：当前 TTS_Request 提供 text、ref_audio_path、prompt_text、速度及采样参数等，未见 emotion/style 或逐片段情绪时间线字段。prompt_text 是参考音频的转录文字，不是自由风格指令。
- [TTS.py](https://github.com/RVC-Boss/GPT-SoVITS/blob/main/GPT_SoVITS/TTS_infer_pack/TTS.py)：沿参考音频/文本提取提示与语音特征；检查未发现显式句内情绪标签控制路径。V3/V4/V5 使用有参考文字的推理流程。

**结论范围：**根据本次官方接口与文档，不能将 v5 记为已支持可指定各文字片段情绪的原生句内控制。合成自然带情绪、参考音频引导整体表达、显式指定句内变化是不同能力。若后续需要，可试验同角色的不同情绪参考音频预设和分段合成，实际情绪还原、音色一致性及衔接需试听；该方案未锁定。

**需求影响：**优先承接用户已有微调模型资产，具体兼容性后续按版本和权重组合检查；不默认升级、重训或更换音色。表达功能依实际 TTS 能力开放，移除先让用户选定固定情绪能力的前提。见 PRD §5.4.6 / INTERFACE-001 §2。


## 13. 2026-11-18 承接旧版 TTS 库管理

用户要求按之前版本维护 TTS 库。本次检查本地源码，未启动服务或验证远程模型。

- `backend/routes/tts_models_api.py:1`、`:38`：模型 CRUD 与权重、参考目录、默认情绪、参数、启停字段；旧实现说明数据库管理主要用于 GSV，其他引擎仍有静态配置，不据此要求新版沿用混合存储。
- `backend/tts/gsv.py:381`：从情绪标签选择参考项，构造 ref_audio_path 与 prompt_text 请求；情绪参考库并非待从零探索的新能力。它是参考音频路由，不代表原生句内情绪控制。
- `backend/routes/tts_api.py:638` 附近：参考项列表、本地标注文字上传；远端 wav 上传仍有占位，不能声称旧版已完整实现远程素材上传。
- `frontend/src/components/character/VoicePicker.tsx:332`、`:381`、`:627`：角色引擎/模型/音色选择和试听界面。
- `docs/adding-new-tts-model.md` 为旧注册表流程参考；其中静态 JSON 新增模型步骤早于现有数据库 CRUD，不能覆盖当前源码。

新版需求承接库管理、角色选择和试听，细节按各 TTS 实际能力提供，接口见 INTERFACE-001 §2。用户自有模型纳入库；具体素材上传、存储与迁移后定。


## 14. 2026-11-18 语音插件即时复核：直接采用、二开还是自做

用户要求现在确认有没有，而非只留下将来调研。本轮在线读取官方仓库 README、树和关键源码，未安装或实际录音。

### 优先评估的两项

1. **[qishuilalala/dsh-voice-mode](https://github.com/qishuilalala/dsh-voice-mode/blob/main/plugin/dsh-voice-mode/README.md)**，树 `eace2a1116d4dc3dce68d647455125e55cd1f5b3`，README 当前标 v0.7.39（2026-11-18）。持续收听、静音分段/发送、字幕、按句播放、VAD 打断、短应答让话均已有；新增 OpenAI 兼容远程 TTS。
   - `plugin/dsh-voice-mode/src/tts-openai.ts:135`：实际发送 model/input/voice/response_format，可选 instructions，具备远程声音服务接入点；不等于原生支持 GPT-SoVITS 的模型/参考素材库。GSV 原生 /tts 协议不同，需已有兼容服务或适配。
   - `src/client.tsx:2662`：hardBreak 清播放、作废在途音频并取消运行回合。`:2702` 的 resumeOnce 是恢复收音，不能据此认定能从原文播放位置续读。
   - `src/client.tsx:1492`：短应答让位期间会丢弃部分音频帧，随后继续播放；不是保留全部未读内容的暂停/续播。
   - 当前仍为 zipformer2/SenseVoice；启用唤醒词后每句回待机，朗读期 VAD 开口即打断。已选 faster-whisper、持续激活、后台判断说话对象、插问后续读及 Momo 自有前端仍需适配/验证。
2. **[ZG2017/dsh-plugin-voice-loop](https://github.com/ZG2017/dsh-plugin-voice-loop)**，树 `8183183a858366c56a2c37dea3757ae8100efb8d`。已有连续语音循环、自适应噪声阈值、停顿提交、打断、分别配置本地/远程 ASR 和 TTS。
   - [src/index.ts](https://github.com/ZG2017/dsh-plugin-voice-loop/blob/main/src/index.ts):149：ASR 上传 file/model，接收 JSON text；:315：TTS 发 model/input/voice/lang_code/response_format，接收音频。若 faster-whisper/GSV 服务提供相应协议，可经此接入；本轮未验证具体服务兼容性。
   - [src/client.js](https://github.com/ZG2017/dsh-plugin-voice-loop/blob/main/src/client.js):1576：打断后放弃当前回合、暂停音频并恢复收音；:2754 调宿主 sessions.cancel。须区分停播与取消正在执行的任务。
   - ASR 整段提交，回复完成后整段合成播放，非逐句流式。没有查到按陪伴场景判断说话对象或保存播报位置续读的现成实现。全屏界面不作为 Momo 前端设计约束。

### 次选与判断

[PerryLink/dsh-talk README](https://github.com/PerryLink/dsh-talk/blob/main/README.md) 已列语音输入、VAD、打断、speak 和事件播报；识别支持 Web Speech/FunASR/whisper.cpp，合成支持 browser/edge-tts/piper。对已选 faster-whisper 与自有 GSV 接入没有比上述两项更直接的已查优势，本轮未继续源码审计，不列为首选。

**当前判断：**基础语音回合已有可复用插件；本轮没有找到可直接覆盖全部 Momo 要求的现成方案。优先以 voice-mode 的回合控制与 voice-loop 的可换服务方案作为两个独立候选验证，不预先混合两套实现。候选能直接满足的采用；替换少量接口/策略即可满足的二开；若必须重写核心流程、持续与宿主耦合冲突且维护代价高，再采用成熟 ASR/TTS/VAD 组件自行实现 Momo 控制部分。此处为技术建议，未锁定插件或批准实现。
