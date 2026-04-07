# Phase: Frontend Implementation

> Execute frontend tasks for the Vben Admin app following ApiHug UI patterns.

---

## Prerequisites

- `## Backend Phase Completed` marker MUST exist in Dev Agent Record
- If not present, HALT: "Backend must be completed before frontend phase"

---

<workflow>

  <critical>FOLLOW STORY FILE FRONTEND TASKS SEQUENCE EXACTLY</critical>
  <critical>Execute continuously without pausing until all frontend tasks are complete or explicit HALT condition</critical>

  <!-- ==================== -->
  <!-- STEP 11: CONTEXT     -->
  <!-- ==================== -->

  <step n="11" goal="Load frontend context">
    <action>Verify "## Backend Phase Completed" exists in Dev Agent Record; if not, HALT</action>
    <action>Load {ui_config} (ui.yml) and {app_apis_config} (apis.yml)</action>

    <!-- Determine frontend app -->
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

  <step n="12" goal="Implement frontend task">

    <!-- L2: Load frontend rule -->
    <action>Load COMPLETE rule file: {apihug_impl_front_vben_guide}</action>
    <critical>HALT if ANY MANDATORY rule violated — See rules file MANDATORY EXECUTION RULES section</critical>
    <critical>PRE-WRITE CHECK: LIST_PAGE? -> Check rules PAGE PATTERNS -> LIST_PAGE</critical>
    <critical>PRE-WRITE CHECK: FORM_PAGE? -> Check rules PAGE PATTERNS -> FORM_PAGE</critical>
    <critical>PRE-WRITE CHECK: DETAIL_PAGE? -> Check rules PAGE PATTERNS -> DETAIL_PAGE</critical>

    <action>Identify first incomplete FRONTEND task</action>
    <check if="no incomplete frontend tasks"><goto step="15">Frontend completion</goto></check>

    <action>Determine task type: "list page"/"query" -> LIST_PAGE; "form page"/"add/edit" -> FORM_PAGE; "detail page"/"view" -> DETAIL_PAGE; "component" -> COMPONENT</action>

    <check if="task type == LIST_PAGE">
      <action>Generate: List page with useVbenVxeGrid, proxyConfig.ajax.query, page index conversion (currentPage - 1)</action>
    </check>

    <check if="task type == FORM_PAGE">
      <action>Generate: Form modal with useVbenForm + useVbenModal, modalApi.lock()/unlock()</action>
    </check>

    <check if="task type == DETAIL_PAGE">
      <action>Generate: Detail view with hideInMenu: true, data loading in onMounted</action>
    </check>

    <check if="task type == COMPONENT">
      <action>Generate: {components_dir}/{{ComponentName}}.vue with Props/Events</action>
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
    <action>Verify Vue files exist with correct defineOptions and Service imports</action>
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
