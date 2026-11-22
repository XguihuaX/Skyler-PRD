> 旧 MomoOS-v2 项目材料快照；其中实施状态和路径按原工作区理解。新需求范围见 [主 PRD](docs/product/PRD-momo-next.md)，执行顺序见[任务清单](docs/product/SKYLER-TASKS.md)。

# Skyler Roadmap

This roadmap separates the code currently published on GitHub from the
maintainer's local validation batch. A documentation update does not make an
unpublished implementation part of the public baseline.

Status definitions:

- **Published baseline**: the code path exists on the current GitHub `main`.
- **Local validation**: implementation and automated evidence exist in the
  maintainer's working tree, but the corresponding code has not yet been
  published as one reviewed implementation batch.
- **Ready for human acceptance**: automated review has passed; product or real
  conversation acceptance is still required.
- **In progress / partial**: the foundation exists, but the behavior, safety,
  or product loop is not complete.
- **Planned**: a design direction only. Do not describe it as current behavior.

## Product feature roster

Detailed product requirements and acceptance evidence remain in the local
`docs/product/` workspace for now. Raw conversation logs are intentionally not
part of this public documentation sync.

| ID | Feature | Local R&D status | Current GitHub baseline |
|---|---|---|---|
| F-001 | 时间一致性与对话重返 | Ready for human acceptance | Not published in this batch |
| F-002 | 社交世界锚点与 DailyAgent | Stage 2A + activity refinement; C-005～C-007 pending | Stage 1 only |
| F-003 | 持续生活线 | Ready for human acceptance | Not published in this batch |
| F-004 | 分享储备与主动分享 | Ready for human acceptance | Not published in this batch |
| F-005 | 体验驱动的偏好成长 | Ready for human acceptance | Not published in this batch |
| F-006 | 角色状态机第一版 | Ready for human acceptance; social/assistant prompt contract reconciled on 2026-11-17 | Not published in this batch |
| F-007 | 上下文仲裁与回复自然度 | Partial; known intent-routing false positives; needs human decision | High-level foundation only |
| F-008 | ComfyUI 独立连接器 | Connector ready for human acceptance; not role-facing | Not published in this batch |

## Current active work

Agent navigation: [Project architecture and feature map](docs/PROJECT-MAP.md).
Current audit: [INV-29](docs/INV-29-global-architecture-review-2026-11.md).

### 2026-11-22 · Skyler 首轮验证准备

现行需求为 [PRD v0.95](docs/product/PRD-momo-next.md)：基于麦麦主程序、按需委托 DSH。用户新增确认先验证出差前交给 Mac、返程交回主机（功能明细 §5.12），尚未实测或开发。后续建议顺序见 PRD §7；最近任务是主机原版私聊、重启回看与 Mac 同一主人会话续聊，按[验收清单](docs/product/SKYLER-5070TI-ACCEPTANCE.md)留证据。原生记忆/日程基线之后验证基础交接，再接 DSH、语音和各业务场景。正式前端后续共同设计。

下方 11 月 17–18 日的“当前”“最新”保留为当时记录，现行范围与建设顺序以上述入口为准。本轮仅文档更新，未安装、实施、提交或推送。

### 2026-11-17 · 架构评估与助手主线

