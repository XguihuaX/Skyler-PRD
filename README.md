# Skyler 产品需求与研究材料

这里集中维护 Skyler（原 MomoOS-v2）的需求、复用依据和后续验证任务。**当前主 PRD：v0.95，2026-11-22。** 产品范围与主要复用方向已经整理，可以开始麦麦原版验证；目标设备的整条流程仍待实测。

## 先读什么

1. [主 PRD](docs/product/PRD-momo-next.md)：产品目标、十个功能域、计划交接、待决与后续顺序。
2. [功能行为明细](docs/product/SPEC-momo-next-details.md)：保留原来谈过的语音、记忆、提醒、操作、角色等具体行为。
3. [执行任务](docs/product/SKYLER-TASKS.md)：最近从 T-01 的主机与 Mac 同一主人会话开始。
4. [设备验收](docs/product/SKYLER-5070TI-ACCEPTANCE.md)与[验收日志](docs/product/SKYLER-VALIDATION-LOG.md)：怎么测，以及哪些真的测过。

完整导航：[需求材料入口](docs/product/README.md)。给另一台 Codex：[接手说明](docs/product/HANDOFF-momo-requirements-2026-11-18.md)。

## 当前方向

麦麦作为 Skyler 主程序，已有能力直接使用，适合 DSH 的任务按需委托。旧 Momo 的客户端、角色和声音资产按实际缺口复用。

基础语音、手机摄像头、主屏与消息感知、任务/文件/网页、游戏陪看与两种游戏操作保留首轮范围。多人语音等独立测试，Life、DailyAgent、多角色和运行中追加任务要求等保持后置。计划交接先验证“出差前主机交给 Mac、回家后交回”。

## 材料分层

| 材料 | 用途 |
| --- | --- |
| [复用总表](docs/product/SKYLER-FUNCTION-REUSE.md) | 哪些已有、哪些先试、哪些路径有明确差异 |
| [当前架构](docs/product/ARCH-001-runtime-renewal.md)、[接口草案](docs/product/INTERFACE-001-frontend-backend-contract.md) | 分工、输入输出与状态；未冻结为全部实现接口 |
| [多端与语音依据](docs/product/SKYLER-CHANNELS-AND-HANDOFF.md) | 微信、手机、Mac、原生语音和换机机制的核对 |
| [十一项案例](.agents/skills/momo-refactor-cases/references/cases.md) | 用户情境与预期；未实测项明确保留 |
| docs/INV-*、docs/research/ | 上游版本、源码依据、局部回放和插件目录 |
| HISTORY 文件、旧 F 系列及设计 | 历史决定与旧实现背景，按原日期理解 |
| [来源清单](references/PACKAGE-MANIFEST.json)、[源码引用索引](references/SOURCE-REFERENCES.md) | 材料来源和未随包代码/本地证据的定位 |

## 双机协作

主机与 Mac 使用相同文档提交，按运行设备分工并将真实结果写入验收日志。当前服务端、客户端设备和模型运行位置分别记录。文档/代码 Git 同步与真实业务数据交接是两项操作。

本仓库不含旧产品运行时代码、运行配置、密钥、真实聊天数据、录音或模型。原工作区源码有未提交改动，相关引用保留路径索引，不假定 GitHub 上旧代码与本地完全相同。

## 本次整理

主 PRD 保留范围与关键规则，详细行为移入配套规格；旧架构/交接和 v0.94 全文各留快照。新增 T-01 至 T-09 任务、空的真实验收日志和多端接入说明。已完成决定对照、BMad 内容及结构/语言核对，具体检查见[整理检查记录](references/REVIEW.md)。
