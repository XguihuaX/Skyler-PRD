# INV-27 · 后台清理、隔离回归与记忆批次复用

日期：2026-10-13。基线：`codex/code-slimming-baseline-20261001` / `00a164da`，含既有未提交改动。承接 [INV-26](INV-26-project-map-and-optimization-review.md)。本文记录调查和验证，实施状态见 [ROADMAP](../ROADMAP.md)。

## 1. 本轮结论

启动失败/退出的清理边界、退役测试的隔离入口、记忆去重重复编码、DailyAgent 无调用辅助代码这四个切口已完成对应自动验证。没有运行生产应用启动、真实工具写操作或真机语音验收；没有修改 `config.yaml`、commit 或 push。

## 2. 控制流证据与改动

### 后台任务和服务清理

原 `lifespan` 的通用清理在 MCP `yield` 块之后；启动或运行阶段抛异常时可能跳过。5 个模型预载、镜像探测、catch-up、权限检查 task 没有统一所有者。

- [main.lifespan](../references/SOURCE-REFERENCES.md#source-df5184bf1f) 用外围 `AsyncExitStack` 注册任务和服务清理，启动失败也能进入清理。
- [BackgroundTasks](../references/SOURCE-REFERENCES.md#source-37bdde9258) 保存 task 引用、记录未处理异常，退出时取消并等待，超时记录具体名称；关闭后拒绝新任务。
- MCP enabled/disabled 分支共享初始化与收尾，初始化前注册 client 清理，部分连接失败也会关闭；后台 catch-up 先取消，随后关闭 MCP client/server。
- player、extractor、activity、cron 独立清理；一个回调异常或取消不会跳过其余回调。
- 18 个 boot mark 的名称和首次出现顺序与 HEAD 对照相同；迁移调用和 appReady gate 未改。

[生命周期测试](../references/SOURCE-REFERENCES.md#source-7e739f892c) 执行从生产源码提取的真实 lifespan/cleanup 函数和 MCP 尾段，用模拟资源注入启动异常、运行异常、取消、清理失败及 MCP 部分初始化失败。它不是完整应用集成启动测试。取消 asyncio 包装 task 不能强停其线程内的模型加载；超时也不应报告为线程已结束。

### 记忆去重的重复计算

[validator._is_duplicate](../references/SOURCE-REFERENCES.md#source-3389389ff0) 原来每条候选都逐条请求已有记忆编码。现在仅在一次 `validate_and_filter_entries` 内复用已成功编码的结果，上限 256 项，批次结束释放，不缓存失败。

[受控对照](../references/SOURCE-REFERENCES.md#source-ed1f2e4b22)：200 条已有内容 × 10 条新候选，编码请求从 **2010 降到 210（约 89.6%）**，判定结果相同；另外覆盖阈值相等、批内去重、缓存容量、跨批释放和编码失败容错。这里的数字是 `_encode` 请求数，**不是实际模型耗时、CPU 或总体延迟降幅**。`long_term._encode` 已有全局 LRU/TTL，真实收益还取决于规模与命中率。记忆来源、角色范围、召回阈值、tombstone 和失败游标语义未改。

### DailyAgent 无调用链

`rg` 核对 backend/frontend/tests/scripts 中无外部消费者后，移除 `_format_calendar`、`_iso_to_hhmm`、`_load_today_calendar`、`_load_recent_summary`，以及 `_format_yesterday_rhythm` 返回后的不可达解析片段。当前 Prompt 有意忽略用户日历、画像和聊天摘要，这些接口参数与默认角色兼容常量仍保留。

去除文档字符串并排除已识别不可达尾段后，保留下来的所有函数 AST 与 HEAD 相同。现有 DailyAgent 运行时 41 项、Prompt 6 项验证通过；不能把删死代码解释为当前日程功能提速。

## 3. 测试治理与实际结果

统一入口：[run_slimming_checks.py](../references/SOURCE-REFERENCES.md#source-cfb0f20117)。显式列出 16 个已审查测试文件，分 lifecycle/memory/chat/tools 四组：每文件一个独立进程、一个临时 SQLite 路径；模型下载设为 offline；按文件明确使用 pytest 或脚本 `main()`。失败/超时返回非零，POSIX 超时清理进程组，包括嵌套测试子进程。**这是经过审查的定向检查入口，不是任意测试的网络沙箱，也不是全仓测试通过证明。**

5 份旧测试原文移到 [legacy_source](../references/SOURCE-REFERENCES.md#source-55eddb797b)，以 `.py.txt` 保存：3 份仍导入不存在的 MemoryAgent/PlannerAgent/AlarmScheduler；旧 WS 和 ToolAgent 断言分别迁到当前持久化/工具暴露契约。未通过把旧断言 skip 来宣称通过。

首轮统一运行 **13/16 文件通过**。另外 3 个文件暴露测试隔离缺口：

1. DailyAgent Prompt 测试落入本机用户画像 DB；补齐非目标依赖的模拟，保留原注入次数/私有活动断言。
2. DailyAgent 坐标校验测试未提供前置“社交卡”身份；显式提供该身份，保留 409/422 断言。
3. MCP transport/dispatch 单测未隔离凭证与确认状态 DB；明确模拟这些依赖，数据库状态仍由另外两个临时数据库测试验证。

此外，validator 脚本原来依靠 tombstone 缺表的 fail-open 才通过，现由明确模拟依赖承担单位测试边界；extractor 静态接线测试直接读源码，不再为读文本而 import 完整 main。

最终分组复验：**16/16 文件通过**，其中 pytest 报告 **128 项**（含 1 个运行子进程记忆契约的包装测试，不把内部断言重复计数）；3 份独立脚本报告 **17 + 34 + 24 = 75 条检查**。本轮 Python 编译与 `git diff --check` 通过。前端本轮无新改动，未重复构建。

```sh
# 完整定向入口；--list 可先查看清单
.venv/bin/python scripts/run_slimming_checks.py --list
.venv/bin/python scripts/run_slimming_checks.py

# 本轮最终结果来自以下两组命令，均通过
.venv/bin/python scripts/run_slimming_checks.py --suite lifecycle --suite memory
.venv/bin/python scripts/run_slimming_checks.py --suite chat --suite tools
```

运行结果摘要存于 [regression results](../references/SOURCE-REFERENCES.md#source-e212af48f1)。入口的超时路径另用 0.001 秒强制验证，3 个文件均记录 timeout 且退出码 1；这是预期故障注入，不是功能回归失败。修复后的功能问题均在首轮复验解决，未触发两轮回退阈值。

锚点复核：

```sh
rg -n 'async def lifespan|async def _shutdown_services|tasks.spawn|mcp_lifetime' backend/main.py
rg -n 'class BackgroundTasks|async def close' backend/utils/background_tasks.py
rg -n '_BATCH_EMBEDDING_CACHE_LIMIT|embedding_cache' backend/utils/memory_entry_validator.py
rg -n '_format_calendar|_iso_to_hhmm|_load_recent_summary|_load_today_calendar' backend frontend/src tests scripts
rg -n 'SUITES|DATABASE_URL|start_new_session' scripts/run_slimming_checks.py
```

## 4. 导航时效和验收边界

[PROJECT-MAP](PROJECT-MAP.md) 已同步本轮入口、测试和状态。机器图谱保留上一批的真实分析快照，在 meta 标注 `stale: true` 和变更路径；没有仅换时间戳冒充重新分析。新 agent 先用 PROJECT-MAP + 实际源码定位，机器图谱的旧节点只作参考，刷新前核对 source-snapshot 哈希。

本轮工程收尾有明确闭环；下列事项仍分别属于性能测量或产品行为评估，不能笼统写成“只差个人测试”：

- 真机验收：启动/退出、正常语音→打断→再次说话、切角色、MCP 真实连接、权限/休眠。
- 性能证据：历史会话数量增长下的恢复开销、全进程内存/CPU、真实对话延迟。
- 产品行为：短句记忆召回、抽取失败可重放、按任务选择工具 schema。这些改变召回/失败处理/工具可见性，属于任务 5 及独立质量对照；本轮保留现有行为。

先前 renderer 并发读取在两轮真实 DB 对照无稳定收益后已回退；本轮没有重试或重新宣称收益。
