---
name: android-mvvm-scaffold
description: 生成 Android MVVM（MMVM）基础架构，技术栈为 Kotlin + Compose Material3 + Hilt + Retrofit。包含 BaseViewModel 通用页面状态、BaseCompose 通用状态渲染、Compose 路由导航中心、ApiCenter 接口请求中心、接口基础返回数据类、自定义异常体系与协程请求封装。用户提到 Android 架构搭建、MVVM/MMVM 脚手架、基础框架时使用。
---

# Android MVVM 基础脚手架

## 目标
生成一套可复用的 Android MVVM 基础框架，必须包含：
1. `BaseViewModel` + 通用页面状态（`data class`）
2. `BaseCompose` 统一处理通用状态显示
3. Compose 路由导航中心
4. `ApiCenter`（统一存放接口请求）
5. 接口基础返回 `data class`
6. 自定义异常体系
7. 在 Compose 中通过 Hilt 注入 ViewModel
8. Kotlin + Compose Material3 + Retrofit 规范

## 强制技术栈
- Kotlin
- Jetpack Compose + Material3
- kotlinx-coroutines
- Hilt（`@HiltViewModel`、`hiltViewModel()`）
- Retrofit（接口定义、网络请求发起、与协程配合）

## 输出要求
执行本 Skill 时，必须输出：
- 清晰的目录结构
- 可编译的基础代码骨架
- 最小可运行示例（`SampleScreen` + `SampleViewModel`）
- 简短验证步骤（如何运行/验证）

## 推荐目录结构
可按项目实际包名微调，但保持职责清晰：

- `core/base/`
  - `BasePageState.kt`
  - `BaseViewModel.kt`
  - `BaseCompose.kt`
- `core/network/`
  - `ApiResponse.kt`
  - `AppException.kt`
  - `ApiCenter.kt`（接口中心入口）
  - `api/`（按业务拆分 Retrofit Service）
    - `UserApiService.kt`
    - `CommonApiService.kt`
  - `retrofit/`
    - `RetrofitFactory.kt`
    - `NetworkModule.kt`（Hilt 提供 Retrofit/Service）
- `core/navigation/`
  - `AppRoute.kt`
  - `NavCenter.kt`
- `feature/sample/`
  - `SampleViewModel.kt`
  - `SampleScreen.kt`

## 1) 通用状态模型（BasePageState）
创建 `BasePageState`，必须包含以下字段：

- `showLoading: Boolean`  // 网络加载框是否显示
- `showServiceDialog: Boolean`  // 是否显示客服弹窗
- `requiredPermissions: List<String>`  // 必须申请的权限列表
- `isNetworkError: Boolean`  // 是否网络错误
- `isLoginExpired: Boolean`  // 是否登录失效
- `errorMessage: String?`  // 要显示的错误信息

要求：
- 状态不可变，统一通过 `copy` 更新
- 默认值必须代表页面初始空闲状态

## 2) BaseViewModel 必须能力
`BaseViewModel` 需要提供：
- 对外只读的 `StateFlow<BasePageState>`
- 通用状态更新方法：
  - `showLoading() / hideLoading()`
  - `setServiceDialogVisible(Boolean)`
  - `setRequiredPermissions(List<String>)`
  - `setNetworkError(Boolean)`
  - `setLoginExpired(Boolean)`
  - `setErrorMessage(String?)`

并封装统一请求方法（等价签名即可）：
`launchRequest(showLoading: Boolean, handleCommonFailure: Boolean, onSuccess, onFailure)`

行为约束：
- `showLoading=true` 时自动处理加载状态显示/隐藏
- `handleCommonFailure=true` 时统一失败处理：
  - 网络异常 -> `isNetworkError=true`
  - 登录失效 -> `isLoginExpired=true`
  - 其他异常 -> 设置 `errorMessage`
- 支持调用方传入 `onFailure` 进行单独失败处理（回调时机保持一致并写清规则）

额外必须：
- 支持多个请求并发执行（如 `async/awaitAll`）
- 协程异常统一映射到自定义异常类
- 禁止吞异常（空处理）

## 3) ApiCenter（接口请求中心）要求
必须新增 `ApiCenter`，用于统一管理接口请求入口：

- 对外提供按业务聚合后的请求方法（如 `login()`、`getProfile()`、`fetchConfig()`）
- 内部依赖 Retrofit Service 接口（`*ApiService`）
- ViewModel/Repository 不直接散落调用多个 Service，实现“单入口调用”
- `ApiCenter` 可作为 facade，屏蔽底层接口拆分细节
- 支持后续扩展（多模块、多 baseUrl、鉴权头统一处理）

