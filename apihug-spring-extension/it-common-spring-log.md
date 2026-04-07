# it-common-spring-log

`it-common-spring-log` 为 ApiHug 体系提供**结构化日志**扩展支持，统一服务间的日志格式、请求链路追踪日志采集及 Hope 上下文信息注入，兼容 Servlet 和 WebFlux 双运行时。

> **当前状态**：模块骨架已建立，结构化日志能力正在持续完善中。

---

## 设计原则

- **结构化优先**：日志以 JSON 结构输出，包含 `requestId`、`userId`、`tenantId`、`application`、`domain` 等标准字段，便于 ELK/Loki 等日志平台采集与检索。
- **Hope 上下文自动注入**：在 MDC（Mapped Diagnostic Context）中自动注入当前请求的 Hope 上下文信息（用户、租户、应用名等），无需业务代码手动传递。
- **轻依赖**：仅依赖 Spring Context/Web/WebFlux，不强制引入特定日志实现，与 Logback/Log4j2 均兼容。

---

## 技术栈

| 组件 | 说明 |
|------|------|
| `spring-context` | Spring 上下文支持 |
| `spring-web` | Servlet Web 支持 |
| `spring-webflux` | Reactive Web 支持 |
| `spring-boot-autoconfigure` | 自动配置支持 |

---

## 依赖

```gradle
optional "org.springframework:spring-context"
optional "org.springframework:spring-web"
optional "org.springframework:spring-webflux"
optional 'org.springframework.boot:spring-boot-autoconfigure'
```

---

## 路线图

- [ ] MDC 自动注入过滤器（Servlet + WebFlux）
- [ ] 请求/响应结构化日志记录
- [ ] 慢请求日志告警
- [ ] 链路 TraceId 传播（与 Micrometer Tracing 集成）
- [ ] 日志级别动态调整端点