用户最新确定采用自上而下的顺序：**产品定位 → 功能定义 → 功能需求与验收 →
技术选型与架构 → 详细设计 → Astra 分批实现**。
当前入口是 [Momo 重构 PRD](docs/product/PRD-momo-next.md)，正在功能需求定义阶段。
最新 v0.23：按用户要求撤回受 DSH 社区项目影响的需求重排，保留十项一级能力及已确认需求。DSH 插件加外置项目作为自主建设方向另记于 ARCH-001，未选社区母项目；版本、模块与接口待定。未开始实现、迁移或退役旧功能。
v0.19 明确至少两类设置：能力启停、模式/模型/角色声音选择；对应 PRD §5.10.1 与接口草案 §1.1，仍在需求阶段。
2026-11-18 / v0.18：二级定义同步前后端交互契约，语音模块可配置、说话人区分可开关；[接口草案](docs/product/INTERFACE-001-frontend-backend-contract.md) 已建立，正式 UI 后置，未进入实现。
2026-11-18 / PRD v0.17：对话与理解专题留到最后，当前进入语音二级功能范围（§5.4.1），候选规则待共同讨论。
最新 v0.15 确认后端优先、简陋测试前端配合验证，正式前端重设计后置；仍先完成二级功能定义与技术设计。
v0.16 开始对话与理解二级草案（PRD §5.11）：八个候选子功能及行为样例待讨论，未转为实施承诺。
PRD v0.11 补入后续多角色共同会话与彼此交互目标，首期单角色优先；未启动多角色实施或选定框架。
PRD v0.12 明确主动性、渐进学习与可开关感知的一级范围（§5.8），具体情景和规则进入后续二级定义。
PRD v0.13 补入长期个人资料库、任务驱动/持续操作和在线持续工作要求（§5.9），未开始实现或确定游戏技术路线。
最新主线为需求驱动重构，代码精简归入重构过程；用户自然描述需求，
`bmad-agent-pm` 沿用现有文档整理补缺，不重启访谈。提醒与持续跟进纳入任务与工作流；
PRD v0.14 将一级能力整理为十项，设置与用户控制承接角色选择、能力配置与技能开关，角色 Prompt 随角色管理；二级功能与首版范围待细化。当前未启动重构代码实施。
用户新增明确保留立绘馆/Live2D、前端重设计与模块化技术文档要求；语音全/半双工和说话人区分进入扩展需求。
[INV-33](docs/INV-33-legacy-capabilities-and-technology-refresh.md) 提供旧能力取舍与新技术核对，当前没有迁移或退役代码。
[INV-30](docs/INV-30-momo-capability-landscape-2026-11.md) 已完成本轮功能可能性调研：
36 份官方资料、15 个讨论主题、62 个二级候选；首版范围待共同确定。
[ARCH-001](docs/product/ARCH-001-runtime-renewal.md) 的此前比较作为技术候选资料；
旧UI和代码不作为新设计的硬约束，复用程度后定。DSH 接入方向保留，插件/外置项目的具体组合未安装或验证。

| 切片 | 状态 | 交付 / 下一步 |
| --- | --- | --- |
| 全局审计、能力抽取、三条路线比较 | 本轮完成 | ARCH-001 + INV-29；推荐局部核心重建，保留整体重搭选项 |
| 已确认观测与测试问题 | 本地修正已验证 | Token call_id/完整请求估数/unknown 状态；F-006 卡片边界契约 |
| 新版产品定位与功能地图 | **当前阶段；能力调研完成** | INV-30 候选清单已形成；共同确定产品重心、真实任务与首版范围，邮件等仍为候选 |
| 功能需求与验收 | 功能范围确定后 | 正常/异常流程、体验指标、数据与权限、用户可观察验收 |
| DSH版本/接入及模块选型 | 自主建设、具体组合待定，尚未实施 | 先细化产品需求，再确定插件/外置模块职责与接口；当前不写代码 |
| 记忆服务与首个任务迁移 | 选型和任务明确后 | 可靠写入、逐条来源、中文检索与任务恢复；避免叠加两套执行所有者 |
| DSH 电脑操作能力复用 | 独立候选 | 验证 CUA MCP 的图片、权限询问和取消适配 |

F-002 C-005～C-007、F-003～F-006 的社交验收与 F-008 连接器验收保留原状态，
不作为助手运行时评估的前置依赖。高级角色自治、更多应用连接器、图记忆和全双工
仍为候选，只有具体场景或对照结果支持时才进入当前切片。

## Published baseline

### Desktop UI

- Full main panel for chat, history, character management, capability settings,
  and app configuration.
