# INV-38 · MaiBot 记忆与学习机制核对

日期：2026-11-19。状态：官方文档及关键源码研究，未运行。用户希望适当学习 MaiBot，尚未决定 Momo 自动长期记忆、写入阈值或纠错默认方式。

## 1. 核对依据

- 官方仓库已从 MaiM-with-u 重定向至 [Mai-with-u/MaiBot](https://github.com/Mai-with-u/MaiBot)。本轮源码固定提交 `59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1`。
- [A_Memorix 配置](https://docs.maibot.chat/manual/configuration/amemorix-config)、[记忆管理](https://docs.maibot.chat/manual/webui/memory-management)、[Bot 配置的学习系统](https://docs.maibot.chat/manual/configuration/bot-config)，页面标注更新 2026-11-18。
- 关键代码：`src/config/official_configs.py`、`src/services/memory_service.py`、`src/maisaka/builtin_tool/query_memory.py`、`src/A_memorix/core/runtime/services/feedback_correction_service.py`、`prompts/zh-CN/main/memory/a_memorix_chat_summary.prompt`。文档说明、配置默认值与运行效果分别判断。

## 2. 实际机制

| 方面 | 核对结果 | 对 Momo 的启发，尚未确认采用 |
| --- | --- | --- |
| 自动积累 | 长期记忆总开关默认关闭；启用后人物事实写回和聊天摘要写回的子开关默认开启 | 不应把记忆简化为每次手动说“记住”；是否默认启用由产品决定 |
| 提取内容 | 增量摘要提示词筛选新增/明确变更且未来有用的信息，要求去重、保留计划/考虑/否定，不把临时安排泛化为长期偏好 | 学习有来源、保留状态的事实提取规则 |
| 内容组织 | 摘要、事实、关系、情景记忆和人物画像分别组织；情景记忆后台分段生成，画像随证据更新 | 先明确“发生过的事”和“对用户的长期认识”各自用途，不直接照搬全部存储组件 |
| 回忆 | 主动查询工具与人物画像注入分别控制；根据聊天印象自动拉起记忆为另一个默认关闭选项 | 结合已确认的按需回忆方向，分清稳定背景与本轮需要的往事 |
| 更正 | 管理页自然语言修正生成方案、确认执行、可回滚；查询后观察用户反馈的自动纠错是独立默认关闭功能 | 接住当前对话更正、更新长期记忆、失效旧证据是不同环节，需一起验证 |
| 删除 | 管理页可按来源抹除并恢复，涉及共享引用和回收站 | 需定义用户所说“忘记”的实际范围；目前不能承诺聊天一句话即彻底清除所有副本 |
| 表达学习 | 与事实记忆独立；学习和使用分别开关，可配置来源与审核 | 学到用户的表达方式，不等于把听到的话都当事实，也不等于模型微调 |

源码定位：`official_configs.py:1525`、`:1538` 的人物事实/摘要写回默认 true，`:1581` 的模糊修正自动执行默认 false，`:1633` 的反馈纠错默认 false，`:1832` 起长期记忆总开关默认 false。`feedback_correction_service.py:683` 起入队方法在功能关闭时直接返回 disabled，故不能把聊天自动纠错描述成开箱默认效果。

提取提示词还明确：引用/转发与发送者自述分开，机器人建议不能单独证明用户事实；按消息日期理解相对时间；排除要求不记的内容；没有新增信息则输出空结果。这是实现意图，真实准确性需实测。提示词中的“不要再写旧值”也不等于已经将历史索引中的旧信息全部失效。

依据：[固定版本提取提示词](https://github.com/Mai-with-u/MaiBot/blob/59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1/prompts/zh-CN/main/memory/a_memorix_chat_summary.prompt)、[默认配置](https://github.com/Mai-with-u/MaiBot/blob/59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1/src/config/official_configs.py)、[反馈纠错实现](https://github.com/Mai-with-u/MaiBot/blob/59cc6a9c4ff1e7a52f1885c5c458a4af3ea219b1/src/A_memorix/core/runtime/services/feedback_correction_service.py)。

## 3. 本轮需求结论

用户确认的是参考 MaiBot 的研究方向，不是认可助手此前提出的自动长期记忆默认方案。保留既有近期连贯、远期按需回忆、可追溯来源方向；先借鉴信息筛选、人物认识、表达学习、回忆和修正的机制，再逐项讨论 Momo 行为。不要复刻群聊共享、固定消息阈值或整套管理页。

下一步可用少量案例评议需求：明确未来计划、临时安排、稳定偏好、引用别人经历、否定/更正、要求不记。后续比较插件或实现时，用这些案例检查实际写入和再次召回，而非只检查提示词或界面存在。当前未选择 A_Memorix 作为 Momo 依赖，未启动测试。

## 4. 与旧 MomoOS-v2 的同场景对照（2026-11-19）

| 环节 | 旧 Momo 当前工作区实现 | MaiBot 固定版本的行为设计 |
| --- | --- | --- |
| 记下来 | 后台分批读取用户的普通发言，提取事实、偏好、事件、承诺，过滤并去重。随口提一次的稳定信息也可入库。另有显式保存工具。 | 启用长期记忆后可分别写回人物事实与增量聊天摘要。摘要提取明确区分本人和被引用者、计划和已发生、旧事实和新变化。 |
| 形成画像 | 近期原文、滚动摘要、长期条目、用户画像均已有。画像根据近期用户发言和旧画像生成七字段档案，支持手动编辑或重建。 | 画像按人组织，结合已有证据刷新。同时可从相关段落生成情景记忆，便于按一件事回忆。 |
| 下次用到 | 当前聊天路径以用户本轮输入查询其有效长期记忆，少于本机阈值 10 字跳过该路。通过筛选的结果按相似度、访问和时间等排序，最多五条随其他上下文进入回复。 | 画像注入、主动查询往事、根据聊天印象自动联想分别配置。普通查询默认按聊天流限定范围，自动联想默认关闭。 |
| 更正与追溯 | 识别为纠正时抑制本轮旧记忆注入。自动提取对同批条目使用末条发言 ID，定位原句精度有限。可管理或删除记忆，删除标记可抑制重新写回。 | 有修正预览、执行、回滚及关联画像/情景处理机制。查询后的聊天反馈纠错另有开关，默认关闭。 |

旧 Momo 依据：[提取规则](../references/SOURCE-REFERENCES.md#source-963723fca3)、[写入来源](../references/SOURCE-REFERENCES.md#source-9abb47cb4b)、[画像重建](../references/SOURCE-REFERENCES.md#source-93eed532f9)、[检索与排序](../references/SOURCE-REFERENCES.md#source-521d60a11a)、[聊天注入](../references/SOURCE-REFERENCES.md#source-6a784c600a)、[历史调查](INV-24-memory-and-tool-routing.md)。两者未用同一批对话做效果对测。本节比较实现机制，产品选型待定。

## 5. 优先复用长期记忆的可行范围

2026-11-19 讨论提案。建议以 MaiBot v1.4.6 为行为参照，先验证能否用同源组件承担长期事实、情景、画像与回忆。MaiBot 将 A_memorix 的专用分支同步进主仓，聊天消息何时写回由宿主调度。独立上游也提供 Python 入口和 MCP 接口，但核对时 README 标为 2.0.0a4，处在适配验证阶段。可先试独立入口，用 MaiBot 的实际写回与召回行为作对照，再决定需补哪些能力。[MaiBot 同步说明](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/docs/a_memorix_sync.md)、[聊天写回入口](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/services/memory_flow_service.py)、[独立上游说明](https://github.com/A-Dawn/A_memorix/blob/28db65dd96f546f736d65c043a39254cbae2c4e9/README.md)。

| 需求 | 建议归属 | 对用户可见的行为 |
| --- | --- | --- |
| 跨会话记住用户和往事 | 优先复用 MaiBot 的长期记忆机制。验证通过后，以它维护长期事实与用户画像，旧 Momo 的提取与画像用来对照效果 | 下次聊到同一件事时能接上，用户改口后能按新说法回应 |
| 当前对话与记录 | 保留 Momo 的文字及 ASR 转写记录、最近对话和需要时的滚动摘要 | 窗口能翻到原话，仅角色模式切回窗口也能接上同一段话 |
| 记忆来自哪里 | 由 Momo 给出谁说的、来自哪次聊天或哪个外部事件，以及角色当时能知道什么 | 邮件里的发件人说“我明天出差”，出差的人归到发件人；用户和角色一起经历的游戏才作为共同回忆 |
| 角色怎么回应 | 保留 Momo 的身份、来历、关系边界及当下状态。参考 MaiBot 独立配置行为、回复风格和表达学习的做法 | 同一份用户偏好能引导 Momo 调整相处方式。角色是否随着长期相处形成新习惯，留待人格专题决定 |

MaiBot 的人设预设可切换性格、行为和回复风格等字段，表达学习另有开关。旧 Momo 的角色设定还供角色日常和对话共同使用，用户画像则描述用户。建议用两次独立会话验证这条边界：第一次说“输的时候陪我吐槽一句”，第二次遇到输局时检查能否想起；用户改为“今天这局先让我静静”时，检查这次能否安静、下次能否分清临时要求与长期偏好，同时查看实际记忆来源、延迟和内存。本轮为源码调查，尚无这组对测结果。[MaiBot 人设切换](https://github.com/Mai-with-u/MaiBot/blob/d531b7eac5686f42bd63ea08fcb462a33c85981c/src/config/personality_service.py)、[Momo 角色资料与状态的使用](INV-16-current-prompt-and-persona-field-map.md)。

## 6. 时间与连续性（MaiBot v1.4.6、旧 Momo）

两边都有聊天记录、角色设定和时间信息。§5 的归属是新版复用建议。“沿用 Momo”表示该项目已有符合桌面陪伴需求的实现，不能据此认定 MaiBot 缺少同类能力。

| 疑问 | 核对结果 |
| --- | --- |
| 昨天说的“明天”怎么算？ | MaiBot 为聊天请求提供当前日期时间。长期记忆的摘要规则要求从原消息日期换算，引用按被引用消息的日期理解；查询工具支持起止时间与关键词组合。[当前时间](https://github.com/Mai-with-u/MaiBot/blob/v1.4.6/src/maisaka/chat_loop_service.py#L870)、[提取规则](https://github.com/Mai-with-u/MaiBot/blob/v1.4.6/prompts/zh-CN/main/memory/a_memorix_chat_summary.prompt#L7)、[查询参数](https://github.com/Mai-with-u/MaiBot/blob/v1.4.6/src/maisaka/builtin_tool/query_memory.py#L47) |
| 聊天中断过，它知道吗？ | MaiBot 启动恢复历史时，从最后一条可恢复记录算到当前，按间隔给角色不同的续聊提示。适用场景是有历史可恢复的启动过程。[上下文恢复](https://github.com/Mai-with-u/MaiBot/blob/v1.4.6/src/maisaka/runtime.py#L451) |
| 一件事现在还有效吗？ | MaiBot 的事实记录可以标明生效与失效时间，查询有效事实时会按时间筛选。普通摘要提示的输出格式没有要求每条事实填写有效期；自然对话里每种时长是否都能走通，尚待同例实测。旧 Momo 另有逐轮计算，会对可解析的时长或钟点给出预计截止、有效或过期。这个结果可供本轮决定是否打扰，现实中的会议是否真的结束仍需其他证据。[MaiBot 事实筛选](https://github.com/Mai-with-u/MaiBot/blob/v1.4.6/src/A_memorix/core/storage/metadata_fact.py#L872)、[Momo 时间计算](../references/SOURCE-REFERENCES.md#source-e035356118) |
| 角色自己这段时间经历了什么？ | 旧 Momo 的社交角色将当前日程活动、已经过和预计剩余分钟提供给对话，持续事项另有进度结算。MaiBot 有可持久化、到点唤醒的日程；本次核对的主仓路径未发现与 Momo 这套角色日常完整对应的流程。社区插件未纳入此项对照。[Momo 角色活动](../references/SOURCE-REFERENCES.md#source-3efd9e5e2c)、[持续事项结算](../references/SOURCE-REFERENCES.md#source-f9feb8a98b)、[MaiBot 日程](https://github.com/Mai-with-u/MaiBot/blob/v1.4.6/src/services/schedules/service.py) |
