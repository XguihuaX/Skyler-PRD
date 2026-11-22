# INV-30 · Momo 功能可能性与产品范围调研

调研日期：2026-11-17。状态：**功能候选研究；待与用户共同取舍**。

产品主入口：[新版 PRD](product/PRD-momo-next.md)。本报告回答“现在能让 Momo 做什么”，支持功能定义；不决定框架、不启动重建。下列功能是新版候选，不表示当前 Momo 已实现。

## 1. 调研结论与证据口径

本轮核对 36 份官方产品文档、开发者文档与官方仓库资料，涉及 Google、Microsoft、Anthropic、OpenAI、DeepSeek、MaiBot、Hermes、Letta、OpenClaw、LiveKit、Zotero、Notion、Raycast、Home Assistant、n8n、Spotify 及开源语音项目。完整来源及适用边界见 §8。

目前可以认真考虑的产品目标是：**有持续记忆与稳定人格，能够理解当前资料、协助完成日常事务，并在约定条件下继续跟进的个人助手。** 聊天、办事、记忆、语音、主动性可以在同一个产品中组合，但每项需要单独定义结果和使用边界。

与前期讨论相比，需要补充的功能方向包括：可查证的个人知识检索、真正可编辑的文件交付、可管理的后台任务、记忆纠错、经验复用、跨设备入口，以及随时接管执行。仅以“聊天＋工具＋向量记忆”描述产品，无法表达这些用户体验。

下文使用三种**研究判断**，不是现成产品的质量认证：

- **有明确路径**：有公开接口或可用组件支持核心操作，适合形成具体需求。仍需集成和验收。
- **需场景验证**：技术组成已存在，但效果依赖数据、账号、应用、中文表达、设备或流程复杂度。
- **探索项**：可以做限定实验，尚缺足够证据承诺广泛、持续可靠的效果。

官方产品展示证明某种体验已能提供；官方 API/开源实现证明存在可集成入口；本文对 Momo 的组合方案属于设计建议。三者在来源表中分开记录。本轮未安装候选、运行成功率测试或测量本机 RAM/语音延迟。上游动态文档按本次访问内容记录，进入选型时再固定版本与提交。

## 2. 功能候选总览

15 个讨论主题、62 个二级候选，映射到 PRD 原有 8 个功能域；这些主题是用户场景分组，不等于 15 个服务或进程。

| 主题 | 编号 | 用户得到什么 | 对应 PRD |
| --- | --- | --- | --- |
| 自然交流与理解 | C01–C04 | 能随口交代、追问、更正，获得合适回应 | N-01 |
| 人格与陪伴 | C05–C08 | 有一致性格、表达与参与分寸 | N-07、N-01 |
| 个人记忆与回忆 | C09–C14 | 记得相关事情，能回查、纠错、忘记 | N-04 |
| 联网搜索与调研 | C15–C18 | 有来源的答案、比较与更新 | N-03、N-02 |
| 个人资料与知识 | C19–C22 | 找到、读懂并管理自己的资料 | N-03、N-04 |
| 文件与内容产出 | C23–C26 | 得到能继续编辑和使用的文件 | N-02 |
| 邮件与沟通协助 | C27–C30 | 整理信息，形成回复和后续事项 | N-02、N-03 |
| 日历、待办与提醒 | C31–C34 | 事情有时间、有状态、能改能取消 | N-05 |
| 主动服务与持续跟进 | C35–C38 | 在约定范围内替用户持续留意 | N-05、N-02 |
| 网页与电脑操作 | C39–C42 | 在真实应用里完成操作并核验结果 | N-03 |
| 语音与共同看屏 | C43–C46 | 自然口述、听回复、打断、指着屏幕交流 | N-06、N-01 |
| 学习与工作辅导 | C47–C50 | 学懂、练习、复习，接续上次进度 | N-02、N-04 |
| 可复用流程与小工具 | C51–C54 | 重复事务逐渐变成可靠的一键流程 | N-02、N-08 |
| 多端、家庭与生活连接 | C55–C58 | 手机入口、家庭设备、媒体与生活信息联动 | N-03、N-05 |
| 用户控制与资源体验 | C59–C62 | 掌握执行、数据、模型、费用与资源 | N-08 |

### 2.1 自然交流与理解

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C01 | 自然对话与追问 | “刚才那个再简单一点”；延续当前话题并接受更正 | 有明确路径；长对话需处理主题切换 |
| C02 | 多模态输入 | 给文字、截图、图片或文件，提取信息并回答 | 有明确路径；不同模型支持的媒体类型不同 |
| C03 | 从模糊请求补全任务 | “帮我把这周的事情理一下”；先整理已有资料，再询问缺项 | 需场景验证；询问次数与自主判断程度待定 |
| C04 | 交流与办事自然切换 | 闲聊中提到正事可转为任务；办事中仍可追问或改目标 | 需场景验证；不能把每句闲聊都变成待办 |

