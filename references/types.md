# TypeScript Type Definitions

These are the core interfaces for the course app. Write to `src/lib/types.ts`.

```typescript
export interface Lesson {
  id: number;
  title: string;
  type: "concept" | "sql_fill" | "sql_write" | "scenario";
}

export interface LessonContent {
  id: number;
  title: string;
  type: Lesson["type"];
  customerScenario: string;
  sections: ContentSection[];
  exercises: Exercise[];
}

export interface ContentSection {
  title?: string;
  body?: string; // markdown-like text with **bold**, *italic*, `code`, [links](url)
  columns?: { left: string; right: string }; // side-by-side layout
}

export interface Exercise {
  id: string;
  type: "radio" | "text_input" | "sql_write" | "text_area";
  question: string;
  options?: string[]; // for radio type
  correctAnswer: string | string[]; // string for radio/text, string[] for sql keyword checks
  hint?: string;
  hintDocLink?: string;
  feedback: {
    correct: string;
    incorrect: string;
  };
}

export interface QuizState {
  answers: Record<string, string>;
  score: number | null;
  passed: boolean;
  attempts: number;
}

export interface Progress {
  lessonId: number;
  lessonName: string;
  completed: boolean;
  score: number;
  maxScore: number;
  attempts: number;
}

export interface SurveyAnswers {
  q1: number;
  q2: number;
  q3: number;
  q4: number;
}
```

## Exercise Type Guidelines

| Type | Use When | correctAnswer Format |
|------|----------|---------------------|
| `radio` | Multiple choice, 2-4 options | Single string matching the correct option |
| `text_input` | Short answer (one word/phrase) | Single string (case-insensitive match) |
| `sql_write` | SQL writing with keyword validation | `string[]` of required keywords |
| `text_area` | Longer freeform responses | `string[]` of required keywords |

## SQL Keyword Matching

For `sql_write` and `text_area` exercises, `correctAnswer` is an array of keywords. The validation logic:
- Convert user input to uppercase
- Check that every keyword in the array appears in the user's input
- Order doesn't matter — just presence of all keywords
