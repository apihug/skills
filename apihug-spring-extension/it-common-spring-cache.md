# it-common-spring-cache

`it-common-spring-cache` 是 ApiHug 体系的**统一缓存抽象层**，在 Spring Cache 基础上提供多后端（Caffeine / Redis / EhCache）的声明式配置、动态 TTL、缓存名称前缀键生成及统一错误策略，避免各微服务重复实现缓存配置样板代码。

---

## 设计原则

- **配置驱动**：通过 `hope.cache.*` 属性统一管理所有缓存行为，开发者只需在 `@Cacheable` 中声明缓存名称，无需编写 `CacheManager` Bean。
- **分层 TTL**：全局默认 TTL（`hope.cache.time-to-live`）→ 后端全局 TTL（`hope.cache.caffeine.time-to-live`）→ 具名缓存 TTL（`hope.cache.caffeine.caches.{name}.expire-after-write`），细粒度覆盖。
- **缓存名内嵌 TTL**：支持 `@Cacheable("myCache#300s")` 语法，在缓存名中直接指定过期时间，无需配置文件修改。
- **安全失败**：`HopeCacheErrorHandler` 实现 Spring Cache `CacheErrorHandler`，可配置吞掉 get/put/evict/clear 异常，防止缓存故障影响主业务。
- **按需启用**：每种后端通过注解激活（`@EnableCaffeineCaching` / `@EnableRedisCaching` / `@EnableEhcacheCaching`），不引入多余依赖。

---

## 核心组件

### `HopeCacheProperties`

配置前缀：`hope.cache`

**顶层属性：**

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `hope.cache.time-to-live` | `30s` | 全局默认 TTL（秒为单位） |
| `hope.cache.allow-null-values` | `true` | 是否允许缓存 null 值 |
| `hope.cache.allow-in-flight-cache-creation` | `true` | 是否允许动态创建未声明的缓存 |

**Caffeine 配置（`hope.cache.caffeine.*`）：**

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `time-to-live` | `30s` | Caffeine 全局 TTL |
| `maximum-size` | `-1` | 缓存最大条目数 |
| `initial-capacity` | `-1` | 初始容量 |
| `expire-after-write` | - | 写入后过期时间 |
| `expire-after-access` | - | 访问后过期时间 |
| `refresh-after-write` | - | 写入后刷新时间 |
| `record-stats` | `false` | 是否记录命中率统计 |
| `allow-in-flight-cache-creation` | `true` | 是否允许动态创建缓存 |
| `caches.{name}.*` | - | 具名缓存独立配置（覆盖全局） |

**Redis 配置（`hope.cache.redis.*`）：**

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `time-to-live` | `30s` | Redis 全局 TTL |
| `serializer` | - | 序列化方式（`JAVA`/`JSON`） |
| `key-prefix` | - | 缓存 Key 前缀 |
| `use-key-prefix` | `true` | 是否使用 Key 前缀 |
| `cache-null-values` | `true` | 是否缓存 null |
| `enable-statistics` | `false` | 是否统计 |
| `allow-in-flight-cache-creation` | `true` | 是否允许动态创建缓存 |
| `caches.{name}.*` | - | 具名缓存独立配置 |

**EhCache 配置（`hope.cache.eh-cache.*`）：**

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `duration` | `5m` | 全局过期时间 |
| `heap.size` / `heap.unit` | `1000` / `ENTRIES` | 堆内缓存大小 |
| `off-heap.size` / `off-heap.unit` | `10` / `MB` | 堆外缓存大小 |
| `disk.size` / `disk.unit` | `10` / `MB` | 磁盘缓存大小 |
| `caches.{name}.*` | - | 具名缓存独立配置 |

**错误策略（`hope.cache.error-policy.*`）：**

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `swallow-get-exception` | `true` | 吞掉 get 异常 |
| `swallow-put-exception` | `true` | 吞掉 put 异常 |
| `swallow-evict-exception` | `true` | 吞掉 evict 异常 |
| `swallow-clear-exception` | `true` | 吞掉 clear 异常 |
| `log-stack-traces` | `true` | 是否打印异常堆栈 |

---

### `CaffeineAutoCacheManager`

扩展 Spring `CaffeineCacheManager`，支持：

- **具名配置合并**：先应用全局配置，再用具名配置覆盖
- **缓存名 TTL 语法**：`@Cacheable("userCache#5m")` 中 `#5m` 解析为 5 分钟过期
- **安全创建**：`allowInFlightCacheCreation=false` 时若无配置则抛出 `HopeErrorDetailException`

### `RedisAutoCacheManager`

基于 Spring `RedisCacheManager`，支持：

- JSON/Java 序列化切换
- 每个缓存独立 TTL、Key 前缀
- 事务支持

### `PrefixedKeyGenerator`

实现 Spring `KeyGenerator`，缓存 Key 格式：

```
{className}#{methodName}#{args...}
```

避免不同方法的相同参数产生 Key 冲突。

---

## 激活方式

### Caffeine（本地内存缓存）

```java
@SpringBootApplication
@EnableCaffeineCaching
public class MyApplication { ... }
```

```yaml
hope:
  cache:
    caffeine:
      time-to-live: 60s
      maximum-size: 5000
      caches:
        userCache:
          maximum-size: 1000
          expire-after-write: 10m
```

### Redis（分布式缓存）

```java
@SpringBootApplication
@EnableRedisCaching
public class MyApplication { ... }
```

```yaml
hope:
  cache:
    redis:
      serializer: JSON
      caches:
        orderCache:
          time-to-live: 30m
          key-prefix: "order:"
```

### EhCache（多层缓存）

```java
@SpringBootApplication
@EnableEhcacheCaching
public class MyApplication { ... }
```

```yaml
hope:
  cache:
    eh-cache:
      heap:
        size: 2000
        unit: ENTRIES
      off-heap:
        size: 100
        unit: MB
```

---

## 使用缓存

启用缓存后，直接使用标准 Spring Cache 注解：

```java
@Service
public class UserService {

    @Cacheable(value = "userCache", key = "#id")
    public UserDTO getUser(Long id) { ... }

    @CacheEvict(value = "userCache", key = "#id")
    public void deleteUser(Long id) { ... }

    // 缓存名内嵌 TTL（仅 Caffeine 支持）
    @Cacheable("userCache#5m")
    public UserDTO getUserShortLived(Long id) { ... }
}
```

---

## 依赖

```gradle
implementation project(":it-common-proto")
optional "com.github.ben-manes.caffeine:caffeine"
optional libs.redisson
optional libs.ehcache
optional "org.springframework.boot:spring-boot-starter-cache"
optional 'org.springframework.boot:spring-boot-starter-data-redis'
```
