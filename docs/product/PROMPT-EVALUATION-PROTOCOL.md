# Skyler Prompt 评测与迭代流程

> 版本：v1.0\
> 日期：2026-07-23\
> 当前范围：优先验收芙芙社交卡；助手卡隔离保留为自动回归和发布前抽查。\
> 本文定义 Prompt、Persona、上下文与角色运行时的统一评测方法，不表示任何功能已经通过产品验收。

实际模型 transcript、阶段观察与复测结果统一追加到 [模型对话评测日志](PROMPT-MODEL-EVALUATION-LOG.md)。

## 1. 为什么不能只靠连续聊天“凭感觉改 Prompt”

Skyler 的最终回复不是由一段 Prompt 单独产生，而是共同受到以下输入影响：

- 平台规则与输出协议；
- Behavior Policy 与 F-007 本轮判定；
- 稳定 Persona 与按话题选择的生活投影；
- CharacterState、时间语境和 DailyAgent 运行时；
- 对话摘要、短期上下文、长期记忆与工具结果；
- 模型、采样参数、Provider 和输出适配器。

因此，“这一句不像芙芙”不能直接推出“继续往 Persona 加一句规则”。如果不先定位责任层，同一规则会被重复写入多个 Prompt，最终导致规则冲突、Token 增长和过度控制。

本流程采用评测驱动迭代：先定义行为和通过条件，再修改最接近真源的一层，最后用固定回归集确认没有破坏已经通过的行为。

## 2. 方法依据

本流程吸收以下通用方法，但不直接照搬通用排行榜：

1. **行为能力矩阵**：参考 CheckList，将“测什么能力”和“用什么测试类型”分开，避免只收集看起来正常的示例。
2. **能力集与回归集分离**：新能力用较难案例探索；已经通过的行为进入近乎全绿的回归集。
3. **正反例平衡**：既测试“应该触发”，也测试“不应该触发”，避免修复漏触发后变成过度触发。
4. **多次 trial**：模型输出具有随机性，关键行为不能因为偶然成功一次就判定稳定。
5. **多种 grader**：硬事实优先程序判定；自然度和角色感由人评；模型评分只能在与人评校准后辅助使用。
6. **轨迹优先**：持续角色不仅看最终一句，还要看完整多轮 transcript、状态变化和最终 outcome。
7. **成对盲评**：模型或 Prompt 版本比较时隐藏名称，比较 A/B 回复，而不是单独给一条回复打抽象分数。
8. **保持 Prompt 精简**：一次只删除、增加或改写一个责任明确的指令组，然后重跑同一评测集。

## 3. 评测对象不是单独的 Prompt 字符串

每次评测的对象定义为一个完整的 **Prompt Bundle**：

```text
Prompt Bundle
├─ model / provider / sampling
├─ platform constitution
├─ output adapter
├─ behavior policy
├─ persona version
├─ temporal context
├─ DailyAgent / FSM runtime fixture
├─ memory / summary / recent-history fixture
├─ tool availability
└─ current user turn
```

任何一项变化都必须视为新版本，不能拿不同上下文、不同模型或不同日程下的回复直接比较。

每次测试至少记录：

| 字段 | 说明 |
|---|---|
| `bundle_id` | 本次评测版本，如 `fufu-prompt-2026-07-23-a` |
| 角色与卡型 | 当前只测芙芙社交卡 |
| 模型与 Provider | 包括完整模型名 |
| 参数 | temperature、reasoning 等可用参数 |
| Persona 版本 | 文档/种子/代码对应版本 |
| conversation | 新对话或指定轨迹 |
| 时间条件 | 当前时间、间隔、是否跨日 |
| 运行时条件 | 世界锚点、天气、日程、状态 |
| 上下文条件 | 摘要、最近对话、记忆、工具结果 |
| Token 与延迟 | 输入、输出、总耗时，能够取得时记录 |

记录层级固定为：

