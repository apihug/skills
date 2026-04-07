---
name: apihug-database-modeling-guide
description: >
  Expert guide for mapping Protocol Buffer messages to database table structures.
  Covers table/column annotations, SQL reserved words avoidance, naming conventions,
  constraints, indexes, and DDL generation.
version: 1.1.0
author: APIHug Team / H.O.P.E. Infra
created: 2026-03-01
updated: 2026-03-09
---


## Import

```proto
import "apihug/protobuf/domain/annotations.proto";
import "apihug/protobuf/domain/persistence.proto";
```

---

## ⚠️ CRITICAL: SQL Reserved Words Avoidance

### Never Use These Column/Table Names

| Category | Reserved Words | Conflict Resolution |
|----------|---------------|---------------------|
| **Keywords** | `user`, `order`, `group`, `index`, `key`, `value`, `number`, `size`, `type`, `status`, `level`, `mode`, `range`, `default`, `option`, `comment`, `access`, `action`, `admin`, `audit`, `backup`, `blob`, `cache`, `client`, `code`, `data`, `date`, `description`, `detail`, `domain`, `duration`, `email`, `event`, `file`, `flag`, `format`, `function`, `grant`, `host`, `image`, `input`, `instance`, `interface`, `item`, `job`, `kind`, `label`, `language`, `limit`, `link`, `list`, `location`, `lock`, `log`, `map`, `match`, `member`, `message`, `method`, `model`, `module`, `name`, `namespace`, `node`, `object`, `operation`, `output`, `owner`, `page`, `parameter`, `parent`, `partition`, `path`, `pattern`, `permission`, `plan`, `pool`, `port`, `position`, `prefix`, `privilege`, `procedure`, `process`, `profile`, `property`, `protocol`, `provider`, `public`, `query`, `queue`, `quota`, `rate`, `ratio`, `read`, `record`, `reference`, `region`, `relation`, `release`, `remote`, `repeat`, `report`, `request`, `resource`, `response`, `result`, `return`, `role`, `rule`, `schedule`, `schema`, `scope`, `script`, `section`, `segment`, `sequence`, `serial`, `server`, `service`, `session`, `setting`, `signal`, `site`, `sort`, `source`, `spec`, `stack`, `stage`, `standard`, `start`, `state`, `statement`, `statistics`, `step`, `stop`, `storage`, `stream`, `string`, `structure`, `style`, `subject`, `subscription`, `suffix`, `summary`, `switch`, `symbol`, `system`, `table`, `tag`, `target`, `task`, `template`, `term`, `test`, `text`, `thread`, `threshold`, `time`, `timeout`, `token`, `topic`, `trace`, `track`, `transaction`, `transform`, `trigger`, `tuple`, `unit`, `uri`, `url`, `usage`, `user`, `version`, `view`, `volume`, `warning`, `window`, `work`, `write`, `zone` | Add prefix: `user_name`, `order_no`, `group_code`, `idx_name`, `key_value`, `type_code` |
| **MySQL Specific** | `auto_increment`, `bigint`, `binary`, `bit`, `blob`, `bool`, `boolean`, `char`, `column`, `columns`, `create`, `database`, `databases`, `day`, `day_hour`, `day_minute`, `day_second`, `dec`, `decimal`, `delayed`, `delete`, `describe`, `distinct`, `distinctrow`, `div`, `double`, `drop`, `dual`, `else`, `enclosed`, `escaped`, `exists`, `exit`, `explain`, `float`, `float4`, `float8`, `force`, `foreign`, `from`, `fulltext`, `grant`, `group`, `having`, `high_priority`, `hour`, `hour_minute`, `hour_second`, `identified`, `if`, `ignore`, `in`, `index`, `infile`, `inner`, `innodb`, `insert`, `int`, `int1`, `int2`, `int3`, `int4`, `int8`, `integer`, `interval`, `into`, `is`, `iterate`, `join`, `key`, `keys`, `kill`, `leading`, `leave`, `left`, `like`, `limit`, `linear`, `lines`, `load`, `localtime`, `localtimestamp`, `lock`, `long`, `longblob`, `longtext`, `loop`, `low_priority`, `master_ssl_verify_server_cert`, `match`, `mediumblob`, `mediumint`, `mediumtext`, `middleint`, `minute`, `minute_second`, `mod`, `modifies`, `natural`, `not`, `no_write_to_binlog`, `null`, `numeric`, `on`, `optimize`, `option`, `optionally`, `or`, `order`, `out`, `outer`, `outfile`, `precision`, `primary`, `procedure`, `purge`, `raid0`, `range`, `read`, `reads`, `read_write`, `real`, `references`, `regexp`, `release`, `rename`, `repeat`, `replace`, `require`, `restrict`, `return`, `revoke`, `right`, `rlike`, `schema`, `schemas`, `second`, `select`, `separator`, `set`, `show`, `smallint`, `spatial`, `specific`, `sql`, `sqlexception`, `sqlstate`, `sqlwarning`, `sql_big_result`, `sql_calc_found_rows`, `sql_small_result`, `ssl`, `starting`, `straight_join`, `table`, `terminated`, `then`, `tinyblob`, `tinyint`, `tinytext`, `to`, `trailing`, `trigger`, `undo`, `union`, `unique`, `unlock`, `unsigned`, `update`, `usage`, `use`, `using`, `utc_date`, `utc_time`, `utc_timestamp`, `values`, `varbinary`, `varchar`, `varcharacter`, `varying`, `when`, `where`, `while`, `write`, `x509`, `xor`, `year_month`, `zerofill` | Use uppercase with prefix |
| **PostgreSQL Specific** | `all`, `analyse`, `analyze`, `and`, `any`, `array`, `as`, `asc`, `asymmetric`, `both`, `case`, `cast`, `check`, `collate`, `column`, `constraint`, `create`, `current_catalog`, `current_date`, `current_role`, `current_time`, `current_timestamp`, `current_user`, `default`, `deferrable`, `desc`, `distinct`, `do`, `else`, `end`, `except`, `false`, `fetch`, `for`, `foreign`, `from`, `grant`, `group`, `having`, `in`, `initially`, `intersect`, `into`, `leading`, `limit`, `localtime`, `localtimestamp`, `new`, `not`, `null`, `off`, `offset`, `old`, `on`, `only`, `or`, `order`, `placing`, `primary`, `references`, `returning`, `select`, `session_user`, `some`, `symmetric`, `table`, `then`, `to`, `trailing`, `true`, `union`, `unique`, `user`, `using`, `variadic`, `when`, `where`, `with` | Use uppercase with prefix |

