# 麦麦怎样记住人，又怎样保持自己的说话方式

2026-11-19，面向 Momo 首轮「记忆与人格」的需求讨论。核对时 GitHub 最新正式版为 **MaiBot v1.4.7**，标签与当时的 main 均指向 `b7f36825463413c47e61d44fe8a93427f6fe5a6f`。正式版于北京时间 11 月 19 日 01:15 发布，发布说明主要涉及语音、日程和适配器。本次重新读取该版本的源码、中文提示词、配置、官方设计说明和相关测试用例，并实际调用了一项不依赖模型的记忆衰减函数。自然对话、写入准确性和整机资源占用仍待实测。[正式版](https://github.com/Mai-with-u/MaiBot/releases/tag/v1.4.7) · [固定源码树](https://github.com/Mai-with-u/MaiBot/tree/b7f36825463413c47e61d44fe8a93427f6fe5a6f) · [上一版整体架构](INV-39-maibot-v1.4.6-overview.md) · [旧 Momo 记忆对照](INV-38-maibot-memory-and-learning.md)

## 1. 从一句话看全程

假设用户先说「周五要做汇报」，隔天又说「汇报改到周一了」。麦麦有几处机会接住这件事。当前聊天的原文能让她马上接话；较早内容可以压缩为话题、背景、进度、未完事项和约定。开启长期记忆且达到写入条件时，后台可以把新计划及变更整理进摘要。下次用户问「上次那场汇报呢」，规划回复的模型可以按需查往事，再决定答什么。如果用户同时建过提醒，提醒有自己的日程记录和修改入口。一次聊天回答正确，只能说明这一轮用到了足够的信息；要判断长期更正是否成功，需换会话并检查实际召回。这个例子用于解释代码路径。[近期对话压缩](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/context/compaction.py#L48) · [写回时机](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/services/memory_flow_service.py#L1042) · [私聊回忆指引](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/prompts/zh-CN/instructions/memory/planner_memory_private.prompt) · [日程工具](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/builtin_tool/schedule.py)

## 2. 麦麦究竟记什么

| 内容 | 来源与用途 |
| --- | --- |
| 原话和当前进展 | 聊天记录保留原始消息；运行中的对话使用近段原文和滚动摘要。启动恢复历史时，代码会根据距最后一条可恢复消息的实际间隔提示怎样续聊。[接收与落库](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/chat/heart_flow/heartflow_message_processor.py) · [恢复间隔](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/runtime.py#L451) |
| 人物事实 | 发送回复后，后台可以找出相关的已知人物，收集该用户本人的发言及上下文。模型提出事实，宿主把实际取到的证据消息 ID 随事实交给写入路径。同一窗口可能含多条证据，追溯时应看原话。[人物事实写回](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/services/memory_flow_service.py#L148) |
| 增量摘要和关系 | 另一条队列按聊天流处理新窗口，提取新增或明确变化的信息、涉及对象和关系。中文提取规则要求保留计划、考虑、否定和原始说话人，相对日期按原消息时间换算。玩笑、转述、机器人建议各有筛选或归属规定。输出可以为空。[摘要调度](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/services/memory_flow_service.py#L574) · [提取提示词](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/prompts/zh-CN/main/memory/a_memorix_chat_summary.prompt) |
| 一段经历 | Episode 在后台从来源段落中划分事件，按来源跟踪和重建。它支持「那次发生了什么」的查找。上游测试覆盖来源变更后复用未变段落、重算新增部分，以及无新增事实的情况。[事件分段](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/A_memorix/core/utils/episode_segmentation_service.py) · [增量用例](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/pytests/A_memorix_test/test_chat_summary_episode_incremental.py) |
| 对一个人的认识 | 画像以人物 ID 为入口，结合名字或别名、关系证据和检索到的文本，形成有版本的画像。私聊自动参考当前用户；群聊可选近期发言者、被提及者、被回复者。自动注入路径在没有画像正文时不会拿未确定候选补位。[画像服务](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/A_memorix/core/utils/person_profile_service.py) · [注入](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/memory/person_profile.py#L197) · [群聊选人](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/scenes/group/memory.py) |
| 图片 | 收到消息时有单独的图片写回队列，包含任务记录、失败重试和描述补偿；规划回复时也有图片记忆参考入口。桌面游戏中持续观察屏幕、判断角色当时实际看见了什么，是 Momo 还需定义的输入场景。[图片写回](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/services/memory_flow_service.py#L768) · [规划时的三类参考](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/reasoning_engine.py#L282) |

**对共同经历有一个实质影响。** 用户和角色昨天一起笑过的梗，可能同时包含三件事：一次值得记住的互动、一种以后可以模仿的说法、以及用户在当时的反应。麦麦的事实摘要会过滤玩笑，表达学习则可以提取梗的用法。两条路径都存在，依然可能丢掉「为什么这个梗对我们俩有意义」。这是 Momo 值得优先用真实对话检验的差异。[事实筛选](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/prompts/zh-CN/main/memory/a_memorix_chat_summary.prompt) · [表达提取](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/prompts/zh-CN/main/learning/learn_style.prompt)

## 3. 下次怎么想起来，什么情况下会想错

- **主动查往事。** 私聊的行动指引鼓励在「上次」、先前承诺、共同经历或长期偏好影响回复时查询。工具支持主题、时间、主题加时间、事件、聚合五种查法；底层可结合文本向量、BM25 关键词和图关系。指定人物重名且无法确认时返回错误。已确定人物但查无命中时，一条后备路径会撤掉人物和时间过滤，改用关键词，保留聊天范围并标识发生过降级。涉及「是谁、哪天」的回答需要重新核对证据。[查询入口](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/builtin_tool/query_memory.py#L200) · [范围与命中过滤](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/A_memorix/core/runtime/services/memory_search_service.py#L19) · [检索实现](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/A_memorix/core/retrieval/dual_path.py)
- **带入背景。** 人物画像可以在规划前自动带入。另一个独立选项先用模型概括最近聊天形成的印象，再按这个印象寻找往事，受新消息数、最短间隔、缓存和范围限制。有这项机制才有机会在用户没说「还记得吗」时联想到旧事；选择主动查询也能实现部分场景。[画像与联想的调用点](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/reasoning_engine.py#L282) · [联想触发](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/memory/heuristic_injector.py#L58)
- **跨聊天范围。** 记忆、表达、黑话可以分别加入「相通组」。共享的是读取范围，来源仍归原会话。A 与 B 相通、B 与 C 相通，A 不会因此读到 C。身份合并有独立的预览与确认机制。Momo 的 ASR 和文字若本来就属于同一次对话，不需要仅因换了输入方式就划出两个群组。[共享规则](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/docs/sharing-groups.md) · [人物资料合并](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/person_info/individual.py)
- **事件时间、事实有效期、记忆衰减各管一件事。** 可以查「周五说过什么」，也有事实记录的生效与失效时间；口语里的「半小时」能否稳定写成有效期仍要实测。后台还有关系保留强度随时间变化的机制，包括访问强化、冻结、归档及相关保护。查得频繁能影响保留强度，不由此增加原事实可信度。[事实时间筛选](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/A_memorix/core/storage/metadata_fact.py#L872) · [维护服务](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/A_memorix/core/runtime/services/memory_maintenance_service.py) · [保留强度与置信度用例](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/pytests/A_memorix_test/test_memory_lifecycle_policy.py#L395)
- **更正与遗忘。** 用户当场改口、后台写入新值、停用旧证据、刷新画像和重建事件，各有生效时机。管理端可以预览自然语言修改方案、执行和回滚，也可按来源处理删除与恢复。另一条反馈纠错路径需要先有召回命中，再观察后续反馈，并根据配置与置信度处理。验收「忘了它」要观察之后还能在哪些地方被查到，包括原始聊天与派生材料。[管理入口](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/webui/routers/memory.py) · [反馈纠错](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/A_memorix/core/runtime/services/feedback_correction_service.py#L683) · [上游回滚用例](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/pytests/A_memorix_test/test_feedback_correction_core.py#L270)

## 4. 麦麦的人格实际在哪里起作用

| 位置 | 它影响的行为 |
| --- | --- |
| 人设预设 | 有名字、人格与身份、行为风格、表达风格、群聊与私聊提示、备用风格及概率。WebUI 可以预览、编辑和保存，导入后由用户选择是否切换。运行时切换在当前实例的所有会话生效，已经开始的回复可继续生成。[导入格式](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/docs/personality-import.md) · [切换语义](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/docs/personality-sdk.md) |
| Planner 决定如何参与 | 规划输入包含人格和行为风格，也有会话场景提示、工具以及本轮取得的画像或回忆。它可以回复、查资料、等待或收尾。比如角色的「不硬找话题」在这里可以影响是否开口；实际执行还取决于会话调度和模型。[规划输入](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/chat_loop_service.py#L850) · [私聊行动模板](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/prompts/zh-CN/main/planner/default/private/planner_default_private.prompt) · [群私聊差异](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/docs/chat-scenes.md) |
| Replyer 决定怎样说 | 普通私聊路径读取名字、别名和人格，再结合近期对话、回复对象、当前时间、表达风格及本轮指导生成正文。备用风格按配置概率临时替换基础表达风格。这里的回复指导是宿主组装的输入；B24 那种把角色视角写进模型推理前缀的实验需单独验证。[人设与风格选择](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/chat/replyer/maisaka_generator_base.py#L109) · [回复组装](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/chat/replyer/reply_prompt.py#L318) · [私聊正文模板](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/prompts/zh-CN/main/reply/private/reply_private.prompt) |
| 表达随使用变化 | 表达学习从真实发言里抽取「某种场景里，有人这样说」的样例，带来源，过滤角色自己的话，可配置来源黑名单与审核。使用端有快速加权抽样和向量召回后用模型筛选两档；Mai Replier 专用回复路径跳过表达习惯块。选用哪种回复模型，会改变学到的表达能否进入最终输入。[学习器](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/learners/expression_learner.py) · [表达选择配置](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/config/official_configs.py#L4107) · [模型适配](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/docs/replyer-prompt-profiles.md) |
| 听懂黑话 | 单独提取词语，用语境推断含义，匹配后可给规划模型提供参考。「知道我们说的词指什么」与「她也学着这么说」可以独立控制。普通聊天积累的是可检索、可管理的资料；本次追踪的路径里没有看到它按轮重写当前人设或更新模型权重。[黑话学习](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/learners/jargon_miner.py) · [本轮匹配](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/maisaka/jargon_context_matcher.py) · [管理端表达审核](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/webui/routers/expression.py) |

因此，同一个「周一的汇报」能供不同环节使用：记忆给出前情，规划决定现在适不适合提，表达决定她是认真提醒还是顺着你的玩笑聊下去。一个人被系统标为认识，或者两个人物在图里有关联，都不能单独推出角色已进入某种亲密关系阶段。本次追踪的核心路径中，没有找到通过统一好感分按等级切换人格的依据。[人物资料](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/person_info/individual.py) · [人设切换](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/config/personality_service.py)

## 5. 哪些行为默认就有，哪些需要开启

以下是发布源码的默认值。某个实例保存过的配置和实际模型选择会改变运行结果。[记忆集成](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/config/official_configs.py#L1304) · [总开关](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/config/official_configs.py#L1832) · [事件和画像](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/config/official_configs.py#L2994) · [学习配置](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/config/official_configs.py#L4107)

| 配置 | v1.4.7 默认与实际含义 |
| --- | --- |
| A_Memorix 长期记忆 | 总开关关。下列记忆子项的默认值要在长期记忆可用时理解。 |
| 人物事实、聊天摘要写回 | 两项都开。摘要消息触发阈值与配置窗口均为 36；达到调度条件也可能提取不到新信息。人物事实另走独立路径。 |
| 事件生成、画像、画像注入、查询工具 | 子开关开。确认对象、有效来源和可用数据仍影响这一轮能否取得内容。 |
| 根据聊天印象自动联想、跨聊天联想 | 默认关。主动用工具查询与自动联想分别控制。 |
| 管理端自然语言修改、聊天反馈纠错 | 前者可用，自动执行默认关；后者默认关。管理端能改不代表用户每次随口纠正都会自动传播到各层。 |
| 表达学习和使用 | 缺省学习配置项把 learn 和 use 都设为开，可以按聊天来源调整。写入前 AI 审核默认开，仅限人工精选默认关，向量精细选择默认关。与长期记忆分开配置。 |

## 6. 复用边界与验证依据

宿主用 Git subtree 同步 A_memorix 的 MaiBot 专用分支。库负责记忆存储、检索和维护；MaiBot 宿主提供会话、人物身份、写回时机、模型任务、工具和界面。单独引入记忆库时，需核对 Momo 对这些入口的接入能力。上游还单列了适用于 MaiBot 的许可说明，可在实际选定复用代码时核对对应版本。[同步说明](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/docs/a_memorix_sync.md) · [宿主调用](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/services/memory_service.py) · [随版许可说明](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/A_memorix/LICENSE-MAIBOT-GPL.md)

本次单独运行了 v1.4.7 的关系保留强度函数。设初始强度为 1、半衰期 24 小时、冻结阈值 0.1，无新证据或访问强化时，第 0、24、72、96 小时分别得到 1、0.5、0.125、0.0625，最后一次判定为不活跃。它说明代码确实按经过时间计算这项强度，无法据此估算用户的偏好在真实对话里多久会被忘记。[实际调用的函数](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/src/A_memorix/core/utils/memory_lifecycle_policy.py#L96)

另行阅读了上游关于写回、事件增量、纠错回滚、共享范围和表达选择的测试用例；未运行该套件或端到端模型服务。原作者的实现意图、已追到的调用路径、本次函数检查与用户最终体验应各按相应证据判断。[写回用例](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/pytests/A_memorix_test/test_person_memory_writeback.py) · [范围用例](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/pytests/test_sharing_groups.py) · [表达选择用例](https://github.com/Mai-with-u/MaiBot/blob/b7f36825463413c47e61d44fe8a93427f6fe5a6f/pytests/maisaka/test_replyer_expression_selection.py)

## 7. 回到 Momo，先定义哪几件事

**已经确认的方向。** 首轮以麦麦相关的完整 Prompt 为底稿，择取旧 Momo 的特有规则并提炼后融入。近期交流连贯、远期按需回忆；用户一侧的时间仍需照顾。Life 首轮后置，DailyAgent 后置，角色推理前缀另行对照。具体 Prompt、长期记忆组件和自动化默认值在各自环节确定。[产品定义](product/PRD-momo-next.md) · [旧 Momo 对信息来源的处理](INV-16-current-prompt-and-persona-field-map.md)

| 待共同定义的二级能力 | 用来判断是否做对的场景 |
| --- | --- |
| 记住用户与记住共同经历 | 两人曾为一场翻盘笑过。隔天提「昨天那场」能回忆到具体经历；一次笑过不足以断定用户每次输局都想听吐槽。 |
| 写入、改口与时效 | 「以后都这样」与「今天这一次」影响范围不同。把汇报从周五改到周一，下次应能找回新计划；若用户已经建了提醒，另查日程是否需要变更。 |
| 回忆的时机与来源 | 用户说「还记得吗」时可以主动查。用户随口接一个梗时，是否允许自动想起，再看打扰程度和效果。用户转述或邮件里的经历，要能说清是谁说的。 |
| 角色怎样决定开口、怎样表达 | 用户忙着操作时可以先听；用户过来开玩笑时能接话；用户改问如何导出录音时能转而帮忙。先描述体验，后端搭好再试不同 Prompt。 |
| 聊久之后什么可以变 | 分别观察对用户的了解、共同回忆、听懂的词和学来的说法；是否允许角色自身形成新的长期习惯单列讨论。 |

建议从第一行开始。它会决定记忆需要保存哪些材料，也会决定人格能拿什么依据来回应；余下四行可以在此基础上依次讨论。

## 8. 公开部署中实际发生过什么

2026-11-19 复核，最新正式版仍为 v1.4.7。以下是官方仓库收录的用户实机反馈，版本随案例标注。本次检索未找到可核验为默认 v1.4.7 的完整跨天陪伴对话。第 1、7 节的汇报和游戏案例是 Momo 的讨论场景。

| 场景与来源 | 观察到的行为 | 对定义 Momo 的意义 |
| --- | --- | --- |
| 多人接梗，1.3.19 定制部署 | A 讲自己有六个账号被封，麦麦先接住了这句话。B 随后调侃麦麦，麦麦回 B 时把六个账号算到了 B 身上。提交者查过消息和模型请求，回复对象正确，这轮也没有注入共享记忆。| 回应对象与经历的主人要分别检查。桌面同样会有用户转述和外部来信。[脱敏聊天和请求核查](https://github.com/Mai-with-u/MaiBot/issues/2445) |
| 一句提及变成长期关系，1.3.19 | 用户提到麦麦的主人伤心，提交者发现其人物画像把这位说话者记为主人，且该信息进入下一步规划。| 验证提及、转述、本人自述如何进入人物画像。v1.4.7 的写回路径仍把本次取到的用户证据消息 ID 一并保存；该版对此例的实际回答未测。[部署记录](https://github.com/Mai-with-u/MaiBot/issues/2366) · [v1.4.7 写回](https://github.com/Mai-with-u/MaiBot/blob/v1.4.7/src/services/memory_flow_service.py#L192) |
| 一对一长期陪伴，1.3.9 个人分支 | 部署者报告，带具体多步说话指导时，回复容易固定为回应、补充、追问。试着去掉措辞指导后，已确认模型输入发生变化，但自然度没有稳定胜过原来。文中聚餐、下雨等例子是归纳或合成说明。| 验收人格时应看整段交谈，特别是角色能否自然收尾。[私聊部署反馈与实验边界](https://github.com/Mai-with-u/MaiBot/issues/2215) |
