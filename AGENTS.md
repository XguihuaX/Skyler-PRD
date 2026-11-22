# Skyler-PRD 协作说明

这是 Skyler 的需求与研究材料仓库。先读 README.md 和 docs/product/PRD-momo-next.md，再按任务查 SPEC-momo-next-details.md、SKYLER-TASKS.md 与验收清单。

- 主 PRD 是当前范围；行为明细保留已定细节。HISTORY、旧 F 系列和旧架构用于追溯，不覆盖当前决定。
- 用户决定、技术候选、局部回放和目标设备验收分别记录。没有实测记录不写通过。
- 不重启已经完成的一级需求访谈，不把案例台词、游戏或参数变为额外硬规则。
- 文档更新优先沿用 bmad-agent-pm（环境已安装时）；无需为读取本仓库安装完整开发环境。
- Git 只列具体文件进行 add；默认不 commit，用户明确要求提交或上传时按范围提交；push 需用户明确授权。不要 force push、amend 或跳过 hooks。
- 不提交运行配置、凭据、聊天数据库、原始录音或模型权重。不得 stage config.yaml。
- 修改后检查需求/状态一致性、相对链接及受影响目录入口。每轮验证在 SKYLER-VALIDATION-LOG.md 留真实结果。
