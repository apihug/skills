# Phase: Discover & Prepare

> Find the next ready story, load project context, detect review continuation, and mark story in-progress.

---

<workflow>
  <critical>Communicate all responses in {communication_language} and language MUST be tailored to {user_skill_level}</critical>
  <critical>Generate all documents in {document_output_language}</critical>
  <critical>Only modify the story file in these areas: Tasks/Subtasks checkboxes, Dev Agent Record, File List, Change Log, and Status</critical>

  <!-- ==================== -->
  <!-- STEP 1: STORY DISCOVERY -->
  <!-- ==================== -->

  <step n="1" goal="Find next ready story and load it" tag="sprint-status">
    <check if="{{story_path}} is provided">
      <action>Use {{story_path}} directly</action>
      <action>Read COMPLETE story file</action>
      <action>Extract story_key from filename or metadata</action>
      <goto anchor="phase_detection" />
    </check>

    <!-- Sprint-based story discovery -->
    <check if="{{sprint_status}} file exists">
      <critical>MUST read COMPLETE sprint-status.yaml file from start to end to preserve order</critical>
      <action>Load the FULL file: {{sprint_status}}</action>
      <action>Read ALL lines from beginning to end - do not skip any content</action>
      <action>Parse the development_status section completely to understand story order</action>

      <action>Find the FIRST story (by reading in order from top to bottom) where:
        - Key matches pattern: number-number-name (e.g., "1-2-user-auth")
        - NOT an epic key (epic-X) or retrospective (epic-X-retrospective)
        - Status value equals "ready-for-dev" OR "review" (review = resume after code-review)
      </action>

      <check if="no ready-for-dev or in-progress story found">
        <output>No ready-for-dev stories found in sprint-status.yaml

          **Current Sprint Status:** {{sprint_status_summary}}

          **What would you like to do?**
          1. Run `create-story` to create next story from epics with comprehensive context
          2. Run `validate-create-story` to improve existing stories before development
          3. Specify a particular story file to develop (provide full path)
          4. Check {{sprint_status}} file to see current sprint status
        </output>
        <ask>Choose option [1], [2], [3], or [4], or specify story file path:</ask>

        <check if="user chooses '1'"><action>HALT - Run create-story to create next story</action></check>
        <check if="user chooses '2'"><action>HALT - Run validate-create-story to improve existing stories</action></check>
        <check if="user chooses '3'">
          <ask>Provide the story file path to develop:</ask>
          <action>Store user-provided story path as {{story_path}}</action>
          <goto anchor="phase_detection" />
        </check>
        <check if="user chooses '4'">
          <output>Loading {{sprint_status}} for detailed status review...</output>
          <action>Display detailed sprint status analysis</action>
          <action>HALT - User can review sprint status and provide story path</action>
        </check>
        <check if="user provides story file path">
          <action>Store user-provided story path as {{story_path}}</action>
          <goto anchor="phase_detection" />
        </check>
      </check>
    </check>

    <!-- Non-sprint story discovery -->
    <check if="{{sprint_status}} file does NOT exist">
      <action>Search {implementation_artifacts} for stories directly</action>
      <action>Find stories with "ready-for-dev" status in files</action>
      <action>Look for story files matching pattern: *-*-*.md</action>
      <action>Read each candidate story file to check Status section</action>

      <check if="no ready-for-dev stories found in story files">
        <output>No ready-for-dev stories found

          **Available Options:**
          1. Run `create-story` to create next story from epics with comprehensive context
          2. Run `validate-create-story` to improve existing stories
          3. Specify which story to develop
        </output>
        <ask>What would you like to do? Choose option [1], [2], or [3]:</ask>

        <check if="user chooses '1'"><action>HALT - Run create-story</action></check>
        <check if="user chooses '2'"><action>HALT - Run validate-create-story</action></check>
        <check if="user chooses '3'">
          <ask>Provide the full path to the story file:</ask>
          <action>Store user-provided story path as {{story_path}}</action>
          <action>Continue with provided story file</action>
        </check>
      </check>

      <check if="ready-for-dev story found in files">
        <action>Use discovered story file and extract story_key</action>
      </check>
    </check>

    <action>Store the found story_key (e.g., "1-2-user-authentication") for later status updates</action>
    <action>Find matching story file in {implementation_artifacts} using story_key pattern: {{story_key}}.md</action>
    <action>Read COMPLETE story file from discovered path</action>

    <anchor id="phase_detection" />
    <action>Parse sections: Story, Acceptance Criteria, Tasks/Subtasks, Dev Notes, Dev Agent Record, File List, Change Log, Status</action>

    <!-- Detect phase from Dev Agent Record markers -->
    <action>Check Dev Agent Record for markers: "## Backend Phase Completed", "## Backend Phase: APPROVED", "## Frontend Phase Completed"</action>

    <check if="## Frontend Phase Completed exists">
      <action>Read and follow: ./phase-complete.md</action>
    </check>
    <check if="## Backend Phase: APPROVED exists AND {frontend_enabled} == true">
      <action>Set {{current_phase}} = "frontend"</action>
      <action>Read and follow: ./phase-frontend.md</action>
    </check>
    <check if="## Backend Phase Completed exists AND ## Backend Phase: APPROVED NOT exists">
      <output>Backend done, awaiting code-review. [1] Run code-review, [2] Skip (NOT RECOMMENDED)</output>
      <ask>Choose [1] or [2]:</ask>
      <check if="user chooses '1'"><action>HALT - Run code-review workflow</action></check>
      <check if="user chooses '2'">
        <action>Set {{current_phase}} = "frontend"</action>
        <action>Read and follow: ./phase-frontend.md</action>
      </check>
    </check>

    <action>Set {{current_phase}} = "backend"</action>
    <action>Identify first incomplete task (unchecked [ ]) in Tasks/Subtasks</action>
    <action if="no incomplete tasks">Read and follow: ./phase-backend.md (jump to Step 9 — Backend completion)</action>
    <action if="story file inaccessible">HALT: "Cannot develop story without access to story file"</action>
  </step>

  <!-- ==================== -->
  <!-- STEP 2: CONTEXT LOAD  -->
  <!-- ==================== -->

  <step n="2" goal="Load project context and ApiHug configuration">
    <critical>Load all available context to inform implementation</critical>

    <action>Load {project_context} for coding standards and project-wide patterns (if exists)</action>
    <action>Load domain configuration from {domain_module}</action>
    <action>Load {hope_wire} config: parse packageName, domain, application, module</action>
    <action>Load proto module meta index (if exists, generated from previous build):
      - {meta_index}: Compressed components and entities index in CSV format
      Note: This CSV file is auto-generated after wire, providing complete module snapshot with minimal token overhead
    </action>
    <action>Load comprehensive context from story file's Dev Notes section</action>
    <action>Extract developer guidance from Dev Notes: architecture requirements, previous learnings, technical specifications</action>
    <output>Context: Story={{story_key}}, Domain={{domain_name}}, Module={{module_gradle_module}}, Frontend={{frontend_enabled}}</output>
  </step>

  <!-- ==================== -->
  <!-- STEP 3: REVIEW CHECK  -->
  <!-- ==================== -->

  <step n="3" goal="Detect review continuation and extract review context">
    <critical>Determine if this is a fresh start or continuation after code review</critical>

    <action>Check if "Senior Developer Review (AI)" section exists in the story file</action>
    <action>Check if "Review Follow-ups (AI)" subsection exists under Tasks/Subtasks</action>

    <check if="Senior Developer Review section exists">
      <action>Set review_continuation = true</action>
      <action>Extract from "Senior Developer Review (AI)" section:
        - Review outcome (Approve/Changes Requested/Blocked)
        - Review date
        - Total action items with checkboxes (count checked vs unchecked)
        - Severity breakdown (High/Med/Low counts)
      </action>
      <action>Count unchecked [ ] review follow-up tasks in "Review Follow-ups (AI)" subsection</action>
      <action>Store list of unchecked review items as {{pending_review_items}}</action>

      <output>Resuming Story After Code Review ({{review_date}})

        **Review Outcome:** {{review_outcome}}
        **Action Items:** {{unchecked_review_count}} remaining to address
        **Priorities:** {{high_count}} High, {{med_count}} Medium, {{low_count}} Low

        **Strategy:** Will prioritize review follow-up tasks (marked [AI-Review]) before continuing with regular tasks.
      </output>
    </check>

    <check if="Senior Developer Review section does NOT exist">
      <action>Set review_continuation = false</action>
      <action>Set {{pending_review_items}} = empty</action>

      <output>Starting Fresh Implementation

        Story: {{story_key}}
        Story Status: {{current_status}}
        First incomplete task: {{first_task_description}}
      </output>
    </check>
  </step>

  <!-- ==================== -->
  <!-- STEP 4: MARK STATUS   -->
  <!-- ==================== -->

  <step n="4" goal="Mark story in-progress" tag="sprint-status">
    <check if="{{sprint_status}} file exists">
      <action>Load the FULL file: {{sprint_status}}</action>
      <action>Read all development_status entries to find {{story_key}}</action>
      <action>Get current status value for development_status[{{story_key}}]</action>

      <check if="current status == 'ready-for-dev' OR review_continuation == true">
        <action>Update the story in the sprint status report to = "in-progress"</action>
        <action>Update last_updated field to current date</action>
        <output>Starting work on story {{story_key}}
          Status updated: ready-for-dev -> in-progress
        </output>
      </check>

      <check if="current status == 'in-progress'">
        <output>Resuming work on story {{story_key}}
          Story is already marked in-progress
        </output>
      </check>

      <check if="current status is neither ready-for-dev nor in-progress">
        <output>Unexpected story status: {{current_status}}
          Expected ready-for-dev or in-progress. Continuing anyway...
        </output>
      </check>

      <action>Store {{current_sprint_status}} for later use</action>
    </check>

    <check if="{{sprint_status}} file does NOT exist">
      <output>No sprint status file exists - story progress will be tracked in story file only</output>
      <action>Set {{current_sprint_status}} = "no-sprint-tracking"</action>
    </check>
  </step>

</workflow>

---

## NEXT

Read fully and follow: `./phase-backend.md` to begin backend implementation.
