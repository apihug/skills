# Domain Strategic Design Prompt Pack

This file provides standard prompt skeletons for the executor.

Purpose:
- force `Phase A` to run with `bmad-agent-architect` modeling discipline
- stabilize first-draft generation
- standardize the three audit rounds
- avoid ad hoc prompting in each conversation

## 1. Phase A Initial Modeling Prompt
```text
You are now performing DDD strategic boundary modeling in the style of `bmad-agent-architect`.
This is not template filling and not freeform document writing.

Goal:
Build the strategic model required for `context-map.md`, then write it down only after the model converges.

Source set:
- PRD
- locked domain-model / bounded-context inputs if they exist
- input-artifacts
- project-context
- existing architecture or old context-map only as constrained references

Hard constraints:
1. Identify macro business domains first, then attach bounded contexts under them.
2. Do not use technical layer names as macro business domain names.
3. If Domain Model or Bounded Context definitions are already locked, only refine, map, clarify, and correct expression. Do not redraw them unless the user explicitly asks.
4. Treat existing `context-map.md` only as a weak reference or correction target, never as the truth source by default.
5. Boundary categories must be source-driven. Do not assume every project naturally has fixed categories such as platform/business/governance domains.
6. External systems entering internal business boundaries must explicitly state whether they pass through `ACL / Adapter`.
7. Unknown or unsupported conclusions must be written into `Open Questions`, not hallucinated.

You must converge at least:
- macro business domain identification
- locked BC and candidate BC identification
- BC-to-domain mapping
- context relationship types and dependency direction
- source-supported boundary-category rules
- logical slicing constraints for a single-module monolith if applicable

Only after these converge may you draft `context-map.md`.
Output style:
- primarily Chinese
- tables + Mermaid
- optimized for downstream `architecture.md` and tactical design
```

## 2. Source Alignment Audit Prompt
```text
Review the current `context-map.md` as a source-alignment auditor.
Focus only on:
1. whether it faithfully covers PRD / input-artifacts / domain-model / bounded-context inputs / architecture references
2. whether it incorrectly treats an old `context-map.md` as a truth source
3. whether it misses current-stage key domains, key boundary categories, or mislabels status
4. whether it turns project-specific patterns into fake universal rules

Output requirements:
- findings first
- for each finding include:
  - issue
  - affected section
  - downstream impact
  - recommended correction
```

## 3. DDD Consistency Audit Prompt
```text
Review the current `context-map.md` as a DDD strategic design auditor.
Focus on:
1. whether macro domains are real business domains
2. whether fake technical domains are mixed in
3. whether BCs are incorrectly promoted, split, merged, or assigned
4. whether relationship types are used correctly
5. whether tactical elements are mixed into the strategic model
6. whether source-driven boundary categories are incorrectly written as universal default project structure

Output requirements:
- findings first
- report only structural issues that materially affect strategic boundaries or downstream modularization
```

## 4. Implementation Readiness Audit Prompt
```text
Review the current `context-map.md` from the perspective of upstream constraints for modularization and code-structure design.
Judge:
1. whether macro domains can guide first-level module boundaries, or first-level logical slices in a single-module monolith
2. whether dependency directions are sufficient to guide module dependency control
3. whether source-supported boundary-category rules are strong enough to constrain `architecture.md`
4. whether there are structural contradictions likely to cause rework in `architecture.md` or tactical design

Output requirements:
- return `READY / READY WITH GAPS / NOT READY`
- if not `READY`, provide the minimum correction set
```

## 5. Finalization Prompt
```text
Finalize the current `context-map.md` based on the three audit rounds.
Requirements:
1. only fix issues already confirmed by audit
2. do not add new unsupported structure
3. keep domains, BCs, relationships, and boundary rules internally consistent
4. record the three audit results in the `Audit Record` section
5. clearly separate:
   - locked-input conclusions
   - source-supported conclusions
   - current-stage reasonable inferences
6. explicitly state:
   - which sources were used
   - which inputs were locked
   - which points remain open
   - whether `docs/project-context.md` has been synced
```