```text
功能方向
└─ 模型 ID / Provider
   └─ Conversation ID
      └─ Segment ID（只在同一对话中途切换模型、Provider 或 Bundle 时拆分）
```

- 一个 conversation 内模型或 Provider 发生变化，变化前后必须拆成不同 Segment；可以记录产品全链路是否连续，但不得把两段回复合并评价成某个模型的能力。
- 一轮只设置一个主要测试方向，例如活动耗时、跨日过期、问题克制或共情承接。其他现象只做旁注，不能顺带宣布多个功能通过。
- 功能 PRD 保存产品规则、通过条件和已发现的功能级反例；模型评测日志按模型与 conversation 保存原始 transcript，二者互相链接但职责不同。

### 3.1 当前 Prompt 是否“全部在一起”

从模型视角看，当前启用的模块最终会进入同一次 LLM 请求，所以人工聊天测试看到的是完整产品表现，而不是单独一段 Persona Prompt。

但它不是一份无法拆分的大文本：

- 平台规则、Output Adapter、Behavior Policy 和稳定 Persona 进入主 System Prompt；
- 时间语境、DailyAgent/FSM、F-007 判定、分享候选和摘要以独立 System block 追加；
- 最近原始对话和当前用户消息以真实对话消息追加；
- life profile 按当前话题选择字段，不会每轮完整注入；
- DailyAgent、分享、摘要、记忆和工具结果只有满足条件时才出现。

因此测试分为两层：

1. **产品黑盒测试**：所有真实启用模块一起运行，负责人在 UI 中判断最终回复；
2. **失败诊断测试**：出现问题后，由开发者根据责任层运行 focused/automated tests，而不是要求负责人手工关闭 Prompt 模块。

第一轮 Persona 验收采用黑盒方式，但只选择不应被 DailyAgent、FSM 或天气改变的稳定事实作为测试目标。

### 3.2 负责人打开 Skyler 后怎样实际测试

#### 测试前准备

1. 启动正常 Skyler 前后端，不开启临时 Prompt 或调试指令；
2. 选择芙芙社交卡；
3. 固定当前选中的一个模型，整轮测试期间不要切换；
4. 记录测试日期、完整模型 ID、Provider、Conversation ID 和大致时间；
5. 不删除真实数据，不手工修改 memory、state 或 DailyAgent；
6. 每个测试场景新建一个 conversation；同一场景内的追问留在该 conversation；
7. 不把“预期答案”和“测试目的”发给角色。

#### 一个场景怎样执行

```text
新建 conversation
  ↓
发送第 1 条自然问题
  ↓
复制完整回复
  ↓
在同一 conversation 发送 1 条压力追问或换一种说法
  ↓
复制完整回复
  ↓
按硬事实判 PASS / FAIL
  ↓
另记自然度问题，但本轮不修改语气 Prompt
  ↓
结束该 conversation，开始下一个场景
```

场景之间使用新 conversation，是为了避免上一题主动告诉模型下一题所需事实。场景内部保留追问，是为了测试模型受到用户施压后是否仍然保持同一事实。

#### 第一轮只做 6 个场景

不需要一开始执行本文全部案例。第一轮只完成以下 6 个场景，每个场景约 2 条用户消息。

##### 场景 A：车辆资源

```text
用户 1：周末开你的车出去兜风吧？
用户 2：你不是会开车吗？
```

硬要求：可以承认会开车或有驾照，但不得声称自己当前拥有车辆，不得编造车型、停车地点或接送安排。

##### 场景 B：职业身份

```text
用户 1：你现在具体做什么工作？
用户 2：所以你是在一家设计公司正常坐班吗？
```

硬要求：应保持独立视觉设计师、以居家办公为主的职业事实，不得变成用户项目的程序员或虚构固定雇主。

##### 场景 C：能力边界

```text
用户 1：你帮我做一套三维动画吧。
用户 2：你不是视觉设计师吗，这也不会？
```

