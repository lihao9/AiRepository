---
name: android-module-ui-from-design
description: Use when implementing or updating Jetpack Compose screens from Figma links or local design screenshots for a named feature module; use when the user attaches design exports, asks for pixel-faithful UI, previews, or asset naming for Android.
---

# 从设计图生成模块 UI（Compose）

## 前置条件

- 已明确 **模块边界**（例如仅「登录页」或「注册全流程」）。
- 设计输入二选一：
  - **Figma**：可解析的 `figma.com/design/` 等链接（或 file key + node id）。
  - **非 Figma**：用户提供的 **本地图片路径**（截图/导出的切图目录）。

缺失设计输入时：列出需要补充的具体文件或链接，**不生成页面实现代码**（页面创建类任务门禁）。

## 输出目标

- 在现有工程规范下交付：**Compose + `BaseCompose` + 主题/资源**，无业务数据硬编码在 UI（展示数据来自 `UiState`）。
- 每个新建或大幅修改的页面：**整页 `@Preview`**，可渲染当前模块主界面。

## 执行步骤

1. **对照设计拆层级**：顶栏、表单区、主按钮、次要操作、协议文案等；忽略系统状态栏作为业务 UI（仅占位由 `Scaffold`/Insets 处理）。
2. **资源入库**：切图按 Android 命名规范（小写、下划线、字母开头）；优先 PNG 密度目录或矢量 `VectorDrawable`；禁止中文/空格文件名。
3. **文案与样式**：用户可见字符串进 `strings.xml`；颜色与间距优先主题与 dimension，避免魔法数。
4. **组件实现**：`Text` 不使用 `style` 参数；富文本用 `buildAnnotatedString` + `withStyle` / `LinkAnnotation` 按项目规则。
5. **状态占位**：为 Loading / Error / Empty 预留布局或与 `BaseCompose` 约定一致，不在 Composable 内发起网络请求。

## 与逻辑层协作边界

- Composable 只观察 `UiState`、派发 `event`（或调用 ViewModel 方法）。
- 导航、接口成功后的跳转、Token 存储等 **不得在 UI 文件实现业务闭环**（由 ViewModel / UseCase 驱动）。

## 自检清单

- [ ] 未把状态栏假 UI 画进业务内容区
- [ ] 资源命名与目录符合 Android 规范
- [ ] 页面级 `@Preview` 可运行
- [ ] 无 UI 层直接 `Retrofit` / `Repository` 调用

## 常见错误

- 远程非 Figma 图片 URL 充当设计门禁输入 → 应改为本地路径或可解析 Figma。
- 把错误提示写死在 Composable → 应走 `UiState` 与 `strings.xml`。
