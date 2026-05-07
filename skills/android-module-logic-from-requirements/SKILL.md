---
name: android-module-logic-from-requirements
description: Use when translating product requirements, acceptance criteria, or user-flow documents into Android module behavior; use when wiring navigation, ViewModel state machines, validation rules, or cross-screen flows for a named feature like login or registration.
---

# 从需求文档串联模块逻辑（ViewModel / 领域层）

## 前置条件

- 已有 **书面需求**：PRD、飞书/Confluence、脑图、用户故事 + 验收标准均可。
- 已明确 **模块范围** 与 **参与页面列表**。

若需求仅有口头描述：先输出「待确认问题清单」，再仅给出假设性草案并标注 **ASSUMPTION**。

## 输出目标

- **可追踪 UI 状态**：`Loading` / `Success` / `Error` / `Empty`（以及表单域 `idle/editing/submitting` 等子状态，若需求要求）。
- **单一数据源**：`StateFlow`/`MutableStateFlow` 或明确 `UiState` data class；异步用协程，ViewModel 内结构化并发。
- **UI 与数据解耦**：ViewModel 通过 UseCase/Repository 获取数据；禁止 Composable 直接访问网络或数据库。

## 执行步骤

1. **抽取用户旅程**：逐步列出用户操作 → 系统响应 → 下一屏；标注异常分支（网络失败、鉴权失效、字段校验、业务错误码）。
2. **定义 `UiState` 与事件**：每个页面一个主 `UiState`（或密封类分状态）；用户操作映射为 `fun onXxx()` 或 `sealed interface UiEvent`。
3. **校验与边界**：按需求实现校验顺序、防抖提交、重复点击保护；错误信息映射到可读文案（由资源或领域层提供 key）。
4. **导航契约**：成功/取消/需额外步骤（如二次验证）时，由 ViewModel 暴露 `effect`（如 `SharedFlow<NavigationEffect>`）或由路由层统一订阅，避免 Composable 内散落 `navController` 逻辑。
5. **与接口文档对齐**：字段、枚举、分页、空列表语义与 `android-module-data-from-api` 产出的 DTO 一致；发现需求与接口冲突 → **停止实现**，列出冲突表让用户决策。

## 与接口层协作

- ViewModel 只依赖 **Repository 接口** 或 UseCase；单元测试优先覆盖 UseCase 与状态转移。

## 自检清单

- [ ] 主路径与需求文档逐步对应可勾选
- [ ] 每个异常路径有明确 `UiState` 或 `effect`
- [ ] 无 `GlobalScope`；无吞掉的 `catch`
- [ ] 需求中的「空数据」与接口实际返回一致

## 常见错误

- 只实现 happy path → Error/Empty 未定义，上线后不可预期 UI。
- 在多个地方重复同一业务规则 → 应沉到 UseCase。
