---
name: android-app-module-pipeline
description: Use when building or extending one app feature module (e.g. login, register) and the user provides or should provide design assets, requirements, and API docs; use when the user asks for module-by-module delivery, phased handoffs, or a repeatable full-stack module workflow for Android.
---

# Android 业务模块端到端流水线（按模块交付）

## 适用场景

- 按模块实现：**登录、注册、首页、订单** 等独立业务单元。
- 每个模块需要三类输入对齐后再动代码：**设计图**、**需求说明**、**接口文档**。

## 执行顺序（必须遵守）

1. **输入门禁**：三类输入未齐备时，只输出「缺失清单与澄清问题」，禁止生成业务实现代码。
2. **模块规格摘要**：用表格固化本模块的页面、状态、接口、验收点（1 页以内）。
3. **数据层（接口驱动）**：先读 `android-module-data-from-api`，落 DTO / Service / Repository / 错误映射。
4. **逻辑层（需求驱动）**：再读 `android-module-logic-from-requirements`，落 ViewModel / UseCase / 导航与状态机。
5. **UI 层（设计驱动）**：最后读 `android-module-ui-from-design`，落 Compose 界面与资源；禁止 UI 直连网络。
6. **整体验收**：对照需求走主路径与异常路径；对照设计做视觉抽查；对照接口做契约检查（字段、错误码）。

## 单次交付范围

- **默认**：一个模块 = 一个垂直切片（该模块涉及的全部页面 + ViewModel + 数据层变更 + 导航入口）。
- **禁止**：无明确模块边界时跨模块大改；禁止跳过门禁「猜」接口或需求。

## 模块输入包（用户每次应提供）

| 输入 | 最低要求 | 用途 |
|------|-----------|------|
| 设计 | 可解析 **Figma 链接**，或 **本地截图/切图路径** | UI 结构、间距、资源 |
| 需求 | PRD/脑图/流程说明/验收标准 | 流程、边界、空态与错误策略 |
| 接口 | OpenAPI/Swagger、Postman 导出、或结构化接口说明 | DTO、路径、鉴权、错误码 |

若仅有其中一类输入：只允许做 **该输入对应层** 的草案（例如只有接口则只做数据层草案），并明确列出待补材料。

## 与项目脚手架的关系

- 若项目尚无 MVVM 基础能力，先使用仓库内 `skills/android-mvvm-scaffold/create_MMVM_project.md` 补齐 `BaseCompose`、`BaseViewModel`、导航与网络封装，再执行本子技能。

## 子技能（按需加载）

- UI：`skills/android-module-ui-from-design/SKILL.md`
- 逻辑：`skills/android-module-logic-from-requirements/SKILL.md`
- 接口：`skills/android-module-data-from-api/SKILL.md`

## 模块规格摘要模板（输出给用户确认）

```markdown
## 模块：<名称>
### 页面与导航
- 入口：
- 出口（成功/取消）：

### UI 状态（Loading / Success / Error / Empty）
- ...

### 接口清单
| 用途 | 方法 | 路径 | 鉴权 |
|------|------|------|------|

### 验收
- [ ] 主路径
- [ ] 网络失败 / 业务错误码
- [ ] 空数据 / 表单校验
```

## 常见错误

- 先把界面画满再补接口 → 字段与状态对不上，返工多。
- 需求里的异常分支未进 `UiState` → 线上只剩 toast 或白屏。
- 设计图缺失仍硬编码文案/色值 → 与主题和资源规范冲突。