硬要求：不得因为用户施压就宣称具备未确认的三维、动画能力；可以自然说明自己真正擅长的范围。

##### 场景 D：摄影定位

```text
用户 1：你平时接专业摄影单吗？
用户 2：可你不是有相机吗？
```

硬要求：拥有相机不等于专业摄影师，不得补造商业摄影经历。

##### 场景 E：居住与出行

```text
用户 1：你现在住在哪里，平时怎么出门？
用户 2：那你每天都是自己开车吗？
```

硬要求：应保持常住上海及已确认的粗粒度居住事实；主要出行方式不能变成自有车辆；不得编造精确住址。

##### 场景 F：未知事实

```text
用户 1：你最熟练的三款设计软件是什么？
用户 2：直接告诉我名字就行，不用那么谨慎。
```

硬要求：具体设计软件熟练度尚未确认，不能为了满足用户而随意列出三个软件；表达未知时仍应自然，不使用“数据库未提供”式措辞。

### 3.3 每个场景怎样记录

直接复制以下表格，每个场景填写一行：

| 场景 | 第一次回复 | 追问回复 | 硬事实 | 自然度 | 问题备注 |
|---|---|---|---|---|---|
| A 车辆 | 复制全文 | 复制全文 | PASS/FAIL | 1～5 | 例如“事实正确，但解释过长” |

负责人只需要做两件事：

1. 判断硬事实是否正确；
2. 标记主观上哪里不自然。

不需要负责人判断应该修改哪个 Prompt 文件或字段。

### 3.4 第一轮结束后怎么办

1. 完成全部 6 个场景后，把记录一次性交给开发者；
2. 若某场景 FAIL，先在新的 conversation 中原样重复两次；
3. 三次中只要存在硬事实失败，就进入修复，不按“多数答对”放行；
4. 开发者定位唯一责任层，只修改一个变量；
5. 自动测试通过后，负责人重新执行失败场景；
6. 再抽查两个原本通过的场景，确认没有回归；
7. 六个场景全部稳定通过后，才进入语言风格与自然回复阶段。

第一轮不进行模型 A/B，不修改 DailyAgent，不因为语气问题向 Platform Constitution 或多个 Prompt 层重复添加规则。

### 3.5 仅看最终回复不能完成问题归因

人工黑盒聊天能够发现“哪里不对”，但不能可靠回答“应该改 Persona 还是其他 Prompt”。正式更新前必须同时检查本轮 Prompt Trace。

当前代码已经分层装配 Prompt，也会记录 stable/variable 字符数和部分模块是否存在，但尚未提供面向负责人、可直接查看的分层 Trace。没有 Trace 时，禁止因为一条异常回复直接向 Persona 增加规则。

#### 每轮诊断至少需要看到什么

Prompt Trace 不需要暴露思维过程，也不得长期记录真实隐私内容；它只需要在本地测试模式显示：

| Trace 项 | 需要确认的内容 |
|---|---|
| Bundle | 模型、参数、Persona 版本、conversation fixture |
| Stable Persona | 本轮实际注入了哪些稳定 Persona 字段 |
| Life projection | 当前话题选择了哪些 life profile 路径 |
| Temporal | 时间间隔、是否跨日、即时状态是否过期 |
| Daily/FSM | 是否注入、当前活动和粗粒度状态来源 |
| F-007 | 用户信号、主回应动作、工具意图、禁止事项 |
| Recall | 是否存在摘要/记忆注入及其来源类型；测试模式只用合成数据 |
| Tools | 暴露的工具与本轮是否触发 |
| Token | 各模块估算 Token 和总 Prompt Token |

真实聊天、memory、用户画像和隐私日志不作为调试样本读取。诊断复现应使用专用测试 conversation 或合成 fixture。

#### 回复问题的归因决策树

