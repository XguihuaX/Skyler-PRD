# INV-22 · DailyAgent × RP 相关研究定向综述

> 调研日期：2026-10-02\
> 问题：在不微调、可调用商业 API 的条件下，角色日常规划与运行状态怎样影响长期角色扮演对话？\
> 范围：2023–2026 年角色代理、日常行为模拟、长期对话和 RP 评测。优先核对 ACL Anthology、ICLR Proceedings、arXiv 原文页面。\
> 性质：定向文献扫描，不是穷尽式系统综述；“未见”只表示本轮检索未发现，不构成绝对新颖性证明。

## 1. 本轮结论

1. **日程生成和角色行为模拟已是成熟方向。** Generative Agents 已实现记忆、反思、计划和行动循环；D2A 用动态需求选择日常活动；Ella 把结构化日程、环境观察、重排和对话接在一起。因此，单独宣称“让角色自主过一天”或“把日程注入聊天”难以构成新方法。
2. **动态 RP 和推理时规则也已有强先例。** Dynamic Persona Coherence、PersonaForge、PHASE-Tree 都研究长对话中的可变状态；MRPrompt 研究角色知识的选择、边界和表达。新增一组普通 Prompt 规则需要与这些方法直接比较。
3. **时间与经历的真实性已有专项研究。** TimeChara 测角色在剧情时间点的越界知识；LoCoMo 用时间事件图构造长对话；TReMu 研究多会话时间推理。不能把“时序一致性”本身当新问题。
4. **现有评测正在转向多轮、动态场景和用户目标。** PersonaArena、RMTBench、Persistent Personas、LifeSide 分别覆盖动态社会环境、多轮用户意图、超长对话退化和长期陪伴。只用单轮角色事实题，不足以支持 DailyAgent 对 RP 的效果。
5. **本轮最值得检验的交叉问题**：在真实墙钟驱动的虚构角色生活里，把“计划、模拟进行中、记录为完成、未知”的状态及其来源交给对话层，是否比同等信息量的普通日程提示更能减少错误经历声明、跨时段矛盾，并改善角色自然度？这是待验证的研究假设，不能预先称为无人做过的贡献。

## 2. 最接近的日常代理研究

