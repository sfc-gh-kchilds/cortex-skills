---
name: gitlab-to-react-course
description: "Convert GitLab labs, GitHub labs, or Snowflake Quickstarts into interactive React course apps with Snowflake branding and gamification. Use when: user provides a GitLab/GitHub lab link or Snowflake quickstart URL to convert into a course, wants to build an interactive learning app, needs to assess which lab sections should become interactive vs stay as-is. Triggers: convert lab, build course, react course, interactive course, gamification, lab to app, gitlab lab, github lab, quickstart, sfguide, sfquickstart, developers/guides, DataCamp style, SE enablement course."
---

# Lab / Quickstart to Interactive React Course

Convert SE enablement labs (GitLab, GitHub) or Snowflake Quickstarts into polished, interactive React course apps with Snowflake 2026 branding, DataCamp-inspired gamification, and SPCS deployment.

## Intent Detection

| Intent | Triggers | Route |
|--------|----------|-------|
| TRIAGE | "analyze", "which parts", "should this be", "recommend", "assess", "review lab" | `triage/SKILL.md` |
| BUILD | "build", "create", "convert", "make interactive", "scaffold", "generate course" | `build-course/SKILL.md` |

If the user provides a link without specifying intent, default to **TRIAGE** first — analyze the lab content and recommend what to convert before building.

## Workflow

### Step 1: Collect Input

**Ask the user:**
1. **Source URL**: GitLab repo, GitHub repo, Snowflake Quickstart URL, or local path
2. **Intent**: Analyze & recommend, or build the full course?

### Step 2: Fetch Lab Content

**Detect the source type from the URL:**

- **Snowflake Quickstart** (`snowflake.com/en/developers/guides/<slug>` or `quickstarts.snowflake.com/guide/<slug>`):
  The published page is client-rendered and won't yield content via `web_fetch`. Instead, derive the raw markdown source from the slug:
  1. Extract `<slug>` from the URL (e.g., `getting-started-with-hybrid-tables` from `.../guides/getting-started-with-hybrid-tables/`)
  2. Strip any anchor fragment (`#0`, `#1`, etc.)
  3. Fetch: `https://raw.githubusercontent.com/Snowflake-Labs/sfquickstarts/master/site/sfguides/src/<slug>/<slug>.md`
  4. The markdown uses `## Step N Title` headers (H2) as section boundaries, with `Duration: N` lines for timing
  5. Also check for companion files in the same directory (SQL scripts, images, etc.)

- **GitHub** (`github.com`): Fetch raw README via `raw.githubusercontent.com/<owner>/<repo>/<branch>/README.md`. Also check for subdirectories with additional markdown/SQL files.

- **GitLab** (`gitlab.com` or self-hosted): Fetch raw README via `<repo-url>/-/raw/<branch>/README.md`. Also check for subdirectories.

- **Local path**: Read files directly.

Use `web_fetch` for URLs. Traverse the repo tree to find all `.md`, `.sql`, and `.py` files that form the lab content.

### Step 3: Route to Sub-Skill

**If TRIAGE:** Load `triage/SKILL.md` — analyze content, classify sections, present recommendation.

**If BUILD:** Load `build-course/SKILL.md` — scaffold project, extract lessons, add gamification, deploy.

**If TRIAGE then BUILD:** After triage approval, automatically continue to `build-course/SKILL.md` with the approved lesson plan.

## Stopping Points

- After Step 1: Confirm URL and intent
- After triage: Approve recommendation before building
- After build: Verify locally before deploying

## Output

- **Triage mode**: Recommendation table (which sections become interactive lessons vs stay as lab)
- **Build mode**: Complete Next.js course app deployed to SPCS with gamification
