---
title: 'ApiHug Unified Dev Story Definition of Done Checklist'
validation-target: 'Story markdown ({{story_path}})'
validation-criticality: 'HIGHEST'
required-inputs:
  - 'Story markdown file with Dev Notes'
  - 'Completed Tasks/Subtasks section'
  - 'Updated File List section'
  - 'Updated Dev Agent Record with phase markers'
validation-rules:
  - 'Only permitted story sections modified'
  - 'Phase markers must be present: Backend Phase Completed (required), Frontend Phase Completed (if enabled)'
  - 'ApiHug rules followed: proto guides, impl golden rule, frontend guide'
---

# ApiHug Unified Dev Story Definition of Done Checklist

## Phase Detection

- [ ] **Phase Detection Complete:** Workflow correctly identified current phase (backend/frontend)
- [ ] **Backend Phase Completed Marker:** Present if backend done
- [ ] **Backend Phase APPROVED Marker:** Present if code-review passed (optional but recommended)
- [ ] **Frontend Phase Completed Marker:** Present if frontend done

---

## PHASE 1: Backend Validation

### Context & Requirements
- [ ] **Story Context Loaded:** Dev Notes contains backend requirements
- [ ] **Domain Configuration:** hope-wire.json loaded, package/module identified
- [ ] **Architecture Compliance:** Onion Architecture followed (Adapter -> Domain -> Infrastructure)

### ApiHug Contract-First (Proto Phase)
- [ ] **Feature Package Organization:**
  - [ ] **Feature Name Extracted:** {{feature_name}} identified from story (or {{feature_names}} for multi-feature)
  - [ ] **API Package:** `{proto_api_path}/{{feature_name}}/{{feature_name}}_service.proto` + `_dto.proto`
  - [ ] **Domain Package:** `{proto_domain_path}/{{feature_name}}/{{feature_name}}_entity.proto`
  - [ ] **Infra Package:** `{proto_infra_path}/{{feature_name}}/{{feature_name}}_constant.proto` + `_error.proto`
  - [ ] **NO Root Level Files:** No proto files placed directly in `api/`, `domain/`, `infra/` root
- [ ] **Core Constraints Verified (inline critical rules):**
  - [ ] **NEVER:** `page`/`size`/`sort` in Request Message -> use `pageable: true`
  - [ ] **NEVER:** Wrap response in `PageableXxx` -> framework handles wrapping
  - [ ] **NEVER:** Use `string` for enum values -> use `EnumType` with full path
  - [ ] **NEVER:** Add file-level options (`java_package`, `go_package`)
  - [ ] **NEVER:** Import `domain/` in `api/` layer -> no cross-layer references
  - [ ] **NEVER:** Use SQL reserved words -> prefix: `user_name`, `order_no`
  - [ ] **NEVER:** Use relative type references -> full package: `com.example.Order`
  - [ ] **NEVER:** Request Message with 1-2 fields -> use `Empty` + `parameters{}`
  - [ ] **NEVER:** Single repeated field message -> use `input_repeated`/`output_repeated`
  - [ ] **NEVER:** Use `DTO` in message names -> real business meaning required
  - [ ] **NEVER:** Inner message types -> all messages at file level
  - [ ] **NEVER:** Redundant "ENUM" suffix
  - [ ] **NEVER:** Modify `_full.xml` migration file -> auto-generated
  - [ ] **NEVER:** Use `ORDINAL` enum persistence -> use `STRING` or `CODE`
  - [ ] **NEVER:** Duplicate `code` values within same enum
  - [ ] **ALWAYS:** `snake_case` for proto fields
  - [ ] **ALWAYS:** Table prefix `SYS_`/`BIZ_`/`CFG_`/`LOG_`
  - [ ] **ALWAYS:** Full package path for types
  - [ ] **ALWAYS:** Run `gradlew :{module}:wire` after proto changes
