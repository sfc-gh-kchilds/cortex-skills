# Snowflake 2026 Branding Reference

## Brand Colors

| Name | Hex | CSS Variable | Usage |
|------|-----|-------------|-------|
| Snowflake Blue | #29B5E8 | `--color-sf-blue` | Primary brand, links |
| Star Blue | #75CDD7 | `--color-sf-star` | Accents, highlights |
| Valencia Orange | #FF9F36 | `--color-sf-orange` | Points, awards, CTAs |
| Purple Moon | #7254A3 | `--color-sf-purple` | Accent, confetti |
| First Light | #D45B90 | `--color-sf-pink` | Accent, confetti |
| Mid-Blue | #11567F | `--color-sf-mid` | Dark accents |
| Dark Navy | #1B2A4A | `--color-sf-dark` | Text on light bg |
| Deep Navy | #0F1B2D | `--color-sf-navy` | Darkest backgrounds |
| Light Blue | #E8F4FD | `--color-sf-light` | Light backgrounds |
| Accent Purple | #6D5CE7 | `--color-sf-accent` | Secondary accent |
| Success Green | #2ECC71 | `--color-sf-success` | Correct answers |
| Warning Yellow | #F39C12 | `--color-sf-warning` | Caution states |
| Error Red | #E74C3C | `--color-sf-error` | Incorrect answers |

## globals.css

```css
@import "tailwindcss";
@plugin "daisyui" {
  themes: corporate --default, dark;
}

@theme {
  --color-sf-blue: #29b5e8;
  --color-sf-dark: #1b2a4a;
  --color-sf-navy: #0f1b2d;
  --color-sf-light: #e8f4fd;
  --color-sf-accent: #6d5ce7;
  --color-sf-success: #2ecc71;
  --color-sf-warning: #f39c12;
  --color-sf-error: #e74c3c;
  --color-sf-star: #75cdd7;
  --color-sf-orange: #ff9f36;
  --color-sf-purple: #7254a3;
  --color-sf-pink: #d45b90;
  --color-sf-mid: #11567f;
}

/* Smooth transitions */
main { scroll-behavior: smooth; }
.card { transition: box-shadow 0.2s ease, transform 0.2s ease; }
.card:hover { box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08); }
pre { border: 1px solid oklch(var(--b3)); }
circle { transition: stroke-dashoffset 0.7s ease-out; }
label:has(input[type="radio"]):hover { background-color: oklch(var(--b2)); }
aside nav::-webkit-scrollbar { width: 4px; }
aside nav::-webkit-scrollbar-thumb { background: oklch(var(--b3)); border-radius: 2px; }

/* Gamification animations */
@keyframes point-bump {
  0% { transform: scale(1); }
  30% { transform: scale(1.3); }
  60% { transform: scale(0.95); }
  100% { transform: scale(1); }
}
@keyframes float-up {
  0% { opacity: 1; transform: translateY(0); }
  100% { opacity: 0; transform: translateY(-40px); }
}
@keyframes confetti-pop {
  0% { transform: scale(0) rotate(0deg); opacity: 1; }
  50% { transform: scale(1.2) rotate(180deg); opacity: 1; }
  100% { transform: scale(0.8) rotate(360deg); opacity: 0; }
}
@keyframes slide-in-right {
  0% { transform: translateX(60px); opacity: 0; }
  100% { transform: translateX(0); opacity: 1; }
}
@keyframes badge-reveal {
  0% { transform: scale(0) rotate(-10deg); opacity: 0; }
  60% { transform: scale(1.1) rotate(2deg); opacity: 1; }
  100% { transform: scale(1) rotate(0deg); opacity: 1; }
}
.animate-point-bump { animation: point-bump 0.4s ease-out; }
.animate-float-up { animation: float-up 1.2s ease-out forwards; }
.animate-confetti { animation: confetti-pop 0.8s ease-out forwards; }
.animate-slide-in { animation: slide-in-right 0.3s ease-out; }
.animate-badge-reveal { animation: badge-reveal 0.6s ease-out forwards; }
```

## DaisyUI Theme

Use `corporate` as the default theme (clean, professional). Set via:
```html
<html lang="en" data-theme="corporate">
```

DaisyUI 5 classes used throughout:
- `bg-base-100`, `bg-base-200`, `bg-base-300` — background levels
- `text-base-content` — primary text
- `btn btn-primary`, `btn btn-ghost` — buttons
- `card`, `card-body` — content cards
- `badge` — status indicators
- `progress` — progress bars
- `range` — slider inputs (surveys)
- `alert` — feedback messages

## Design Patterns

- Cards with `rounded-xl shadow-sm border border-base-200`
- Success state: `bg-success/10 border-success/30 text-success`
- Error state: `bg-error/10 border-error/30 text-error`
- Points: `text-sf-orange` with `tabular-nums` for aligned numbers
- Code blocks: `bg-base-200 rounded-lg p-4 text-xs font-mono`
- Sidebar: `w-72 bg-base-200 border-r border-base-300`