### Naming Conflict Resolution Pattern

```proto
// ❌ WRONG — conflicts with reserved words
message User {
  string user = 1;        // 'user' is reserved
  string name = 2;        // 'name' is reserved
  string order = 3;       // 'order' is reserved
  string group = 4;       // 'group' is reserved
  string status = 5;      // 'status' is reserved
  string type = 6;        // 'type' is reserved
}

// ✅ CORRECT — prefixed to avoid conflicts
message UserEntity {
  option (hope.persistence.table) = {
    name: "SYS_USER";     // Table: prefix + uppercase
    wires: [IDENTIFIABLE, AUDITABLE, DELETABLE];
  };

  string user_name = 1 [(hope.persistence.column) = { name: "USER_NAME"; }];     // Column: prefix
  string full_name = 2 [(hope.persistence.column) = { name: "FULL_NAME"; }];     // Alternative name
  string order_no = 3 [(hope.persistence.column) = { name: "ORDER_NO"; }];       // Add suffix
  string group_code = 4 [(hope.persistence.column) = { name: "GROUP_CODE"; }];   // Add suffix
  string status_code = 5 [(hope.persistence.column) = { name: "STATUS_CODE"; }]; // Add suffix
  string type_code = 6 [(hope.persistence.column) = { name: "TYPE_CODE"; }];     // Add suffix
}
```

---

## 1. Import and Basic Syntax

```proto
import "apihug/protobuf/domain/annotations.proto";

message EntityName {
  option (hope.persistence.table) = {
    name: "TABLE_NAME";
    description: "Table description";
    wires: [IDENTIFIABLE];
  };

  string field_name = 1 [(hope.persistence.column) = {
    name: "COLUMN_NAME";
    nullable: false;
    type: VARCHAR;
    length: 32;
  }];
}
```

---

## 2. Table Extension

### 2.1 Basic Fields

| Field | Type | Purpose |
|-------|------|---------|
| `name` | string | Table name (uppercase + prefix: `SYS_USER`, `BIZ_ORDER`) |
| `description` | string | Business description |
| `catalog` | string | Database catalog (multi-db) |
| `schema` | string | Database schema (Oracle/PostgreSQL) |

### 2.2 Constraints and Indexes

```proto
unique_constraints: {
  name: "UK_USER_EMAIL";
  column_list: ["email"];
}

indexes: {
  name: "IDX_USER_TENANT";
  column_list: ["tenant_id"];
}
```

### 2.3 Built-in Features (`wires`)

