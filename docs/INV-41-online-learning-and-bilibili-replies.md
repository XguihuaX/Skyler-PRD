# INV-41 · Momo 上网学习与 B 站大肥鱼回评

2026-11-21。Momo 可以先读用户指定的学习仓库，找出对当前项目有用的变化，在原聊天反馈。DSH 社区已有保存学习结果和来源的插件，也有接收 B 站评论并回到原楼层的插件。下文记录各自能直接用到哪一步。

## 本轮新知识对 Momo 有什么用

「规划多平台新知学习汇总」本轮结束后，已核对 GitHub 远端 main 的提交 `9c02d0a`，并从这一提交读取日报和两篇讲义。扫描范围是收藏夹的 36 条视频，10 条首次建档，其中 2 条完成音轨、关键画面和时间点整理，其余 8 条为线索。[本轮日报与阅读状态](https://github.com/XguihuaX/multiplatform-ai-learning/blob/9c02d0ac094e95bf77c66f8b0b0d9218f3410d44/reports/daily/2026-11-21-bilibili.md)

| 内容 | 对 Momo 的用途 | 可以怎样试 |
| --- | --- | --- |
| B29 · video-watch-plus | 优先用于学习插件读有授权的录屏或教程。先用 FFmpeg 找切换、静止段和运动，再让模型读关键画面，遇到小字回到原尺寸补读，并把画面与转写对齐。 | 用带有一闪而过的菜单和静态代码的一段录屏，对比固定间隔抽帧与定向取帧，记录漏掉的步骤、图片数和耗时。实时摄像头的延迟要在另一条链路测。[学习讲义](https://github.com/XguihuaX/multiplatform-ai-learning/blob/9c02d0ac094e95bf77c66f8b0b0d9218f3410d44/lessons/video-watch-plus.md) · [上游方法](https://github.com/xinhailaopo/video-watch-plus/blob/8ca266f9cf9e5ffce7e9d424aa0e937a897b608d/SKILL.md) |
| B28 · Fleshy | 交给独立 3D 研究线。它改善已有动作里的接触，例如手掌托住脸颊时皮肤轻微受压。作者提供 Blender 内的碰撞和烘焙，列出的宿主为 64 位 Windows。 | 在同一段托腮动作上比较开启前后的接触效果，再导出到目标引擎检查形变是否保留。桌宠运行时支持情况待测。[学习讲义](https://github.com/XguihuaX/multiplatform-ai-learning/blob/9c02d0ac094e95bf77c66f8b0b0d9218f3410d44/lessons/fleshy-contact-deformation.md) · [作者产品说明](https://www.patreon.com/moyu_ch/posts/fleshy-contact-172631791) |

剩余条目中，B34 的视频与网页感知、B37 的上下文整理值得下一轮优先读正文。B36 的角色动作控制可作为 3D 研究线线索。三项在上述提交中均只有建档资料。

## 上网学习插件可以直接复用什么

用户点名收藏或仓库时，先读指定材料。对于这次任务，输入就是学习仓库新增的日报和讲义。同一个提交只处理一次，并把原文位置带进反馈。需要主动寻找新项目时，可接公开仓库或发布记录。OpenBiliClaw 已能通过 GitHub 官方接口搜索和筛选公开仓库，它的 GitHub 来源模块提供的是发现与推荐。[OpenBiliClaw 的 GitHub 来源](https://github.com/whiteguo233/OpenBiliClaw/blob/aa4af1843756445847027083b0fc81337b764774/docs/modules/github.md)

DSH 社区的 `dsh-agent-growth` v0.3.0 能接收材料、登记出处、保存结论、回读和修订。它还能提出下一步要查什么，实际获取网页或仓库内容由外部工具完成。Momo 需要接的几步是读取指定来源的增量、把成果和当前需求对照，再送回发起任务的对话。先用现有知识仓库就能验证这条路径。[插件使用说明](https://github.com/XiaoZheBrother/dsh-agent-growth/blob/f46cd4380f0436714a7db49861583ae160a7e914/docs/agent-guide.md) · [探索请求与外部读取](https://github.com/XiaoZheBrother/dsh-agent-growth/blob/f46cd4380f0436714a7db49861583ae160a7e914/README.md#L147-L154)

一次反馈可以很具体，例如“这版改进了静态画面的重复抽取，值得用上次那段教程对照一下，看短暂出现的菜单有没有漏掉”，并附来源。用户也可以把交付物指定为知识仓库或最小实验仓库，保存笔记、样例和测试结果。新建远端仓库、提交改动和推送走相应的 Git 工具与用户给定的权限，学习结果本身可独立交付。[GitHub 建库接口](https://docs.github.com/en/rest/repos/repos#create-a-repository-for-the-authenticated-user)

**本地验证。** 用上游核心实际读入 B28、B29 两篇完整讲义，提交相同版本两次，插件只建一个任务。将本次调研写出的两条建议存入后，可以回读并找回两份来源，任务状态为完成。建议由本次调研撰写。这次验证覆盖材料进入、去重、保存和召回，自动联网与模型自主判断仍待 Momo 接入验证。此外，学习核心的 13 项离线用例全部通过，覆盖来源是否读过、局部阅读标记、结论修订等行为。[学习核心](https://github.com/XiaoZheBrother/dsh-agent-growth/blob/f46cd4380f0436714a7db49861583ae160a7e914/core.js) · [上游用例](https://github.com/XiaoZheBrother/dsh-agent-growth/tree/f46cd4380f0436714a7db49861583ae160a7e914/test)

## B 站评论区的大肥鱼怎么回复

作者公开的一个例子是，用户在视频下 @ 角色，角色读取通知、核对对象，再结合转写和抽帧理解视频，最后回复到原评论。仓库的实际效果截图里，一条 Agent 教程的回评分出了从 00:00 到 15:00 的章节，06:37 对应工具调用循环。另一种实现只读相关评论和上下文就能接话。单凭某个账号的头像或口吻，无法判断那条回评采用了哪个插件，或是否由人代发。[作者公开的回评截图](https://github.com/bronyacccxz/dsh-bilibili-reply/blob/85a92410e41b2110920f7510446b79a9106b985b/examples/01-timestamps.png) · [该示例的处理流程](https://github.com/bronyacccxz/dsh-bilibili-reply/blob/85a92410e41b2110920f7510446b79a9106b985b/README.md#L79-L109)

| 想做的事 | 已有做法 |
| --- | --- |
| 有人回复或 @ 时，让自己的角色接话 | `dsh-bilibili-plugin` 用已登录账号定时读取回复和 @ 通知，补上根评论、已有对话、视频标题或动态正文，再交给 DSH。允许自动回复时，将生成文本发回对应楼层并记录结果。每轮仅读一页通知，高峰时可能遗漏。[通知与上下文](https://github.com/seiuna/dsh-bilibili-plugin/blob/9f622f9bd6d7e894d57ed7247532dc260d839050/README.md#L89-L99) · [回复处理](https://github.com/seiuna/dsh-bilibili-plugin/blob/9f622f9bd6d7e894d57ed7247532dc260d839050/src/poller-service.ts#L486-L555) |
| 守自己的评论区，或让角色挑到视频后留言 | `dsh-bili-pond` 提供不同模式。守护模式可处理自己视频下的新评论，在他人视频下可限于白名单用户的 @；选片后的主动评论另有开关。自动运行与公开评论默认关闭。其评论进度的离线测试在本机通过 21 条断言，包括一轮只处理部分评论时如何在下一轮接续。[评论模式](https://github.com/YOYOLnFu/dsh-bili-pond/blob/57765435e33a8697c7f3b3b9f317c6146cd05faa/README.md#L209-L245) · [进度测试](https://github.com/YOYOLnFu/dsh-bili-pond/blob/57765435e33a8697c7f3b3b9f317c6146cd05faa/tools/test-guard-watermark.mjs) |

按已定设计，Momo 可以把“收到一条评论”交给同一角色会话。自用学习可以只把结果发回 Momo 聊天。公开回评还涉及平台授权，B 站用户协议对自动程序取得平台服务、内容或数据要求事先明确书面许可。Momo 与 B 站账号之间的在线收发尚未联测。本次使用作者公开材料和本地样例，未使用用户账号发评。[B 站用户使用协议](https://www.bilibili.com/blackboard/protocal/licence.html)
