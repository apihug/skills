# it-common-spring-ai-mcp

`it-common-spring-ai-mcp` 为 ApiHug 体系提供 **MCP（Model Context Protocol）Server** 支持，将 Spring AI MCP Server 与 Hope 的 JWT 鉴权、会话管理、切面体系深度整合，使业务服务能够以最小代价暴露为 AI Agent 工具。

---

## 设计原则

- **桥接 MCP 与 Hope 安全体系**：MCP 工具调用时，通过会话 ID 找到对应 `Customer`，自动注入 `HopeContextHolder`，使工具执行与普通 HTTP 请求共享一套认证/鉴权上下文。
- **双传输模式支持**：同时支持 WebMVC SSE（`HopeWebMvcSseServerTransportProvider`）与 WebFlux SSE（`HopeWebFluxSseServerTransportProvider`），根据应用类型自动选择。
- **可插拔认证**：通过 `McpServerAuthProvider` SPI 接口支持自定义认证；默认提供 `JWTMcpServerAuthProvider`，从 MCP 握手请求中提取 JWT Token 并解析出 `Customer`。
- **切面透传**：`AbstractMcpService` 内部调用 `AspectManager`，工具执行前后自动触发 `Aspect.before/after/exception`，与 HTTP 接口的切面行为一致。

---

## 核心组件

### `AbstractMcpService`
所有 MCP 服务的基类，实现 `ToolCallbackProvider`。

**工具执行生命周期：**
```
[收到工具调用]
    → contextBridge(exchange)    // 从 MCP Session 恢复 Customer 到 ThreadLocal
    → validate(input)            // Jakarta Validation 校验输入
    → AspectManager.before()
    → doCall(args)               // 子类实现具体业务
    → AspectManager.after()      // 或 exception()
    → 序列化结果为 JSON 返回
```

**自定义工具示例：**
```java
@Component
public class UserQueryMcpService extends AbstractMcpService {

    @Override
    public ToolCallback[] getToolCallbacks() {
        return new ToolCallback[]{ new QueryUserTool() };
    }

    class QueryUserTool extends MyToolCallback<QueryInput, UserDTO> {
        @Override protected String name() { return "query_user"; }
        @Override protected String description() { return "Query user by id"; }
        @Override protected String inputSchema() { return "..."; }
        @Override protected Class<QueryInput> inputType() { return QueryInput.class; }
        @Override protected ServiceMethodContext path() { return UserApiModule.QUERY_USER; }

        @Override
        protected UserDTO doCall(QueryInput args) {
            return userService.query(args.getId());
        }
    }
}
```

### `McpSessionManager`
并发会话管理接口，维护 `sessionId → McpServerSession` 及对应的 `McpServerSessionContext`（含 `Customer`）。

```java
public interface McpSessionManager extends ConcurrentMap<String, McpServerSession> {
    Optional<McpServerSession> session(String sessionId);
    Optional<McpServerSessionContext> sessionContext(String sessionId);
    void put(String sessionId, McpServerSession session, McpServerSessionContext context);
}
```

默认实现 `DefaultMcpSessionManager` 使用 `ConcurrentHashMap`，支持应用级替换。

### `McpServerAuthProvider`
MCP 握手认证 SPI：

```java
public interface McpServerAuthProvider {
    McpServerAuthProvider NO_AUTH = context -> {};  // 无认证默认实现
    void auth(McpServerSessionContext context);
}
```

`JWTMcpServerAuthProvider` 从 HTTP 请求头（`Authorization`，可配置）中提取 JWT，解析后设置 `Customer` 到 `McpServerSessionContext`。

---

## 配置属性

配置前缀：`hope.ai.mcp`

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `hope.ai.mcp.enabled` | `false` | 是否启用 Hope MCP 扩展 |
| `hope.ai.mcp.authKey` | `Authorization` | MCP 握手时提取 JWT 的 Header 名称 |

**application.yaml 示例：**
```yaml
hope:
  ai:
    mcp:
      enabled: true
      authKey: Authorization
```

---

## 依赖

```gradle
platform(libs.spring.ai.bom)
optional(libs.spring.ai.starter.mcp.server)
optional(libs.spring.ai.starter.mcp.server.webmvc)
optional(libs.spring.ai.starter.mcp.server.webflux)
implementation project(":it-common-spring-plus:it-common-spring-security")
implementation project(":it-common-spring-plus:it-common-spring-core")
```

---

## 自动配置条件

- `HopeMcpServerCondition`：检查 `hope.ai.mcp.enabled=true`
- `ConditionalOnClass(McpSchema.class, McpSyncServer.class)`：classpath 存在 MCP 依赖
- 传输层：`ConditionalOnWebApplication(SERVLET)` → WebMVC；`ConditionalOnWebApplication(REACTIVE)` → WebFlux

---

## 与 Hope 安全体系集成

MCP 工具调用时，`contextBridge` 方法自动执行：

```
MCP Exchange → ExchangeSessionHelper.sessionId()
    → McpSessionManager.sessionContext(sessionId)
    → McpServerSessionContext.customer()
    → HopeContextHolder.setContext(CustomerWithContext)
```

若找不到对应会话，则注入匿名 `Customer`（`JwtCustomizer.anonymousCustomer()`）。