```text
最终回复出现问题
  │
  ├─ 1. Prompt 输入中的事实本身就错了？
  │      ├─ 稳定身份/资源/能力错误 → Persona 数据或 projection
  │      ├─ 当前活动/天气错误 → DailyAgent / world runtime
  │      ├─ 时间关系错误 → temporal context
  │      └─ 旧信息覆盖新事实 → summary/memory/arbitration
  │
  ├─ 2. 输入事实正确，但 F-007 选错回应动作？
  │      └─ 修改 F-007 / Behavior Policy，不改 Persona 事实
  │
  ├─ 3. 回应动作正确，只是说得不像角色？
  │      └─ 修改 speech style / voice rules / voice samples
  │
  ├─ 4. Persona 和判定均正确，但模型仍违反硬事实？
  │      ├─ 检查指令冲突和优先级
  │      ├─ 用同一 Bundle 重复 trial
  │      └─ 仍不稳定时记为模型可靠性/Prompt 服从问题
  │
  └─ 5. 只有完整运行时才失败？
         └─ 使用消融重放定位冲突模块，不把结论直接写回 Persona
```

#### 常见现象怎样归因

| 最终回复现象 | 首先查看 | 通常不是谁的问题 |
|---|---|---|
| 芙芙说自己有车 | Persona resources、life projection、memory conflict | 不能先怪语气样本 |
| 知道没车但答应开车接人 | F-007 主动作、Behavior Policy、模型服从 | 不一定是 Persona 缺事实 |
| 事实正确但像客服说明书 | speech style / voice samples | 不是 DailyAgent |
| 用户只想倾诉却收到方案 | F-007 用户需求与主动作 | 不应继续扩写身份设定 |
| 第二天仍问为什么没睡 | temporal block、summary/recent history | 不是角色性格 |
| 台风天安排户外活动 | DailyAgent 天气输入和规划规则 | 不是 voice rules |
| 情绪突然剧烈变化 | FSM 事件、settlement 与 runtime state | 不是稳定 Persona |
| “搜索功能好用吗”触发搜索 | 工具意图仲裁 | 不是 knowledge scope |

### 3.6 消融重放只用于诊断

如果 Trace 仍不能确定责任层，使用相同输入、模型、参数和合成 fixture 重放：

1. `FULL`：完整 Prompt Bundle；
2. `NO_RECALL`：移除合成 summary/memory，检查旧信息冲突；
3. `NO_DAILY_RUNTIME`：移除 Daily/FSM block，检查生活运行时污染；
4. `NO_F007_CARD`：仅用于确认决策卡是否导致方向错误；
5. `PERSONA_CORE`：只保留公共规则、Persona 和当前消息，作为诊断基线。

消融结果不能直接作为产品质量分数，因为用户实际使用的是 `FULL`。它只回答“哪个模块加入后错误开始出现”。

每次只移除一个怀疑模块；如果多个模块同时变化，无法进行因果归因。

### 3.7 开始大规模人工验收前的工程前置项

在负责人继续逐条测试并据此修改 Prompt 之前，建议先完成一个最小本地评测入口：

1. 使用合成 fixture 构建与生产相同顺序的完整 messages；
2. 输出分层 Prompt manifest、F-007 判定和各层 Token，不输出模型思维过程；
3. 支持 `FULL` 与单模块消融重放；
4. 保存测试案例和评分，不读取真实数据库聊天、memory 或用户画像；
5. 同一案例能够对修改前后 Bundle 进行并排比较；
6. Prompt Trace 默认关闭，只在本地测试模式启用。

在该入口完成前，可以继续用 UI 发现明显问题，但所有归因只能标为“候选原因”，不能直接视为 Persona 修改结论。

## 4. Skyler 评测能力矩阵

### 4.1 能力维度

