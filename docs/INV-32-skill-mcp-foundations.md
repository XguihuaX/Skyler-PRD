# INV-32 · Momo 使用 DSH / WorkBuddy Skills 与 MCP 的基础能力

日期：2026-11-17。状态：公开资料与当前代码核对完成；未安装技能、启动外部服务或进行端到端兼容实测。

## 1. 本轮范围与结论

用户确定：Momo 是独立智能体，优先复用 DSH、WorkBuddy 生态的 Skills 和 MCP；本轮不需要 Codex Skills，也不以调用完整外部智能体作为方案。“接入并使用 Skills / MCP”单列功能，其他业务功能研究后再逐项讨论。

结论：方向可行。应建立一套 Momo 自有的技能发现、按需读取、工具执行与结果交付能力，同时适配目标技能依赖。无须为两个来源各自运行一套主 Agent。来源相同不保证所有技能拥有相同的执行条件；兼容判定以具体技能包、工具、服务认证和实测结果为单位。

## 2. 上游核对与可复用边界

DSH 资料固定到提交 `39163dd30f703a79079e2d2446545cff3c453738`，避免主分支更新改变本轮证据。WorkBuddy 按本日公开文档核对；公开包格式不证明某个市场条目已经允许下载、导出或在第三方宿主使用。

| 对象 | 官方证据 | 对 Momo 的含义 |
| --- | --- | --- |
| 通用 Agent Skills | S1：SKILL.md、名称/描述、可选脚本与资源、兼容环境声明 | 建立通用读取机制；只给模型必要目录信息，正文和引用资源按需读取 |
| DSH 技能体系 | S2：provider 汇集目录，tool-skill 加载正文，filesystem/office 分别供给技能 | 可借鉴目录与加载分工；不必引入 DSH 主智能体 |
| DSH Office 实际样本 | S3、S4：office-docx 使用 python-docx，并依赖 office_render / office_convert 等；加载器可追加 Python 环境说明 | 文字处理流程可复用；原样使用完整技能必须提供对应工具，或明确改写为 Momo 的等价工具。仅复制 SKILL.md 会遗漏加载器追加的环境信息 |
| DSH 电脑操作 MCP | S5：独立 cua-mcp profile 对外提供工具，不运行模型回合；当前为可选实验能力 | 可作为 Momo 的独立工具服务候选；需要截图输入链路、平台权限及用户询问转发。电脑操作功能尚未因此进入首版 |
| WorkBuddy 技能包 | S6：SKILL.md + references / scripts / templates；有展示字段与 Bash 脚本约定 | 基础结构可适配。保留来源字段，解析相对资源路径；脚本型技能还需命令执行能力 |
| WorkBuddy 连接器 | S7：MCP + Skill、CLI + Skill 两种形式；认证和环境可由 WorkBuddy 托管 | 标准服务可以独立连接；平台托管登录、CLI 安装和运行环境需要独立解决，不能继承 WorkBuddy 的登录状态 |
| WorkBuddy 用户管理 | S8：本地导入、启用/关闭、卸载 | 可参考用户操作，但不把建设市场或自动找技能列为起步前提 |
| MCP 协议 | S9、S10：工具、资源及不同内容结果；客户端可处理用户输入请求 | Momo 负责客户端与模型之间的能力和结果映射；按目标服务选择协议版本及支持项 |

注意：DSH 的 mcp-client 是消费其他服务的客户端插件，不意味着所有 DSH 内部工具都已作为 MCP Server 发布。CUA 有明确独立 MCP 入口；Office 文档描述的是 DSH 内部工具，本轮未确认独立 Office MCP 分发入口。

WorkBuddy 与 CodeBuddy 的文档需区分。本轮以 WorkBuddy 开放平台的技能、连接器规范为主；CodeBuddy 文档出现的专属变量、Hooks 或子 Agent 字段不自动视为 WorkBuddy 的通用执行契约，也不列为 Momo 必须完整模拟的能力。

