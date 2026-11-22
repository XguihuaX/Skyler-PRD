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

## 开发阶段使用哪些技能

| 技能 | 用途 | 入口 |
| --- | --- | --- |
| bmad-agent-po | 将 PRD 或已复现缺口拆成要求、任务与验收条件 | [SKILL.md](.agents/skills/bmad-agent-po/SKILL.md) |
| bmad-architecture | 确定跨模块职责、数据归属和接口约束 | [SKILL.md](.agents/skills/bmad-architecture/SKILL.md) |
| bmad-agent-dev | 开发角色入口，执行明确的功能或修复 | [SKILL.md](.agents/skills/bmad-agent-dev/SKILL.md) |
| bmad-build | 单项功能/修复的计划、实现、审阅和验证 | [SKILL.md](.agents/skills/bmad-build/SKILL.md) |
| bmad-build-auto | 对目标、范围和验收清楚的任务自主执行 | [SKILL.md](.agents/skills/bmad-build-auto/SKILL.md) |
| bmad-code-review | 对代码差异做审阅与问题处理 | [SKILL.md](.agents/skills/bmad-code-review/SKILL.md) |
| bmad-qa-generate-e2e-tests | 为已工作功能补端到端/API 验证 | [SKILL.md](.agents/skills/bmad-qa-generate-e2e-tests/SKILL.md) |

按实际任务选用：原版验证先得到结果，缺口形成任务；需要跨模块决定时用 architecture，具体实现用 build，再做相称的代码审阅和回归。无需每个小改动都走全部技能。

以上技能包含对应的 workflow、步骤、模板、review prompts、参考资料和脚本；需要渲染的工作流通过 `_bmad/scripts/render_skill.py` 生成，依赖由其脚本元数据声明。带有角色、代码编辑和测试的步骤要在目标实现仓库执行，不在本材料仓库凭空启动产品开发。

## 换到主机后怎样接手

克隆本仓库后，让 Codex 先读取根目录 AGENTS.md、主 PRD 和任务清单，再使用 bmad-agent-pm。支持项目技能发现的工具可从 `.agents/skills/` 读取；如果技能未出现在列表，直接指定上面的 SKILL.md 路径读取。是否被具体客户端自动发现以其实际环境为准。

BMad 的配置解析、模板列表、日志、任务脚本已放进 `_bmad/`，项目名为 Skyler-PRD。现有文档仍在 `docs/product/` 维护，不因为默认 output_folder 另建一份平行 PRD。

辅助脚本要求 Python 3.11+；`memlog.py` 声明 PyYAML，`render_skill.py` 声明 Jinja2，使用 uv 可按脚本元数据解析依赖。此处附带脚本，不代表主机已经安装 Python、uv 或产品依赖。Mac 可从仓库根目录验证：

```sh
python3 _bmad/scripts/resolve_customization.py --skill .agents/skills/bmad-agent-pm --project-root . --key agent --key workflow
python3 _bmad/scripts/list_templates.py --libraries-of .agents/skills/bmad-facilitate .agents/skills/bmad-agent-pm/assets --project-root . --root .agents/skills --brief
```

Windows 上可使用提供 Python 3.11+ 的 `python` 或 `py -3` 执行同一脚本；技能流程若调用 uv，则按其声明运行。本仓库已附上述开发链路。交互过程中若另选 party-mode、advanced-elicitation、walkthrough 或 retrospective 等可选流程，再取得相应技能；不预装全部 BMad 生态。

## 当前约定

沿用已确认范围和用户调整；本轮上传技能不改变 PRD 的功能或排期。主 PRD v0.96、行为明细和任务清单继续作为同一套需求依据。先验证 T-01，再按实际结果安排后续接入。