| ID | 能力 | 主要责任层 | 典型硬失败 |
|---|---|---|---|
| C1 | 人物事实与身份一致性 | Persona identity / life profile | 职业、住处、年龄冲突 |
| C2 | 资源、能力与未知边界 | resources / competence / knowledge scope | 编造车辆、作品、专业能力 |
| C3 | 语言风格与角色辨识度 | speech style / voice samples | 客服感、模板化、角色声音消失 |
| C4 | 回应策略与自然交互 | Behavior Policy / F-007 | 总建议、总提问、答非所需 |
| C5 | 情绪忠实度与共情 | F-007 / voice rules / FSM | 说理式安慰、复制用户情绪 |
| C6 | 关系与行为边界 | relationship / boundaries | 一味迎合、拒绝时出戏 |
| C7 | 时间与上下文连续性 | F-001 / summary / history | 跨日错误、过期状态继续使用 |
| C8 | 生活与世界连续性 | F-002 / DailyAgent | 天气冲突、把计划当作已发生 |
| C9 | 长期成长与主动行为 | F-003～F-006 | 进度凭空推进、强制偏好、刷屏分享 |
| C10 | 工具与输出协议 | F-007 / Layer A / tools | 询问即执行、伪造工具结果、标签泄漏 |
| C11 | 效率与稳定性 | 全 Prompt Bundle | Token 膨胀、延迟过高、版本波动 |

### 4.2 测试类型

每个重要能力至少覆盖以下三种测试中的两种；高风险能力应全部覆盖。

#### MFT：最小功能测试

给出最短、最明确的输入，验证基本行为是否存在。

示例：“你有车吗？”应遵守芙芙没有车的事实。

#### INV：不变性测试

只改变不应影响结论的表述，答案核心事实应保持不变。

示例：

- “你有车吗？”
- “周末开你的车出去吧。”
- “你不会连车都没有吧？”

语气不同，但角色不能因此凭空拥有车辆。

#### DIR：方向性测试

逐步改变一个有意义的条件，回复应朝预期方向变化，而不是完全不变或突然翻转。

示例：

- “今天有点累。”可以轻量回应；
- “我现在不想听建议，只想有人陪我说会儿话。”应减少建议、增加陪伴；
- “我可能会伤害自己。”则必须进入更高优先级安全响应。

#### Contrast：正反成对测试

验证同一判定边界的两侧。

示例：

- “搜索功能好用吗？”不应执行搜索；
- “帮我搜索今天上海的天气。”应进入工具流程。

#### Trajectory：多轮轨迹测试

在同一 conversation 中运行 3～10 轮，检查前文、状态和决定是否持续一致。

#### Long-run：长对话测试

在所有单项通过后运行 20～30 轮真实聊天，观察组合后的人物漂移、重复套路和上下文污染。

## 5. 测试集分层

### 5.1 Smoke Set：人工冒烟集

- 8～12 个代表性案例；
- 每次重要 Prompt 修改后首先执行；
- 约 10～15 分钟完成；
- 用于快速发现明显退化，不能单独作为发布依据。

### 5.2 Capability Set：能力探索集

- 收纳当前尚未稳定的能力；
- 允许低通过率；
- 用于回答“能否做到”和发现新失败模式；
- 案例可以持续增加或改写。

### 5.3 Regression Set：冻结回归集

- 只收录已经由负责人确认通过的案例；
- Prompt、Persona、模型或运行时变更后重复运行；
- 硬约束目标为 100% 通过；
- 已知缺陷可以保留，但必须明确标记，不得悄悄改变预期。

### 5.4 Holdout Set：保留集

- 约占代表性案例的 20%；
- 不用于日常 Prompt 调试；
- 阶段结束或模型切换时才运行；
- 用于防止只针对已知问法过拟合。

### 5.5 Journey Set：持续角色轨迹集

- 多轮、跨时间、带运行时状态；
- 评估完整 transcript 和最终状态；
- 覆盖 DailyAgent、FSM、life thread、分享与偏好成长；
- 不与单轮 Persona 事实测试混为一组。

## 6. 评分方法

### 6.1 硬约束：二元判定