## 3. Momo 的基础能力建议

下表是调研建议，尚未逐条获得用户确认。功能归入 PRD N-03.1，跨任务状态和用户控制时关联 N-02/N-08。

| 基础能力 | 用户能得到什么 | 建议范围 |
| --- | --- | --- |
| 技能导入与管理 | 添加技能、看用途/来源/依赖、启停、删除；同名冲突有明确提示 | 第一版先支持本地目录/包；是否支持 ZIP、仓库链接及自动升级随后确定 |
| 发现与按需加载 | 自然语言匹配技能，也能明确指定；正文和引用资料需要时才读 | 基础必需；无相关任务时不加载全文；无需先上向量数据库 |
| 路径与工作区 | 能读技能资源、访问用户指定资料、保存结果 | 基础必需；区分技能只读资源、任务输入与输出目录，解析相对路径 |
| 工具/脚本执行 | 将技能步骤变成真实操作，并读回结果继续处理 | 工具调用为基础；Python/Node/Shell 按选定脚本技能提供，不要求同时预装所有语言 |
| MCP 接入与管理 | 配置服务、查看连接状态、发现/调用工具、停用或重新连接 | 本地 stdio 与远程 Streamable HTTP 为主要目标；SSE 按实际兼容对象保留；资源读取等按样本需求补齐 |
| 依赖与授权处理 | 使用前知道缺库、缺工具、未登录或不支持的平台；能完成必要授权 | 基础必需；通用字段不能描述全部依赖，结合技能正文与兼容测试记录；技能声明不自行提升权限 |
| 执行状态与用户交互 | 看见正在做什么，补充信息、停止任务、理解失败位置 | 基础必需；MCP elicitation、长任务进度等按服务需求支持。停止本地等待不等于撤销已经发生的外部操作 |
| 结果接收与核验 | 得到真实文件、链接或操作结果；失败不报完成 | 基础必需；文本/结构化结果正常保留，图片需接入视觉模型；支持的内容类型显式说明 |

条件能力随技能加入：Office 渲染/转换/公式重算、浏览器会话、截图与鼠标键盘操作、OAuth 流程等。先选样本再补依赖，不以支持全部外部生态为首版门槛。

对 RAM 与 Token 的建议：读取技能文件不需要为每个技能常驻一个进程；脚本按任务运行，MCP 进程记录启动与释放状态。目录与工具说明需要预算，长结果保留文件/引用再按需读取。具体缓存、空闲释放时间和节省幅度另行实测，不能由设计直接宣称 RAM 已下降。

## 4. 当前项目代码基础与缺口

仅核对本地代码，不将旧文档的 ship 标记当作本轮真机结果；本轮未修改这些实现文件。

| 检查点 | 代码锚点 | 当前判断 |
| --- | --- | --- |
| MCP 连接与工具注册 | `backend/mcp/client.py`：`_holder_task`、`_capability_from_external_tool` | 已有 stdio / HTTP / SSE、工具列举与调用、启停和重连，可作为复用基础 |
| Agent 执行循环 | `backend/agents/chat.py`：`stream`、`max_rounds = 5` | 已有多轮工具调用；5 轮上限需要用技能任务验证，不宜只放大数字代替任务预算与停止状态设计 |
| MCP 结果映射 | `backend/mcp/client.py`：`simplified`、`_repr` | 非文本块被变成 repr 字符串；本处理函数未消费 structuredContent。图片和结构化数据需要保真适配，才能支持相关技能 |
| 技能发现/加载 | 定向检索 backend、frontend/src 中的 `SKILL.md / skill_catalog / load_skill / read_skill` | 未找到标准技能目录与加载实现；不是声称所有历史插件均无技能逻辑 |
| 用户输入与资源扩展 | 定向检索 backend/mcp 中的 `oauth / elicitation / sampling / list_resources / read_resource / structuredContent` | 本次未找到对应专门处理；需按目标服务确认所需部分，不能以能列出工具代替完整兼容 |
| 旧扩展指南 | `docs/skills-extension-guide.md` | 讲的是 MCP、Live2D、Python capability；文件名带 skills 不代表已支持 SKILL.md 技能包 |

