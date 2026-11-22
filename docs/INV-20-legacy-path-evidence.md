# INV-20 · 旧路径调用证据

日期：2026-10-02。范围：REF-001 的“记录遗留路径”切口。以下结论只依据当前仓库的静态调用、路由和测试；未据此推断外部客户端流量。

## 可移除：前端 profile_summary 客户端

- `frontend/src/lib/profile.ts` 没有源码导入点，导出的三个 `profile_summary` 操作请求的路由已不在 `backend/routes/users_api.py` 注册表中。该文件已删除。
- 仍被设置页复用的 `ProfileSection` 只读取 `/profile` 的 nickname、language；其旧响应类型中未被使用、后端也不返回的 `profile_summary` 字段已移除，组件本身继续保留。
- 现行用户画像界面 `UserProfileSection.tsx` 调用 `profileData.ts` 的结构化资料接口；后端注册 `/profile_data` 读写、删除和重算。`App.tsx` 所用的同名 `fetchUserProfile` 来自 `lib/config.ts`，请求仍存在的 `/profile` 路由，因此不能随旧模块删除。
- 后端 `users.profile_summary` 数据列保留供老数据库迁移兼容；删除前端无人调用的客户端并不删除用户数据。

## 必须保留：聊天和角色兼容路径

- `chat._build_messages` 在 `character_id is None`、renderer 缺 active persona variant 或 renderer/Jinja/DB 异常时进入 legacy 装配；legacy 中仍调用 `prompt_manager.get_prompt`。这条路径有运行时条件，现阶段不能按 `@deprecated` 标签直接删除。
- `ws.py` 通过 `prompt_manager.get_current_character` 解析缺失角色的消息，`tools/builtin.py` 通过 `prompt_manager.switch_character` 执行工具。`prompt_manager` 模块本身仍有活调用。
- `profile_summary` 旧接口及 `ws._compute_profile_summary` 在运行时代码中已退役，但部分旧测试仍直接调用或 patch 它们（如 `tests/test_profile_api.py`、`tests/test_profile_user_only.py`）。这些测试反映旧契约，需独立确定是否改写或归档；不能把它们的失败当成新切口回归。
- `tests/test_chat_agent.py` 仍导入已不存在的 `backend.database.services.upsert_personality`，在测试设置阶段即失败；`tests/test_ws_helpers.py` 仍导入已不存在的 `_run_plan`。两者需要按现行公开接口重写或归档，不构成本轮运行时代码的回归证据。

## 删除条件

- 聊天 legacy 装配：补齐 `character_id=None`、renderer 异常、persona 缺失时的等价消息和状态回归，再评估能否移除；若要取消 fallback，需单独决定产品行为。
- 旧资料列和迁移：先确认现存数据库升级与回滚路径，以及是否有仓库外客户端仍依赖旧接口；不在代码瘦身切口中直接删列。