依据：实时多模态输入已有公开接口；跨轮状态与工具循环也有公开实现。[Gemini Live（S27）](https://aistudio.google.com/docs/live-api/capabilities)、[DSH 模型接入（S34）](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/providers.md)。C03/C04 的具体体验是 Momo 设计建议。

### 2.2 人格与陪伴

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C05 | 一致的人格与说话风格 | 能干、温柔、幽默程度可调，工作时简洁、闲聊时自然 | 有明确路径；效果需真实对话验收 |
| C06 | 学习用户表达习惯 | 学会称呼、常用表达和约定；能关闭或纠正学习 | 需场景验证；学习范围、错误表达处理待定 |
| C07 | 合适的参与时机 | 在允许的时间问候、接话，用户忙时安静 | 需场景验证；打扰频率必须可控 |
| C08 | 角色卡与桌面呈现 | 助手卡、角色切换、头像/桌宠、语音与表情一致 | 有明确路径；入口形态、角色之间记忆共享待定 |

MaiBot 将身份、人设、行为风格和表达风格分开，提供表达学习及范围控制，也有桌宠入口，值得逐项借鉴。[人设（S21）](https://docs.maibot.chat/manual/personality)、[Bot 配置（S22）](https://docs.maibot.chat/manual/configuration/bot-config)、[桌宠（S25）](https://docs.maibot.chat/manual/desktop-pet)。这些能力可以通过人设、样例、记忆和行为规则实现，是否需要模型微调应由实际效果决定。

### 2.3 个人记忆与回忆

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C09 | 长期偏好与背景 | 记住语言、沟通风格、常用软件、重要项目 | 有明确路径；用户能查看记住了什么 |
| C10 | 随口事件与近期话题 | “上周我提的那家店叫什么”；找回事件、时间和原话 | 需场景验证；短句、别名、指代是重点 |
| C11 | 人物与项目关系 | 分清谁的偏好、谁负责什么、哪个项目发生了什么 | 需场景验证；同名与第三人称不能混归用户 |
| C12 | 低 Token 的按需回忆 | 当前问题只带入少量相关记录，必要时再展开原文 | 有明确路径；检索相关性、漏召回和总费用待测 |
| C13 | 查证、纠错与删除 | “这件事已经变了”“你从哪里记住的”“忘掉这段” | 有明确路径；更正需影响后续检索与回答 |
| C14 | 不记与过期管理 | 临时话题不升格为长期偏好；允许临时会话和到期清理 | 需场景验证；保留规则由用户选择 |

Hermes 提供有界常用记忆与历史搜索；MaiBot 提供来源关联、查询、纠错和删除管理；Letta 当前文档提供常驻内容与按需读取内容的区分。[Hermes Memory（S19）](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/)、[MaiBot 记忆管理（S23）](https://docs.maibot.chat/manual/webui/memory-management)、[Letta Memory（S36）](https://docs.letta.com/agent-sdk/memory)。具体分层建议与随口信息处理见 §3。

### 2.4 联网搜索与调研

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C15 | 搜索并交叉核对 | 查最新信息，回答附日期、来源与未知项 | 有明确路径；无法读取的来源需明确 |
| C16 | 方案与产品比较 | 按用户条件比较软件、设备、旅行选项，交付比较表 | 需场景验证；信息时效和条件完整性决定质量 |
| C17 | 长资料阅读与归纳 | 多网页/PDF/视频资料形成有出处的主题笔记 | 需场景验证；视频需要实际媒体或明确可用的文本来源 |
| C18 | 专题更新 | 只报告某项技术或项目自上次以来有意义的变化 | 需场景验证；依赖持续运行和去重规则 |

资料约束回答及引用已有产品路径；浏览器工具提供网页读取与操作，定时工具提供持续执行。[Gemini Notebook（S07）](https://workspace.google.com/products/gemini-notebook/)、[Playwright MCP（S12）](https://github.com/microsoft/playwright-mcp)、[Hermes Cron（S18）](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron/)。组合研究流程仍需自行定义筛选与证据标准。

### 2.5 个人资料与知识

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C19 | 自然语言找资料 | “找上次那份有预算表的方案”；返回文件和命中位置 | 有明确路径；授权目录、文件解析和索引更新待定 |
| C20 | 私人知识问答 | 对自己的笔记、论文、工作资料回答，带原文来源 | 有明确路径；选定资料范围与版本 |
| C21 | 资料归档与整理 | 下载文件命名、分类、去重候选和整理计划 | 需场景验证；可预览、恢复，不能仅凭相似度删除 |
| C22 | 知识库连接 | 读取或更新 Notion、Zotero 等实际使用的资料库 | 有明确路径；账号、客户端版本与权限逐项确认 |

公开接口已经支持知识库内容读写及全文访问；检索可以同时使用关键词与语义信息。[Notion（S06）](https://developers.notion.com/guides/connections-and-tokens/overview)、[Zotero Local API（S08）](https://www.zotero.org/support/dev/web_api/v3/local_api)、[File Search（S35）](https://developers.openai.com/api/docs/guides/tools-file-search)。是否采用托管检索或本地检索留给后续选型。

### 2.6 文件与内容产出

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C23 | 文档写作与修改 | 根据资料与模板输出可编辑报告、总结、说明或简历 | 有明确路径；以文件内容和渲染效果验收 |
| C24 | 表格与数据处理 | 清洗表格、计算、核对、画图，保留公式与处理记录 | 需场景验证；数据口径和计算正确性需检查 |
| C25 | 演示文稿与图示 | 把材料变成 PPT、流程图、讲解稿 | 有明确路径；可编辑性和排版质量待验收 |
| C26 | 图片与轻量媒体处理 | 生成配图、改图、OCR 后整理、给音视频做摘要 | 需场景验证；媒体模型、费用和实际素材可访问性待定 |

Claude 已公开提供文档、表格、演示文件生成体验；Gemini 有图片生成编辑接口。它们支持把“交付可用文件”列为功能，但 Momo 仍需自己的生成、预览与检查流程。[Claude 文件（S09）](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude)、[Gemini 图片接口（S10）](https://aistudio.google.com/docs/image-generation)。

### 2.7 邮件与沟通协助

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C27 | 收件与消息摘要 | 汇总未读、重要变化和待回应事项，保留原消息入口 | 有明确路径；先选用户真实使用的服务 |
| C28 | 回复草稿与发送 | 结合相关资料拟回复，按授权规则保存草稿或发送 | 有明确路径；草稿、发送结果分别显示 |
| C29 | 沟通转行动 | 从邮件或聊天提取约定、材料清单、截止时间与附件 | 需场景验证；提及日期不一定构成真实承诺 |
| C30 | 会议辅助 | 会前材料、会后摘要、决定与待办，能返回记录证据 | 需场景验证；音频获取、语言与录制条件需要明确 |

Gmail、Microsoft Graph 已提供邮件资源操作；Google Meet 的会议笔记展示了会后文档化体验。[Gmail（S01）](https://developers.google.com/workspace/gmail/api/reference/rest)、[Outlook（S02）](https://learn.microsoft.com/en-us/graph/outlook-mail-concept-overview)、[Meet（S11）](https://support.google.com/meet/answer/14754931?hl=en)。本轮没有验证个人微信、QQ、飞书或钉钉的具体接入路径，不能将这些平台统称为已支持；入选后按实际账号与接口补查。

### 2.8 日历、待办与提醒

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C31 | 自然语言设置提醒 | “明早九点提醒我带材料”；显示解析后的时间和事项 | 有明确路径；时区、模糊日期、重复请求需处理 |
| C32 | 人性化延后与完成 | “再过十分钟”“这件事完成了”；修改现有提醒并确认状态 | 有明确路径；不因改时间再造一条重复提醒 |
| C33 | 日程与任务安排 | 查空闲、发现冲突、安排工作块、联动日历待办 | 需场景验证；时间估算及外部日历同步待定 |
| C34 | 条件触发提醒 | “如果周五仍未回复再提醒我”；满足条件才通知 | 需场景验证；条件来源、检测频率、失联行为待定 |

Google Calendar 支持单次及重复事件；Graph 支持日历、提醒与 To Do；Hermes 支持一次/周期/事件任务。[Calendar（S03）](https://developers.google.com/workspace/calendar/api/guides/overview)、[Outlook Calendar（S04）](https://learn.microsoft.com/en-us/graph/outlook-calendar-concept-overview)、[To Do（S05）](https://learn.microsoft.com/en-us/graph/api/resources/todo-overview?view=graph-rest-1.0)、[Cron（S18）](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron/)。

要区分“通知式提醒”和“必须响起的闹钟”：电脑休眠、关机、退出以及系统勿扰会改变交付条件。后者若入选，需要定义系统级/手机侧或常在线执行位置及可验收行为。

### 2.9 主动服务与持续跟进

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C35 | 日报与每日简报 | 在约定时间汇总日程、重点消息与未完成事项 | 有明确路径；无变化时是否沉默可配置 |
| C36 | 事项闭环跟进 | 追踪“等回复、等材料、等用户决定”，提醒后允许继续执行 | 需场景验证；外部状态必须有可读取依据 |
| C37 | 订阅与变化监测 | 跟进网页、文档、项目发布或价格变化，仅通知相关变化 | 需场景验证；来源访问、频率与重复通知控制 |
| C38 | 主动建议与关心 | 根据已知安排建议准备材料、休息或调整任务 | 需场景验证；无约束全天主动规划归探索项 |

定时/事件执行已有实现；人格化的主动参与也有参考，但“值得打扰”是需要用户体验验证的产品判断。[Hermes Cron（S18）](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron/)、[MaiBot 行为配置（S22）](https://docs.maibot.chat/manual/configuration/bot-config)。

### 2.10 网页与电脑操作

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C39 | 浏览器读取和填表 | 打开页面、查找、填写、上传指定文件，返回完成证据 | 有明确路径；登录/验证码等需接管入口 |
| C40 | 本机应用操作 | 打开应用、定位窗口、点击输入，处理选定应用中的事务 | 需场景验证；各应用可访问性与权限差异明显 |
| C41 | 跨应用任务 | 从网页下载材料、整理文件、填表、形成可审阅结果 | 需场景验证；流程越长越需要中间结果和失败恢复 |
| C42 | 观察、暂停和接管 | 用户看见做到了哪一步，能停止、改目标、接手后让 Momo 继续 | 有明确路径；已经发生的外部动作需按实际状态报告 |

Playwright MCP 支持结构化页面操作；OpenClaw 提供独立浏览器及已登录会话接入路径；DSH 的 macOS/Windows CUA 在官方文档中仍标为可选实验能力；OpenAI 的电脑操作接口要求应用提供实际执行环境与结果验证。[S12](https://github.com/microsoft/playwright-mcp)、[S15](https://docs.openclaw.ai/tools/browser)、[S13](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/computer-use.md)、[S14](https://developers.openai.com/api/docs/guides/tools-computer-use)。

**对“能否像当前助手一样顺畅操作”的判断：**可以按具体网站和应用追求接近的体验，不能仅凭换模型或框架承诺全局同等流畅。需要比较同一任务的实际完成率、耗时、人工接管次数与副作用；允许一口气完成多少步、何时停下来，由场景需求决定。

### 2.11 语音与共同看屏

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C43 | 听写与口述整理 | 说一段想法，形成文字、任务或结构化笔记 | 有明确路径；中文、专有名词和噪声需测 |
| C44 | 自然朗读与角色音色 | 按偏好读结果、调语速、使用合适且有权使用的音色 | 有明确路径；本地/云端、音色与资源之间需选取 |
| C45 | 可打断的语音对话 | 说话中插入更正，Momo 停播并按新信息继续 | 需场景验证；外放回声、误打断和停播后的状态需测 |
| C46 | 边看屏边交流 | 分享当前页面或画面，“这里是什么意思”可指向可见内容 | 需场景验证；持续全双工与全天常看屏归探索项 |

LiveKit 已包含轮次与打断处理；Gemini Live 提供实时音视频输入接口，但页面明确标注预览且有帧率/会话限制。Hermes 展示多入口语音交互。[S26](https://docs.livekit.io/agents/build/turns/)、[S27](https://aistudio.google.com/docs/live-api/capabilities)、[S28](https://hermes-agent.nousresearch.com/docs/user-guide/features/voice-mode/)。

GPT-SoVITS V5 和 CosyVoice 值得进入后续音色/流式对照，但不在此预选引擎。前者列出的 CUDA 加速不能直接推导 M2 提速；后者的双向流式能力与官方低延迟数字，不能代替 Momo 从用户停顿到听到回复的整链路测量。[S29](https://github.com/RVC-Boss/GPT-SoVITS)、[S30](https://github.com/QwenAudio/CosyVoice)。

### 2.12 学习与工作辅导

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C47 | 基于资料讲解 | 按用户基础解释文章、软件流程、课程内容，能追问 | 有明确路径；与资料引用结合 |
| C48 | 练习与反馈 | 出题、口头演练、代码/文稿反馈、错因复盘 | 需场景验证；评分标准与题目正确性需检查 |
| C49 | 学习进度与复习 | 分开记录“已准备材料、已讲、已练、已掌握”，接续上次进度 | 需场景验证；不能仅因看过就判断掌握 |
| C50 | 项目上下文助手 | 回答项目进展、决定依据、待办和下一步，带资料入口 | 需场景验证；当前状态应来自项目记录 |

资料问答、跨会话记忆与经验文档提供实现基础。[Gemini Notebook（S07）](https://workspace.google.com/products/gemini-notebook/)、[Hermes Skills（S20）](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills/)。学习状态和项目状态的具体记录规则是 Momo 的候选产品设计。

### 2.13 可复用流程与小工具

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C51 | 保存做事方法 | “以后都按刚才这个格式整理”；保存可查看、修改的流程 | 有明确路径；成功实例需核对后再复用 |
| C52 | 一键重复任务 | 周报、资料收集、文件转格式、每日简报变成固定入口 | 有明确路径；固定步骤可以不逐步消耗 LLM |
| C53 | 小脚本与批处理 | 生成并运行限定目录的数据整理、转换、检查脚本 | 需场景验证；先明确输入输出与可修改范围 |
| C54 | 工具与能力扩展 | 添加新应用连接或流程，按任务需要加载，失败有明确反馈 | 有明确路径；用户选能力，不必面对大量内部工具名 |

Hermes Skills 支持按需加载及从材料/经验形成技能；n8n 展示可在指定动作等待人工决定；Raycast 有扩展工具及条件确认机制。[S20](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills/)、[S33](https://docs.n8n.io/build/integrate-ai/ai-examples/human-in-the-loop-for-tools.md)、[S17](https://developers.raycast.com/api-reference/tool)。这里的“学习”主要是更新经验和流程，不自动等于更新模型权重。

### 2.14 多端、家庭与生活连接

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C55 | 手机与电脑连续使用 | 手机上交代任务、接收结果，电脑上接着处理 | 需场景验证；手机入口、执行机器、后台存活待定 |
| C56 | 智能家居协助 | 查询或控制已接入的灯、空调、场景 | 有明确路径；需要用户实际设备与家庭服务 |
| C57 | 媒体与生活偏好 | 播放/暂停、维护歌单、按场景推荐内容 | 需场景验证；服务接口、账号与播放设备不同 |
| C58 | 出行与生活计划 | 汇总天气、出行资料、行程和准备清单；必要时协助填预约表 | 需场景验证；预订/支付等动作单独定义授权 |

OpenClaw Nodes 提供跨设备命令通道参考；Home Assistant MCP 提供设备控制；Spotify API 提供媒体元数据、歌单和播放操作。它们证明相关连接可实现，不证明已覆盖用户的所有设备或本地生活平台。[S16](https://docs.openclaw.ai/nodes)、[S31](https://www.home-assistant.io/integrations/mcp_server)、[S32](https://developer.spotify.com/documentation/web-api)。

### 2.15 用户控制与资源体验

| ID | 候选功能 | 具体使用方式与结果 | 判断 / 需要明确 |
| --- | --- | --- | --- |
| C59 | 任务状态与操作记录 | 看得见等待、执行、需用户处理、失败、完成；能取消 | 有明确路径；完成必须有对应结果 |
| C60 | 数据与权限管理 | 管理可访问目录/账号、记忆范围、导出删除和撤销授权 | 有明确路径；不同存储副本的删除行为需说明 |
| C61 | 模型与费用选择 | 选择服务商/模型，显示任务用量与可用能力 | 有明确路径；登录方式、配额、媒体/工具能力分别验证 |
| C62 | 轻量运行与更新 | 空闲时资源可控，重任务按需开启，异常可诊断、更新可恢复 | 需场景验证；RAM、CPU、耗电、延迟需真实测量 |

DSH 已展示多模型/协议接入；记忆管理、调度状态和工具确认也有现成参考。[S34](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/providers.md)、[S23](https://docs.maibot.chat/manual/webui/memory-management)、[S18](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron/)、[S17](https://developers.raycast.com/api-reference/tool)。Momo 的资源预算和更新体验仍需自行建设。

## 3. 记忆应怎样定义成用户功能

### 3.1 先区分要记住什么

以下是候选规则，尚未变成用户确认的保留策略。

| 用户表达 / 信息类型 | 建议的产品行为 | 后续回答应做到 |
| --- | --- | --- |
| “以后回答简短一点” | 作为可修改的长期偏好 | 不用每轮重复解释偏好来源 |
| “昨天去了某家餐厅” | 作为带时间、来源的经历候选 | 可以回忆发生过什么，不自动推导“最喜欢这家” |
| “明天下午要交材料” | 识别为可能的任务/日程；按已约定自动化规则处理 | 区分只记录这句话、已建任务、已设提醒 |
| “上次那家店叫什么” | 搜近期相关事件，再找原话 | 返回候选和依据；多家同样符合时澄清 |
| “北京明天下雨吗” | 作为问题保留在会话中，是否长期保留由策略决定 | 不据此推导用户住在北京 |
| “我朋友喜欢猫” | 归属于朋友的描述 | 不写成用户偏好 |
| “刚才是开玩笑”“现在换工作了” | 纠正或使旧记录过期，保留必要来源关系 | 后续回答采用新状态 |
| 上传的文章或网页 | 保留为资料来源 | 文中观点不自动当成用户事实 |
| “这段不要记” | 按临时会话/排除规则处理 | 明确影响哪些历史、索引和后台任务 |

日程、任务执行状态要有明确记录；不能只靠“模型好像记得”。随口信息可以先作为事件或对话记录，等重复出现、用户确认或实际任务需要时再形成长期事实。

### 3.2 低 Token 回忆的功能契约

建议把 C12 定义为：**能够跨会话找回相关信息，回答能返回证据，成本不随全部历史线性塞入每次提示词。**

候选处理过程：当前问题 → 按人物/项目/时间/来源缩小范围 → 关键词与语义检索 → 少量候选 → 必要时读原始片段 → 回答并关联来源。少量稳定偏好可以常驻；大量经历和资料按需读取。

这条路线有公开依据：OpenAI File Search 同时支持语义和关键词检索；MaiBot 有分层检索及最终条数限制；Hermes 将有界常用记忆与历史搜索分开；Letta 将 system 内容与按需读取文件分开。[S35](https://developers.openai.com/api/docs/guides/tools-file-search)、[S24](https://docs.maibot.chat/manual/configuration/amemorix-config)、[S19](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/)、[S36](https://docs.letta.com/agent-sdk/memory)。

本地索引查询本身不需要把全库发给聊天模型；但嵌入、抽取、重排、摘要及回答仍可能产生计算或 API 费用。后续应测每次回忆的总成本、命中率、误归属、延迟及删除后的残留，而非只看最终提示词有多短。固定“只取 5 条”也不能保证该找的内容都在里面。

## 4. 把功能组合成可以体验的场景

下面用于选择产品方向，不是已经批准的实施计划。

| 场景 | 用户的一句话 | 完整结果 | 小规模可行性验证 |
| --- | --- | --- | --- |
| 每日开始 | “今天哪些事情需要我注意？” | 汇总日程、已授权信息源和未完成事项，能追问并打开原记录 | 给定一组真实但可用的事项，检查遗漏、误报、重复与来源 |
| 资料变交付物 | “把这些资料整理成明天要讲的文件” | 阅读资料、询问受众/格式、形成可编辑产物与引用 | 检查引用、关键内容、文件可打开及排版 |
| 随口交代后续 | “周五还没收到回复就提醒我” | 存明确条件、展示任务、按条件通知，能改或取消 | 用受控事件验证触发、不触发、重复、取消与休眠恢复 |
| 记得过去 | “我前阵子比较过的那两个方案，最后为什么选这个？” | 找回相关讨论与决定依据，区分记载和推测 | 混入相似话题、更正与第三人信息，检查证据及归属 |
| 一起操作电脑 | “帮我完成这个网页上的材料登记” | 读取要求、准备内容、填表、按约定节点交接、核验结果 | 在一个目标网站记录完成情况、时间、接管和重复提交 |
| 有人格的语音助手 | “先停一下，我刚刚说错了” | 及时停播，理解更正，继续正确任务并保留一致风格 | 耳机/外放、短插话/真打断、噪声、说话中取消分别试用 |

评估每个候选场景时记录：使用频率、当前耗时/痛点、期望结果、所需账号和数据、允许的自主程度、资源/费用可接受范围。先拿近期真实例子，不用功能数量代替价值判断。

## 5. 向各项目学习什么

| 参考 | 值得借鉴的用户能力 | 不宜直接照搬的假设 |
| --- | --- | --- |
| MaiBot | 人设与行为分开；表达学习；参与时机；可管理的记忆与桌宠 | 群聊目标、主动频率和复杂记忆结构未必适合个人办事 |
| Hermes | 有界常用记忆、历史查找、按需技能、定时任务及多入口 | 默认记忆容量、渠道与执行环境不自动适合 Momo |
| DeepSeek Harness | 可插拔能力、多模型、工具执行与电脑操作入口 | CUA 实验能力不代表任意软件已稳定；框架本身不确定功能需求 |
| OpenClaw | 独立/已登录浏览器、多端节点、持续在线助手的组织方式 | 跨设备部署复杂度不应默认加入首版 |
| Letta | 常驻与按需记忆分开，跨会话状态和后台整理 | 某种存储/后台整理机制是否值得使用，要看实际召回和成本 |
| Gemini Notebook / Claude 文件 | 引用可回查、资料转交付物、结果可以继续编辑 | 产品端体验不等于提供同等能力的嵌入 API |
| Raycast / n8n / Home Assistant | 窄工具、可复用流程、按动作授权、设备连接 | 连接器多不代表用户每天会用；选实际服务再实现 |

以上均由 §8 官方资料支持。对于 Momo，我建议优先研究这些项目的**可观察行为、交互和验收办法**，技术选型时再决定采用、适配或自行实现哪些组件。

## 6. 能力范围与资源取舍

- **可以认真纳入需求讨论**：自然交流、文件问答、资料产出、邮件/日历操作、提醒、可管理记忆、限定应用操作、语音输入输出及可复用流程。
- **需要针对场景验证**：中文随口回忆、长任务恢复、主动服务时机、跨应用办事、外放可打断语音、多端连续使用。
- **保留探索方向**：任意应用无人值守操作、全天看屏并自主规划、无限期准确记住全部经历、在低资源机器上同时保持高质量本地多模态与低延迟全双工。本轮资料不足以把这些写成可靠承诺；可为其中有价值的部分设计限定实验。

本机低 RAM 目标应影响部署选择和资源生命周期，不必先缩减为只能聊天的功能范围。云端推理＋本地操作、混合语音和按需启动都是候选；选择取决于用户的联网、隐私、费用和离线要求。本轮不填未经测量的 RAM 或延迟数字。

商业产品质量建议具体化为：任务完成有证据、失败能解释、操作能停止、数据能管理、重启行为明确、费用资源可见。体验指标在功能需求阶段确定，通过同一真实任务对照候选方案。

## 7. 接下来共同定义什么

建议先讨论产品重心：**办事为主且有人格，还是办事与人格陪伴同等重要。** 这会影响默认入口、主动性、语音和角色表现的优先级。用户尚未选择，本文不代选。

随后逐组讨论 §2：标注“核心 / 后续 / 不需要 / 需体验后决定”，给核心项补充一个近期真实例子。功能定义完成后再写流程、输入输出、正常/异常情况、权限、资源和验收要求，最后进入技术选型。

候选应有进入与退出理由；新技术发布只有在改善入选功能的具体问题时才引起范围或架构变更。当前不把 62 项全部写成首版待办。

## 8. 官方来源登记

以下 36 项均在 2026-11-17 打开核对。类型“产品”说明已有体验；“接口/实现”说明公开可集成机制；“文档”说明官方设计与配置。检索摘要只用于找到页面，结论以打开内容为依据。Apple Shortcuts 文档多次返回 429，未纳入这 36 项的正文证据。

| ID | 官方来源 | 本次支持的事实 / 边界 |
| --- | --- | --- |
| S01 | [Gmail REST](https://developers.google.com/workspace/gmail/api/reference/rest) | 接口：消息、线程、标签、附件、草稿与发送；摘要和优先级由 Momo 实现 |
| S02 | [Outlook Mail](https://learn.microsoft.com/en-us/graph/outlook-mail-concept-overview) | 接口：Graph 可整合邮件及相关资源；账号授权仍需核对 |
| S03 | [Google Calendar](https://developers.google.com/workspace/calendar/api/guides/overview) | 接口：单次/重复事件、日历、设置；不保证本机闹钟交付 |
| S04 | [Outlook Calendar](https://learn.microsoft.com/en-us/graph/outlook-calendar-concept-overview) | 接口：事件、提醒与空闲查询；部分智能排期能力限工作/学校账号 |
| S05 | [Microsoft To Do](https://learn.microsoft.com/en-us/graph/api/resources/todo-overview?view=graph-rest-1.0) | 接口：任务、列表、子项、关联资源及增量更新 |
| S06 | [Notion connections](https://developers.notion.com/guides/connections-and-tokens/overview) | 接口：能力权限和内容访问分别控制；不能默认读全工作区 |
| S07 | [Gemini Notebook](https://workspace.google.com/products/gemini-notebook/) | 产品：当前官方页使用此名称并注明 formerly NotebookLM；基于上传资料回答并提供引用 |
| S08 | [Zotero Local API](https://www.zotero.org/support/dev/web_api/v3/local_api) | 接口：本机资料访问；写入支持有版本及授权条件，需检查用户客户端 |
| S09 | [Claude 创建编辑文件](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude) | 产品：可生成 Word、Excel、PowerPoint、PDF；不等于 Momo 可直接嵌入完整产品 |
| S10 | [Gemini 图片生成](https://aistudio.google.com/docs/image-generation) | 接口：图片生成与编辑；具体模型、费用和质量后续选型 |
| S11 | [Google Meet 笔记](https://support.google.com/meet/answer/14754931?hl=en) | 产品：会后笔记及共享；有套餐、语言及会议设置条件 |
| S12 | [Playwright MCP](https://github.com/microsoft/playwright-mcp) | 实现：基于可访问性快照的网页操作；成功率依具体任务 |
| S13 | [DSH Computer Use](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/computer-use.md) | 实现：macOS/Windows 可选实验组件，原生可访问性与权限机制 |
| S14 | [OpenAI Computer Use](https://developers.openai.com/api/docs/guides/tools-computer-use) | 接口：模型与执行环境交替工作；应用提供驱动、会话、权限、清理及验证 |
| S15 | [OpenClaw Browser](https://docs.openclaw.ai/tools/browser) | 实现：独立浏览器、已登录会话接入、标签页、动作、下载 |
| S16 | [OpenClaw Nodes](https://docs.openclaw.ai/nodes) | 实现：设备节点及相机/通知/系统等命令通道；各平台能力不同 |
| S17 | [Raycast Tool](https://developers.raycast.com/api-reference/tool) | 接口：扩展工具、条件确认与副作用说明；可借鉴用户控制方式 |
| S18 | [Hermes Cron](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron/) | 实现：一次、周期、事件任务，暂停修改及结果交付；需存活的调度与执行环境 |
| S19 | [Hermes Memory](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory/) | 文档：有界常用记忆及历史搜索；当前页面也解释会话边界和真实写入的重要性 |
| S20 | [Hermes Skills](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills/) | 实现：按需知识、从经验/材料建立技能；仍需检查生成流程的正确性 |
| S21 | [MaiBot 人设](https://docs.maibot.chat/manual/personality) | 文档：人设、行为、表达区分，多套保存切换 |
| S22 | [MaiBot Bot 配置](https://docs.maibot.chat/manual/configuration/bot-config) | 文档：参与行为、表达学习、学习范围及使用控制 |
| S23 | [MaiBot 记忆管理](https://docs.maibot.chat/manual/webui/memory-management) | 产品/实现：来源、画像、纠错、删除与恢复；其删除可能是逻辑删除，不应照搬为彻底抹除承诺 |
| S24 | [MaiBot A_Memorix](https://docs.maibot.chat/manual/configuration/amemorix-config) | 文档：写读链路、稀疏/向量与最终检索数量；组件多不保证实际召回质量 |
| S25 | [MaiBot 桌宠](https://docs.maibot.chat/manual/desktop-pet) | 实现：桌面入口、皮肤及共享主人会话；不决定 Momo 必须保留桌宠 |
| S26 | [LiveKit Turn handling](https://docs.livekit.io/agents/build/turns/) | 实现：轮次、打断及误打断处理；整链路体验还需设备验证 |
| S27 | [Gemini Live capabilities](https://aistudio.google.com/docs/live-api/capabilities) | 接口：实时多模态；文档标预览并列出视频帧率和会话限制 |
| S28 | [Hermes Voice Mode](https://hermes-agent.nousresearch.com/docs/user-guide/features/voice-mode/) | 实现：CLI/消息入口语音，识别和合成有依赖及平台条件 |
| S29 | [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) | 实现：音色克隆与 V5 更新；CUDA 优化不直接证明 M2 低延迟低 RAM |
| S30 | [CosyVoice](https://github.com/QwenAudio/CosyVoice) | 实现：文本输入/音频输出流式、多语种等；官方延迟不能直接当 Momo 实测 |
| S31 | [Home Assistant MCP](https://www.home-assistant.io/integrations/mcp_server) | 接口：家庭状态与设备动作；不同工具集合访问范围不同 |
| S32 | [Spotify Web API](https://developer.spotify.com/documentation/web-api) | 接口：媒体元数据、歌单与播放；当前文档列有 Premium 前提，其他音乐服务需另查 |
| S33 | [n8n 工具人工决策](https://docs.n8n.io/build/integrate-ai/ai-examples/human-in-the-loop-for-tools.md) | 实现：指定工具可暂停等待批准/拒绝；无需所有动作都加确认 |
| S34 | [DSH Providers](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/providers.md) | 实现：多服务商及 Chat Completions/Responses/Anthropic/Gemini 协议；具体端点兼容性仍需测试 |
| S35 | [OpenAI File Search](https://developers.openai.com/api/docs/guides/tools-file-search) | 接口：托管语义与关键词检索；需要上传/索引，不自动解决个人记忆的事实归属 |
| S36 | [Letta Agent SDK Memory](https://docs.letta.com/agent-sdk/memory) | 文档：system 常驻、其他记忆按需读及后台整理；旧 memory 链接跳到 legacy，已改用当前 SDK 页面 |

## 9. 本轮交付边界

完成能力调研与候选定义，同步 PRD/调查索引/路线图的文档入口。未变更运行时代码、未安装新框架、未创建新对话、未 commit 或 push。下一阶段需要用户与助手共同确认功能重心和范围；技术方案随后细化。