重跑检查入口：

```sh
rg -n 'stdio|streamablehttp|sse_client|list_tools|call_tool|_repr' backend/mcp/client.py
rg -n 'max_rounds|tool_calls' backend/agents/chat.py
rg -n 'SKILL\.md|skill_catalog|load_skill|read_skill' backend frontend/src
rg -n 'oauth|elicitation|list_resources|read_resource|structuredContent' backend/mcp
```

最后两条无匹配时 rg 返回 1，是本次查询结果，非运行故障。

## 5. 后续兼容验证建议

这是未来技术验证方法，不是已执行测试，也不提前决定其他业务功能。

1. 纯指引技能：导入→发现→自然语言/明确指定触发→读取正文；无关对话不加载全文，停用后不再可选。
2. 脚本技能：读取附带资源→检查解释器/库→执行→打开产物核验；缺依赖时报告具体缺项。
3. Skill + MCP：实际列举并调用一个选定服务；保留成功/失败和结构化结果；断开、取消、凭证失效各有清楚状态。
4. DSH Office 样本：记录原版依赖与 Momo 替代工具，验证产物；需渲染时实际接通渲染，否则标记该操作不支持。
5. 如后续选择电脑操作：验证截图确实到达模型、授权询问到达用户、操作结果可回查；不以文本工具调用成功替代验证。

兼容记录至少包含来源/版本、原样或修改、依赖、支持与不支持的操作、实测输入输出、RAM/Token/耗时及失败。可先选各一份可公开获取的 DSH / WorkBuddy 技能做验证；当前还没有“两个市场全部兼容”的结论。

## 6. 官方证据入口