- Transparent desktop-pet window for low-friction companion presence.
- Live2D rendering, model scanning, model management, framing, idle motion,
  blink / breathe, and lipsync hooks.

### Interaction

- Text input.
- Voice input through Silero VAD or manual recording.
- ASR, LLM streaming, TTS, and Live2D expression hooks.
- Image and file input for the current model turn.
- Attachment inputs are not yet a mature long-term memory system.

### Character system

- `character_personas` schema and active variant.
- Structured Persona fields such as identity, personality core, speech style,
  relationship, voice samples, forbidden phrases, taboo topics, lore, and
  capability overrides.
- Persona loading into the character prompt.
- `card_type` foundation for social cards and assistant cards.
- Assistant-card specific fields are present in the prompt path, but full
  runtime policy separation is still in progress.

### State and memory context

- `CharacterState`: mood, intimacy, current thought, current activity, and
  interaction timestamps.
- `<state_update>` write-back from model output.
- Short-term conversation window.
- Conversation summary.
- Long-term semantic memory.
- User profile context.
- Activity context.

### Activity context

- ActivityWatcher records foreground app, browser URL/title, document path, and
  activity sessions.
- ActivityTimeline writes local `activity_sessions`.
- Activity summaries can be injected into the ChatAgent prompt.
- Early proactive triggers exist, with idle gate, throttle, daily cap, and active
  conversation guard.

### DailyAgent Stage 1

- Daily plan generation.
- Current-activity ticker.
- State write-back into `current_activity`.
- This is Stage 1, not a complete FSM.

### Desktop perception

- Read-only AX / UI Tree summary for the current foreground app.
- The default strategy is on-demand foreground-app reading, mainly to control
  token use, latency, and privacy risk.
- The current implementation should not be described as continuous monitoring of
  all windows.

### Capability layer

- `CapabilityRegistry`.
- Native capability registration.
- MCP client for external tools.
- MCP server exposing Skyler capabilities.
- Per-tool switches.
- Provider replacement for LLM / TTS paths.
- Confirm-gate framework.

## Local validation batch

The following work has implementation and focused automated evidence in the
maintainer's local workspace. It is not part of the current public code baseline
until the corresponding implementation is reviewed and published separately.

### Temporal, DailyAgent, and activity refinement

- F-001 computes elapsed-time, cross-day, and conversation-return context.
- F-002 Stage 2A adds per-character world anchors, local time, real-weather
  snapshots, structured daily schedules, current/next activity runtime, restart
  recovery, and social-card isolation.
- DailyAgent still creates coarse schedule events. Event-level activity
  refinement selects a controlled kind, variant, detail, and phase plan shortly
  before an event starts.
- Activity phases advance from persisted real-time boundaries; the model cannot
  directly declare a phase complete or write an arbitrary activity value.
- The current product gate is three independent human conversations verifying
  that a two-minute follow-up cannot turn “preparing food” into “already cooked
  and finished eating.”

### Character state and continuous life

- FSM v1 maintains energy, valence, and arousal through deterministic reducers,
  event records, bounded changes, decay, and restart catch-up.
- The model may emit a controlled interaction event; it cannot directly set FSM
  scores or intimacy.
- FSM state currently changes how a social character expresses herself. It does
  not choose or rewrite the DailyAgent plan.
- F-003 tracks verifiable cross-day reading, watching, and creative-project
  progress. Planning an activity does not count as completing it.
- F-004 builds traceable sharing candidates from verified experiences.
- F-005 builds specific, reversible preferences from evidence without rewriting
  the core Persona.

### Context and media infrastructure

- F-007 has a production-connected evidence/action decision card, but known
  question-versus-action false positives remain and the feature is partial.
- Topic-level memory, on-demand multi-hop retrieval, and RAG routing are design
  requirements only; they are not implemented.
- F-008 is an isolated, mock-tested ComfyUI connector. It is not registered as a
  role tool and does not implement role-triggered image generation or sharing.

