# it-common-spring-security

`it-common-spring-security` 是 ApiHug 体系的**JWT 安全模块**，基于 Spring Security OAuth2 JOSE 提供 JWT 令牌的签发、解析、过滤及 Hope 用户上下文注入，同时支持 Servlet 和 Reactive（WebFlux）双运行时，与 `it-common-spring` 中的 `SecurityContext` 深度整合。

---

## 设计原则

- **JWT 集中管理**：`JwtCustomizer` SPI 将 JWT 的编码（`encode`）、解码（`decode`）、Claims 组装（`prepareJwt`）、解码后补全（`postDecode`）统一封装为一个可扩展接口，业务只需实现一个 Bean。
- **Hope 上下文透明注入**：JWT 过滤器解析 Token 后，自动将 `Customer` 注入 `HopeContextHolder`（ThreadLocal），业务代码通过 `HopeContextHolder.getContext()` 无感知获取当前用户。
- **内部资源访问控制**：`InternalResourceAccessChecker` 检查请求头 `x-hope-source`，内部服务调用（无 `external` 标记）可访问内部接口，外部请求被拒绝。
- **双运行时无缝切换**：`ServletSecurityConfiguration`（`@ConditionalOnWebApplication(SERVLET)`）和 `ReactiveSecurityConfiguration`（`@ConditionalOnWebApplication(REACTIVE)`）自动选择，无需额外配置。
- **可插拔 JWT 配置**：`JwtTrivialConfigurationProvider` SPI 允许自定义 MAC 算法、JWT 异常处理逻辑。

---

## 核心组件

### `JwtCustomizer<T extends Customer>`

JWT 操作核心 SPI，业务必须实现此接口并注册为 Spring Bean：

```java
@Component
public class MyJwtCustomizer implements JwtCustomizer<MyCustomer> {

    @Autowired
    private JwtEncoder jwtEncoder;
    @Autowired
    private JwtDecoder jwtDecoder;

    @Override
    public String encode(MyCustomer customer, boolean rememberMe) {
        JwtClaimsSet.Builder claims = JwtClaimsSet.builder();
        prepareJwt(claims, customer);
        // 设置过期时间...
        return jwtEncoder.encode(JwtEncoderParameters.from(claims.build())).getTokenValue();
    }

    @Override
    public void prepareJwt(JwtClaimsSet.Builder claims, MyCustomer customer) {
        claims.subject(String.valueOf(customer.getId()));
        claims.claim("roles", customer.getRoles());
        claims.claim("authorities", customer.getAuthorities());
        claims.claim("tenantId", customer.getTenantId());
    }

    @Override
    public MyCustomer decode(String token) {
        Jwt jwt = jwtDecoder.decode(token);
        return postDecode(new MyCustomer(), jwt);
    }

    @Override
    public MyCustomer postDecode(MyCustomer customer, Jwt jwt) {
        Map<String, Object> claims = jwt.getClaims();
        customer.setId(Long.valueOf(jwt.getSubject()));
        customer.setRoles((Collection<String>) claims.get("roles"));
        customer.setTenantId(safe2Long(claims.get("tenantId"), 0L));
        return customer;
    }

    @Override
    public MyCustomer customer() { return new MyCustomer(); }

    @Override
    public MyCustomer anonymousCustomer() {
        return (MyCustomer) new MyCustomer().setId(0L).setActive(false);
    }
}
```

### `HopeSecurityManager`

运行时安全管理门面，整合 `SecurityContext`（权限规则）与 `JwtCustomizer`（用户解析），提供统一的权限检查方法供 JWT 过滤器调用。

### `SecurityAspect`

`Aspect` 接口实现，作为切面在每次 API 方法调用前执行权限校验，与 `SecurityContext` 中定义的路径规则联动。

### `InternalResourceAccessChecker`

内部资源访问检查器，通过请求头区分请求来源：

```
x-hope-source: external  →  外部请求，只能访问公开资源
x-hope-source: [其他]    →  内部请求，可访问内部资源
```