- S1 [Agent Skills 格式规范](https://agentskills.io/specification)。
- S2 [DSH skill family](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/skill/README.md)；[skills subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/subsystems/skills.md)。
- S3 [DSH Office provider](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/skill/skill-office/README.md)。
- S4 [DSH office-docx 实际技能](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/skill/skill-office/assets/office-docx/SKILL.md)。
- S5 [DSH computer-use subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/subsystems/computer-use.md)；[MCP family](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/mcp/README.md)。
- S6 [WorkBuddy 开放平台：技能](https://open.workbuddy.cn/docs/skill)。
- S7 [WorkBuddy 开放平台：连接器](https://open.workbuddy.cn/docs/connector)。
- S8 [WorkBuddy 技能管理](https://intl.cloud.tencent.com/zh/document/product/1316/84503)。
- S9 [MCP 架构与结果类型](https://modelcontextprotocol.io/docs/learn/architecture)。
- S10 [MCP 客户端与用户输入](https://modelcontextprotocol.io/docs/learn/client-concepts)。

本轮另核对了 [CodeBuddy Skills](https://cloud.tencent.com/document/product/1831/137020) 的所属产品，仅用于避免混淆，不作为 WorkBuddy 全量兼容证明。WorkBuddy 旧 codebuddy.ai 文档路径本轮未返回可读正文，已改用开放平台与腾讯云官方文档。

## 7. 新版提醒、办事习惯、接管与权限

2026-11-20 核对麦麦 v1.4.8、DSH 预发布版 0.2.2-rc.5，以及下列社区插件的固定源码。项目原有麦麦调研基线为 v1.4.7。当天发布的 v1.4.8 修复了低频率下持续思考、延长触发时多次判定两项问题。升级项目依赖仍由用户选择。[麦麦发布说明](https://github.com/Mai-with-u/MaiBot/releases/tag/v1.4.8) · [DSH 发行版](https://github.com/deepseek-ai/deepseek-harness/releases/tag/dsh-v0.2.2-rc.5)

| 用户想做什么 | 现成能力和接入时要验证的地方 |
| --- | --- |
| 有新版时再决定要不要换 | DSH 在刷新插件清单时，会检查从软件包仓库安装的插件有无新版。可复用检查结果，交给 Momo 提醒。普通资料需有可核对的来源、版本或新文件，现有插件检查只覆盖插件自身。[DSH 版本检查](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.2-rc.5/packages/client/ui-plugin-manager/README.md#use-this-package) |
| 记住偏好和办事方法 | 麦麦可按需回忆明确说过的偏好。DSH 社区的 dsh-learn 0.3.13 会从纠正、失败和修正办法中提取候选，模型选中后写成技能，供下次同类任务加载。建议用“比报价时把运费算上”回放两次，看第二次能否主动把运费纳入计算。[麦麦记忆核对](INV-40-maibot-memory-and-personality.md) · [dsh-learn 的写入与使用方式](https://github.com/xiex16070-jpg/dsh-learn/blob/6a13e316e21b28b4bd00edb23c7b5982e568f71b/README.md#它是什么) |
| 操作到一半由用户接手 | Windows 社区插件 dsh-computer-use-win 在控制桌面输入时可按 Esc 停止。浏览器沿用已选的 DSH Playwright 方案，可将可见页面交给用户，交还后重新观察同一页。DSH macOS 驱动支持取消工具调用。跨设备发出“我来”、目标设备实际停手、交还后读到新状态，这三步要在 Momo 端联测。[Windows 停止设置](https://github.com/Yu-tao-Li/dsh-computer-use-win/blob/37a6a3c6170c0adecd59d9e4fd9ec7f04a3c5cb2/README.md#L134-L140) · [DSH 浏览器](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.2-rc.5/packages/experimental/browser-use-playwright-mcp/README.md) · [macOS 驱动](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.2-rc.5/packages/experimental/cua-macos/README.md) |
| 自然接话与主动陪伴 | 麦麦的私聊会把消息交给规划流程判断，回复沿用同一角色的人格与记忆。DSH 生态的游戏、镜头插件分别提供赛况和通话画面。建议由麦麦决定是否开口、如何回应，先回放场景再调节表达。[私聊与触发](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/maisaka/turn_trigger/README.md) · [游戏和摄像头接入](INV-36-scene-perception-reuse.md) |
| 谁能使用扩展、怎样授权 | 麦麦可按插件限制允许的用户或会话。管理端助手能查市场、核对版本、提出安装请求，由用户在管理界面确认。DSH 有会话权限档（只读、工作区可写、完全权限）和敏感操作的一次性确认。Momo 可复用这些入口，仍要把自动下载授权落实到实际下载动作；新版替换遵循用户的更新选择。[麦麦使用权限](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/core/plugin_usage.py#L10-L26) · [麦麦安装提议](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/webui/services/ai_search_market.py#L68-L95) · [DSH 权限档](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.2-rc.5/packages/client/ui-permission-presets/README.md#summary) · [DSH 单次确认](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.2-rc.5/packages/interaction/user-approval/README.md#summary) |

旧 Momo 还有一项可直接参考的做法。每个 MCP 工具可以单独保存“调用前确认”，下次使用时仍按该设置执行。DSH 的单次放行只适用于当次请求。两者分别覆盖用户长期设定和本次决定。[旧 Momo 的设置](../references/SOURCE-REFERENCES.md#source-9f52d9c090) · [调用前读取设置](../references/SOURCE-REFERENCES.md#source-9aa1d0cadd) · [DSH 单次放行](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.2-rc.5/packages/interaction/user-approval/README.md)

以上为上游功能与 Momo 接入建议。本轮完成文档和源码核对，跨端接管、下一次任务沿用做法和新版提醒的实际效果待接入后回放。
