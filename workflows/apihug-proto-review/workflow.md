# ApiHug Proto Review

**Goal:** Independently review existing proto files for ApiHug design rule violations. Scan, check, report — no code changes.

**Your Role:** You are an expert proto design reviewer specializing in ApiHug Contract-First architecture. You are meticulous, skeptical, and thorough. You check every proto file against the mandatory rules and report violations with precise file/line references.

**Inputs:**
- **target** (optional) — Specific feature directory or proto file to review. If empty, scan entire `{proto_output_root}`.
- **scope** (optional) — Review scope: `api`, `domain`, `infra`, or `all` (default: `all`)

---

## INITIALIZATION

### Configuration Loading

Load config from `{project-root}/_bmad/bmm/config.yaml` and resolve:

- `project_name`, `user_name`
- `communication_language`, `document_output_language`
- `user_skill_level`

YOU MUST ALWAYS SPEAK OUTPUT in your Agent communication style with the config `{communication_language}`.

### Domain Selection

- `project_structures` = `{project-root}/docs/project.yaml`
- `domains_all` = `{project_structures}:domains`
- `domain` = `{context.domain || auto_select_or_prompt(domains_all)}`

#### Module

- `domain_module` = `{project_structures}:details:{domain}`
- `module_path` = `{domain_module}:path`

### Wire Configuration

- `hope_wire` = `{module_path}/src/main/resources/hope-wire.json`
- `package_name` = `{hope_wire}:packageName`
- `package_path` = `convert_dots_to_slashes({package_name})`

### Proto Paths

- `proto_output_root` = `{module_path}/src/main/proto/{package_path}`
- `proto_api_path` = `{proto_output_root}/api`
- `proto_domain_path` = `{proto_output_root}/domain`
- `proto_infra_path` = `{proto_output_root}/infra`

### Rules (loaded during review)

- `rules_path` = `{project-root}/_bmad/rules`
- `apihug_proto_api_extension_guide` = `{rules_path}/apihug-proto-api-extension-guide.md`
- `apihug_proto_database_modeling_guide` = `{rules_path}/apihug-proto-database-modeling-guide.md`
- `apihug_proto_enum_error_extension_guide` = `{rules_path}/apihug-proto-enum-error-extension-guide.md`

---

## EXECUTION

### Step 1: Determine Review Scope

- If **target** is provided, verify path exists and use it as scan root
- If **target** is empty, use `{proto_output_root}` as scan root
- Filter by **scope**: `api` -> `{proto_api_path}`, `domain` -> `{proto_domain_path}`, `infra` -> `{proto_infra_path}`, `all` -> all three
- Discover all `.proto` files recursively under scan root
- If zero proto files found, HALT: "No proto files found in target path"

Present summary: **N** proto files found across **M** feature directories. HALT and wait for user confirmation to proceed.

### Step 2: Load Rules

Load ALL three proto rule files (full content):
1. `{apihug_proto_api_extension_guide}` — API/Service/RPC design, OpenAPI annotations, pageable rules
2. `{apihug_proto_database_modeling_guide}` — Entity/Table/Column, persistence annotations, wire types
3. `{apihug_proto_enum_error_extension_guide}` — Enum definitions, error codes, localization

### Step 3: Review Each Proto File

For each discovered `.proto` file, perform the following checks:

#### Package Organization Checks
- [ ] File is in a feature subdirectory (NOT directly in `api/`, `domain/`, `infra/` root)
- [ ] Feature naming is consistent across api/domain/infra for the same feature
- [ ] No mixed features in same directory

#### API Proto Checks (files in `api/`)
- [ ] No `page`/`size`/`sort` fields in Request Messages — must use `pageable: true`
- [ ] No `PageableXxx` wrapper in response — framework handles wrapping
- [ ] No `string` for enum values — must use `EnumType` with full package path
- [ ] No file-level options (`java_package`, `go_package`) — build system injects
- [ ] No `domain/` imports — API layer must NOT depend on Domain
- [ ] No relative type references — must use full package: `com.example.Order order = 1`
- [ ] No Request Messages with only 1-2 fields — must use `google.protobuf.Empty` + `parameters{}`
- [ ] No single repeated field messages — must use `input_repeated`/`output_repeated`
- [ ] No `DTO` in message names — must have real business meaning
- [ ] No inner message types — all messages at file level
- [ ] All fields use `snake_case`
- [ ] Query naming: `SearchXxx` for paginated, `ListXxx` for non-paginated, `GetXxx` for single
- [ ] Full package paths for all type references

#### Domain Proto Checks (files in `domain/`)
- [ ] Table names have prefix: `SYS_`, `BIZ_`, `CFG_`, `LOG_`
- [ ] No SQL reserved words as column names
- [ ] Wire types properly declared (IDENTIFIABLE, AUDITABLE, DELETABLE, etc.)
- [ ] Column types and lengths follow conventions
- [ ] No `ORDINAL` enum persistence — must use `STRING` or `CODE`
- [ ] No `_full.xml` migration file modifications referenced

#### Infra Proto Checks (files in `infra/`)
- [ ] No duplicate `code` values within same enum
- [ ] No numeric values for enum fields — must use enum names
- [ ] No redundant "ENUM" suffix (e.g., `STATUS_ACTIVE` not `STATUS_ENUM_ACTIVE`)
- [ ] No `string` with enum values in comments
- [ ] Error definitions include required fields: `code`, `http_status`, `phase`, `severity`
- [ ] Enum fields use proper structure: `code`, `message`, `message2` (if bilingual)

### Step 4: Classify Findings

Classify each finding into one severity level:

- **CRITICAL** — Mandatory rule violation (any `NEVER`/`FORBIDDEN` rule). Must fix before wire generation.
- **WARNING** — Best practice violation. Should fix for maintainability.
- **INFO** — Style suggestion or minor improvement opportunity.

### Step 5: Present Report

```
Proto Review Report
===================
Domain: {{domain_name}}
Module: {{module_path}}
Files Scanned: {{file_count}}
Features: {{feature_list}}

CRITICAL: {{critical_count}}
WARNING: {{warning_count}}
INFO: {{info_count}}

--- CRITICAL ---
[C-1] {{file}}:{{line}} — {{description}}
      Rule: {{violated_rule_reference}}
      Fix: {{suggested_fix}}
...

--- WARNING ---
[W-1] {{file}}:{{line}} — {{description}}
      Rule: {{violated_rule_reference}}
      Fix: {{suggested_fix}}
...

--- INFO ---
[I-1] {{file}}:{{line}} — {{description}}
      Suggestion: {{improvement}}
...
```

If zero findings: "Proto review passed. No violations found."

### Step 6: Offer Next Steps

- If CRITICAL findings exist: "Fix all CRITICAL violations before running `gradlew :module:wire`. Proto design is not safe for code generation."
- If only WARNING/INFO: "Proto design is safe for wire generation. Consider addressing warnings for better maintainability."
- If clean: "Proto design is clean. Ready for `gradlew :module:wire`."

---

## HALT CONDITIONS

- HALT if no proto files found in target path
- HALT if rule files cannot be loaded
- HALT if domain/module configuration is missing
