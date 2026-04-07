# it-common-spring-api

`it-common-spring-api` 为 ApiHug 服务提供 **OpenAPI 元数据端点**，将 Proto 定义生成的 API 文档、错误码、权限、字典、版本信息通过 HTTP 端点对外暴露，同时支持 MCP Server 工具描述集成。

---

## 设计原则

- **Proto 元数据直接暴露**：API 文档来自 Proto 编译时生成的 JSON 资源文件（classpath），无运行时扫描，启动快、内存占用小。
- **WebMVC / WebFlux 双适配**：`WebMvcConfiguration` 与 `WebFluxConfiguration` 分别注册 Servlet 和 Reactive 版本的端点，由 Spring Boot 条件化自动选择。
- **可扩展 Provider 机制**：`ApiProvider` SPI 支持多来源 API 文档聚合，`ApiProviderManager` 负责调度；内置 `ResourcesApiProvider` 从 classpath 加载。
- **生产环境可关闭**：通过 `hope.api.enable=false` 彻底关闭 Swagger 端点，避免生产暴露 API 信息。

---

## 核心组件

### `ApiProvider`
API 文档提供者接口：

```java
public interface ApiProvider {
    String api();                                    // 返回 JSON 格式 API
    String api(boolean yaml);                        // 返回指定格式
    String apiWithVersion(String version);           // 返回指定版本
    String apiWithVersion(String version, boolean yaml);  // 主接口
}
```

内置实现 `ResourcesApiProvider` 直接读取 classpath 中由 Proto 插件生成的 `hope-wire.json`。

### `ApiMetaResource`
抽象元数据资源基类，负责懒加载以下资源（均来自 Proto 编译产物）：

| 资源 | 说明 |
|------|------|
| `api()` | OpenAPI/Swagger JSON 或 YAML |
| `errors()` | 错误码字典 |
| `authorities()` | 权限定义列表 |
| `dictionaries()` | 枚举/字典数据 |
| `versions()` | API 版本历史 |

子类 `ApiWebMvcResource` / `ApiWebFluxResource` 分别实现 Servlet 与 Reactive HTTP 端点。

### `HopeMetaApiConfiguration`
元数据端点自动配置，注册 `/meta/errors`、`/meta/authorities`、`/meta/dictionaries` 等端点。

---

## 配置属性

配置前缀：`hope.api`

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `hope.api.enable` | `true` | 是否启用 API 端点（生产可设 `false`） |
| `hope.api.path` | `/v3/api-docs` | OpenAPI 文档路径 |
| `hope.api.showActuator` | `false` | 是否在 Swagger UI 显示 Actuator 端点 |
| `hope.api.showMeta` | `true` | 是否启用元数据端点 |
| `hope.api.project.name` | - | 关联的 Proto 项目名称 |
| `hope.api.project.latest` | `true` | 是否取最新版本文档 |
| `hope.api.mcp.mcpServer` | `openapi-server` | MCP Server 名称 |
| `hope.api.mcp.toolPrefixName` | `""` | MCP 工具名前缀 |
| `hope.api.mcp.serverUrl` | - | 运行时服务器 URL（如 `http://localhost:8080`） |

**application.yaml 示例：**
```yaml
hope:
  api:
    enable: true
    path: /v3/api-docs
    show-meta: true
    project:
      name: my-api-proto
    mcp:
      server-url: http://localhost:8080
```

---

## 暴露的端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `{path}` | GET | OpenAPI JSON（默认 `/v3/api-docs`） |
| `{path}.yaml` | GET | OpenAPI YAML 格式 |
| `{path}/{version}` | GET | 指定版本的 OpenAPI 文档 |
| `/meta/errors` | GET | 错误码清单 |
| `/meta/authorities` | GET | 权限定义清单 |
| `/meta/dictionaries` | GET | 枚举字典数据 |
| `/meta/versions` | GET | API 版本列表 |

---

## 依赖

```gradle
implementation project(":it-common-proto")
optional "org.springframework:spring-web"
optional "org.springframework:spring-webflux"
optional 'org.springframework.boot:spring-boot-autoconfigure'
```

---

## 使用方式

引入依赖后自动配置生效（`hope.api.enable=true`）。通常在 ApiHug 应用中无需手动配置，Proto 插件生成的 wire 包会自动注册 `ApiProvider` Bean。

**禁用 API 端点（生产环境）：**
```yaml
hope:
  api:
    enable: false
```

**自定义 API 文档来源：**
```java
@Bean
public ApiProvider myApiProvider() {
    return (version, yaml) -> myCustomApiJson;
}
```
