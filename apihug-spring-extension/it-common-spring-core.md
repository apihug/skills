# it-common-spring-core

`it-common-spring-core` 是 ApiHug Spring 体系的**核心基础设施层**，提供统一异常处理、分页请求守卫、参数校验扩展、切面管理器及 Web 头部上下文传播，同时兼容 Servlet 与 Reactive（WebFlux）双运行时。

---

## 设计原则

- **异常处理可插拔**：`ApiExceptionHandler` SPI 允许业务模块注册专属异常处理器；框架内置覆盖 Spring MVC / WebFlux 所有常见异常类型，开箱即用。
- **统一响应格式**：所有异常最终通过 `SimpleResultBuilder` 转换为标准 `Result<?>` 响应，与正常业务响应格式一致。
- **防御式分页**：`PageRequestGuardian` 强制校验分页参数边界，防止恶意超大 `pageSize` 导致的数据库压力。
- **双运行时并行支持**：Servlet 路径（`servlet` 包）与 Reactive 路径（`reactive` 包）分别实现，通过 `@ConditionalOnWebApplication` 自动选择。
- **切面链集中管理**：`AspectManager` 作为 `Aspect` 执行链的统一入口，在 MCP 工具调用、HTTP 请求拦截中复用同一切面逻辑。

---

## 核心组件

### 异常处理体系

**`ApiExceptionHandler`**（SPI 接口）：

```java
public interface ApiExceptionHandler {
    boolean canHandle(Throwable exception);   // 是否能处理此异常
    boolean exactly(Throwable exception);     // 是否精确匹配（用于排序）
    void handle(Throwable exception, SimpleResultBuilder builder, Locale locale);
}
```

**内置处理器（`handler` 包）：**

| 处理器 | 覆盖异常 |
|--------|---------|
| `HopeErrorDetailExceptionHandler` | `HopeErrorDetailException`（业务异常） |
| `BindExceptionHandler` | 参数绑定/校验失败 |
| `ConstraintViolationExceptionHandler` | Jakarta Bean Validation 违规 |
| `HttpMessageNotReadableExceptionHandler` | 请求体解析失败 |
| `HttpMediaTypeExceptionHandler` | 媒体类型不支持/不接受 |
| `TypeMismatchExceptionHandler` | 参数类型转换失败 |
| `MissingRequestValueExceptionHandler` | 必填参数缺失 |
| `SpringSecurityExceptionHandler` | Spring Security 认证/授权异常 |
| `ServerErrorExceptionHandler` | 服务端 5xx 错误 |
| `ObjectOptimisticLockingFailureApiExceptionHandler` | 乐观锁失败 |
| `AsyncRequestTimeoutExceptionHandler` | 异步请求超时 |
| `ServerWebInputExceptionHandler` | WebFlux 输入异常 |

**使用示例（自定义异常处理器）：**

```java
@Component
public class MyExceptionHandler implements ApiExceptionHandler {

    @Override
    public boolean canHandle(Throwable exception) {
        return exception instanceof MyBusinessException;
    }

    @Override
    public void handle(Throwable exception, SimpleResultBuilder builder, Locale locale) {
        MyBusinessException ex = (MyBusinessException) exception;
        builder.badRequest().error(ex.getError());
    }
}
```

**`HopeProblemProperties`**（配置前缀：`hope.problem`）：

| 属性 | 说明 |
|------|------|
| `log-level` | 异常日志级别（DEBUG/INFO/WARN/ERROR） |
| `log-stack-trace` | 是否打印堆栈 |
| `forbidden-explicit-http-error-code` | 禁止显式 HTTP 错误码（统一返回 200） |

---

### `PageRequestGuardian` 分页守卫

防止非法分页参数，静态工厂方法：

```java
// 在业务方法入口使用
PageRequest safeRequest = PageRequestGuardian.guard(rawPageRequest);
```

**`HopePageProperties`**（配置前缀：`hope.page`）：

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `default-page-size` | `20` | 默认每页条数 |
| `max-page-size` | `200` | 最大每页条数（超出强制截断） |
| `default-batch-size` | `50` | 默认批处理大小 |
| `max-batch-size` | `500` | 最大批处理大小 |

---

### 参数校验扩展（`validator` 包）

内置自定义 Jakarta Validation 约束：

| 注解 | 说明 |
|------|------|
| `@ChinaMobile` | 中国大陆手机号格式校验 |
| `@Enums` | 枚举值合法性校验（值必须在指定枚举中） |

**使用示例：**

```java
public class RegisterRequest {
    @ChinaMobile
    private String phone;

    @Enums(enumClass = StatusEnum.class)
    private String status;
}
```

**`ValidationUtils`** 提供编程式校验工具方法。

---

### `AspectManager`

全局切面链管理器，聚合所有注册的 `Aspect` Bean 并按 `priority` 排序：

```java
// 在拦截器或 MCP 工具调用中使用
AspectManager.get().before(methodContext, parameters);
// ... 执行业务 ...
AspectManager.get().after(methodContext, parameters, result);
```

---

### Web 头部上下文传播

- **Servlet**：`HopeServletHeaderContextAppender` —— 从请求头提取 `x-hope-source`、`x-request-id` 等标准头，注入运行时上下文。
- **Reactive**：`HopeReactiveHeaderContextAppender` —— 同上，基于 `WebFilter` 实现。

---

### Actuator 集成

`HopeProjectInfoContributor` 实现 `InfoContributor`，将 Proto 项目元数据（版本、应用名等）注入 `/actuator/info` 端点：

```json
{
  "hope": {
    "application": "my-service",
    "version": "1.0.0",
    "domain": "user"
  }
}
```

---

## 配置示例

```yaml
hope:
  problem:
    log-level: WARN
    log-stack-trace: true
    forbidden-explicit-http-error-code: false
  page:
    default-page-size: 20
    max-page-size: 100
```

---

## 依赖

```gradle
implementation project(":it-common")
implementation project(":it-common-proto")
implementation project(":it-common-spring-plus:it-common-spring")
optional "org.springframework.boot:spring-boot-starter-web"
optional "org.springframework.boot:spring-boot-starter-actuator"
optional "org.springframework.boot:spring-boot-starter-security"
optional "org.springframework:spring-webflux"
optional "org.springframework.boot:spring-boot-starter-validation"
```
