# it-common-spring-data

`it-common-spring-data` 是 ApiHug 体系的**数据持久化基础层**，提供多租户、可审计、软删除的标准实体基类，并在 Spring Data JDBC 和 MyBatis 基础上提供增强封装，统一多数据库方言的类型映射与分页查询。

---

## 设计原则

- **标准化实体契约**：通过 `Domain<T, Identify, Tenant>` 基类提供完整的审计（创建/更新人、时间）、软删除、乐观锁版本、多租户字段，与 Spring Data 审计注解深度集成。
- **多数据库方言透明**：`PersistenceContext` SPI 定义身份 ID/租户类型（Long/Integer/String），`ColumnTypeMapper` 根据运行时数据库方言自动映射 Java 类型与 SQL 类型，无需手动处理方言差异。
- **MyBatis 列表类型支持**：为所有基本 Java 类型的 `List<T>` 提供 MyBatis `TypeHandler`，数据库以 JSON/序列化字符串存储，透明反序列化。
- **安全防护**：`HopeJdbcRepository.deleteAll()` 强制抛出 `UnsupportedOperationException`，避免误操作清空整表。
- **DSL 链式查询**：`DSL` 工具类提供类型安全的查询条件构建，与 Spring Data Criteria 无缝衔接。

---

## 核心组件

### 实体基类与接口契约

**`Domain<T, Identify, Tenant>`**

所有业务实体的基类，内置标准字段：

| 字段 | 列名 | 说明 |
|------|------|------|
| `id` | `ID` | 自增主键 |
| `createdAt` | `CREATED_AT` | 创建时间（`@InsertOnlyProperty`） |
| `createdBy` | `CREATED_BY` | 创建人 ID（`@InsertOnlyProperty`） |
| `updatedAt` | `UPDATED_AT` | 更新时间 |
| `updatedBy` | `UPDATED_BY` | 更新人 ID |
| `version` | `VERSION` | 乐观锁版本（`@Version`） |
| `deleted` | `DELETED` | 软删除标志，默认 `false` |
| `deletedAt` | `DELETED_AT` | 删除时间 |
| `deletedBy` | `DELETED_BY` | 删除操作人 |
| `tenantId` | `TENANT_ID` | 租户 ID（`@InsertOnlyProperty`） |

**Wire 接口（`wire` 包）：**

| 接口 | 说明 |
|------|------|
| `Entity<T>` | 基础实体接口 |
| `Identifiable<T>` | 提供 `Long id` |
| `Auditable<T, Identify>` | 提供创建/更新人+时间 |
| `Deletable<T, Identify>` | 提供软删除相关字段 |
| `Versionable<T>` | 提供 `Long version` 乐观锁 |
| `Tenantable<T, Tenant>` | 提供 `tenantId` |

**使用示例：**

```java
@Table("T_USER")
public class UserDomain extends Domain<UserDomain, Long, Long> {
    public String username;
    public String email;
}
```

---

### `PersistenceContext<Identify, Tenant>`

持久化上下文 SPI，每个微服务实现一个，提供默认用户 ID、默认租户 ID、命名策略等运行时配置：

```java
@Component
public class UserPersistenceContext implements PersistenceContext<Long, Long> {

    @Override
    public Long defaultUserId() { return 0L; }

    @Override
    public Long defaultTenant() { return 1L; }

    @Override
    public String domain() { return "user"; }

    @Override
    public NamingStrategyOption namingStrategy() {
        return NamingStrategyOption.CAMEL_2_UNDER_SCORE_UPPER;  // 驼峰转大写下划线
    }
}
```

`PersistenceContextManager` 管理多个 `PersistenceContext` 实例（多数据源场景）。

---

### `HopeJdbcRepository<T, ID>`

扩展 Spring Data JDBC `SimpleJdbcRepository`，增强能力：

```java
// 按条件统计
long count = userRepo.count(Criteria.where("deleted").is(false));

// 按条件查询
Optional<User> user = userRepo.findOne(Criteria.where("id").is(userId));

// 分页查询
Page<User> page = userRepo.findAll(
    Criteria.where("tenantId").is(tenantId),
    PageRequest.of(0, 20)
);

// 排序查询
List<User> users = userRepo.findAll(
    Criteria.where("deleted").is(false),
    DSL.SortColumn.of(UserTable.CREATED_AT, Direction.DESC)
);

// 投影查询
Page<UserDTO> dtos = userRepo.findAll(criteria, UserDTO.class, pageable);

// deleteAll() 会抛出 UnsupportedOperationException（安全保护）
```

---

### `EasyCriteria`

链式条件构建工具，简化复杂查询：

```java
Criteria criteria = EasyCriteria.builder()
    .eq("tenantId", tenantId)
    .eq("deleted", false)
    .like("username", keyword)
    .build();
```

---

### MyBatis 类型处理器（`mybatis` 包）

为以下类型提供 `TypeHandler`，数据库字段以 JSON 数组存储：

`StringList`、`IntegerList`、`LongList`、`BigDecimalList`、`BooleanList`、`DoubleList`、`FloatList`、`ByteList`、`ShortList`、`DateList`、`DateTimeList`、`TimeList`

**使用示例（MyBatis Mapper）：**

```xml
<result column="tags" property="tags"
        typeHandler="hope.common.spring.data.persistence.mybatis.StringListTypeHandler"/>
```

---

### Liquibase 异步支持

`AsyncSpringLiquibase` 将数据库变更迁移移入线程池异步执行，不阻塞应用启动：

```java
@Bean
public SpringLiquibase liquibase(DataSource dataSource, TaskExecutor taskExecutor) {
    AsyncSpringLiquibase liquibase = new AsyncSpringLiquibase(taskExecutor, environment);
    liquibase.setDataSource(dataSource);
    liquibase.setChangeLog("classpath:db/changelog/master.xml");
    return liquibase;
}
```

---

### `HopeListConverters`

为 Spring Data JDBC 注册 `List<T>` 到 `String`（JSON）的双向 Converter，支持字段级 JSON 列表序列化。

---

## 配置属性

配置前缀：`hope.data`（`HopeDataProperties`）

| 属性 | 说明 |
|------|------|
| `hope.data.naming-strategy` | 全局命名策略（`CAMEL_2_UNDER_SCORE_UPPER` 等） |

---

## 依赖

```gradle
implementation project(":it-common")
implementation project(":it-common-proto")
implementation project(":it-common-spring-plus:it-common-spring")
optional 'org.liquibase:liquibase-core'
optional 'org.springframework.data:spring-data-jdbc'
api(libs.mybatis)
api(libs.mybatisDynamicSql)
implementation libs.jakartaPersistence
optional "com.fasterxml.jackson.core:jackson-databind"
optional 'org.springframework.boot:spring-boot-autoconfigure'
```
