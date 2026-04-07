# it-common-spring-tool

`it-common-spring-tool` 为 ApiHug 体系提供**通用工具集**，包含 Spring Web 场景下的常用辅助工具函数，兼容 Servlet 和 WebFlux 双运行时。

> **当前状态**：模块骨架已建立，工具类能力正在持续完善中。

---

## 设计原则

- **零侵入**：工具类仅封装通用能力，不引入业务依赖，可在任意 Spring 服务中引用。
- **轻量依赖**：所有依赖均为 `optional`，按需引入，不强制传递依赖。
- **双运行时支持**：Servlet（`spring-web`）与 Reactive（`spring-webflux`）场景的工具方法并行提供。

---

## 技术栈

| 组件 | 说明 |
|------|------|
| `spring-context` | Spring 上下文工具 |
| `spring-web` | Servlet Web 工具 |
| `spring-webflux` | Reactive Web 工具 |
| `jackson-databind` | JSON 序列化/反序列化工具 |
| `spring-boot-autoconfigure` | 自动配置支持 |

---

## 依赖

```gradle
optional "org.springframework:spring-context"
optional "org.springframework:spring-web"
optional "org.springframework:spring-webflux"
optional "com.fasterxml.jackson.core:jackson-databind"
optional "jakarta.validation:jakarta.validation-api"
optional 'org.springframework.boot:spring-boot-autoconfigure'
```

---

## 路线图

- [ ] HTTP 请求/响应工具类（Header 提取、参数解析）
- [ ] IP 地址解析工具（支持代理透传）
- [ ] JSON 工具类（基于 Jackson，统一序列化配置）
- [ ] Bean 复制/转换工具
- [ ] 时间日期工具类（统一时区处理）
- [ ] Spring 上下文工具（Bean 查找、环境属性读取）
