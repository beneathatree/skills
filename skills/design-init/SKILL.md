---
name: design-init
version: 1.0.0
description: >
  Interview-driven design system establishment. Conducts a guided conversation about your
  product's soul, audience, and constraints, then produces DESIGN.md (Google-spec tokens),
  PRODUCT.md (product identity), and a preview HTML style tile. Runs once per product.
  Every subsequent /mock call reads these artifacts as ground truth.
agents:
  - pi
  - claude-code
  - cursor
  - codex
  - copilot
  - gemini-cli
  - windsurf
  - zed
  - roo-code
  - trae
category: design
tags:
  - ui
  - ux
  - design-system
  - frontend
  - product-design
  - mockup
requires:
  - none
produces:
  - DESIGN.md
  - PRODUCT.md
  - previews/token-preview.html
---

# `/design-init` — Design System Establishment

> **Run once per product.** This skill captures your product's soul in a design system that every subsequent `/mock` call reads as ground truth. Don't repeat it unless your product fundamentally changes direction.

---

## What This Skill Does

1. **Conducts an interview** — a focused back-and-forth (8–12 questions) about what your product *is*, who it serves, how it should feel, and what it must avoid.
2. **Produces `DESIGN.md`** — a Google-spec design system file with typed tokens (colors, typography, spacing, elevation, shapes, components) and rationale.
3. **Produces `PRODUCT.md`** — your product's persistent identity: soul, audience, posture, anti-references.
4. **Produces a preview HTML** — a style tile + rough composition page you can open in a browser to validate tokens before committing.

---

## Output Rules

- **No emoji in any output** -- conversation responses, artifact files, preview HTML comments, commit messages. Plain text only.
- **Exception:** The user explicitly asks for emoji, or the product's brand voice (established in PRODUCT.md) calls for it.
- **Tone:** Professional, direct, concise. Not stiff, not chatty.

## Phase 1: The Interview

This is **not a form**. It's a conversation. Probe where answers change the design; infer what you can. If the user says "internal tool for data analysts," don't ask about brand personality — ask about information density and scan patterns.

### Question Flow (Adaptive)

Ask these in order. Skip or merge questions when the user's earlier answers already cover the territory. **Target: 8–12 focused questions total.**

#### Round 1 — Product Soul (2–4 questions)

**Q1. What is this product, in one sentence?**
> Not a feature list. The *thing* it does in the world.
> Good: "A real-time incident management tool for on-call SRE teams."
> Bad: "A dashboard with alerts, metrics, and runbooks."

Probe deeper if needed:
- "What problem does it solve that existing tools don't?"
- "If this product disappeared tomorrow, what would its users miss most?"

**Q2. Who uses this, and in what mental state?**
> Situations and cognitive posture, not demographics.
> Good: "An on-call SRE at 3am, one hand on coffee, scanning fast for anomalies."
> Bad: "Male, 25–34, tech-savvy."

Probe deeper if needed:
- "Are they stressed, bored, focused, distracted?"
- "How much time do they spend here per session? Seconds? Minutes? Hours?"

**Q3. How should using this product *feel*?**
> 3–5 adjectives describing cognitive posture, not decoration.
> Examples: calm, clinical, dense, airy, surgical, warm, quiet, precise, fast, reassuring.

Probe deeper if needed:
- "Should it feel like a cockpit (dense, instrument-panel) or a living room (airy, spacious)?"
- "When something goes wrong, should the UI feel urgent or composed?"

**Q4. What existing products feel close to what you want? And what should this absolutely NOT look like?**
> References and anti-references. These are gold — they constrain generation more than any positive instruction.
> Good: "Like Linear's restraint. NOT like a Stripe dashboard clone. NOT pastel baby colors."

#### Round 2 — Constraints (2–4 questions)

**Q5. What platform(s) does this target?**
> Web (desktop-first), web (mobile-first), native mobile, desktop app, all of the above?

**Q6. How information-dense are the screens?**
> Data-heavy dashboards need tight spacing and small type. Editorial/content pages need breathing room.
> Options to calibrate: cockpit (max density), workspace (moderate), editorial (generous whitespace).

