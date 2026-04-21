# Phase: Final Completion

> Validate Definition of Done, generate completion summary, and provide next steps.

---

<workflow>

  <step n="16" goal="Final completion and user support">
    <action>Execute the definition-of-done checklist using ../checklist.md</action>

    <check if="## Frontend Phase Completed exists">
      <output>Story Fully Complete (Backend + Frontend)

        Story: {{story_key}}
        Proto: {{proto_count}}, Services: {{service_count}}, Vue Pages: {{page_count}}, Components: {{component_count}}
      </output>
    </check>

    <check if="## Backend Phase Completed exists AND ## Frontend Phase Completed NOT exists">
      <output>Story Complete (Backend Only)

        Story: {{story_key}}
        Proto: {{proto_count}}, Services: {{service_count}}
      </output>
    </check>

    <action>Confirm File List includes every changed file</action>
    <action>Verify Change Log has summary of changes</action>

    <action>Communicate to {user_name} that story implementation is complete and ready for review</action>
    <action>Summarize key accomplishments: story key, title, key changes made, tests added, files modified</action>

    <action>Based on {user_skill_level}, ask if user needs any explanations</action>

    <action>Recommended next steps:
      - Run `code-review` workflow for peer review (preferably with different LLM)
      - Review the implemented story and test the changes
      - Verify all acceptance criteria are met
    </action>

    <check if="{sprint_status} file exists">
      <action>Suggest checking {sprint_status} to see project progress</action>
    </check>

    <!-- Final validation gates -->
    <action if="any task is incomplete">HALT - Complete remaining tasks</action>
    <action if="regression failures exist">HALT - Fix regression issues</action>
    <action if="File List is incomplete">HALT - Update File List</action>
    <action if="definition-of-done validation fails">HALT - Address DoD failures</action>
  </step>

</workflow>