- [ ] **Conditional Rule Loading (L2 deep rules):**
  - [ ] **API Design:** {apihug_proto_api_extension_guide} loaded (if service/rpc involved)
  - [ ] **Database Design:** {apihug_proto_database_modeling_guide} loaded (if entity/table involved)
  - [ ] **Enum/Error Design:** {apihug_proto_enum_error_extension_guide} loaded (if enum/error involved)
- [ ] **Wire Generation Success:** `./gradlew :module:wire` generates expected classes

### Implementation (Impl Phase)
- [ ] **Rule Loaded:** {apihug_impl_golden_rule}
- [ ] **MANDATORY Rules Followed:** All FORBIDDEN rules avoided
- [ ] **MANDATORY Rules Applied:** All MUST DO rules implemented
- [ ] **SUCCESS METRICS Met:** See rules file SUCCESS METRICS section
- [ ] **NO FAILURE MODES:** See rules file FAILURE MODES section — all avoided
- [ ] **Build Success:** `./gradlew :module:build` passes

### Testing
- [ ] **Unit Tests:** All core functionality tested
- [ ] **Integration Tests:** Component interactions tested
- [ ] **All Tests Pass:** `./gradlew :module:test` succeeds

### Backend Completion
- [ ] **All Backend Tasks Complete:** Every proto/impl task marked [x]
- [ ] **File List Updated:** All proto and Java files listed
- [ ] **Dev Agent Record:** Contains "## Backend Phase Completed"
- [ ] **Story Status:** Set to "review"

---

## PHASE 2: Frontend Validation (if enabled)

### Prerequisites
- [ ] **Backend Completed:** "## Backend Phase Completed" present
- [ ] **Code Review:** Backend approved OR user confirmed skip
- [ ] **Frontend Enabled:** project.yaml has frontend enabled for domain

### Context Loading
- [ ] **UI Config Loaded:** ui.yml parsed
- [ ] **API Dependencies:** apis.yml parsed, domain SDK identified
- [ ] **Services Available:** apihug.idx.csv exists, Services importable

### Frontend Implementation
- [ ] **Rule Loaded:** {apihug_impl_front_vben_guide}
- [ ] **MANDATORY Rules Followed:** All FORBIDDEN rules avoided
- [ ] **MANDATORY Rules Applied:** All MUST DO rules implemented
- [ ] **SUCCESS METRICS Met:** See rules file SUCCESS METRICS section
- [ ] **NO FAILURE MODES:** See rules file FAILURE MODES section — all avoided

### Generation
- [ ] **UI Generation Success:** `gradlew :ui_module:ui` completed
- [ ] **Routes Generated:** router/routes/modules/*.ts exists
- [ ] **Menus Generated:** menus/*.ts exists

### Frontend Completion
- [ ] **All Frontend Tasks Complete:** Every Vue task marked [x]
- [ ] **File List Updated:** All Vue files listed
- [ ] **Dev Agent Record:** Contains "## Frontend Phase Completed"
- [ ] **Story Status:** Set to "review"

---

## Final Status Verification

- [ ] **Story Structure Preserved:** Only permitted sections modified
- [ ] **Sprint Status Updated:** If sprint tracking enabled
- [ ] **Quality Gates Passed:** All validations complete
- [ ] **No HALT Conditions:** No blocking issues
- [ ] **Review Follow-ups:** All review follow-up tasks completed and corresponding review items marked resolved (if applicable)

---

## Final Validation Output

```
Definition of Done: {{PASS/FAIL}}

Story: {{story_key}}

Backend Phase:
   Proto files: {{proto_count}}
   Services: {{service_count}}
   Test coverage: {{coverage}}%
   Status: {{backend_status}}
   Rules Loaded: {{loaded_proto_rules}} + {apihug_impl_golden_rule}

Frontend Phase: {{enabled_status}}
   Vue pages: {{page_count}}
   Components: {{component_count}}
   Status: {{frontend_status}}
   Rules: {apihug_impl_front_vben_guide}
```

**If FAIL:** List specific failures and required actions — reference rules file FAILURE MODES

**If PASS:** Story ready for code-review