**Q7. Are there accessibility requirements beyond WCAG AA?**
> Keyboard-only workflows? Screen reader patterns? High-contrast mode? Motion sensitivity?

**Q8. What tech stack will implement this?**
> HTML + CSS only? A framework (React, Vue, Svelte)? Tailwind? Design tokens output format preference?

#### Round 3 — Content & Edge Cases (2–3 questions)

**Q9. What kind of content lives in these screens?**
> Data tables? Forms? Charts? Text articles? Media galleries? User-generated content?
> This determines component priorities.

**Q10. What are the trickiest states or edge cases your users encounter?**
> Empty states, error states, loading states, zero-data scenarios, overflow scenarios.
> Good designers design for these explicitly.

**Q11. Is there anything you feel strongly about that I haven't asked?**
> Open floor. Often reveals hidden constraints or strong opinions.

### Interview Behavior Rules

- **Listen more than you ask.** One good follow-up question is worth three scripted ones.
- **Don't ask about things you can infer.** If they said "internal tool," assume enterprise constraints unless told otherwise.
- **Respect terse users.** Some people give one-word answers and expect you to be smart. Be smart.
- **Never ask about surface aesthetics** ("what color do you like?") — that's what the token system produces from their emotional-register answers.
- **Summarize before producing.** Before writing DESIGN.md, read back your understanding: "Here's what I heard: [summary]. Does that feel right?" This catches misunderstandings at the cheapest stage.

---

## Phase 2: Produce Artifacts

After the interview (and summary confirmation), produce three files in the project root:

### Artifact 1: `DESIGN.md`