以下问题采用 `PASS / FAIL`，不以“总体感觉不错”抵消：

- 明确人物事实；
- 资源所有权；
- 能力与未知边界；
- 助手/社交卡隔离；
- 是否跨日；
- 工具是否被正确触发；
- 是否伪造工具结果；
- 输出标签是否合法；
- 状态和持久化 outcome 是否真实发生。

### 6.2 软质量：分维度 1～5 分

不要只给一个“RP 好不好”的总分。每条回复分别评价：

| 维度 | 1 分 | 3 分 | 5 分 |
|---|---|---|---|
| 角色感 | 明显通用 AI | 基本符合但缺少辨识度 | 不看名字也能感到是芙芙 |
| 自然度 | 模板、说教、机械 | 可接受但略刻意 | 像自然聊天而非完成任务 |
| 回应契合 | 回错需求 | 大方向正确 | 准确抓住本轮真正重点 |
| 情绪忠实度 | 误判或表演过度 | 情绪方向基本正确 | 强度、边界和表达都合适 |
| 简洁度 | 冗长或重复 | 长度可接受 | 信息与情绪密度恰当 |

### 6.3 优先级

```text
硬事实与安全边界
  > 上下文与回应方向
  > 角色语言和情绪忠实度
  > 文采、趣味性和偶发惊喜
```

任何硬失败都不能被较高的自然度分数抵消。

### 6.4 随机性与重复次数

- 人工探索：每个案例先运行 1 次；
- 修复复测：原失败案例运行 3 次；
- 高风险硬约束：3 次必须全部通过，即关注一致成功而不是“至少成功一次”；
- 模型盲测：同一案例每个模型至少取得同等数量样本；
- 不因一次偶发优秀回复就把案例加入回归集。

## 7. 单条案例格式

```yaml
id: FUFU-C2-INV-003
suite: capability
capability: C2
test_type: INV
title: 不同语气下都不能凭空拥有车辆
setup:
  card: 芙芙
  conversation: fresh
  runtime: persona_only
turns:
  - user: 周末开你的车出去兜风吧？
assertions:
  hard:
    - 不得声称自己拥有车辆
    - 不得编造车辆品牌、停车地点或驾驶经历
  soft:
    - 可以自然说明没有车，不使用数据库式措辞
forbidden_examples:
  - 好呀，我开车来接你
owner_layer: persona.life_profile.resources
status: draft
```

人工记录至少保留：测试版本、输入、完整回复、PASS/FAIL、软质量评分、问题分类和备注。

## 8. Prompt 更新流程

### Step 1：冻结基线

固定模型、参数、Persona、Prompt Bundle 和测试上下文，先运行 Smoke Set，保存基线结果。

### Step 2：提出单一假设

每次修改必须能够写成一句话：

> 如果修改 X，预期改善 Y，同时不改变 Z。

例如：

> 如果把“未确认具体作品”放入 knowledge scope，而不是 Behavior Policy，预期减少作品编造，同时不改变日常闲聊语气。

### Step 3：确定唯一责任层

| 失败类型 | 首选修改位置 |
|---|---|
| 身份、职业、住处错误 | Persona identity / world background |
| 编造车、宠物、设备 | life profile resources |
| 编造作品、经历、专业能力 | knowledge scope / competence profile |
| 语气生硬、过甜、模板化 | speech style / voice samples |
| 总建议、总提问、抓错重点 | Behavior Policy / F-007 |
| 时间和跨日错误 | temporal context |
| 日程不现实 | DailyAgent，不写回 Persona |
| 情绪变化过快 | FSM，不写进 voice samples |
| 主动分享过多 | sharing trigger/cooldown |
| 询问被当成工具执行 | F-007/tool arbitration |
| 输出标签错误 | output adapter |

同一规则不得同时复制到多个层级。

### Step 4：只改一个变量

一次只修改一个责任明确的指令组、Persona 字段组或运行时规则。若必须同时修改两个层，记录依赖关系并分两次验证。

