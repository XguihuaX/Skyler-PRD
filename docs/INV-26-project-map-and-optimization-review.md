# INV-26 · 全项目复查与功能导航治理

核对日期：2026-10-13。范围：当前工作区代码、测试、文档与旧图谱；含未提交改动。本轮未启动完整应用、未访问用户聊天内容、未调用真实日历或外部工具执行任务。

> 后续处理见 [INV-27](INV-27-lifecycle-and-regression-gates.md)：生命周期、旧测试、编码复用与无调用辅助链已完成定向验证。本文保留当时调查状态；旧行号按历史快照解读。

## 1. 结论

全项目仍有优化空间，下一阶段应优先提高测量与测试的可信度、生命周期可靠性，再评估记忆与工具成本。单纯减行数不能证明效率改善。

供后续 agent 接手的产物命名为 **项目架构与功能导航图（Codebase Map / Architecture Overview）**：[PROJECT-MAP](PROJECT-MAP.md)。包含系统关系、功能入口、核心符号、数据来源、测试定位、实现状态和维护规则；生成的机器图谱提供进一步的文件/符号依赖查询。

本文保存调查与验证证据；实施状态和后续候选顺序记入 [ROADMAP](../ROADMAP.md)。不把本次发现的全部问题写成“已修复”。

## 2. 新发现与优先级

