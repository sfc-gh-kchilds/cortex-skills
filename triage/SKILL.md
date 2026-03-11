---
name: triage-lab-content
description: "Analyze a GitLab/GitHub lab or Snowflake Quickstart and recommend which sections to convert to interactive React course vs keep as lab format."
parent_skill: gitlab-to-react-course
---

# Triage Lab Content

Analyze lab content from GitLab, GitHub, or a Snowflake Quickstart and produce a recommendation for what becomes an interactive React course lesson vs what stays in traditional lab format.

## When to Load

Main skill routes here when user wants to analyze/assess a lab before building.

## Prerequisites

- Lab content has been fetched (markdown, SQL, config files from the repo)
- User has confirmed the URL and intent

## Workflow

### Step 1: Parse Lab Structure

Break the lab content into discrete sections. Look for:
- **H1/H2 headers** as section boundaries
- **Quickstart `## Step N` headers** — these are the primary section boundaries in sfguides (e.g., `## Step 1 - Overview`, `## Step 2 - Create Tables`). Lines with `Duration: N` immediately after indicate estimated minutes and can be used for course timing.
- **SQL code blocks** — candidate for interactive exercises
- **Step-by-step instructions** with explicit outcomes
- **Conceptual explanations** — candidate for read-through lesson sections
- **Setup/prerequisite blocks** — usually NOT interactive (infra config, account setup)
- **Screenshots/images** — note these; they can become lesson section illustrations
- **Quickstart metadata block** — the YAML-like frontmatter at the top of sfguide markdown (id, summary, categories, environments, status, tags, authors) should be captured for course metadata but is NOT lesson content

Create a section inventory:
```
| # | Section Title | Type | Content Summary |
|---|--------------|------|----------------|
| 1 | Introduction | Concept | Overview of the topic |
| 2 | Create Table | SQL Exercise | CREATE TABLE with specific params |
| ...
```

### Step 2: Classify Each Section

Score each section on **interactivity potential** using these criteria:

**GOOD candidates for React interactive lessons (score 3+):**
- SQL writing exercises where the learner fills in blanks or writes statements
- Multiple-choice questions testing conceptual understanding
- Configuration exercises with specific correct answers
- Scenario-based questions ("the customer asks X, what do you say?")
- Exercises with clear right/wrong answers that can be auto-graded

**POOR candidates — keep as GitLab lab (score 1-2):**
- Environment setup (CREATE WAREHOUSE, GRANT ROLE, etc.) — infra, not learning
- Long sequential CLI workflows (docker build, deploy steps)
- Tasks requiring real Snowflake execution against live data
- Sections that depend on previous section's live state (cascading dependencies)
- Content that's purely reference/lookup (no exercise component)

**Classification types:**
- `concept` — Read-through content with a quiz question (radio/multiple-choice)
- `sql_fill` — SQL template with blanks to fill in (keyword matching)
- `sql_write` — Freeform SQL writing with keyword validation
- `scenario` — Customer scenario with best-response selection
- `skip` — Keep in GitLab lab format, not suitable for interactive

### Step 3: Group Into Lessons

Group related sections into lessons (target 5-8 lessons per course):
- Each lesson should have 1-3 exercises
- Mix exercise types for variety (don't make all lessons the same type)
- Order by learning progression (foundations first, advanced later)
- Include a customer scenario opener for each lesson

### Step 4: Present Recommendation

**Present this to the user:**

```
## Lab Analysis: <LAB_NAME>

### Recommended Interactive Lessons (React App)

| Lesson | Title | Type | Source Sections | Exercises |
|--------|-------|------|----------------|-----------|
| 1 | ... | concept | Sections 1-2 | 1 radio quiz |
| 2 | ... | sql_fill | Section 3 | 1 fill-in-blank |
| ...

### Keep as Original Lab / Quickstart

| Section | Reason |
|---------|--------|
| Setup & Prerequisites | Infra config, not learning content |
| ... | ... |

### Course Metadata
- **Total lessons**: X
- **Exercise types**: X concept, X sql_fill, X sql_write, X scenario
- **Estimated completion time**: ~Y minutes
- **Max possible points**: Z (at 100pts/exercise first try)
```

**STOP**: Wait for user approval. They may want to:
- Move sections between interactive/lab
- Merge or split lessons
- Change exercise types
- Add customer scenarios

### Step 5: Hand Off to Build

After approval, pass the approved lesson plan to `build-course/SKILL.md`:
- Approved lesson structure (titles, types, section mappings)
- Exercise definitions (questions, correct answers, hints)
- Customer scenarios per lesson
- Any user customizations

**Continue** to `build-course/SKILL.md` with the approved plan.

## Stopping Points

- After Step 4: User must approve the recommendation before building
- User can modify the plan at this checkpoint

## Output

Approved lesson plan ready for the build-course sub-skill.
