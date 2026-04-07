# it-common-spring

`it-common-spring` 是 ApiHug Spring 扩展体系的**基础抽象层**，提供与业务领域解耦的核心模型、安全抽象、响应构建及切面契约。所有上层 Spring 模块均依赖此模块。

---

## 设计原则

- **与 Spring Security 解耦**：不使用 `GrantedAuthority` / `UserDetails`，定义了自己的 `Customer` 模型，保持灵活的多租户、多身份类型泛型能力。
- **编译期安全**：`SecurityContext` 的访问控制规则在设计时（Protobuf 定义）确定，代码生成阶段固化，运行时无反射、无表达式解析，性能极高。
- **Builder 模式统一响应**：`SimpleResultBuilder` / `PageableResultBuilder` 封装 `ResponseEntity<Result<T>>`，统一错误码、消息、payload。
- **切面契约标准化**：`Aspect` 接口定义 `before/after/exception` 三阶段钩子，可插拔地挂载日志、指标、鉴权等横切关注点。

---

## 核心组件

### `Customer<T, Identify, Tenant>`
通用用户模型，支持泛型身份 ID 类型（`Long`/`Integer`/`String`）与租户 ID 类型。

| 属性 | 说明 |
|------|------|
| `id` | 用户唯一标识 |
| `tenantId` | 租户 ID |
| `account` | 登录账号 |
| `name` | 用户名 |
| `roles` | 角色集合（如 `admin`/`player`） |
| `authorities` | 权限集合（如 `user:add`/`order:delete`） |
| `active` | 账号是否有效 |

内置变体（`internal` 包下）：9 种 ID×Tenant 类型组合（`Long/Integer/String` × `Long/Integer/String`）。

### `SecurityContext<MODULE>`
抽象安全上下文，管理所有 Proto 定义资源的访问规则。

**路径匹配 DSL：**
```java
// 精确、前缀、后缀、包含、正则、Ant 路径匹配
exactly("/api/admin").deny();
start("/api/public").permit();
ant("/api/user/**").login();
regular("^/api/v\\d+/.*").active();
```

**默认策略：**
```java
denyAsDefault()    // 未配置资源全部拒绝（默认）
passAsDefault()    // 未配置资源全部放行
loginAsDefault()   // 未配置资源需登录
activeAsDefault()  // 未配置资源需登录且账号激活
```

**角色/权限联合校验：**
```java
start("/api/admin")
    .roles("ADMIN")
    .authorities(AuthorityEnum.USER_DELETE)
    .and()
    .security();
```

### `SimpleResultBuilder<PayLoad>`
Spring MVC/WebFlux 统一响应构建器，链式 API：

```java
return new SimpleResultBuilder<UserDTO>()
    .ok()
    .payload(userDTO)
    .done();

// 错误场景
return new SimpleResultBuilder<>()
    .badRequest()
    .error(BizErrorsEnum.USER_NOT_FOUND)
    .done();
```

### `PageableResultBuilder<Payload>`
分页响应构建器，扩展自 `SimpleResultBuilder`：

```java
return new PageableResultBuilder<UserDTO>()
    .ok()
    .setPageIndex(1)
    .setPageSize(20)
    .setTotalCount(200L)
    .setData(users)
    .done();
```

### `AuditContext<Identify, Tenant>`
审计上下文，携带当前操作人 ID 与租户 ID，与 `Customer` 双向适配：

```java
AuditContext ctx = AuditContext.from(currentCustomer);
// 或
AuditContext ctx = AuditContext.builder()
    .identifier(userId)
    .tenant(tenantId)
    .build();
```

### `Aspect` 接口
横切关注点标准契约：

```java
public interface Aspect {
    AspectType type();           // 分类（AUTH/LOGGING/METRICS...）
    default int priority() { return 0; }  // 执行优先级
    void before(ServiceMethodContext method, Map<String, Object> ctx);
    void after(ServiceMethodContext method, Map<String, Object> ctx, ResponseEntity<Result<T>> result);
    void exception(ServiceMethodContext method, Map<String, Object> ctx, Throwable throwable);
}
```

---

## 依赖

```gradle
implementation project(":it-common")
implementation project(":it-common-proto")
optional "org.springframework:spring-context"
optional "org.springframework:spring-web"
optional "jakarta.validation:jakarta.validation-api"
```

---

## 使用方式

引入依赖后，通常无需显式配置。上层模块（`it-common-spring-security`、`it-common-spring-core` 等）会自动拉取此模块的依赖。

如需直接使用响应构建器：

```java
// 在 Controller 或 Service 中直接 new
SimpleResultBuilder<MyDTO> builder = new SimpleResultBuilder<>();
return builder.ok().payload(myDto).done();
```

如需自定义安全上下文，继承 `SecurityContext` 并在 `@Configuration` 中注册：

```java
@Component
public class MyModuleSecurityContext extends SecurityContext<MyModule> {
    @Override
    protected MyModule module() { return MyModule.INSTANCE; }

    @Override
    public void afterPropertiesSet() {
        super.afterPropertiesSet();
        start("/api/public").permit();
        denyAsDefault();
    }
}
```