| 优先级 | 发现及证据 | 影响与建议验证 | 本轮状态 |
|---|---|---|---|
| 高 | [system.collect](../references/SOURCE-REFERENCES.md#source-e596941acb) 原来每次 `psutil.Process(os.getpid())` 再调用 `cpu_percent(None)`，丢失对象上的上次采样 | 同进程忙循环下，新对象三次均 0.0%，复用对象为 99.4/99.6/95.6%。先修测量，才能比较后续 CPU 优化 | **已修并验证**：保留采样对象；PID 变化时重建；异常与无 psutil 仍报告未知 |
| 高 | [test_integration.py:138](../references/SOURCE-REFERENCES.md#source-3fceb03e2c) 导入 `backend.agents.memory`，`:214` 导入 `backend.scheduler.task`；两个模块均不存在 | 旧脚本继续使用退役 MemoryAgent/AlarmScheduler/API，不能作当前集成门禁。应分类 isolated / integration / external / retired，逐步迁移有效断言 | 已确认，尚未改测试语义；导航中明确标记 |
| 高 | [main.py:355](../references/SOURCE-REFERENCES.md#source-df5184bf1f)、`:383–384`、`:504`、`:594` 的 5 个 `create_task` 表达式未保存 task 引用；`:786–823` 的通用清理位于 `yield` 所在 try/finally 之后 | MCP 关闭有 finally；其他清理在 lifespan 异常向外传播时可能跳过。统一任务所有权和外围 finally，并以启动中断、catch-up 失败、关闭异常注入验证；取消 asyncio task 也不等于能强停其线程内模型加载 | 静态控制流风险，**未复现真实泄漏或重启故障**；未改运行时 |
| 中高 | [main.py:242](../references/SOURCE-REFERENCES.md#source-df5184bf1f) 启动时枚举用户全部 `(character_id, conversation_id)`，逐桶分页恢复 4000-token 窗口 | 当前开销随历史会话桶数量增长。可比较“仅当前会话恢复，其余首次访问恢复”；必须保持分桶、摘要和最近原文契约 | 规模风险，**未取得大规模启动耗时证据**，不提前改策略 |
| 中高 | 旧知识图谱元数据为 2026-07-10 / `d9a11ab5`；[DailyAgent 模块](../references/SOURCE-REFERENCES.md#source-ba51ee23ae) 修改前顶部称单角色 Stage1，[ticker](../references/SOURCE-REFERENCES.md#source-72d43fcc5d) 修改前顶部称纯 DB / 0 LLM，但已有多角色调度和 `ensure_activity_instance` | 新 agent 容易错判能力和后台费用。导航需附时间/状态/真源，图谱要包括工作区，并排除生成物自身 | **已补导航和入口，已纠正两个模块说明**；没有改这两个模块的可执行逻辑 |
| 中 | [observability/system.py](../references/SOURCE-REFERENCES.md#source-e596941acb) 只采 backend RSS，Tauri、WebView、MCP 与模型进程不在总量内 | 先补进程树测量基线，再考虑 ASR/模型卸载。无总量证据不能把入口 JS 变小称为整机 RAM 降低 | 测量边界已写入导航；方案与 [INV-25](INV-25-commercial-assistant-research.md) 对齐 |

复查同时确认下列原有优化候选仍存在：

- **抽取失败恢复：**[extractor.py:366](../references/SOURCE-REFERENCES.md#source-9abb47cb4b) 调用处理后推进游标；处理函数遇 `raw is None` 返回。现有测试明确接受失败后前进以避免卡死。候选是有限重试 + 失败记录 + 可重放，而不是无限重试或无条件停住游标。
- **检索与去重：**[long_term.py:307](../references/SOURCE-REFERENCES.md#source-521d60a11a) 的短输入门槛可能漏掉“上次那件事”等指代；[validator.py:196](../references/SOURCE-REFERENCES.md#source-3389389ff0) 对旧内容逐条编码。先建立短句/纠正/跨会话召回样本与规模基线，再考虑复用已存向量。当前记忆规模不足以支持“必须上独立向量库”的结论。
- **工具上下文成本：**[chat.py](../references/SOURCE-REFERENCES.md#source-6a784c600a) 的 `_get_all_tools` 仍组合记忆工具与全部已注册 schema；`ChatAgent.stream` 每轮提供 schema。`tool_addendum` 的固定文案不能代表动态工具状态。按需选择必须测漏选率、补召回率、成功率、Token 与延迟，不能只测省了多少字。
- **职责拆分：**当前 `chat.py` 2350 行、`ws.py` 1580 行、`main.py` 929 行、`SettingsPanelLegacy.tsx` 1636 行。行数只是导航复杂度信号；先确定会话状态、工具执行、上下文预算等边界并补契约，再按边界拆分。运行中的兼容分支不能仅凭名称删除。
- **低优先旧辅助路径：**`daily_plan._iso_to_hhmm` 对有效字符串缺少返回，解析代码位于 `_format_yesterday_rhythm` 的 return 后，无法执行。当前 `build_daily_plan_prompt` 有意忽略用户日历，因此不能把此问题描述为当前 F-002 主链故障；重用旧日历 formatter 前应补契约并修复或退役。本轮仅记录。

## 3. 本轮具体改动

1. 新增 [PROJECT-MAP](PROJECT-MAP.md)：16 类功能导航、F-001～F-008 实现位置和状态、系统总图、记忆/工具链图、验证入口与维护规则。
2. 在 AGENTS、ROADMAP、FEATURE-INDEX、INVESTIGATION-INDEX 增加导航入口；历史索引保留并明确其时间边界。
3. 修正 `backend/observability/system.py` 的 CPU 采样对象生命周期；新增 [4 项采样契约](../references/SOURCE-REFERENCES.md#source-04fe509c46)。首个采样仍为 0.0，后续基于两次采样间的 CPU 时间，不代表整机 CPU 百分比。
4. 更新 DailyAgent 两个模块及调度/Prompt 入口说明，消除单角色/纯 DB 的过期描述；原实现保留。
5. 刷新 `.understand-anything` 图谱，保留原图、元数据和指纹备份。扫描继续排除本机配置、密钥、DB、聊天记录、媒体资源、依赖、个人简历输出与图谱自身。

本轮没有新增 commit 或 push，没有修改 config.yaml；工作区原有改动属于各自先前批次。

## 4. 已执行验证与限制

- `.venv/bin/python -m pytest -q tests/test_system_resource_sampling.py`：**4 passed**。覆盖连续采样、PID 改变、缺少 psutil、采样异常；无 DB、模型或应用启动。
- 实际 `system.collect()` 受控验证：stub 模型信息读取，首次 0.0%，约 0.2 秒 CPU 忙循环后第二次 **99.3%**。证明采样连续性恢复，**不是应用性能提升幅度**。
- 说明编辑以 AST 去除 docstring 后逐项比较，保证两个 DailyAgent 文件可执行结构不变。
- 本轮不改前端可执行代码，因此未重复整个前端构建；先前构建结果仍只适用于其当时改动快照。
- 图谱最终结构校验：696 个文件（551 code / 119 docs / 18 config / 8 markup），4864 节点、10339 条边、10 层、12 步导览；1067 条内部 imports 与扫描精确一致，0 个结构错误、0 条悬空边、0 个失效文件路径。118 个独立节点没有关联边，作为非阻断警告保留；增量继承的部分旧摘要较概括，深入修改仍须读源码。测试节点和 `tested_by` 关系仅提供位置，不能解释为已通过测试或完整覆盖率。
- 两份新文档的 181 个本地链接全部有效；人工语义复核未发现阻断问题。`git diff --check` 与本轮 Python 文件编译通过。

图谱生成过程说明：初始扫描曾把旧图谱自身纳入，已加入排除并重扫；已删除 `profile.ts` 的 ENOENT 为预期跳过。补批曾误用 `backend/agents/daily_plan.py`，按实际 `backend/services` 路径纠正后 bundled 提取成功。插件增量指引使用的 `batch-existing.json` 被其合并脚本忽略，已改为该脚本支持的 `batch-0.json` 并重新合并，保留旧图的未变节点。Dashboard 的 pnpm 包装入口因非 TTY 依赖检查失败，随后使用已安装的 TypeScript 直接构建 core 成功，未重装依赖。以上生成问题均未改动产品行为。

锚点复核命令（仓库根目录）：

```sh
rg -n '_sample_process|def collect' backend/observability/system.py
rg -n 'backend.agents.memory|backend.scheduler.task' tests/legacy_source/test_integration.py.txt
rg -n 'create_task|yield|shutdown|stop_polling' backend/main.py
rg -n 'history_keys|page_size|select_recent_history' backend/main.py
rg -n '_social_dispatch_targets|ensure_activity_instance' backend/services/daily_plan.py backend/services/daily_ticker.py
rg -n '项目架构与功能导航图|INV-26' AGENTS.md ROADMAP.md docs/INVESTIGATION-INDEX.md docs/product/FEATURE-INDEX.md
```

真机待验收项沿用原清单：正常/打断/切角色后的语音释放，MCP 真实连接，全启动与关闭、权限和休眠场景，以及 F 系列对话质量。用户在车上无法测试，保留待验收状态，不用自动检查替代。

## 5. 下一轮如何作出优化决策

每项建立同输入、同模型、同数据的对照：正确性 → 失败恢复 → 耗时/资源/Token。每轮结果注明范围和副作用隔离条件。修改后连续两轮仍未解决同一问题，回退该项并记录，不持续盲改。

优先把测试与指标变成可用的基线，再做生命周期、记忆和工具路由的窄改动。整项目调查覆盖不等于整项目每一行都已优化，也不等于所有候选方案都应立即实施。