Follows the **[Google DESIGN.md specification](https://github.com/google-labs-code/design.md)** exactly. This format is machine-parseable, lintable (`npx @google/design.md lint`), and consumed by Google Stitch, Impeccable, and other tools.

#### Required Structure

```markdown
---
# YAML FRONTMATTER — Typed Design Tokens

colors:
  ink: "#1a1a1a"            # Primary text
  paper: "#fafaf7"           # Background
  accent: "#3a5a40"          # Primary actions, links
  accent-hover: "#2d4732"
  subtle: "#6b7280"         # Secondary text
  border: "#e5e7eb"
  success: "#059669"
  warning: "#d97706"
  error: "#dc2626"
  # Add semantic roles as needed (info, muted, etc.)

typography:
  heading:
    fontFamily: "...", sans-serif
    fontSize: "1.75rem"
    fontWeight: 700
    lineHeight: 1.2
    letterSpacing: "-0.02em"
  body:
    fontFamily: "...", sans-serif
    fontSize: "1rem"
    fontWeight: 400
    lineHeight: 1.6
  label:
    fontSize: "0.875rem"
    fontWeight: 500
    letterSpacing: "0.01em"
  caption:
    fontSize: "0.75rem"
    fontWeight: 400
    lineHeight: 1.4

spacing:
  xs: "0.25rem"   # 4px
  sm: "0.5rem"    # 8px
  md: "1rem"      # 16px
  lg: "1.5rem"    # 24px
  xl: "2rem"      # 32px
  2xl: "3rem"     # 48px

elevation:
  # Either shadow-based or flat (border/color) strategy
  strategy: "shadow"  # or "flat" or "border"
  levels:
    - name: "resting"
      value: "0 1px 2px rgba(0,0,0,0.05)"
    - name: "raised"
      value: "0 4px 12px rgba(0,0,0,0.08)"
    - name: "floating"
      value: "0 12px 32px rgba(0,0,0,0.12)"

radii:
  sm: "4px"
  md: "8px"
  lg: "12px"
  full: "9999px"

components:
  button_primary:
    background: "{colors.accent}"
    color: "{colors.paper}"
    padding: "{spacing.sm} {spacing.lg}"
    radius: "{radii.md}"
    fontWeight: 600
  button_secondary:
    background: "transparent"
    color: "{colors.accent}"
    border: "1px solid {colors.border}"
    # ... define key component styles via token references
---

# MARKDOWN BODY — 8 Sections in Fixed Order

## 1. Overview
Brand personality, emotional register, target audience, design philosophy.
Why these choices exist.

## 2. Colors
Full palette with semantic roles. Hex values, usage rules,
accessibility contrast ratios noted.

## 3. Typography
Type scale with roles (heading, body, label, caption).
Font pairing rationale. Size ratios (must be ≥1.25 between steps).

## 4. Layout
Grid system, max-width constraints, spacing rhythm,
responsive breakpoints, density strategy.

## 5. Elevation & Depth
Shadow strategy or flat alternative. When to use which level.
How layering communicates hierarchy.

## 6. Shapes
Corner radii scale. Shape language consistency.
When to use sharp vs rounded.

## 7. Components
Key component styles (buttons, inputs, cards, badges, alerts)
defined via token references ({path.to.token} syntax).

## 8. Do's and Don'ts
Explicit guardrails. Anti-patterns specific to this product.
What makes this design system *yours* vs generic.
```

#### Token Rules

- **All visual values must be tokens.** No magic numbers anywhere in the body either.
- **Type scale ratio ≥1.25** between adjacent steps (Refactoring UI rule). No flat hierarchies.
- **Contrast ratios must meet WCAG AA** minimum (4.5:1 for normal text, 3:1 for large text). Note them.
- **Use `{token.reference}` syntax** in component definitions so they stay linked.
- **Choose fonts intentionally.** Avoid the overused defaults (Inter, Roboto, Fraunces, Geist, Plus Jakarta Sans, Space Grotesk) unless there's a deliberate reason. Pair a display face with a body face — never one font for everything.

### Artifact 2: `PRODUCT.md`

Captures the product's **soul** — identity, audience, posture, anti-references. This is the persistent context every `/mock` call reads. JTBD and pain points live in `/mock` briefs (per-screen), not here.

```markdown
# PRODUCT.md

## What This Is
One sentence. The product's reason for existing.
The *thing*, not a feature list or pitch.

## Who It's For
Situations and cognitive posture, not demographics.
Describe the user's state of mind when they open this product.

## How It Should Feel
3–5 attributes describing the UI's cognitive posture.
Each attribute gets a one-line explanation of what it means visually.

## What It's Not (Anti-References)
Explicit rejections. "NOT X. NOT Y."
These are guardrails — they exclude default behaviors.

## What Good Looks Like (References)
Products, designs, or aesthetics to emulate.
Be specific enough to steer decisions.

## Constraints
- Platform: ...
- Density: ...
- Accessibility: ...
- Tech stack: ...

## Design Mode
Product. Design serves the task.
```

### Artifact 3: Preview HTML (`previews/token-preview.html`)

A single self-contained HTML file (inline styles, no external dependencies except optional web fonts) with two sections:

**Section A — Style Tile (top half):**
- Full color palette swatches with hex labels
- Typography specimens at every scale level (heading through caption)
- Button states: primary, secondary, ghost, disabled, hover (noted via comment)
- Input field examples (default, focus, error)
- Card example (with elevation)
- Spacing scale visualization
- Badge/tag variants
- Alert/notification examples (success, warning, error, info)

**Section B — Rough Composition (bottom half):**
- A representative page layout using the tokens
- Placeholder-but-realistic content matching the product domain
- Enough density to judge whether the palette and type work at actual scale
- Should look like a plausible screen from this product, not a generic template

**HTML quality rules for the preview:**
- Semantic HTML (`<header>`, `<main>`, `<nav>`, `<section>`, `<button>`, `<input>`)
- Inline `<style>` block with CSS custom properties mapped from tokens
- Responsive (looks reasonable from 375px to 1440px viewport)
- No external CDN dependencies (except Google Fonts if a web font was chosen)
- Include `<!-- comments -->` indicating hover/focus/error/loading states

---

## Phase 3: Iterate Until It's Right

After producing all three artifacts:

1. **Tell the user** to open `previews/token-preview.html` in a browser.
2. **Wait for feedback.** Common responses and how to handle them:

| User says | You do |
|---|---|
| "Colors work" | Done. Confirm artifact locations. |
| "Accent too aggressive, try warmer" | Adjust accent color in DESIGN.md, update token references, re-render preview. |
| "Body text feels too small" | Increase body font size, adjust scale ratios (maintain ≥1.25), re-render. |
| "Needs more density" | Reduce spacing scale values, tighten line heights, re-render. |
| "Feels too generic" | Push further into anti-references. Make bolder token choices. Re-render. |
| "I hate the font" | Propose 2–3 alternatives with rationale. Pick one, update, re-render. |
| "Can you try [specific thing]?" | Do it. Update DESIGN.md + re-render. |

3. **Loop until the user approves.** Each revision updates DESIGN.md and regenerates the preview HTML. PRODUCT.md rarely changes after initial creation (it's identity, not styling).

4. **On approval:** Confirm the final artifact locations and remind the user they can now run `/mock` for any screen.

---

## Knowledge Base — Baked Into Every Generation

This skill carries design expertise. It's not a thin prompt wrapper — it encodes principles that shape every decision.

### A. Design Principles (Product Mode — Active Constraints)

1. **Function before form.** Every visual decision must reference a user task or hierarchy goal. If it doesn't help the user do the thing, it doesn't belong.
2. **Hierarchy is the master skill.** Most "bad design" is flat hierarchy, not ugliness. Squint test: can you rank the top 5 elements by importance in 3 seconds?
3. **Start with the feature, not the layout.** (Refactoring UI.) Design the primary interaction first, then arrange around it.
4. **De-emphasize, don't over-emphasize.** When the primary element isn't standing out, make competing elements quieter — don't add more weight to the primary.
5. **Conventions > novelty.** (Jakob's Law.) Users expect nav where nav goes, search where search goes. Deviate only with explicit justification.
6. **Content is the design.** Layout is shaped around real copy and real data. Never the inverse.
7. **Tokens are the only source of visual values.** No hardcoded hex, no magic pixels. If you need something, add it to DESIGN.md first.
8. **Density serves scanning.** In product mode, screens are tools. Users extract information or complete actions. Respect their time.

### B. UX Laws (Applied, Not Academic)

| Law | How it shapes DESIGN.md decisions |
|---|---|
| **Hick's** | Primary screens ≤7 top-level choices. If more, group/nest. Affects navigation token count. |
| **Fitts's** | Primary CTA ≥44px touch target. Destructive actions separated spatially. Affects button sizing tokens. |
| **Miller's** | Chunk info into ≤7 groups. Affects spacing/grouping tokens. |
| **Jakob's** | Follow conventions. Nav = top or left. Search = top-right or header. Deviate only with justification. |
| **Von Restorff** | Use distinctiveness sparingly — for the single most important action. One accent color, used judiciously. |
| **Tesler's** | Complexity lives somewhere. Don't hide it behind minimal UI; solve the right problem instead. |

### C. Gestalt Principles (Visual Grouping)

| Principle | Application in Tokens/Layout |
|---|---|
| **Proximity** | Related items closer than unrelated. Spacing scale must have clear steps (xs < sm < md < lg). |
| **Similarity** | Same-role items share treatment. Consistent component tokens enforce this. |
| **Common Region** | Containers only when grouping needs reinforcement. Don't wrap everything in cards/borders. |
| **Figure/Ground** | Primary content separable from background. Sufficient contrast between ink/paper tokens. |
| **Continuation** | Alignment guides the eye. Grid system with consistent gutters. |

### D. The Slop Catalog — Generation Constraints (Not Detection Rules)

These patterns are **avoided during production**, not checked afterward. The skill knows not to generate them.

**Visual slop:**
- No purple-to-blue gradients unless the user/product explicitly wants that palette
- No glassmorphism / backdrop-blur unless elevation strategy calls for blur-based layering
- No side-tab accent borders on cards (thick colored stripe on left edge)
- No rounded rectangles with generic drop shadows as the only shape language
- No gradient text on headings
- No sparklines as decoration (tiny charts conveying no real information)
- No modals by reflex — if content needs scrolling and columns, it's a page

**Typography slop:**
- No flat type hierarchy — font sizes MUST have ≥1.25 ratio between steps
- No icon tiles stacked above headings (universal AI feature-card template)
- No monospace as "developer/technical" shorthand (unless the product IS a code editor)
- No single font for everything — always pair display + body faces
- No overused defaults (Inter, Roboto, Fraunces, Geist, Plus Jakarta Sans, Space Grotesk) without explicit user choice AND awareness they're ubiquitous
- No all-caps body text — uppercase for short labels/headings only

**Layout slop:**
- No card-in-card-in-card nesting ("Cardocalypse") without clear hierarchy justification
- No copy-paste section layouts where every section uses identical template
- No massive icon containers larger than the content they introduce
- No hero→metrics→features→CTA→FAQ→footer formulaic page structure as default

**Content slop:**
- No redundant UX writing (label + sublabel + helper + hint all saying the same thing)
- No lorem ipsum — ever. Realistic placeholder copy matching the product domain
- No generic marketing fluff ("The best solution for your needs", "Welcome to our platform")
- No empty state that just says "No data" — empty states must be actionable

**Interaction slop:**
- No animation without purpose — motion is feedback, not decoration
- No bouncing/wiggling elements as attention-grabbers

### E. Refactoring UI Rules (Practical Application)

- **Design in grayscale first.** Establish hierarchy through size, weight, and spacing BEFORE applying color. (Apply mentally — output is colored, but hierarchy was proven without color.)
- **Add color deliberately.** Color has a job: identify primary actions, indicate state, provide brand recognition. It is never filler.
- **Use font weight for hierarchy.** Never below 400 for body text. Use 600–700 for emphasis. Never above 700 for body copy.
- **Align to a grid.** Don't center everything. Left-alignment creates stronger vertical scan lines.
- **Consistent spacing scale only.** Every margin and padding comes from DESIGN.md spacing tokens. No magic pixels.

### F. Nielsen's 10 Usability Heuristics (Quality Floor)

Every design decision must satisfy these at minimum:

1. **Visibility of status** — Keep users informed about what's happening
2. **Match to real world** — Speak the user's language, not system jargon
3. **User control freedom** — Easy undo/exit/abort
4. **Consistency standards** — Same words/situations → same behavior
5. **Error prevention** — Better than good error messages is careful design that prevents errors
6. **Recognition over recall** — Show options, don't make users remember
7. **Flexibility efficiency** — Shortcuts for experts without hiding basics from novices
8. **Minimalist aesthetic** — Nothing irrelevant. If it doesn't serve a task, cut it.
9. **Error recovery** — Error messages in plain language, specific, constructive
10. **Help documentation** — Contextual help available without leaving the task

---

## What This Skill Is NOT

- **Not a brand identity exercise.** It's optimized for product UI (dashboards, tools, interfaces). For marketing landing pages with bespoke illustration, it will produce competent but not exceptional work.
- **Not a component library generator.** Components in DESIGN.md are style definitions, not code packages.
- **Not a Figma replacement.** Medium is HTML viewed in a browser. No canvas, no vector editing.
- **Not a code generator.** Mocks are production-viable HTML scaffolds, but routing/state/data-fetching are separate concerns.
- **Not multi-step detection.** Slop prevention is baked into generation constraints, not a post-hoc checking pipeline.

---

## Relationship to `/mock`

`/design-init` **feeds** `/mock`. The workflow is intentionally split:

| | `/design-init` | `/mock` |
|---|---|---|
| **Runs** | Once per product | Many times per product |
| **Input** | Conversation with user | Brief + DESIGN.md + PRODUCT.md |
| **Output** | DESIGN.md, PRODUCT.md, preview HTML | 2–3 HTML iterations → frozen mock |
| **Focus** | Product soul, design system, identity | Specific screen, layout, content |
| **Changes when** | Product pivots | Every new screen or revision |

Running `/mock` without `/design-init` first will fail — there are no tokens or product context to ground the generation.