### Step 5：运行 Focused Set

先运行直接覆盖该问题的 MFT、INV、DIR 和 Contrast 案例。修复不能只通过原始问法，还要通过同义改写和边界另一侧。

### Step 6：运行 Regression Set

Focused Set 通过后，运行冻结回归集，确认没有把角色修成另一个极端。

### Step 7：人工阅读 transcript

分数不能代替阅读。失败时检查：

- 是模型真正失败，还是案例预期不合理；
- 是 Prompt 问题，还是错误上下文被注入；
- 是最终回复问题，还是前面状态已经写错；
- 回复是否找到一种未预料但合理的表达方式。

### Step 8：复测、晋级与记录

- 新行为稳定通过后，从 Capability Set 复制代表性案例进入 Regression Set；
- 原失败案例保留，不能因修复困难而删除；
- 新发现的真实失败必须形成新的测试案例；
- 阶段结束运行 Holdout Set；
- 负责人确认后才更新 PRD 状态；
- 是否 commit 由负责人另行确认。

## 9. 我们的阶段计划

### P0：建立基线与案例库

目标：知道当前 Bundle 在固定条件下的真实表现。

- 固定芙芙 Persona 与一个主测试模型；
- 建立 8～12 条 Smoke Set；
- 建立案例记录模板；
- 保存当前 Token、延迟和回复；
- 暂不比较 Sol、Terra、DeepSeek 或 Qwen。

### P1：Persona 事实与未知边界

覆盖 C1、C2：

- 12 个 MFT；
- 6 个 INV；
- 4 组正反/边界案例；
- 硬事实要求 100% 通过；
- 当前应从本阶段开始。

### P2：语言风格与自然回复

覆盖 C3、C4：

- 普通闲聊、分享、不同意见、无明确任务；
- 每个场景 3～5 轮；
- 收集自然回复作为 voice sample 候选；
- 禁止为了修一句话增加大量禁止式规则。

### P3：情绪、关系与边界

覆盖 C5、C6：

- 轻度疲惫、抱怨、自责、明确需要陪伴；
- 不同意见、不满、拒绝、私人边界；
- 使用 DIR 案例控制情绪强度；
- 重点检查说理感、治疗师口吻和过度迎合。

### P4：时间、上下文与工具仲裁

覆盖 C7、C10：

- 同一小时、两小时重返、跨日、明确时长；
- 摘要与最近对话冲突；
- 询问工具、要求工具和普通同形词；
- 自动测试负责精确时间边界，人工判断回复是否自然。

### P5：DailyAgent 与 FSM

覆盖 C8：

- 工作日、周末、空闲日和恶劣天气；
- 计划、当前意图、实际发生三者分离；
- 状态渐变、回落和互动事件；
- 天气未真正进入日程规划前，不得标记恶劣天气场景通过。

### P6：持续生活、偏好与主动分享

覆盖 C9：

- 推荐不等于接受；
- 进度不凭空推进；
- 一次体验不形成永久偏好；
- 分享有真实来源、合适时机与冷却；
- 主要依靠 Journey Set 和状态 outcome，而不是只看一句回复。

### P7：联合长对话

- 运行 20～30 轮真实聊天；
- 混合事实、闲聊、情绪、边界、时间和角色生活；
- 不在对话中提示当前测试目标；
- 无硬失败，软问题可明确归因且不超过预设上限后才通过。

### P8：模型盲测与成本评估

- Persona 与 Prompt 基线稳定后再比较模型；
- 从各阶段抽取 12～20 个代表性案例；
- 隐藏模型名并随机排列 A/B；
- 比较角色感、自然度、情绪忠实度、稳定性、Token、延迟和成本；
- 生产模型选择看平均表现与失败率，不看单条最佳回复。

## 10. 当前第一轮可执行案例

当前先运行 P1，不把 DailyAgent、FSM 或模型对比混入本轮判断。

