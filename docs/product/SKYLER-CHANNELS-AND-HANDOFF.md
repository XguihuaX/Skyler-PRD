# Skyler 多端入口、原生语音与换机依据

2026-11-22 定向核对。源版本为 MaiBot v1.4.8，固定提交 `edf42446f123149c57fc79765dd5a8b004b8524c`；这是文档/源码证据，不是目标设备通过记录。

## 发消息的入口

| 入口 | 已查到的能力 | 接入时需要验证 |
| --- | --- | --- |
| 微信 ClawBot | 内置适配器，扫码绑定、私聊文字和图片双向收发，接收微信提供的语音转写；重启恢复连接 | 该版不发语音，不读取普通群或其他私聊；无转写的音频无法读取。主动发送受微信服务限制。身份与内部主人会话另行关联 |
| 网页与 Mac 桌宠 | 网页普通模式和官方桌宠共用 internal/owner；Mac 客户端可连接宿主 | Mac 构建存在，上游文档注明尚未 Mac 实机验证；采音、读屏和播放分别验。主屏实际取客户端设备 |
| 手机 | WebUI 提供手机页面；Android 另有悬浮桌宠，支持聊天、历史、待办和探索 | 可达地址、认证与真机体验；所查 Android 客户端未实现自动读屏/应用监测。同一个 Bot 同时只有一个官方远端桌宠客户端 |

来源：[微信适配器](https://github.com/Mai-with-u/MaiBot/blob/edf42446f123149c57fc79765dd5a8b004b8524c/src/plugins/built_in/weixin_clawbot_adapter/README.md)、[主人会话](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/internal-chat.md)、[桌宠平台](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/desktop-pet.md)、[Android](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/pet-clients/android/README.md)。

麦麦相通组可共享记忆、表达和黑话的读取范围；共享不等于把消息复制为同一条历史。人物身份迁移另有平台限制，不能直接当作微信与 internal/owner 的自动合并工具。[相通组](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/sharing-groups.md) · [身份迁移](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/person-migration.md)

## 麦麦原生语音

原版能把收到的语音消息转写交给聊天流程；回复可整段合成为语音，选择仅语音或文字加语音；模型库支持 ASR/TTS 提供方与 TTS 音色、试听，聊天语音播放器支持播放/暂停、进度与速度。默认 ASR/TTS 模型列表为空，需配置服务或插件。

这些已有入口可用于接入选定 ASR 和自训声音；持续麦克风、独立唤醒、场景让话、插问续播和防回声仍按选定客户端/插件逐项验收。所查官方网页 ChatComposer 没有录音入口。麦麦社区桌宠、DSH 语音插件与旧 Momo 采音属于不同来源，见复用总表。

来源：[识别](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/common/utils/utils_voice.py)、[回复语音](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/maisaka/builtin_tool/reply_voice.py)、[插件协议](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/docs/plugin-provider-formats.md)、[默认配置](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/src/config/default_model_config.py)、[播放器](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/dashboard/src/components/VoicePlayer.tsx)、[聊天输入](https://github.com/Mai-with-u/MaiBot/blob/v1.4.8/dashboard/src/routes/chat/ChatComposer.tsx)。

## 换机交接起点

上游提供配置、数据、插件的导出/恢复及换机流程，要求核对工作区、版本、路径、权限和地址。可作为主机与 Mac 计划交接验证的起点；未查到本项目可直接声明通过的自动双机接替。Mac 服务端运行与模型是否本地推理分别决定；DSH 数据和依赖需另核对。

产品已选择先验证主动往返交接，见[交接明细](SPEC-momo-next-details.md#512-主机与-mac-的计划交接2026-11-22-已确认验证方向)与[测试步骤](SKYLER-5070TI-ACCEPTANCE.md#主机与-mac-计划交接验证)。[上游备份与迁移说明](https://docs.maibot.chat/faq/backup-and-migration)
