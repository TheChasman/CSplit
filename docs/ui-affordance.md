# UI Affordance Mandate — Skeuomorphism Revival

> Load when: building any UI component, reviewing any frontend code.

---

## Core Principle

**If it's interactive, it must look interactive.**

Flat design stripped the visual cues that made interfaces self-documenting. We bring them back. No flat ambiguous tiles. No hidden tap targets. No guessing games.

---

## Non-Negotiable Rules

1. **Buttons must look like buttons** — visible border, shadow, or gradient. Hover/focus states must visibly change the element (darken, lift, or depress). Never rely on text colour alone.

2. **Tappable cards must signal interactivity** — always include at least one of: chevron (`›`), hover elevation change, or explicit hint text. Never present a card as flat/static if tapping it opens a detail view.

3. **Links must be distinguishable** — underline within body text, or distinct colour + underline-on-hover. Never colour alone.

4. **Form inputs must have visible boundaries** — borders or inset shadows always. Never borderless underline-only inputs. Focus states must be obvious.

5. **Toggles must show state clearly** — colour + position + secondary indicator. Never a subtle colour shift alone.

6. **Disabled elements must look disabled** — reduce opacity, desaturate, remove shadow. Never leave disabled identical to enabled.

---

## Standard Tailwind Patterns

| Element | Classes |
|---|---|
| Button (resting) | `border border-black/15 shadow-sm rounded-md` |
| Button (hover) | `hover:shadow-md hover:-translate-y-px active:shadow-sm active:translate-y-0` |
| Interactive card | `border border-black/10 shadow-sm rounded-lg cursor-pointer hover:shadow-lg hover:-translate-y-0.5 transition-all` |
| Input field | `border border-black/20 rounded-md shadow-inner focus:ring-2 focus:ring-primary/20 focus:border-primary` |
| Disabled | `opacity-50 saturate-50 shadow-none cursor-not-allowed` |

---

## Dark Mode

- Lighter top-edge highlights, darker bottom shadows
- Borders: `rgba(255, 255, 255, 0.08–0.1)` instead of dark borders
- Elevation communicated through lighter surface colours, not just shadow

---

## Enforcement Checklist

Before committing any UI component:
- [ ] Every interactive element — does it visually communicate interactivity?
- [ ] Test with fresh eyes — would someone unfamiliar know what's tappable?
- [ ] Default to over-communicating — heavier self-documenting beats "clean" needing a tutorial

**No exceptions. No "but flat looks modern." Flat looks broken.**
