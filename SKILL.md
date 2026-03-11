---
name: gitlab-to-react-course
description: "Convert GitLab or GitHub labs into interactive React course apps with Snowflake branding and gamification. Use when: user provides a GitLab/GitHub lab link to convert into a course, wants to build an interactive learning app, needs to assess which lab sections should become interactive vs stay as-is. Triggers: convert lab, build course, react course, interactive course, gamification, lab to app, gitlab lab, github lab, DataCamp style, SE enablement course."
---

# GitLab/GitHub Lab to Interactive React Course

Convert SE enablement labs hosted on GitLab or GitHub into polished, interactive React course apps with Snowflake 2026 branding, DataCamp-inspired gamification, and SPCS deployment.

## Intent Detection

| Intent | Triggers | Route |
|--------|----------|-------|
| TRIAGE | "analyze", "which parts", "should this be", "recommend", "assess", "review lab" | `triage/SKILL.md` |
| BUILD | "build", "create", "convert", "make interactive", "scaffold", "generate course" | `build-course/SKILL.md` |

If the user provides a link without specifying intent, default to **TRIAGE** first — analyze the lab content and recommend what to convert before building.

## Workflow

### Step 1: Collect Input

**Ask the user:**
1. **Lab URL**: GitLab or GitHub repository link (or local path)
2. **Intent**: Analyze & recommend, or build the full course?

### Step 2: Fetch Lab Content

**Detect the host from the URL:**

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
