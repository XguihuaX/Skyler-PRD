# 本仓库的协作规则与技能

2026-11-22。根目录 [AGENTS.md](AGENTS.md) 已包含用户提供的“决策与执行原则”和“主张、边界与预设质疑”原文：约束需有具体依据、可逆细节合理推进、用验证回答未知、不替用户构造更强主张、不为假设批评削弱目标。

## 需求工作使用哪些技能

| 技能 | 用途 | 入口 |
| --- | --- | --- |
| bmad-agent-pm | 继续维护、整理和核对产品需求 | [.agents/skills/bmad-agent-pm/SKILL.md](.agents/skills/bmad-agent-pm/SKILL.md) |
| bmad-facilitate | PM 调用的文档更新流程、模板与决定日志 | [.agents/skills/bmad-facilitate/SKILL.md](.agents/skills/bmad-facilitate/SKILL.md) |
| bmad-review | 配套结构/语言及其他指定视角审阅 | [.agents/skills/bmad-review/SKILL.md](.agents/skills/bmad-review/SKILL.md) |
| momo-refactor-cases | 把用户例子记入案例册，区分预期与实测 | [.agents/skills/momo-refactor-cases/SKILL.md](.agents/skills/momo-refactor-cases/SKILL.md) |

`bmod-core-tools`、`bmod-method` 随附模块元数据和相关模板，不主动调用。上面的技能保留本机已安装版本的完整文件，来源与逐文件哈希见 [SKILLS-MANIFEST.json](references/SKILLS-MANIFEST.json)，BMad 版权和许可见 [third_party/bmad](third_party/bmad/README.md)。

## 换到主机后怎样接手

克隆本仓库后，让 Codex 先读取根目录 AGENTS.md、主 PRD 和任务清单，再使用 bmad-agent-pm。支持项目技能发现的工具可从 `.agents/skills/` 读取；如果技能未出现在列表，直接指定上面的 SKILL.md 路径读取。是否被具体客户端自动发现以其实际环境为准。

BMad 的配置解析、模板列表、日志、任务脚本已放进 `_bmad/`，项目名为 Skyler-PRD。现有文档仍在 `docs/product/` 维护，不因为默认 output_folder 另建一份平行 PRD。

辅助脚本要求 Python 3.11+；`memlog.py` 的脚本元数据声明 PyYAML 依赖，使用 uv 可按声明解析。此处附带脚本，不代表主机已经安装 Python、uv 或产品依赖。Mac 可从仓库根目录验证：

```sh
python3 _bmad/scripts/resolve_customization.py --skill .agents/skills/bmad-agent-pm --project-root . --key agent --key workflow
python3 _bmad/scripts/list_templates.py --libraries-of .agents/skills/bmad-facilitate .agents/skills/bmad-agent-pm/assets --project-root . --root .agents/skills --brief
```

Windows 上可使用提供 Python 3.11+ 的 `python` 或 `py -3` 执行同一脚本；技能流程若调用 uv，则按其声明运行。这里不附带完整 BMad 其他角色或构建技能，后续需要时再取得对应依赖。

## 当前约定

沿用已确认范围和用户调整；本轮上传技能不改变 PRD 的功能或排期。主 PRD v0.95、行为明细和任务清单继续作为同一套需求依据。先验证 T-01，再按实际结果安排后续接入。
