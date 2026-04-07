# it-common-spring-integration

`it-common-spring-integration` 为 ApiHug 体系提供**消息集成（AMQP/RabbitMQ）**支持，基于 Spring Integration 和 Spring AMQP，提供统一的消息发布、消费及事件驱动架构基础设施。

> **当前状态**：模块骨架已建立，核心消息集成能力正在持续完善中。

---

## 设计原则

- **松耦合消息传递**：通过 Spring Integration 通道（Channel）将业务逻辑与消息基础设施解耦，业务代码无需直接依赖 RabbitMQ 客户端。
- **Hope 上下文透传**：消息发布时，自动将当前 `Customer`（用户身份）、`tenantId`、`requestId` 等 Hope 上下文信息序列化到消息头，消费端自动还原上下文。
- **统一错误处理**：复用 `it-common-spring-core` 的错误处理体系，消息消费异常遵循统一的 `Result` 响应格式。

---

## 技术栈

| 组件 | 说明 |
|------|------|
| `spring-boot-starter-amqp` | RabbitMQ AMQP 支持 |
| `spring-boot-starter-integration` | Spring Integration 核心 |
| `spring-integration-amqp` | Spring Integration AMQP 适配器 |
| `spring-integration-file` | Spring Integration 文件适配器 |

---

## 依赖

```gradle
implementation project(":it-common")
implementation project(":it-common-proto")
implementation project(":it-common-spring-plus:it-common-spring")
optional 'org.springframework.boot:spring-boot-starter-amqp'
optional 'org.springframework.boot:spring-boot-starter-integration'
optional 'org.springframework.integration:spring-integration-amqp'
optional "org.springframework.integration:spring-integration-file"
```

---

## 路线图

- [ ] AMQP 消息发布器基类（携带 Hope 上下文头）
- [ ] AMQP 消息消费器基类（还原 Hope 上下文）
- [ ] 死信队列（DLQ）统一处理策略
- [ ] 消息幂等性支持（基于消息 ID 去重）
- [ ] Spring Cloud Stream 适配（RabbitMQ Binder）
