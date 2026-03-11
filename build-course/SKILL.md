---
name: build-react-course
description: "Full course build workflow: scaffold Next.js project, extract lessons, add gamification, deploy to SPCS."
parent_skill: gitlab-to-react-course
---

# Build Interactive React Course

Generate a complete interactive course app from an approved lesson plan.

## When to Load

- After triage approval (with lesson plan), OR
- Directly when user says "build" and provides lab content

## Prerequisites

- Lab content fetched and parsed
- Lesson plan approved (from triage) or user-provided structure
- Node.js v20+ and Docker installed

## Workflow

### Step 1: Collect Parameters

**Ask user for:**
```
1. Course name (kebab-case): e.g., "open-format-sharing"
2. Course title (display): e.g., "Open Format Sharing"
3. Course description: One-liner for metadata
4. Snowflake connection name: For SPCS deployment
5. Target database.schema: Where course tables live
6. Compute pool name: SPCS compute pool
7. Project directory: Where to create the app (default: ~/<course-name>-react)
```

**STOP**: Confirm parameters before scaffolding.

### Step 2: Scaffold Project

**Load** `references/stack.md` for exact dependency versions.

1. Create Next.js project:
```bash
npx create-next-app@latest <project-dir> --typescript --tailwind --eslint --app --src-dir --import-alias="@/*" --turbopack
```

2. Install additional dependencies:
```bash
cd <project-dir>
npm install snowflake-sdk
npm install -D daisyui@5 @tailwindcss/postcss
```

3. Configure `next.config.ts`:
```typescript
import type { NextConfig } from "next";
const nextConfig: NextConfig = { output: "standalone" };
export default nextConfig;
```

4. Configure `postcss.config.mjs`:
```javascript
const config = { plugins: { "@tailwindcss/postcss": {} } };
export default config;
```

5. **Load** `references/snowflake-branding.md` and write `globals.css` with Snowflake 2026 brand colors, DaisyUI corporate theme, and gamification animations.

6. Write `src/app/layout.tsx` with course title, corporate theme, base-100 background.

### Step 3: Generate Core Files

**Load** `references/types.md` and write `src/lib/types.ts` — the TypeScript interfaces.

**Load** `references/stack.md` and write `src/lib/snowflake.ts` — Snowflake connection with SPCS OAuth token detection. Key pattern: do NOT set `accessUrl` when running in SPCS (detect via `/snowflake/session/token` file existence).

Write `Dockerfile` — multi-stage node:22-alpine build with standalone output on port 8080.

Write `.dockerignore` — exclude node_modules, .next, .git, .env.

### Step 4: Extract Lessons

This is the core content generation step. For each approved lesson:

1. **Create the lesson object** in `src/lib/lessons.ts` following the `LessonContent` interface:
   - `id`: Sequential number (1, 2, 3...)
   - `title`: From the approved plan
   - `type`: "concept" | "sql_fill" | "sql_write" | "scenario"
   - `customerScenario`: A realistic customer quote that frames the lesson
   - `sections`: Array of `ContentSection` objects extracted from the lab markdown
   - `exercises`: Array of `Exercise` objects

2. **For each exercise**, determine:
   - `type`: "radio" (multiple choice), "text_input" (short answer), "sql_write" (freeform SQL), "text_area" (long answer)
   - `question`: Clear prompt — for sql_fill/sql_write, reference blanks in the template
   - `correctAnswer`: String for radio/text, string[] of keywords for SQL
   - `hint`: Guiding text that helps WITHOUT giving the answer
   - `hintDocLink`: Link to relevant Snowflake documentation
   - `feedback.correct`: Explains WHY the answer is right and reinforces the learning
   - `feedback.incorrect`: Nudges toward the right direction

3. **Exercise design rules:**
   - Hints NEVER contain the full answer — guide with questions and partial info
   - SQL templates use blank underscores (`____________`) not TODO placeholders
   - Questions reference "fill in the blanks" not "replace the TODOs"
   - Each lesson should have 1-3 exercises, mixing types when possible
   - Correct feedback should add insight beyond just "correct"

Export as `const LESSONS: LessonContent[] = [...]`.

### Step 5: Build Components

Generate these components in `src/components/`:

**Sidebar.tsx** — Left navigation panel:
- Course title and progress bar (completed/total lessons)
- Points display (star icon + pts count, visible when > 0)
- Clickable lesson list with completion checkmarks
- Lock uncompleted lessons (sequential progression)

**LessonContent.tsx** — Content renderer:
- Renders markdown sections (bold, italic, code, links)
- Supports two-column layouts via `columns` property
- Export `formatInline()` for reuse in other components

