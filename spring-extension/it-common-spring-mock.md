# it-common-spring-mock

`it-common-spring-mock` 为 ApiHug 微服务提供**基于 WireMock 的 Contract Mock Server**，通过解析 Proto 定义的 API 元数据自动生成 Mock 桩（Stub），使下游服务的集成测试无需真实依赖上游服务即可执行。

---

## 设计原则

- **契约驱动 Mock**：`HopeContractConfiguration` 从 Proto 生成的 `Module`（服务元数据）中读取所有 API 路径、HTTP 方法及响应结构，自动注册 WireMock Stub，无需手工编写桩文件。
- **Mock 数据自动生成**：通过 `WireContext` 和 `RuntimeContext` 根据 Proto 定义的响应类型自动生成合理的 Mock 响应数据，包括枚举值、数字、字符串等。
- **可定制 Stub**：`WireMockStubCustomizer` SPI 允许测试代码注入自定义 Stub，在自动生成的基础上叠加特殊场景的响应。
- **测试生命周期管理**：`WireMockApplicationListener` 与 Spring 测试生命周期集成，测试前启动 WireMock Server，测试后自动关闭，可配置是否在每次测试后重置 Stub。

---

## 核心组件

### `AutoConfigureWireMock`

测试类注解，一键启用 WireMock Server 自动配置：

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureWireMock(port = 0)  // port=0 表示随机端口
class UserServiceTest {
    // WireMock Server 已自动启动并注册到 Spring 上下文
}
```

### `WireMockProperties`

配置前缀：`wiremock`

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `wiremock.server.port` | `8080` | WireMock 监听端口 |
| `wiremock.server.https-port` | `-1` | HTTPS 端口（-1 禁用） |
| `wiremock.server.port-dynamic` | `false` | 是否使用随机端口 |
| `wiremock.server.https-port-dynamic` | `false` | 是否使用随机 HTTPS 端口 |
| `wiremock.server.stubs` | `[]` | Stub 配置目录路径 |
| `wiremock.server.files` | `[]` | 响应文件目录路径 |
| `wiremock.reset-mappings-after-each-test` | `false` | 是否每次测试后重置 Stub |
| `wiremock.rest-template-ssl-enabled` | `false` | RestTemplate 是否启用 SSL |

### `HopeContractConfiguration`

契约 Mock 配置基类，继承后通过 `module()` 方法返回要 Mock 的 Proto Module：

```java
@Configuration
public class UpstreamServiceMockConfig extends HopeContractConfiguration {

    @Override
    protected Module module() {
        // 返回上游服务的 Proto Module（由代码生成）
        return UpstreamApiModule.INSTANCE;
    }

    // 可选：自定义 RuntimeContext（如控制 Mock 数据格式）
    @Override
    protected RuntimeContext runtimeContext() {
        RuntimeContext ctx = super.runtimeContext();
        ctx.setResultPlain(true);  // 返回纯 JSON
        return ctx;
    }
}
```

框架自动遍历 `Module` 中所有 `Service` → `ServiceMethod`，生成对应 HTTP 方法的 WireMock Stub：

- **路径参数**（`/api/{id}`）→ `urlPathTemplate` 匹配
- **普通路径**（`/api/users`）→ `urlPathEqualTo` 精确匹配

### `WireMockStubCustomizer`

自定义 Stub SPI，在自动生成的 Contract Stub 之上追加或覆盖：

```java
@Component
public class MyStubCustomizer implements WireMockStubCustomizer {

    @Override
    public String name() { return "my-customizer"; }

    @Override
    public void stub(WireMockServer server) {
        // 追加特殊场景的 Stub
        server.stubFor(
            get(urlPathEqualTo("/api/users/999"))
                .willReturn(aResponse().withStatus(404))
        );
    }
}
```

### `WireMockConfigurationCustomizer`

自定义 WireMock Server 配置（超时、扩展等）：

```java
@Bean
public WireMockConfigurationCustomizer myConfigCustomizer() {
    return config -> config
        .timeout(5000)
        .notifier(new ConsoleNotifier(true));
}
```

---

## 使用场景

### 场景一：集成测试替换上游服务

```java
@SpringBootTest
@AutoConfigureWireMock(port = 0)  // 随机端口
@Import(UpstreamServiceMockConfig.class)  // 加载 Contract Mock
class OrderServiceIntegrationTest {

    @Autowired
    private OrderService orderService;

    @Test
    void shouldCreateOrder() {
        // WireMock 已自动 Stub 所有 UserService API
        // orderService 内部调用 UserService 会命中 WireMock
        OrderDTO order = orderService.create(new CreateOrderRequest().setUserId(1L));
        assertThat(order).isNotNull();
    }
}
```

### 场景二：自定义特定接口响应

```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class ErrorScenarioStub implements WireMockStubCustomizer {

    @Override
    public void stub(WireMockServer server) {
        // 覆盖自动生成的 Stub，模拟 500 错误
        stubFor(get(urlPathEqualTo("/api/users/error-user"))
            .willReturn(serverError()
                .withBody("{\"code\": 500, \"message\": \"Internal Error\"}")));
    }
}
```

---

## 依赖

```gradle
implementation project(":it-common")
implementation project(":it-common-mock")
implementation libs.wiremock
optional 'org.springframework.cloud:spring-cloud-starter-openfeign'
optional "org.springframework:spring-web"
compileOnly "org.springframework.boot:spring-boot-starter-test"
```

---

## 注意事项

1. `it-common-spring-mock` 应仅作为 **测试作用域** 依赖引入（`testImplementation`），不应在生产代码中使用。
2. 端口配置：推荐使用 `port = 0`（随机端口）避免端口冲突，通过 `${wiremock.server.port}` 获取实际端口。
3. Contract Stub 的 Mock 数据基于 Proto Schema 自动生成，如需特定测试数据，请通过 `WireMockStubCustomizer` 覆盖。