| Wire Value     | Auto-Added Columns                                                                    |
|----------------|---------------------------------------------------------------------------------------|
| `ALL`          | All columns below (entity extends `Domain` base class)                                |
| `IDENTIFIABLE` | `ID` (BIGINT)                                                                         |
| `AUDITABLE`    | `CREATED_AT` (TIMESTAMP), `CREATED_BY` (BIGINT/VARCHAR), `UPDATED_AT` (TIMESTAMP), `UPDATED_BY` (BIGINT/VARCHAR) |
| `DELETABLE`    | `DELETED` (BOOLEAN), `DELETED_AT` (TIMESTAMP), `DELETED_BY` (BIGINT/VARCHAR)          |
| `TENANTABLE`   | `TENANT_ID` (BIGINT)                                                                  |
| `VERSIONABLE`  | `VERSION` (BIGINT)                                                                    |

> **Implementation mapping**: Each wire value maps to a Java interface in `hope.common.spring.data.persistence.wire.*`. `ALL` maps to the `Domain` base class which implements all interfaces. See [impl-golden-rule §Entity Wire Interfaces](apihug-impl-golden-rule.md#-entity-wire-interfaces--proto-wire-to-java-mapping) for method details.

```proto
wires: [IDENTIFIABLE, AUDITABLE, DELETABLE]
```

### 2.4 Liquibase Integration

```proto
liquibase: {
  version: 1;
  comment: "MySQL fulltext index";
  dbms: [MYSQL];
  sql: ["CREATE FULLTEXT INDEX IDX_USER_NAME_FT ON SYS_USER(USER_NAME)"];
}
```

---

## 3. Column Extension

### 3.1 Basic Fields

| Field | Type | Purpose |
|-------|------|---------|
| `name` | string | Column name (uppercase + prefix: `USER_NAME`, `ORDER_NO`) |
| `description` | string | Business description |

### 3.2 Constraint Fields

| Field | Type | Purpose |
|-------|------|---------|
| `nullable` | bool | `false` = NOT NULL |
| `unique` | bool | Unique constraint |
| `insertable` | bool | Participate in INSERT |
| `updatable` | bool | Participate in UPDATE |
| `searchable` | bool | Include in filter query conditions |
| `sortable` | bool | Allow this column in `ORDER BY` clauses |

### 3.3 Type and Length

| Field | Type | Purpose |
|-------|------|---------|
| `type` | enum | SQL type |
| `length` | uint32 | String length (VARCHAR/CHAR) |
| `precision` | uint32 | Total digits (DECIMAL) |
| `scale` | uint32 | Decimal places (DECIMAL) |

**Common `type` values**:

| Enum | SQL Type | Use Case |
|------|----------|----------|
| `VARCHAR` | VARCHAR | Strings |
| `CHAR` | CHAR | Fixed-length codes |
| `INTEGER` | INTEGER | Integers |
| `BIGINT` | BIGINT | Large integers, IDs |
| `DOUBLE` | DOUBLE | Floating point |
| `DECIMAL` | DECIMAL | Monetary, exact decimals |
| `DATE` | DATE | Date |
| `TIMESTAMP` | TIMESTAMP | Timestamp |
| `BOOLEAN` | BOOLEAN | Boolean |
| `BLOB` | BLOB | Binary |
| `CLOB` | CLOB | Long text |

### 3.4 Enum Mapping

| `enum_type` | Persisted | Example |
|-------------|-----------|---------|
| `STRING` | Enum name | `"AVAILABLE"` |
| `CODE` | Business code | `1`, `2`, `3` |

> **Prohibited**: `ORDINAL` — deprecated, error-prone.

```proto
StatusEnum status_code = 1 [(hope.persistence.column) = {
  name: "STATUS_CODE";
  enum_type: STRING;
  type: VARCHAR;
}];
```

### 3.5 Transient Fields

Fields that should NOT be persisted to the database:

```proto
string display_label = 10 [(hope.persistence.column) = {
  transient: true;   // Not mapped to any database column
}];
```

> Use for computed fields, temporary values, or fields only needed in the wire layer.

### 3.6 Default Values

```proto
int32 retry_count = 11 [(hope.persistence.column) = {
  name: "RETRY_COUNT";
  default_value: "0";       // Always string type, framework converts
}];

string status_code = 12 [(hope.persistence.column) = {
  name: "STATUS_CODE";
  default_value: "PENDING";  // Enum default as string
}];
```

> `default_value` is always a `string` regardless of the column type. The framework handles type conversion.

### 3.7 Primary Key

```proto
int64 id = 1 [(hope.persistence.column) = {
  name: "ID";
  id: true;
  nullable: false;
  generated_value: { strategy: UUID }  // TABLE/SEQUENCE/IDENTITY/UUID/AUTO
}];
```

---

## 4. Type Distinctions

### 4.1 bool vs string

```proto
// ❌ WRONG
nullable: "false";

// ✅ CORRECT
nullable: false;
```

### 4.2 Proto type vs SQL type

| Proto | Default SQL | Explicit `type` |
|-------|-------------|-----------------|
| `string` | VARCHAR | VARCHAR/CHAR/CLOB |
| `int32` | INTEGER | INTEGER/SMALLINT |
| `int64` | BIGINT | BIGINT |
| `double` | DOUBLE | DOUBLE/DECIMAL |
| `bool` | BOOLEAN | BOOLEAN/BIT |
| `bytes` | BLOB | BLOB/VARBINARY |

---

## 5. Complete Example

```proto
syntax = "proto3";
package com.example.pet.domain;

import "apihug/protobuf/domain/annotations.proto";
import "com/example/pet/enumeration/constants.proto";

message PetEntity {
  option (hope.persistence.table) = {
    name: "BIZ_PET";  // Prefix: BIZ_ for business tables
    description: "Pet information table";
    unique_constraints: {
      name: "UK_PET_NAME_CATEGORY";
      column_list: ["pet_name", "category_code"];
    };
    indexes: { name: "IDX_PET_TENANT"; column_list: ["tenant_id"]; };
    wires: [IDENTIFIABLE, AUDITABLE, DELETABLE, TENANTABLE];
  };

  string pet_name = 1 [(hope.persistence.column) = {
    name: "PET_NAME";      // Prefix to avoid 'name' reserved word
    nullable: false;
    length: 64;
    type: VARCHAR;
  }];

  string category_code = 2 [(hope.persistence.column) = {
    name: "CATEGORY_CODE"; // Suffix to avoid 'category' reserved word
    nullable: false;
    length: 32;
    type: VARCHAR;
  }];

  com.example.pet.enumeration.SizeEnum size_code = 3 [(hope.persistence.column) = {
    name: "SIZE_CODE";     // Suffix to avoid 'size' reserved word
    enum_type: STRING;
    type: VARCHAR;
    length: 16;
  }];

  double weight_value = 4 [(hope.persistence.column) = {
    name: "WEIGHT_VALUE";  // Suffix to avoid 'value' reserved word
    type: DOUBLE;
  }];
}
```

---

## 6. Common Mistakes

### 6.1 Type Confusion

```proto
// ❌ WRONG
nullable: "false";

// ✅ CORRECT
nullable: false;
```

### 6.2 DECIMAL Missing Precision

```proto
// ❌ WRONG
type: DECIMAL;

// ✅ CORRECT
type: DECIMAL;
precision: 10;
scale: 2;
```

---

## 7. Best Practices

### Naming Conventions

| Element | Pattern | Example |
|---------|---------|---------|
| **Table** | `{PREFIX}_{ENTITY}` | `SYS_USER`, `BIZ_ORDER`, `CFG_SETTING` |
| **Column** | `{ENTITY}_{FIELD}` or `{FIELD}_{SUFFIX}` | `USER_NAME`, `ORDER_NO`, `STATUS_CODE` |
| **Index** | `IDX_{TABLE}_{COLUMN}` | `IDX_USER_EMAIL`, `IDX_ORDER_TENANT` |
| **Unique** | `UK_{TABLE}_{COLUMN}` | `UK_USER_EMAIL`, `UK_ORDER_NO` |

### Table Prefixes by Domain

| Prefix | Domain | Example |
|--------|--------|---------|
| `SYS_` | System/Platform | `SYS_USER`, `SYS_ROLE`, `SYS_PERMISSION` |
| `BIZ_` | Business | `BIZ_ORDER`, `BIZ_PRODUCT`, `BIZ_CUSTOMER` |
| `CFG_` | Configuration | `CFG_SETTING`, `CFG_PARAMETER` |
| `LOG_` | Logging/Audit | `LOG_OPERATION`, `LOG_ACCESS` |
| `TMP_` | Temporary | `TMP_IMPORT`, `TMP_EXPORT` |

### Wire Selection

```
Primary key only    → wires: [IDENTIFIABLE]
Audit needed        → wires: [IDENTIFIABLE, AUDITABLE]
Soft delete         → wires: [IDENTIFIABLE, DELETABLE]
Multi-tenancy       → wires: [IDENTIFIABLE, TENANTABLE]
Full features       → wires: [ALL]
No wire columns     → wires: [NONE]   // Entity manages all columns manually
```

> `NONE` means the entity will NOT extend any wire interface. Use for legacy tables or tables with non-standard column layouts.

### Type Selection

```
Short strings       → VARCHAR + length
Long text           → CLOB
Primary key         → BIGINT or UUID
Monetary            → DECIMAL(10,2)
Date/Timestamp      → DATE / TIMESTAMP
Enum                → VARCHAR + enum_type:STRING
```