| 研究 | 已做到什么 | 与 Skyler 的关系和边界 |
|---|---|---|
| [Generative Agents](https://arxiv.org/abs/2304.03442), UIST 2023 | 记忆流、反思、每日计划、环境感知、行动和对话；用消融验证行为可信度。 | 最重要的日常代理先例。发生在 Smallville 模拟世界，论文重点是代理行为与社会涌现。不能再把“计划—执行—反应”本身当新架构。 |
| [Lyfe Agents](https://arxiv.org/abs/2310.02172), arXiv 2023 | option-action、异步自检和压缩记忆，目标是低成本实时社会互动。 | 对“每天少量 API 调用、其余状态确定性推进”的成本方案有直接启发。需避免把低调用成本单独包装成首次贡献。 |
| [Simulating Human-like Daily Activities with Desire-driven Autonomy / D2A](https://proceedings.iclr.cc/paper_files/paper/2025/hash/513cb685f67550dbd133b81a7a24249f-Abstract-Conference.html), ICLR 2025 | 动态需求／价值系统驱动自主候选活动与选择，在 Concordia 中测活动合理性和变化性。 | 与未来 mood/need-aware DailyAgent 高度相关；主要评估活动选择，没有直接回答日常状态如何改变长期 RP 对话。 |
| [Ella: Embodied Social Agents with Lifelong Memory](https://arxiv.org/abs/2506.24019), arXiv 2025 | 3D 社区中的多模态情节记忆、结构化日程、环境观察、计划修订和交谈。 | 是最接近“日程＋执行反馈＋对话”的先例。其重点是具身社区与记忆能力；不能声称此前没有把日程和对话连接。当前按预印本引用。 |
| [iPET](https://aclanthology.org/2025.acl-demo.40/), ACL Demo 2025 | 虚拟宠物的对话、记忆和世界模拟。 | 证明陪伴产品的模块组合已有系统展示先例；若仅展示产品集成，应按 Demo 论文定位。 |

## 3. 最接近的 RP 方法研究

| 研究 | 已做到什么 | 与 DailyAgent × RP 的区别 |
|---|---|---|
| [TimeChara](https://aclanthology.org/2024.findings-acl.197/), Findings ACL 2024 | 在指定剧情时间点检验角色是否知道当时不该知道的事实。 | 直接覆盖“时间点角色幻觉”；其时间轴来自既有故事，不是开放式、持续推进的角色当日生活。 |
| [Beyond Static Persona Consistency](https://aclanthology.org/2026.acl-long.1336/), ACL 2026 | 稳定身份与动态心理状态分层，自动评估与修正；跨 GPT-4o、Claude、DeepSeek 测试。 | 动态人设和无须训练权重的闭环改进已有先例。重点在心理／情绪轨迹，不是计划与模拟执行的证据边界。 |
| [PersonaForge](https://aclanthology.org/2026.findings-acl.386/), Findings ACL 2026 | 心理结构、动态情绪／关系状态与选择性 Think-then-Speak；50 轮轨迹和消融。 | 推理时角色规则与分层架构并非空白；它检验人格漂移，未以日常活动状态为主要变量。 |
| [Memory-Driven Role-Playing / MRPrompt](https://aclanthology.org/2026.findings-acl.1175/), Findings ACL 2026 | 将角色知识的锚定、选择、边界、演绎拆为可测能力，并提出提示架构。 | 与“角色只说有依据的经历”最接近；若提出证据规则，需说明它如何处理随时间变化的日程／活动，而非只是换一种知识检索。 |
| [PHASE-Tree](https://arxiv.org/abs/2608.06975), arXiv 2026 | 不变身份根与可变的 persona/session/moment 状态树，配长程演变评测。 | 已覆盖多时间尺度的角色状态演化；按预印本引用。重点是剧情对话状态，不是墙钟、日程中断与模拟事件推进。 |
| [Character is Destiny](https://aclanthology.org/2025.findings-emnlp.813/), Findings EMNLP 2025 | 测人格如何影响选择，并用 persona 记忆检索改进。 | 如果研究“日程选择符合性格”，需要与此类 persona-driven decision work 区分。 |

## 4. 应直接借鉴的评测研究

| 研究 | 对实验设计的启示 |
|---|---|
| [InCharacter](https://aclanthology.org/2024.acl-long.102/), ACL 2024 | 心理问答测角色人格忠实度；可作辅助，不足以代替跨时间对话。 |
| [LoCoMo](https://aclanthology.org/2024.acl-long.747/), ACL 2024 | 用 persona 与时间事件图构造最长 32 个会话的长对话；可借鉴事件图和跨会话测试。 |
| [TReMu](https://aclanthology.org/2025.findings-acl.972/), Findings ACL 2025 | 时间轴摘要与符号时间计算改善多会话时间推理；应单列时间事实错误，不能把所有错归为“RP 感不强”。 |
| [RAIDEN](https://aclanthology.org/2025.coling-main.735/), COLING 2025 | 定向多轮对话与成对比较，比单条绝对打分更适合 RP。 |
| [RMTBench](https://aclanthology.org/2025.findings-emnlp.730/), Findings EMNLP 2025 | 把用户动机纳入 RP 评测；角色日常不能挤掉用户当前真正的需求。 |
| [Persistent Personas?](https://aclanthology.org/2026.eacl-long.246/), EACL 2026 | 在超过 100 轮的长对话中观察到角色忠实度下降，尤其目标导向对话；应测多轮退化与指令跟随。 |
| [PersonaArena](https://aclanthology.org/2026.findings-acl.471/), Findings ACL 2026 | 动态社会环境、状态更新和多轮角色评测；不能宣称“动态环境 RP 评测”是空白。 |
| [ChatAnime](https://aclanthology.org/2026.acl-long.2179/), ACL 2026 | 用户中心陪伴 RP 同时评估基础对话、角色扮演和情绪支持；证明“角色像不像”不能吞掉用户体验。 |
| [LifeSide](https://arxiv.org/abs/2606.04660), arXiv 2026 | 多会话记忆、情绪、环境循环的陪伴评测；其持续世界主要围绕用户，应与“角色自身日常状态”区分。按预印本引用。 |
| [Rethinking Role-Playing Evaluation](https://aclanthology.org/2026.sigdial-1.15/), SIGDIAL 2026 | 知名角色名称会泄露模型预训练知识；应使用原创／匿名角色和未见场景。 |

## 5. 对当前项目的具体判断

项目现状以 [DESIGN_LITE.md](../DESIGN_LITE.md) 和 [ROADMAP.md](../ROADMAP.md) 为准：公开基线有每日计划、ticker 和 `current_activity` 写回；本地 Stage 2A 已实现世界锚点、真实天气快照、结构化日程、当前／下一活动、重启恢复及事件细化，但仍待真实对话验收。自适应重排、旅行、mood-aware 活动选择等仍是后续工作。设计文档明确将计划与实际模拟行为分开，且不把模型一句话视为事件完成证明。[角色机制](design/character-mechanism.md)；[F-002](product/F-002-social-world-dailyagent.md)。

这意味着当前项目具有一个可做研究的基础，但不能把已经实现的产品模块直接等同论文贡献。最需要先验证的不是“会不会排日程”，而是：

- **因果效果**：有 DailyAgent 的 RP 是否真的优于相同 token 预算的“Persona＋日程文本”？
- **状态证据**：计划、临时覆盖、模拟推进、已完成与未知如何影响回答里的经历声明？
- **双目标**：减少时间／经历错误时，角色感、回应用户需求与自然度是否保住？
- **泛化**：结果是否跨原创角色、跨日、多模型家族稳定，而非只对一个角色或单一模型有效？

这四个问题目前是**候选研究问题**，不是本轮文献已证明的差异性。若后续找不到稳定差异，适合以产品系统或失败分析为产出；若有强消融和跨模型效果，才适合进一步凝练方法论文。

不微调的 API 方法有发表先例，但效果未必在每个模型上一致。例如 MRPrompt 的补充实验覆盖 GPT、DeepSeek、Qwen、GLM 等 API 模型；其中强模型 Qwen3-Max 上的总体增益并不明显。跨模型结果应逐个报告，不宜只展示平均数。

## 6. 检索记录与限制

- 检索日期：2026-10-02。
- 检索入口：ACL Anthology、ICLR Proceedings、arXiv 的原文／论文页；使用 `role-playing`, `persona`, `daily schedule`, `generative agents`, `long-term dialogue`, `temporal consistency`, `companion` 等组合。
- 选择原则：优先与 DailyAgent × RP 直接相交、正式发表、有可核对方法与评测的研究；关键新预印本单独标注。
- 本轮未统计全部命中数、未做双人筛选或正式 PRISMA，因此不能称为系统综述。产品竞争格局和论文投稿策略不在本轮范围。
