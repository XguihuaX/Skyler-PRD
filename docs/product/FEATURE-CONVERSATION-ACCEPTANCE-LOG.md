# Skyler 功能对话验收日志

> 建档日期：2026-07-24\
> 当前角色：芙芙社交卡\
> 固定主测模型：Qwen 3.7 Max\
> 固定 Provider：`heyroute_cn`（前端显示“Qwen 3.7 Max · 国产线路”）\
> 用途：模型选定后，逐个功能、逐个方向保存真实人工验收对话和判定。

模型之间的选择、横向比较和历史候选回复继续记录在
[模型选择评测日志](PROMPT-MODEL-EVALUATION-LOG.md)。本文件不再比较 Terra、Sol、
Qwen-Character 等模型。

## 1. 记录规则

1. 每次固定一个功能、一个测试方向、一个模型、一个 Provider 和一个新 conversation。
2. 记录完整模型 ID、Provider、Conversation ID、时间和原始 transcript。
3. 同一 conversation 中途切换模型、Provider 或 Prompt Bundle 时立即拆 Segment；该轮不能作为干净的单模型功能验收。
4. 一轮只判一个主要方向。其他表现只做旁注，不顺带宣布多个功能通过。
5. 当前功能方向至少连续三个独立 trial 达标，再进入下一个方向。
6. 自动测试通过不等于负责人真机验收通过。

正确路由日志应包含：

```text
model=openai/qwen3.7-max
vendor=heyroute_cn
credential_source=db
endpoint_source=vendor
```

## 2. 固定主测模型决定

从 2026-07-24 起，后续功能验收固定使用 Qwen 3.7 Max，不在功能测试中途切换模型。
这表示它是当前功能开发和验收的统一基线，不等于永久关闭未来的模型成本、延迟或生产选型复评。

## 3. 历史发现 · C-004 · F-002 活动耗时连续性

该 conversation 在 11:53 与 11:55 之间发生 Provider 切换，因此不是合格的固定模型 trial，
只作为产品级反例保留。完整 transcript 见
[Conversation C-004 / Segment B-C](PROMPT-MODEL-EVALUATION-LOG.md)。

- 11:53 接近中午，角色说“准备去吃饭”符合时间语境，活动意图合理。
- 11:55 角色称已经下面并吃完，两分钟内完成整个过程不符合真实耗时。
- 失败点是活动推进过快，不是“中午想吃饭”。

## 4. 当前测试 · C-005～C-007 · F-002 活动耗时连续性

| 字段 | 内容 |
|---|---|
| 功能 | F-002 世界锚点与 DailyAgent |
| 唯一测试方向 | 角色活动是否按真实耗时推进 |
| 模型 | `openai/qwen3.7-max` |
| Provider | `heyroute_cn` |
| Conversation | C-005、C-006、C-007；每个 trial 使用新 conversation |
| 当前状态 | 待负责人执行 |

### 测试步骤

```text
用户：中午啦，你准备吃什么呀？

如果角色表示准备做饭、吃饭或等待外卖，约两分钟后继续：

用户：怎么样，已经吃完了吗？
```

### 判定

- **优秀通过**：保持合理进度，并用角色口吻自然回应，例如“两分钟哪有那么快，我才刚把面下锅”。
- **通过**：仍在决定、准备、烹饪、等待或刚开始吃。
- **失败**：已经做完并吃完，凭空切换成无关活动，或否认上一轮刚刚说过的计划。
- **本轮不判**：共情、长期记忆、主动分享、偏好成长和模型横向质量。

### Trial 结果

| Trial | Conversation | 时间 | 路由已核验 | 结果 | 备注 |
|---|---|---|---|---|---|
| 1 | C-005 |  | 否 | 待测 |  |
| 2 | C-006 |  | 否 | 待测 |  |
| 3 | C-007 |  | 否 | 待测 |  |

## 5. 后续方向队列

只有当前方向完成三个 trial 后才进入下一项：

1. F-002：活动耗时连续性；
2. F-001：跨日状态过期；
3. F-001：两小时重返线；
4. F-007：问题克制与用户边界；
5. F-002 / F-006：角色自述的状态来源与连续性。

## 6. 追加模板

```markdown
## F-XXX · 测试方向

| 字段 | 内容 |
|---|---|
| 模型 / Provider | `openai/qwen3.7-max` / `heyroute_cn` |
| Conversation ID |  |
| Segment ID |  |
| 日期与时间 |  |
| 唯一测试方向 |  |
| Prompt Bundle / Persona 版本 |  |

原始 transcript：

判定：PASS / FAIL / INCONCLUSIVE

功能级结论：

旁观现象（不计入本轮通过条件）：
```
