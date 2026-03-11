# Gamification Reference

DataCamp/Duolingo-inspired gamification system. All CSS-only animations — no third-party libraries.

## Points System

| Attempt | Points Per Exercise |
|---------|-------------------|
| 1st try | 100 pts |
| 2nd try | 50 pts |
| 3rd+ | 25 pts |

**Points are derived, not stored.** Calculate from `quizStates` via `useMemo`:

```typescript
const points = useMemo(() => {
  let total = 0;
  for (const lesson of LESSONS) {
    const qs = quizStates[lesson.id];
    if (qs?.passed) {
      const ptsPerExercise = qs.attempts === 1 ? 100 : qs.attempts === 2 ? 50 : 25;
      total += lesson.exercises.length * ptsPerExercise;
    }
  }
  return total;
}, [quizStates]);
```

## Streak Tracking

Track consecutive first-try passes:
```typescript
const [streak, setStreak] = useState(0);
// In handleQuizSubmit when allCorrect:
if (newAttempts === 1) { setStreak(s => s + 1); } else { setStreak(0); }
```

Display fire icon in PointsBadge when streak >= 2.

## Celebration Triggers

On lesson pass, trigger celebration overlay:
```typescript
const celebType = newAttempts === 1 ? "perfect" : newAttempts === 2 ? "good" : "ok";
setCelebration({ type: celebType, points: earned });
```

Auto-dismiss after 2 seconds. If all lessons + post-survey complete, show CompletionPage after 2.5s delay.

## PointsBadge Component

Header bar showing:
- Star icon (SVG) + total points with tabular-nums font
- Animated bump (scale 1 → 1.3 → 0.95 → 1) on point change
- Fire emoji when streak >= 2

Position: fixed header bar, right-aligned above lesson content.

## CelebrationOverlay Component

Full-screen overlay (fixed inset-0, z-50) with three variants:

**Perfect (1st try):**
- 12 confetti particles in brand colors (star, orange, purple, pink)
- Random positions, sizes, animation delays
- Large "Perfect!" text with point award

**Good (2nd try):**
- Green checkmark circle (SVG)
- "Great job!" text

**OK (3rd+):**
- Thumbs-up emoji
- "Got it!" text

All use `animate-confetti` for particles, fade-in for text. Background: semi-transparent dark overlay.

## CompletionPage Component

Full-screen summary shown after all lessons + post-survey:

**Stats grid (2x2):**
- Total points (with star icon)
- Perfect scores count
- Total attempts
- Completion percentage

**Per-lesson breakdown table:**
| Lesson | Attempts | Points |
|--------|----------|--------|

**SVG completion badge:**
- Hexagonal or circular badge shape
- Course name text
- Brand gradient background
- `animate-badge-reveal` entrance animation

**Actions:** "View Lessons" to go back, "Start Over" to reset.

## Inline Point Awards (QuizCard)

When an exercise is answered correctly:
```tsx
<span className="ml-auto text-xs font-bold text-sf-orange bg-sf-orange/10 px-2 py-0.5 rounded-full animate-float-up">
  +{attempts === 1 ? 100 : attempts === 2 ? 50 : 25}
</span>
```

Total points in success banner:
```tsx
<span className="ml-auto font-bold text-sf-orange text-sm">
  +{exercises.length * (attempts === 1 ? 100 : attempts === 2 ? 50 : 25)} pts
</span>
```

## Sidebar Points Display

Below the progress bar when points > 0:
```tsx
{points > 0 && (
  <div className="flex items-center gap-1.5 mt-2">
    <svg className="w-3.5 h-3.5 text-sf-orange" viewBox="0 0 24 24" fill="currentColor">
      <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
    </svg>
    <span className="text-xs font-bold text-sf-orange tabular-nums">{points} pts</span>
  </div>
)}
```
