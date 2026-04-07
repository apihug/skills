---
name: apihug-enum-error-extension-guide
description: >
  Guide for defining business codes, localized messages, and error metadata
  for enum values using APIHug Protocol Buffers extensions.
version: 1.0.1
author: APIHug Team / H.O.P.E. Infra
created: 2026-03-01
updated: 2026-03-09
---

# Enum and Error Extension Guide

Define business codes, localized messages, and error metadata for enum values.

---

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 **NEVER** add `NA` for **Business Enum** – APIHug will generate `NA` as a placeholder.
- 🛑 **NEVER** add `UNDEFINED` for **Error Enum** – APIHug will generate `UNDEFINED` as a placeholder.


## 0. Two Types of Enums

| Type              | Purpose                         | Has `error{}` block? | File Location                    |
|-------------------|---------------------------------|----------------------|----------------------------------|
| **Business Enum** | Status codes, types, categories | No                   | `infra/{feature}/constant.proto` |
| **Error Enum**    | Error codes with HTTP status    | **Yes**              | `infra/{feature}/error.proto`    |


Both use `(hope.constant.field)` for metadata. The presence of `error{}` block distinguishes them.

### Error Code Range Convention

Each module's error code prefix is defined in `hope-wire.json` → `authority.codePrefix`. Error codes MUST be unique within the module and MUST NOT overlap with other modules.

```
Module A: codePrefix = 10240000 → errors: 10240001, 10240002, ...
Module B: codePrefix = 10250000 → errors: 10250001, 10250002, ...
```

> Business enum codes (non-error) follow a separate range and are typically small integers (1, 2, 3...).

---

## 1. Import and Basic Syntax

```proto
import "apihug/protobuf/swagger/annotations.proto";   // for (hope.swagger.enm)
import "apihug/protobuf/extend/constant.proto";       // for (hope.constant.field)
```

### Enum-Level Metadata: `(hope.swagger.enm)`

Add `title` and `description` to the enum itself for Swagger/OpenAPI documentation:

```proto
enum OrderStatusEnum {
  option (hope.swagger.enm) = {
    title: "Order Status";
    description: "Lifecycle states of an order";
  };

  PLACED = 0 [(hope.constant.field) = {
    code: 1,
    message: "Placed",
    message2: "已经下单"
  }];
}
```

---

## 2. Field Descriptions

| Field      | Type    | Required | Description                        |
|------------|---------|----------|------------------------------------|
| `code`     | int32   | Yes      | Business code (unique within enum) |
| `message`  | string  | Yes      | Primary language (English)         |
| `message2` | string  | No       | Secondary language (Chinese)       |
| `error`    | `Error` | No       | Extended error metadata            |


---

## 3. Error Code Extension

### 3.1 Syntax

```proto
error: {
  title: "<error_title>";           // required
  tips: "<user_guidance>";          // optional
  http_status: <HTTP_STATUS_ENUM>;  // required
  phase: <PHASE_ENUM>;              // required
  severity: <SEVERITY_ENUM>;        // required
}
```

### 3.2 Example

```proto
enum UserError {
  USER_NOT_FOUND = 0 [(hope.constant.field) = {
    code: 10001,
    message: "user not found",
    message2: "用户不存在",
    error: {
      title: "User Not Found",
      tips: "Check if the user ID is correct.",
      http_status: NOT_FOUND,
      phase: SERVICE,
      severity: ERROR
    }
  }];
}
```

### 3.3 Error Sub-Message Fields

| Field | Type | Values |
|-------|------|--------|
| `title` | string | Brief error title |
| `tips` | string | Suggestions for client/user |
| `http_status` | `HttpStatus` | `BAD_REQUEST`, `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `CONFLICT`, `INTERNAL_SERVER_ERROR` |
| `phase` | `Phase` | `CONTROLLER` (input validation), `SERVICE` (business logic), `DOMAIN` (data access) |
| `severity` | `Severity` | `LOW` (info), `WARN` (recoverable), `ERROR` (interrupted), `FATAL` (system failure) |

---

## 4. Type Usage

| Field | Type | ❌ Wrong | ✅ Correct |
|-------|------|----------|------------|
| `code` | int32 | - | `code: 10001` |
| `message` | string | - | `message: "user_not_found"` |
| `http_status` | enum | `http_status: 404` | `http_status: NOT_FOUND` |
| `phase` | enum | `phase: "SERVICE"` | `phase: SERVICE` |
| `severity` | enum | `severity: 2` | `severity: ERROR` |

> **Rule**: Always use enum name (without quotes) for enum fields.

---

## 5. Common Mistakes

### Mistake 1: Incorrect Enum Values

```proto
// ❌ WRONG
error: {
  http_status: 404,    // Must NOT use number
  phase: "SERVICE",    // Must NOT quote
  severity: 2          // Must NOT use numeric index
}

// ✅ CORRECT
error: {
  http_status: NOT_FOUND,
  phase: SERVICE,
  severity: ERROR
}
```

### Mistake 2: Duplicate `code` Values

```proto
// ❌ WRONG
enum UserError {
  ERROR_1 = 0 [(hope.constant.field) = {code: 10001}];
  ERROR_2 = 1 [(hope.constant.field) = {code: 10001}];  // Duplicate
}

// ✅ CORRECT
enum UserError {
  ERROR_1 = 0 [(hope.constant.field) = {code: 10001}];
  ERROR_2 = 1 [(hope.constant.field) = {code: 10002}];
}
```

### Mistake 3: Missing Required Fields

```proto
// ❌ WRONG
USER_NOT_FOUND = 0 [(hope.constant.field) = {
  message: "user not found"   // Missing `code`
}];

// ✅ CORRECT
USER_NOT_FOUND = 0 [(hope.constant.field) = {
  code: 10001,
  message: "user not found"
}];
```

---

## 6. Complete Example

```proto
syntax = "proto3";
package com.example.user.infra.user;

import "apihug/protobuf/extend/constant.proto";
import "apihug/protobuf/swagger/annotations.proto";

// Business Enum — no error{} block
enum Authority {
  option (hope.swagger.enm) = {
    title: "User Authority";
    description: "Permission codes for user management";
  };

  USER_CREATE = 0 [(hope.constant.field) = {
    code: 1001,
    message: "user:create",
    message2: "创建用户"
  }];
  USER_DELETE = 1 [(hope.constant.field) = {
    code: 1002,
    message: "user:delete",
    message2: "删除用户"
  }];
}

// Error Enum — HAS error{} block with http_status, phase, severity
enum UserError {
  USER_NOT_FOUND = 0 [(hope.constant.field) = {
    code: 10001,
    message: "user not found",
    message2: "用户不存在",
    error: {
      title: "User Not Found",
      http_status: NOT_FOUND,
      phase: SERVICE,
      severity: ERROR
    }
  }];
}
```