**QuizCard.tsx** — Exercise renderer:
- Renders exercise question with `formatInline` for markdown
- Radio buttons for multiple choice, text inputs for fill-in, textarea for SQL
- SQL template display (pre-formatted code blocks with blanks)
- Submit button with attempt tracking
- Per-exercise feedback (correct/incorrect) with inline point awards
- Total points banner on lesson pass

**HintAccordion.tsx** — Collapsible hint with doc link:
- Show/Hide toggle
- Hint text with optional documentation link button

**SurveyForm.tsx** — Pre/post confidence survey:
- 4 questions with 1-5 range sliders
- Labels aligned to DaisyUI range thumb positions (px-2 padding)

**Load** `references/gamification.md` and generate:

**PointsBadge.tsx** — Header points counter:
- Animated bump on point gain
- Streak fire indicator (consecutive first-try passes)
- Fixed position in header bar

**CelebrationOverlay.tsx** — Full-screen celebration:
- Perfect (1st try): Confetti particles + "Perfect!" text
- Good (2nd try): Checkmark + "Great job!" text
- OK (3rd+): Thumbs-up + "Got it!" text
- Auto-dismiss after 2 seconds
- CSS-only animations (no third-party libraries)

**CompletionPage.tsx** — End-of-course summary:
- Stats grid: total points, perfect scores, total attempts, completion percentage
- Per-lesson breakdown table (lesson name, attempts, points earned)
- SVG completion badge with course name
- "Start Over" option

### Step 6: Wire Up Main Page

Generate `src/app/page.tsx` — the main SPA orchestrator:

**State management:**
- Current lesson, completed lessons set, quiz states, survey states
- Points derived via `useMemo` from quizStates (no separate state)
- Streak counter, celebration state, completion page toggle

**Points formula:**
- 100 pts per exercise on 1st attempt
- 50 pts per exercise on 2nd attempt
- 25 pts per exercise on 3rd+ attempt

**Flow:**
1. Identity detection (SPCS user header or prompt)
2. Pre-survey → Lessons (sequential) → Post-survey → Completion
3. Progress saved to Snowflake tables via API routes

**API routes** (`src/app/api/`):
- `user/route.ts` — Get/create user identity
- `progress/route.ts` — Save/load lesson completion
- `attempts/route.ts` — Log quiz attempts
- `survey/route.ts` — Save survey responses

### Step 7: Create Snowflake Tables

```sql
CREATE TABLE IF NOT EXISTS <DB>.<SCHEMA>.COURSE_PROGRESS (
  USER_EMAIL VARCHAR, LESSON_ID NUMBER, COMPLETED BOOLEAN DEFAULT FALSE,
  SCORE NUMBER DEFAULT 0, ATTEMPTS NUMBER DEFAULT 0,
  COMPLETED_AT TIMESTAMP_NTZ, CREATED_AT TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP()
);
CREATE TABLE IF NOT EXISTS <DB>.<SCHEMA>.COURSE_ATTEMPT_LOG (
  USER_EMAIL VARCHAR, LESSON_ID NUMBER, ATTEMPT_NUMBER NUMBER,
  ANSWERS VARIANT, PASSED BOOLEAN, CREATED_AT TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP()
);
CREATE TABLE IF NOT EXISTS <DB>.<SCHEMA>.COURSE_CONFIDENCE_SURVEY (
  USER_EMAIL VARCHAR, SURVEY_TYPE VARCHAR, Q1 NUMBER, Q2 NUMBER, Q3 NUMBER, Q4 NUMBER,
  CREATED_AT TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP()
);
```

Grant SELECT, INSERT, UPDATE to the service-owning role.

### Step 8: Test Locally

```bash
npm run dev    # Verify at http://localhost:3000
npm run build  # Must compile with zero TypeScript errors
```

**STOP**: Ask user to verify the app locally before deploying.

### Step 9: Deploy to SPCS

**Load** `references/spcs-deploy.md` for the exact deployment workflow.

1. Docker login to Snowflake registry
2. `docker build --platform linux/amd64 -t <image-tag>:latest .`
3. `docker push <image-tag>:latest`
4. `CREATE SERVICE` (first deploy) or `ALTER SERVICE` (update — preserves URL)
5. `GRANT SERVICE ROLE ... TO ROLE <consumer-role>`
6. Verify service status is READY

**STOP**: Confirm deployment success and share the endpoint URL.

## Stopping Points

- Step 1: Confirm parameters
- Step 4: Review generated lessons before building components
- Step 8: Local test before deployment
- Step 9: Deployment confirmation

## Output

- Complete Next.js course app with gamification
- Deployed to SPCS with public endpoint URL
- Snowflake tables for progress tracking
- Service role granted for consumer access