### REF-001 code slimming (project-wide staged review)

2026-11-17 follow-up:

- Rechecked the original isolated subset: 16/16 files pass. Added token-probe
  coverage; the updated chat group passes 6/6 files, bringing the runner to 17.
- Reconciled the F-006 test with the existing social-only output contract;
  full FSM and output-adapter files pass 26 tests. Human acceptance remains open.
- Request/response observation now has unique call IDs; complete request estimates
  include tool results, and unavailable estimates/usage stay distinguishable.
- Frontend default/build TypeScript checks and four audio lifecycle tests pass.
- Architecture renewal is tracked in ARCH-001, separate from completed slimming.

2026-10-13 engineering follow-up (INV-27):

- Completed task ownership and exception-safe service cleanup, including partial
  MCP initialization failure. Boot marks, migrations and appReady gates preserved.
- Added an explicit isolated regression runner: 16 reviewed files pass across
  lifecycle/memory/chat/tools; old retired tests retained as source artifacts.
- Memory extraction dedup reuses at most 256 successful encodings per batch.
  Controlled 200-history × 10-candidate calls fall from 2010 to 210, with the
  same decisions; actual model latency improvement remains unmeasured.
- Removed four unreferenced DailyAgent helpers and one unreachable tail; existing
  runtime/Prompt contracts pass. Compatibility parameters remain in place.
- Updated PROJECT-MAP; machine graph meta marks the prior snapshot stale and
  lists drift for the next refresh. No full application or real-device acceptance.

2026-10-13 navigation and measurement follow-up:

- Added a maintained feature-to-code/data/test map and entry links for incoming
  agents; refreshed the generated knowledge graph against the working tree.
- Fixed backend CPU polling losing its prior sample by constructing a new
  psutil Process on every poll. Four isolated regression checks pass; an actual
  process busy-loop sample now reports nonzero usage after the first poll.
  This corrects measurement and is not a claim of reduced application CPU use.
- Corrected DailyAgent module summaries that still described a single-character,
  database-only Stage 1 path; current dispatch and refinement are documented.
- Test classification and failure cleanup now have the scoped gate above.
  Remaining measurement candidates: history restore as conversation count grows
  and process-tree resource use. Memory retrieval/extraction and dynamic tool
  selection still need behavior evaluation before changing defaults.

Automated implementation and focused checks cover several main-path cuts,
not the whole project. Real-device voice acceptance and current-production
latency measurement remain open. See `docs/INV-23-project-slimming-scope.md` for
the remaining project-wide hotspots and evidence boundaries.
Tool filtering and removal of live compatibility paths require separate behavior
decisions and are outside this no-behavior-change batch.

- The startup migration sequence now lives in `backend/database/run_migrations.py`;
  `main.py` keeps the original boot mark. Chat context assembly shares one
  conversation-tail builder across renderer and legacy paths.
- The WebSocket attachment filter has one independent boundary with focused
  regression coverage. Turn persistence and TTS ordering remain on their existing
  paths pending broader integration evidence.
- Frontend playback releases active and queued audio on interrupt, touch, and
  unmount. Late microphone permission grants are stopped after ownership changes.
  TypeScript and controlled audio lifecycle checks pass; real-device voice
  acceptance remains open.
- Live2D lip sync now writes amplitude directly to the runtime. Its sampling
  loop starts only while TTS audio sources are connected, so an idle canvas no
  longer creates an output AudioContext or runs its own animation-frame loop.
  Frame-time and memory savings still need real-device measurement.
- The capabilities and settings overlays now load on first open. A matched Vite
  production build reduced the entry JS from 1,459.54 kB (gzip 415.32 kB) to
  1,213.63 kB (gzip 355.09 kB). Startup and first-open latency need device
  measurement; the first open shows a loading fallback while its chunk arrives.
- Character management, system panel, and character gallery now also load on
  first open. The same build's entry JS is 1,073.58 kB (gzip 319.58 kB), a
  total reduction of 385.96 kB (gzip 95.74 kB) from the original entry.