推荐职责：
- `ApiService`：只定义 Retrofit 注解接口
- `ApiCenter`：组织请求、聚合调用、必要的参数预处理
- Repository（可选）：做领域转换与缓存策略

## 4) 自定义异常体系
创建异常层级（sealed class 或等价设计），至少包含：
- `NetworkException`
- `ApiException(code, message)`
- `LoginExpiredException`
- `UnknownAppException`

并提供统一映射：
- `Throwable -> AppException`
- 保证错误提示可读（message 为空时要有兜底文案）

## 5) 接口基础返回模型
创建泛型返回模型 `ApiResponse<T>`，至少包含：
- `code: Int`
- `message: String?`
- `data: T?`

建议补充：
- `isSuccess()` 判断方法
- 或转换为领域层 Result 的方法

## 6) BaseCompose（通用状态渲染）
实现通用页面容器（例如 `BasePage`），接收 `BasePageState` 并统一渲染：

- Loading 遮罩（Material3 风格）
- 客服弹窗
- 网络异常提示（Dialog/Snackbar 均可）
- 登录失效提示（支持“重新登录”回调）
- 普通错误信息提示（Snackbar 或 Dialog）

建议 API 形态：
- `BasePage(state, onDismiss..., onRelogin..., content = { ... })`

要求：
- UI 展示逻辑与业务逻辑解耦
- 提供合理默认回调/插槽，方便复用

## 7) Compose 导航中心
提供统一导航中心：
- 路由定义（`AppRoute`，常量或 sealed class）
- `NavHost` 集中注册（`NavCenter`）
- 至少注册一个示例页面路由
- 路由命名清晰且稳定

## 8）Hilt + Compose 注入（强制）
必须包含：
- `@HiltViewModel` 的 ViewModel
- Compose 页面中使用 `hiltViewModel()`
- 若项目缺失，补齐应用入口配置：
  - `@HiltAndroidApp` Application
  - `@AndroidEntryPoint` Activity

网络与接口层注入必须满足：
- 通过 Hilt 提供 `Retrofit` 实例
- 通过 Hilt 提供各 `ApiService`
- 通过 Hilt 提供 `ApiCenter`
- **`ApiCenter` 必须通过构造函数注入到 ViewModel 内（constructor injection）**
- **禁止 ViewModel 直接依赖或调用 Retrofit `ApiService`**
- **ViewModel 只能通过 `ApiCenter` 发起接口请求**
必须包含：
- `@HiltViewModel` 的 ViewModel
- Compose 页面中使用 `hiltViewModel()`
- 若项目缺失，补齐应用入口配置：
  - `@HiltAndroidApp` Application
  - `@AndroidEntryPoint` Activity

同时必须包含网络注入：
- 通过 Hilt 提供 `Retrofit` 实例
- 通过 Hilt 提供各 `ApiService`
- 通过 Hilt 注入 `ApiCenter` 到上层（ViewModel/Repository）

## 9) 示例功能（必需）
必须提供一个最小示例：
- `SampleViewModel : BaseViewModel`
- `SampleScreen` 使用 `BasePage`
- 演示一个成功请求与一个失败请求（通过 `ApiCenter` 发起）
- 演示通用状态字段的使用（至少覆盖 loading + error）
示例要求（强制）：
- `SampleViewModel @Inject constructor(private val apiCenter: ApiCenter) : BaseViewModel()`
- 示例请求必须由 `apiCenter.xxx()` 发起
- 不允许在示例 ViewModel 中出现 `UserApiService`、`CommonApiService` 等 Service 直接注入

## 10) 编码规范
- 优先 Kotlin 惯用法（`data class`、sealed class、扩展函数）
- 代码简洁、可读、可编译
- 避免硬编码用户可见文案（可逐步替换到资源）
- 清理无用导入与死代码

## 11) 交付前检查清单
输出前逐项确认：
- [ ] 通用状态包含 6 个必填字段
- [ ] 请求封装支持 `showLoading` 与公共失败开关
- [ ] 支持并发请求执行
- [ ] 存在统一异常映射并被实际调用
- [ ] BaseCompose 能渲染全部通用状态
- [ ] 导航中心可运行并包含示例路由
- [ ] 已演示 Hilt 在 Compose 中注入 ViewModel
- [ ] 已集成 Retrofit 并通过 Hilt 注入
- [ ] `ApiCenter` 已作为统一接口请求入口被使用
- [ ] 使用 Material3 组件