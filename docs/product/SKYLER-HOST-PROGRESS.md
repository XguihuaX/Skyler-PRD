# 5070 Ti 主机进度

由 Windows 主机 Codex 维护。最近更新于 2026-11-22 16:06（上海时间）。[Mac 端进度](SKYLER-MAC-PROGRESS.md) · [任务清单](SKYLER-TASKS.md)

**T-01 主机侧已通过，等待 Mac 联测。** 主机 WebUI 的主人私聊收到了真实回复，正常重启宿主后仍能回看原题和两条回复。Windows 本地桌宠已恢复，当前使用 `local` 模式。

## 主机实测

| 时间（上海） | 操作和结果 |
| --- | --- |
| 15:48:57 | 在「麦麦聊聊」发送 `T01-HOST-20261122-154815-a7bf1b`，请 Bot 计算 27 加 16 并带回记号。 |
| 15:49:07 | 聊天昵称「雪」回复「27加16等于43」，随后发回完整记号。该轮实际调用行为模型 `gpt-6.1-sol` 和回复模型 `gpt-6.1-terra`。 |
| 15:54 | 确认本轮已结束后正常停止并启动宿主。重新加载 `/chat`，在同一 Bot、同一主人私聊看到了原题、答案和记号。 |
| 16:00、16:01 | 用户后来加入的回复候选 `gemini-3.8-flash` 在 WebUI 单项能力测试中两次收到上游 HTTP 503 `model_unavailable`。回复分工当前含两个候选，策略为随机；Gemini 的可用性待复测。 |

需求基线为 `2e20017b11d5efabb16bab38166ea113ebb3d66b`。实装 MaiBot `1.4.8`、Dashboard `1.10.8`，宿主实例名「麦麦」，Bot ID 为 `e726094db46a42119b9084e8cc5f89fb`。本轮验收的主人私聊为 `internal / owner`。ASR、TTS 分工目前留空，按 T-05 基础语音接入。

主机本地证据位于 `MaiBot-runtime/instance`。聊天事件为 2（提问）、6（答案）、8（记号）；重启后的截图为 `temp/T01-HOST-20261122-154815-a7bf1b-after.jpg`。完整步骤保存在主机工作副本的[验收日志](SKYLER-VALIDATION-LOG.md)。

## 下一步 · 配合 Mac

1. Mac 先验证能打开同一宿主的 WebUI。主机无线局域网地址为 `192.168.1.46`，WebUI 的 `8001` 当前只监听回环。若 Mac 已有这台主机的 SSH 登录，可在 Mac 上运行以下命令，将 `WINDOWS_USER` 换成实际登录名。保持终端连接，Mac 浏览器和桌宠均使用 `http://localhost:18001/`。主机侧已核实 SSH 监听 `22` 并支持转发，Mac 侧可达性待实测。

   ```sh
   ssh -N -o ExitOnForwardFailure=yes -L 127.0.0.1:18001:127.0.0.1:8001 WINDOWS_USER@192.168.1.46
   ```

2. 若 Mac 没有现成 SSH 登录，先协调局域网接入地址。Mac 网页可达、双方就位后，由主机切到 `remote`，Mac 用配套的官方 **MaiBot Desktop Pet 1.0.1** 连接上述 Bot。
3. 收到 Mac 的测试记号和回复时间后，主机在网页核对同一私聊并更新本页。Mac 端负责记录自己重开客户端后的回看结果。