### `DefaultJwtCustomizer`

`JwtCustomizer` 的默认实现，提供基础 JWT Claims 解析（`sub`、`iat`、`exp`），可作为自定义实现的起点。

### `RunTimeSecurityFactory`

运行时 Customer 工厂，从 `HopeContextHolder` 获取当前请求的 `Customer`，供业务代码调用：

```java
// 获取当前用户
Customer currentUser = RunTimeSecurityFactory.customer();
Long userId = (Long) currentUser.getId();
```

---

## 配置属性

配置前缀：`hope.security`

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `hope.security.enable` | `true` | 是否启用 Hope 安全模块 |
| `hope.security.default-access-style` | `DENY` | 未配置资源的默认访问策略 |
| `hope.security.request-origin-header` | `x-hope-source` | 请求来源 Header 名称 |
| `hope.security.external-origin-value` | `external` | 标识外部请求的 Header 值 |
| `hope.security.bypass-origin-check` | `false` | 是否跳过来源检查（测试用） |
| `hope.security.jwt.secret` | `NwskoUmKHZtzGRKJKVjsJF7BtQMMxNWi` | JWT 签名密钥（**生产必须替换**） |
| `hope.security.jwt.base64-secret` | - | Base64 编码的签名密钥（优先级高于 `secret`） |
| `hope.security.jwt.token-validity-in-seconds` | `604800`（7天） | Token 有效期（秒） |
| `hope.security.jwt.token-validity-in-seconds-for-remember-me` | `2592000`（30天） | 记住我 Token 有效期（秒） |

`DefaultAccessStyle` 枚举值：

| 值 | 说明 |
|----|------|
| `PASS` | 未配置资源全部放行（匿名） |
| `DENY` | 未配置资源全部拒绝（默认） |
| `LOGIN` | 未配置资源需登录 |
| `ACTIVE` | 未配置资源需登录且账号激活 |

**application.yaml 示例：**

```yaml
hope:
  security:
    enable: true
    default-access-style: DENY
    bypass-origin-check: false
    jwt:
      base64-secret: "your-base64-encoded-secret-key-here"
      token-validity-in-seconds: 86400  # 24 小时
```

---

## JWT 过滤器工作流程

```
HTTP 请求
    → JwtFilter（Servlet/Reactive）
    → JwtCustomizer.jwtPicker()       # 从 Header/Cookie 提取 Token
    → JwtDecoder.decode(token)        # Nimbus JWT 解码
    → JwtCustomizer.postDecode()      # 补全 Customer 信息
    → HopeContextHolder.setContext()  # 注入 ThreadLocal
    → SecurityAspect.before()         # 执行权限校验
    → 业务方法执行
    → HopeContextHolder.clear()       # 清理 ThreadLocal
```

---

## 自动配置

**Servlet 环境（`spring-boot-starter-web`）：**

```
ServletSecurityConfiguration
    ├── JwtFilter（OncePerRequestFilter）
    ├── HopeSecurityManager（Bean）
    ├── RunTimeSecurityFactory（Bean）
    ├── SecurityAspect（Bean）
    └── JwtDecoder / JwtEncoder（Bean，基于 Nimbus）
```

**Reactive 环境（`spring-webflux`）：**

```
ReactiveSecurityConfiguration
    ├── JwtFilter（WebFilter）
    ├── HopeSecurityManager
    └── JwtDecoder / JwtEncoder
```

---

## 依赖

```gradle
api project(":it-common")
api project(":it-common-proto")
api project(":it-common-spring-plus:it-common-spring")
optional "org.springframework.boot:spring-boot-starter-web"
optional "org.springframework:spring-webflux"
optional "org.springframework.security:spring-security-oauth2-jose"
```

> **重要**：`it-common-spring-security` 以 `api` 方式导出 `it-common-spring` 依赖，下游模块引入后即可直接使用 `Customer`、`SecurityContext` 等核心类型。
