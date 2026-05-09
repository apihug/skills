# Phase: Frontend Implementation

> Execute frontend tasks for the Vben Admin app by loading the canonical frontend guide first, selecting an approved page pattern, then using `vben-expert` to route to the minimal Vben references needed for implementation.

---

## Prerequisites

- `## Backend Phase Completed` marker MUST exist in Dev Agent Record
- If not present, HALT: "Backend must be completed before frontend phase"

---

<workflow>

  <critical>FOLLOW STORY FILE FRONTEND TASKS SEQUENCE EXACTLY</critical>
  <critical>Execute continuously without pausing until all frontend tasks are complete or an explicit HALT condition is reached</critical>

  <!-- ==================== -->
  <!-- STEP 11: CONTEXT     -->
  <!-- ==================== -->

  <step n="11" goal="Load frontend context">
    <action>Verify "## Backend Phase Completed" exists in Dev Agent Record; if not, HALT</action>
    <action>Load {ui_config} (ui.yml) and {app_apis_config} (apis.yml)</action>

    <check if="{frontend_app} NOT specified">
      <action>Load available apps from {frontend_target_ui_module_apps}</action>
      <check if="apps_count == 0"><output>Configure frontend_app in project.yaml</output><action>HALT</action></check>
      <check if="apps_count == 1"><action>Auto-select single app</action></check>
      <check if="apps_count > 1"><ask>Select app number or name:</ask></check>
    </check>

    <action>Re-resolve paths: app_path, app_apis_config, views_dir, components_dir, stores_dir</action>
    <action>Scan existing code: views, components, stores</action>
    <output>Frontend Context: UI Module={{frontend_ui_module}}, App={{frontend_app}}, Domain SDK={{domain_sdk_module}}</output>
  </step>

  <!-- ==================== -->
  <!-- STEP 12: IMPLEMENT   -->
  <!-- ==================== -->

  <step n="12" goal="Design and implement the current frontend task">
    <action>Load COMPLETE canonical rule file: {apihug_impl_front_vben_guide}</action>
    <critical>HALT if any canonical frontend rule or stop condition is violated</critical>
    <critical>`vben-expert` MUST be invoked for ALL frontend tasks in this phase. Do not treat it as optional.</critical>

    <action>Identify first incomplete FRONTEND task</action>
    <check if="no incomplete frontend tasks"><goto step="15">Frontend completion</goto></check>

    <action>Determine task type: "list page"/"query" -> LIST_PAGE; "form page"/"add/edit" -> FORM_PAGE; "detail page"/"view" -> DETAIL_PAGE; "component" -> COMPONENT</action>
    <action>Select the page pattern from the canonical guide's "Page Pattern Decision" section before writing code</action>
    <action>Run the canonical guide's "Before writing code" checklist before implementation</action>
    <action>Check story Dev Notes for explicit frontend sub-mode hints, per-task UI behavior notes, or Vben reference expectations for the current task</action>

    <critical>HALT if locale coverage is missing for any selected page or schema key</critical>
    <critical>HALT if a required reference field has no backend lookup/search/tree contract</critical>
    <critical>HALT if a hierarchy-maintenance page has no non-pageable tree/list SDK contract</critical>
    <critical>HALT if the backend still requires non-empty input for a backend-managed business code field</critical>
    <critical>HALT if sort UI is required but the backend sort contract is not available</critical>

    <action if="explicit frontend sub-mode hints exist for the current task">Use those hints as the primary source for `frontend_sub_modes[]`</action>
    <action if="explicit frontend sub-mode hints do NOT exist for the current task">Determine one or more frontend sub-modes from the selected page pattern and intended UI behavior:
      - pageable list workbench -> LIST_GRID
      - hierarchy maintenance -> TREE_WORKBENCH
      - split master-detail layout -> SPLIT_LAYOUT
      - field/schema/rules/validation/dependencies/valueFormat -> FORM_SCHEMA
      - modal dialog lifecycle -> MODAL_FLOW
      - drawer lifecycle -> DRAWER_FLOW
      - read-only detail or long-text presentation -> DETAIL_PRESENTATION
      - lookup or inspect-in-place modal -> LOOKUP_MODAL
      - upload/import surface -> UPLOAD_IMPORT
      - diagnostic or test workbench -> DIAGNOSTIC_PANEL
      - alert/result/loading/empty/fallback state work -> RESULT_STATE
      - staged ordered workflow -> STEPPED_FLOW
      - authority/access/permission/button visibility -> ACCESS_CONTROL
      - icon/theme/style polish -> THEME_ICON
      - nearest approved business analog needed -> PLAYGROUND_ANALOG
    </action>
    <action>Store the result as `frontend_sub_modes[]` and allow multiple matches for the same task</action>
    <action>Invoke the `vben-expert` skill as the secondary router for page-pattern references, Vben component behavior, and safe analog selection</action>
    <critical>Priority order for decisions: canonical frontend guide -> `vben-expert` -> routed page-pattern docs -> component docs and playground analogs</critical>
    <critical>Never inherit API, SDK, paging, or request-client conventions from generic Vben demos or playground source</critical>

    <check if="task type == LIST_PAGE">
      <action>Implement the approved list or tree workbench using generated services, schema-driven search, shared action/status helpers, and the routed Vben references</action>
    </check>

    <check if="task type == FORM_PAGE">
      <action>Implement the approved form modal, drawer, or configuration surface using generated schema, proper submit lifecycle, and the routed Vben references</action>
    </check>

    <check if="task type == DETAIL_PAGE">
      <action>Implement the approved detail, trace, or lookup presentation using routed references and explicit read-only behavior</action>
    </check>

    <check if="task type == COMPONENT">
      <action>Implement the page-local component under {components_dir} with only the Vben behavior needed by the selected page pattern and sub-modes</action>
    </check>
  </step>

  <!-- ==================== -->
  <!-- STEP 13: GENERATE    -->
  <!-- ==================== -->

  <step n="13" goal="Generate routes and menus">
    <action>Execute: gradlew :{frontend_ui_module}:ui</action>
    <action>Verify: {router_dir}/*.ts, {menus_dir}/*.ts generated</action>
  </step>

  <!-- ==================== -->
  <!-- STEP 14: VALIDATE    -->
  <!-- ==================== -->

  <step n="14" goal="Validate and mark frontend task complete">
    <action>Verify Vue files exist with correct defineOptions and generated Service imports</action>
    <action>Run the canonical guide's "Before marking the task complete" checklist</action>
    <action>Verify no canonical failure mode is present</action>
    <action>Load and apply the `vben-expert` review checklist for all current frontend sub-modes</action>
    <action>Verify any borrowed pattern was used only for page shape and did not override canonical contracts</action>
    <action>Run: pnpm lint (if configured)</action>
    <action if="lint fails">STOP and fix</action>

    <action>Mark frontend task checkbox [x]; update File List; save story</action>
    <check if="more frontend tasks"><goto step="12">Next frontend task</goto></check>
  </step>

  <!-- ==================== -->
  <!-- STEP 15: PHASE DONE  -->
  <!-- ==================== -->

  <step n="15" goal="Frontend phase completion">
    <action>Verify ALL frontend tasks marked [x]</action>
    <action>Add to Dev Agent Record:
      ## Frontend Phase Completed
      **Date**: {{date}}
      **Vue Pages**: {{page_count}}
      **Components**: {{component_count}}
    </action>
    <action>Update story Status to "review"</action>
    <check if="{sprint_status} file exists AND {{current_sprint_status}} != 'no-sprint-tracking'">
      <action>Update sprint-status: development_status[{{story_key}}] = "review"</action>
    </check>
  </step>

</workflow>

---

## NEXT

Read fully and follow: `./phase-complete.md` for final completion.