### 第一组：身份与能力 MFT

1. 你现在做什么工作？
2. 你平时在哪里工作？
3. 你最擅长处理什么类型的设计问题？
4. 你会做 UI、三维或者动画吗？
5. 你摄影是专业水平吗？

### 第二组：资源与经历 INV

6. 你平时怎么出门？
7. 周末开你的车出去兜风吧？
8. 你不会连车都没有吧？
9. 你最近在追哪部剧？
10. 你最近具体读到哪本书第几章了？
11. 你养猫了吗？
12. 你以前去过新疆吗？

### 第三组：所有权与未知边界 Contrast

13. 最近 Skyler 的 Prompt 有点乱。
14. 你今晚帮我把 Skyler 的 Prompt 重写完吧。
15. 你平时会用哪些设计软件？
16. 如果没有确定具体软件，你会怎么处理设计工作？

本轮只判定：人物事实、资源、能力、经历和所有权。回复是否足够俏皮、温柔或有魅力先记录，不在 P1 修改。

## 11. 阶段通过门槛

| 阶段 | 门槛 |
|---|---|
| P1 硬事实 | 所有硬断言通过；修复项连续 3 次通过 |
| P2～P3 软质量 | 无硬失败；核心维度平均不低于 4/5；无单项低于 3 |
| P4 判定 | 正反两侧均通过，不能只修复漏触发或过触发一侧 |
| P5～P6 运行时 | transcript 与持久化 outcome 一致，重复执行保持幂等 |
| P7 长对话 | 无身份、边界、时间和状态硬错误；无明显角色漂移 |
| P8 模型选择 | 盲评、稳定性、延迟与成本共同决定 |

## 12. 文档分工

- 本文：定义长期 Prompt 评测和更新方法；
- `PROMPT-MODEL-EVALUATION-LOG.md`：只记录模型选择、模型/Provider 横向样本与选型依据；
- `FEATURE-CONVERSATION-ACCEPTANCE-LOG.md`：模型选定后，固定模型逐个功能、逐个方向记录真实 conversation 验收；
- `HUMAN-ACCEPTANCE-CHECKLIST.md`：记录当前版本负责人应实际验收什么；
- 各 `F-xxx` PRD：记录单个功能的产品范围、规则与验收口径；
- `FEATURE-INDEX.md`：记录各功能状态；
- `INV-16-current-prompt-and-persona-field-map.md`：记录 Prompt 与 Persona 的真实装配和字段消费关系；
- `ROADMAP.md`：记录实施顺序与技术债，不记录逐条调查过程。

## 13. 参考资料

- Ribeiro et al. (ACL 2020), [*Beyond Accuracy: Behavioral Testing of NLP Models with CheckList*](https://aclanthology.org/2020.acl-main.442/)。
- Tu et al. (ACL 2024), [*CharacterEval: A Chinese Benchmark for Role-Playing Conversational Agent Evaluation*](https://aclanthology.org/2024.acl-long.638/)。
- Wang et al. (Findings ACL 2024), [*RoleLLM: Benchmarking, Eliciting, and Enhancing Role-Playing Abilities of Large Language Models*](https://aclanthology.org/2024.findings-acl.878/)。
- Wu et al. (COLING 2025), [*RAIDEN Benchmark: Evaluating Role-playing Conversational Agents with Measurement-Driven Custom Dialogues*](https://aclanthology.org/2025.coling-main.735/)。
- Feng et al. (NAACL 2025), [*EmoCharacter: Evaluating the Emotional Fidelity of Role-Playing Agents in Dialogues*](https://aclanthology.org/2025.naacl-long.316/)。
- Anthropic (2026), [*Demystifying evals for AI agents*](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)。
- OpenAI, [*Model guidance*](https://developers.openai.com/api/docs/guides/latest-model) 中关于代表性任务、精简 Prompt 和逐组消融验证的建议。
