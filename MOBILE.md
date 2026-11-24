# Mobile app rules (clariva-hq)

Every mobile app in clariva-hq follows these rules, whatever the stack (React Native/Expo, SwiftUI, Kotlin).
They come from the owner's reviews of real screens. When a review finds a new problem worth a rule, add it here
through a PR, so every app and every future agent picks it up. Adopted 2026-11-24.

Agents: read this file before writing UI code in any app repo. Each app repo links it from its `AGENTS.md`.

## 1. No magic numbers in UI code
- No fixed font sizes, paddings, margins, gaps, corner radii, widths, heights or offsets written as numbers in
  screens and components.
- Sizes are **computed** from:
  - the user's text size (spacing grows with the text);
  - the screen or window size (readable widths, tablets, landscape);
  - the measured layout, e.g. the keyboard offset is measured, never "100 pt". A guessed offset left a large gap
    above the keyboard on iPad.
- The few base values a design system needs (the platform's default text sizes, named ratios) live in **one
  theme module** per app, each with a comment. A test or lint rule fails on numeric sizes anywhere else.

## 2. Dynamic Type everywhere
- Text follows the user's system text size: iOS Dynamic Type, Android font scale. That includes the largest
  accessibility sizes.
- Never disable or cap font scaling (`allowFontScaling={false}`, `maxFontSizeMultiplier`, fixed `.font(.system(size:))`).
- Use the platform's text styles (body, headline, title, footnote…), not raw point sizes.
- Layouts must still work at the largest sizes: text wraps, nothing is cut off, controls stay reachable.

## 3. Every screen size, both platforms
- Layouts work on all iPhone, iPad and Android phone and tablet sizes, in portrait and landscape.
- Long text is capped at a readable line length on wide screens, not stretched edge to edge.
- Respect safe areas (notches, home indicator, Android system bars) and the on-screen keyboard.
- iOS and Android are both first-class: test rounds alternate between them.

## 4. Accessibility basics
- Every button and tappable item has an accessible label and role; alerts and errors are announced.
- Colours come from the theme, with light and dark variants and readable contrast.

## 5. Platform rules from earlier decisions
- **White-label:** no brand-specific text, colours or ids in code; they come from the brand config.
- **Nothing secret in an app:** anything in the binary is public. Apps use limited publishable keys or short-lived
  tokens, never a secret (`sk_`) key.

## How each stack does it

| Rule | React Native / Expo | SwiftUI |
|---|---|---|
| Sizes from one theme | `useTheme()`: `t.space(em)`, `t.lines(n)`, `t.contentWidth`; base values only in `src/theme` | `@ScaledMetric` for spacing; a theme type for the rest |
| Dynamic Type | `<AppText variant>` (iOS `dynamicTypeRamp`, Android font scale) | `.font(.body)` and other text styles; no fixed sizes |
| Measured layout | `onLayout`, `measureInWindow`, `useWindowDimensions` | `GeometryReader`, `ViewThatFits`, safe-area insets |
| Enforcement | a test that fails on numeric sizes outside the theme | SwiftLint custom rule (to add) |

## Changing these rules
Open a PR here. Say what changes and why, and which apps need updating.
