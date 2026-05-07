---
name: android-module-data-from-api
description: Use when implementing or updating Android network and persistence layers from API documentation; use when mapping endpoints to Retrofit services, DTOs, repositories, auth headers, and error codes for a specific feature module.
---

# 从接口文档实现后台交互（Data 层）

## 前置条件

- 持有 **接口说明**：OpenAPI/Swagger JSON、YAML、Postman Collection、或表格化 Markdown（含路径、方法、请求/响应示例、鉴权、错误码）。
- 已知 **环境**：base URL、是否多环境、TLS/证书要求（仅描述，密钥不进仓库）。

若文档缺失关键字段（如鉴权方式、错误体结构）：列出 **阻塞项**，禁止猜测生产路径或 token 头名称。

## 输出目标

- **Retrofit `interface`**：按模块或领域拆分；方法命名与文档一致可读。
- **DTO**：与 JSON 字段对齐；使用 `@SerializedName` 或项目既定序列化方案；可空性与默认值反映真实接口（宁可可空 + 明确映射）。
- **Repository**：对外返回统一 `Result` / `ApiResponse` 包装（与项目现有 `ApiCenter` / 异常体系一致）。
- **错误映射**：HTTP 状态、业务 `code`、字段校验错误 → 领域错误或 `UiState` 可用的错误模型。

## 执行步骤

1. **列出本模块端点表**：方法、路径、用途、是否需登录、幂等性备注。
2. **定义请求/响应模型**：先写数据类再写接口方法；复杂嵌套拆文件避免单文件过大。
3. **鉴权与拦截器**：Token、刷新、公共头、时间戳签名等放在 OkHttp 拦截器或 `NetworkModule`，不在每个 Service 重复。
4. **Mock/契约**：若文档有示例 JSON，至少用单元测试或 fake 解析校验；可选记录 sample 到 `test/resources`（不含密钥）。
5. **与需求对齐**：接口字段是否满足 UI 展示与流程；不满足则上报 **需求-接口差距表**。

## 与 UI 的边界

- Data 层 **不知道** Compose；不返回「已格式化 UI 字符串」除非项目已有 i18n 层。
- 敏感信息：**禁止** 写入日志、示例代码、或提交到 git。

## 自检清单

- [ ] 所有路径与文档一致（含前缀版本号）
- [ ] 错误体与全局异常处理器兼容
- [ ] 可空字段与列表空数组语义已注释或测试
- [ ] 无硬编码密钥；base URL 来自 `BuildConfig` 或配置层

## 常见错误

- 直接把 JSON 字段名当作 Kotlin 属性不设序列化注解 → 发布环境混淆或命名策略下解析失败。
- Repository 返回裸 DTO 泄漏给 UI → 应在领域层映射为 `User`/`Session` 等模型（若项目已有此约定）。
