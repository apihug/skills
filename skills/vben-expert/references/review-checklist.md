# Frontend Review Checklist

Use this checklist before marking any frontend task complete in `apihug-dev-story`.

Apply all sections that match the current `frontend_sub_modes[]`. A single task may require multiple sections.

## Global Checks

- The implementation still follows `{project-root}/_bmad/apihug/rules/apihug-impl-front-vben-guide.md`.
- Generated services are used instead of `requestClient`, `axios`, or handwritten API wrappers.
- `RequestSchema` is used for forms and search forms where applicable.
- `ResponseSchema` is used for table columns where applicable.
- Any page metadata still follows the `defineOptions({ name, meta })` contract.
- Any borrowed Vben example or playground pattern was used only for UI composition.
- No generic Vben demo API, paging, or adapter assumptions were copied into APIHug feature code.

## `LIST_GRID`

- Search form is placed in `formOptions`, not `gridOptions.formConfig`.
- Paging uses `page.currentPage - 1` when building APIHug page requests.
- Grid slots, toolbar slots, tree-table settings, and custom cells are intentional and scoped to the task.
- Table behavior matches the selected Vben pattern instead of defaulting to copied demo code.

## `FORM_SCHEMA`

- Field rules, defaults, dependencies, and `valueFormat` are intentional and aligned with the story task.
- Form schema patches are done in page code, not by editing generated schema files.
- Submit lifecycle and validation flow are explicit and complete.

## `DETAIL_PRESENTATION`

- Read-only presentation is intentional and not implemented as a hidden editable form.
- Long text, tooltip, or expand behavior is handled intentionally where needed.
- Detail page visibility and route behavior still align with the ApiHug route contract.

## `MODAL_FLOW`

- `connectedComponent`, `setData`, `getData`, and `onOpenChange` are used consistently.
- Submit lifecycle uses `lock()` and `unlock()` when async confirmation is involved.
- The modal is used because the interaction is modal-sized, not because it was copied from a demo.

## `DRAWER_FLOW`

- Drawer lifecycle, state handoff, and submit locking are correct.
- Drawer is used because the editing experience benefits from side-panel interaction, not by default.

## `ALERT_PROMPT`

- `alert`, `confirm`, or `prompt` is used only for lightweight interaction.
- A full modal or drawer was not silently replaced by a prompt-based shortcut.

## `ELLIPSIS_TEXT`

- Truncation, tooltip, and expand behavior improve readability instead of hiding important data.
- Overflow handling works intentionally in the actual table or detail layout.

## `ACCESS_CONTROL`

- Access control is implemented in UI visibility or interaction points only.
- Backend authorization logic is not duplicated in frontend code.

## `THEME_ICON`

- Theme and icon changes stay within the product's established visual language.
- Styling changes are scoped and do not silently alter unrelated pages or shared behaviors.