- MCP client transports now prepare concurrently during boot; capability/tool
  registration waits in config order to keep the model-visible schema order.
  A controlled 3 × 80 ms comparison measured 243.2 ms before and 81.3 ms after;
  current three-client warm local runs measured serial 3.28/2.88 s versus
  parallel 2.20/2.09 s, with all three clients connected each time. Full app
  boot and appReady timing remain to be measured. Boot marks and appReady
  semantics are unchanged.
- The unused `profile_summary` frontend client is removed. Chat legacy assembly
  and database compatibility paths remain where runtime or migration calls exist.
- Renderer and legacy chat paths now share the stage-1 proactive sentinel check;
  its prompt behavior is covered by the existing integration checks.
- Correction turns skip a read-only summary lookup whose value was discarded;
  normal turns still read and inject summaries. The local lookup baseline was
  about 0.30 ms, without an end-to-end latency claim.
- Renderer profile, activity, and memory reads stay serial. A trial of parallel
  reads improved a synthetic delay test, but two same-database paired rounds
  did not show a stable practical gain; the trial was reverted. The prompt
  behavior matched in all 13 paired runs.
- Completed and interrupted WebSocket persistence now share one reply sanitizer,
  preserving the interrupted path's whitespace rule and warning wording.
- Efficiency is part of REF-001: repeated work, latency, token cost, and idle
  resource use need matched before/after measurements. Tool-schema cost has a
  historical baseline (see `docs/INV-21-slimming-efficiency-baseline.md`); the
  larger token and embedding hotspots still need controlled comparisons.

## In progress

- Human acceptance and publication planning for the local character-system batch.
- Mood-aware flexible behavior selection beyond FSM v1; FSM v1 itself does not
  control DailyAgent scheduling.
- Multi-character DailyAgent scheduling.
- Runtime strategy split between social cards and assistant cards.
- Better context arbitration across persona, memory, activity, screen context,
  user message, and tool results.
- Window Roster: lightweight app / window / PID roster.
- Watchlist: user-authorized or context-relevant targets.
- Target-PID AX / UI Tree deep read when a watched target still exists, changes,
  or the user asks about it.
- Screenshot or local-crop visual understanding when structured UI information
  is not enough.
- End-to-end validation for dangerous tools and write-action confirmation.
- Credential handling and dangerous-tool governance.
- Long-term persistence and memory consolidation for image / file inputs.
- More complete Persona content for multiple characters.
- Release packaging and update flow.

## Planned

- AX + visual model fusion.
- Continuous visual perception with clear privacy and consent boundaries.
- Multi-character collaboration.
- Long-term autonomous behavior.
- Live2D AI director: expression / motion / action planning.
- Character asset generation pipeline.
- ComfyUI workflow integration.
- Stronger memory architecture and regression tests.
- DMG packaging and auto-update.

## Demo wording boundaries

Safe to say:

- Skyler has a full panel and desktop-pet window.
- Skyler has text, voice, image, and file input for the current turn.
- Persona, `card_type`, and `CharacterState` are connected.
- ActivityWatcher / ActivityTimeline provide local activity context.
- AX / UI Tree is read-only and on demand for the current foreground app.
- MCP / API capabilities are connected through a registry-style capability layer.

Do not say:

- DailyAgent is a complete FSM.
- Skyler continuously reads every window.
- Full VLM screen monitoring is done.
- Write actions are fully safe end to end.
- Social / assistant cards already have complete runtime policy separation.
- Image / file inputs are fully consolidated into long-term memory.
- Multi-character collaboration or long-term autonomy is complete.

## Suggested next order

Use **Current active work** above as the single current order. ARCH-001 owns the
runtime comparison and migration criteria. Earlier social-first and QQ/Watchlist
sequences are historical proposals, superseded by the user's architecture review
and daily-task discovery request on 2026-11-17.
