---
name: check-in
description: Review a user's assigned projects and tasks using the Inovo dashboard MCP to identify stale statuses, missing comments, weak information radiance, framing gaps, and workflow gaps. Use when a developer, PM, or team member needs a project check-in, daily sync review, task hygiene review, or help deciding what statuses or comments should be updated. Follows Inovo communication style and asks for confirmation before making any write actions through the MCP.
---

# Inovo Project Check-In

Use this skill to help a team member review the health of their assigned work and keep the dashboard accurate.

The goal: keep process easy, make progress visible, improve information radiance with minimal friction.

**Requires the Inovo dashboard MCP connection** (`mcp__inovo-dashboard__*` tools). If the MCP is not connected, tell the user to set it up at `/dashboard/settings/mcp-setup` and stop.

## Core operating style

Follow Inovo style, not generic PM language.

Every recommendation should be:
- **minimal**
- **clear**
- **framed**
- **skimmable**
- **specific**

Communication rules:
- lead with the bottom line
- use short bullets
- bold only what matters
- keep comments short
- prefer direct language over explanatory language
- avoid filler, hedging, and long backstory
- sound calm and in control

Do not write like a consultant or a project manager trying to sound formal.
Write like a builder keeping the team aligned.

## Inovo framing rules

Before suggesting any update, frame the work in this order:

1. **State** — what is true right now
2. **Blocker** — what is preventing movement, if anything
3. **Next step** — what should happen next

Good updates make all 3 obvious. If a comment or status does not clarify state, blocker, or next step, it is weak and should be improved.

## What to check for

Look for gaps like:
- task is `Dev Ready` but work has clearly started — should be `In Progress`
- task is `In Progress` but work is complete — should be `In Review`
- task has recent activity but no visible comment
- project has gone quiet and no blocker is documented
- work is being done but not radiated into the dashboard
- comment exists but is vague and does not frame the work
- next step is unclear
- framing document is missing or incomplete for a project in Shaping or later
- task grid is empty for a project past Framing stage
- task is active but still unshaped or under-framed

## Priorities

Surface only the most important 3 to 5 gaps:

1. **status accuracy** — task/project status matches reality
2. **clear framing** — framing doc exists and is complete
3. **information radiance** — recent comments that explain state
4. **blockers** — documented when present
5. **next-step clarity** — obvious what happens next

Do not dump every issue. Focus on what matters most right now.

## Workflow

### 1. Identify the user's projects

Ask the user which project to check in on, or use the MCP to find their work:

```
search_projects → find their project(s)
get_project → get status, appetite, framing status, dates
```

If the user has multiple projects, ask which one to focus on, or do a quick scan of all.

### 2. Pull current state

Use these MCP tools to gather the full picture:

| What | Tool | Key fields |
|------|------|------------|
| Project details | `get_project` | status, appetite, has_framing, dates |
| Framing document | `get_framing` | problem, outcome, out_of_bounds, appetite_justification |
| Shape Up task grid | `get_task_grid` | slices, tasks, completion counts |
| Dev tasks | `list_project_tasks` | status, assignee, priority per task |
| Task details | `get_task` | content, comments, status, dates, assignees |
| Project comments | `get_project_comments` | author, text, timestamps, resolved status |

Pull project + tasks first. Only drill into framing, task grid, or individual task details if gaps surface.

### 3. Evaluate reality vs dashboard

Look for mismatch between:
- actual movement (recent comments, updated dates)
- visible communication (comments, status changes)
- recorded status (task and project statuses)

**Status accuracy checks:**
- Task is `Dev Ready` but has recent comments or activity → should be `In Progress`
- Task is `In Progress` but looks complete → should be `In Review`
- Task is `In Progress` with no activity for days → may be stalled, needs a blocker comment or status correction

**Framing checks:**
- Project is in `Shaping` or later but `has_framing` is false → framing needed
- Framing exists but sections are empty or weak → suggest improvements via `review_framing`

**Comment checks:**
- Tasks with no comments and status changes → missing context
- Last comment is old relative to status → stale signal
- Comments that say "working on this" or "making progress" → weak, need rewriting

### 4. Present recommendations

Use this structure:

```
## Check-in: [Project Name]
One or two sentences. Lead with the biggest truth.

## Recommended updates

- **Gap**: [what looks off]
  **Action**: [exact MCP tool + value]
  **Why**: [one sentence]
```

Example:

```
## Check-in: Client Portal Redesign
Dashboard is behind reality. Three small updates would make current state visible.

## Recommended updates

- **Gap**: Task T-42 is still `Dev Ready` — work started 2 days ago
  **Action**: `update_task_status` → "In Progress"
  **Why**: Status should reflect active development

- **Gap**: Task T-43 has no recent signal
  **Action**: `add_task_comment` → "Started auth flow implementation. Working through token refresh edge cases. Next: testing retries."
  **Why**: Makes current progress visible to the team

- **Gap**: Task T-44 appears stalled with no blocker documented
  **Action**: `add_task_comment` → "Blocked on endpoint clarification from backend. Next: confirm payload shape before continuing."
  **Why**: Makes the blocker and next step obvious
```

### 5. Ask before writing

After presenting recommendations, ask for confirmation:

> "Want me to apply these updates?"

**Never** change statuses or add comments without explicit confirmation.

### 6. Apply confirmed updates

Once confirmed, execute using the correct MCP tools:

| Action | MCP Tool |
|--------|----------|
| Change task status | `update_task_status` (task_id, status) |
| Add task comment | `add_task_comment` (task_id, comment) |
| Assign task to user | `assign_task` (task_id) |
| Update task content | `update_task_content` (task_id, content) |

**Enforced status transitions** (the MCP will reject invalid ones):
- `Dev Ready` → `In Progress` or `In Review`
- `In Progress` → `In Review` or `Dev Ready`
- `In Review` → `Completed` is **human-only** — do not attempt

After applying, confirm what was done in a short summary.

## Comment-writing rules

Every comment should be short and shaped.

Structure: **Now** / **Blocker** (if any) / **Next**

Good:
- "Started API sync work. Working through auth edge cases. Next: testing retries."
- "UI implementation in progress. No blocker. Next: wiring submission flow."
- "Blocked on endpoint clarification. Next: confirm payload shape before continuing."
- "Core implementation complete. Next: review and QA."

Bad:
- "working on this"
- "making progress"
- long explanations
- multiple paragraphs
- comments with no next step

Use `backticks` for file names and code references. Use **bold** for emphasis on key phrases.

## Status decision rules

- if work has clearly started → `Dev Ready` to `In Progress`
- if work is complete → `In Progress` to `In Review`
- if work is blocked and being put back → `In Progress` to `Dev Ready` with a blocker comment
- if uncertain, do not fake precision — present it as a likely recommendation and let the user decide
- never attempt `In Review` → `Completed` — that transition is reserved for humans

## Constraints

- prefer MCP data over assumptions — do not invent details
- do not write without confirmation
- keep output lightweight — optimized for daily use
- always prefer crisp bullets over long prose
- use task numbers in T-### format (never # prefix)
- link to Notion URLs when available from tool responses, never fabricate URLs

## Success criteria

A successful check-in helps the user:
- get the dashboard back in sync in a few minutes
- make status reflect reality
- leave short, useful comments
- communicate progress in a skim-friendly way
- frame work clearly enough that the next person reading is not guessing
