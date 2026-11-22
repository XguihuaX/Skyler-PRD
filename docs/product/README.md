# Skyler 需求材料入口

当前主 PRD 为 v0.96（2026-11-22）。产品范围和主要复用方向已明确，正在准备原版与目标设备验证。基础语音、手机摄像头、游戏陪看/两种操作均保留；计划交接已确认先验证；尚无目标设备整条流程通过记录。

## 从这里读

| 你要了解什么 | 文件 |
| --- | --- |
| 产品要做什么、范围与待决 | [主 PRD](PRD-momo-next.md) |
| 某项功能已经谈定的具体行为 | [功能行为明细](SPEC-momo-next-details.md) |
| 麦麦/DSH/旧 Momo 有什么，何时才二开 | [十域复用总表](SKYLER-FUNCTION-REUSE.md) |
| 当前架构、接入和设备分工 | [架构](ARCH-001-runtime-renewal.md) · [接口草案](INTERFACE-001-frontend-backend-contract.md) |
| 微信/手机/Mac、原生语音、换机的依据 | [多端接入核对](SKYLER-CHANNELS-AND-HANDOFF.md) |
| 下一步具体做什么 | [任务清单](SKYLER-TASKS.md) · [设备验收](SKYLER-5070TI-ACCEPTANCE.md) |
| 哪些真的测过 | [目标设备验收日志](SKYLER-VALIDATION-LOG.md)；局部回放仍看对应 INV |
| 给另一台 Codex 接手 | [接手说明](HANDOFF-momo-requirements-2026-11-18.md) |
| 用户原来的例子 | [十一项案例](../../.agents/skills/momo-refactor-cases/references/cases.md) |

先读主 PRD，再按任务查行为细节和技术证据。最近可执行任务为 T-01：主机原版私聊、重启回看、Mac 同一主人会话续聊。

## 调查材料与历史

INV-29 至 INV-42 提供重构、技能/语音、场景感知、角色制作、麦麦记忆和功能的调查；更早的关联调查、F 系列旧功能、旧设计和插件目录随材料保留，作用是查证和追溯，不作为新版默认范围。

- [v0.94 完整 PRD](HISTORY-momo-prd-v0.94.md)：本次结构整理前的完整内容。
- [旧架构记录](HISTORY-runtime-renewal-20261122.md)：先前执行核心/底座比较。
- [旧交接记录](HISTORY-requirements-handoff-20261122.md)：历次增量和当时的继续位置。
- [v0.22 历史快照](HISTORY-momo-prd-v0.22.md)：已回滚阶段，仅供追溯。
- [决定日志](.memlog.md)：用户决定、建议、事实与后续纠正的记录。

## 如何更新

范围变化写主 PRD；具体行为写明细并保持链接；技术证据写 INV/架构；执行顺序与真实结果写任务/验收日志。每次更新同时核对受影响的入口，避免继续累积多个“最新状态”。

本次为文档整理与 GitHub 交接。主 PRD 中未决项目保持未决，历史中“待做”“当前”等词描述当时状态，不替代主 PRD。

专项与扩展的现成方案、待测项和接入窗口已在主 PRD §2.2 汇总，后续执行从任务清单对应窗口进入。
