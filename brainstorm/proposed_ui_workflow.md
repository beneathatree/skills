# Proposed UI Workflow — Agent-Driven Frontend Design

> A two-command workflow that captures a product's soul in a design system, then produces production-viable HTML mockups grounded in real design practice — not slop.

---

## Design Decisions (from interview)

| Decision | Choice | Rationale |
|---|---|---|
| Mode | **Product mode** — design serves the task | Primary output is app UI, dashboards, tooling. Clarity > decoration. |
| Token preview | **Style tile + rough composition** | Both the abstract system (tile) and how it feels at density (page). |
| Slop prevention | **Baked into generation** | Anti-pattern knowledge lives in the skill's instructions. Prevention, not detection. No separate check step. |
| Mock lifecycle | **Production starting point + optional feedback** | Frozen mocks scaffold real code. Patterns extracted into the skill only when genuinely useful. |

---

## The Workflow — Two Commands

```
/design-init                            /mock
     │                                     │
     ▼                                     ▼
 Conversation ──────► DESIGN.md       Brief ──► 2-3 HTML
 with the user        PRODUCT.md      +          iterations
                      Preview HTML    DESIGN.md     │
                                       PRODUCT.md    ▼
                                         │        User picks
                                         ▼        one → frozen
                                      (iteratable)     │
                                                        ▼
                                                   Production
                                                   starting point
                                                        │
                                                        ▼
                                                Pattern extraction
                                                (if warranted)
```

> **`/design-init` runs once per product.** It establishes the design system and product context that `/mock` reads on every subsequent call. You don't repeat it unless the product fundamentally changes direction.

### Command 1: `/design-init`

**Purpose:** Understand the product well enough to produce a coherent design system. This is what a senior product designer does in a kickoff — distilled into a guided conversation. **Runs once per product.**

**What happens:**

1. The skill conducts a conversation. Not a form. A real back-and-forth that covers:
   - **What this product *is*** — its soul, its reason for existing. Not feature lists — the *thing* it does in the world. "A real-time incident management tool for SRE teams" not "a dashboard with alerts and metrics."
   - **Who it serves** — not demographics, but *situations and cognitive posture*. "An on-call SRE at 3am, one hand on coffee, scanning fast." Not "male, 25-34, tech-savvy."
   - **The emotional register** — should this feel calm, clinical, dense, airy, surgical, warm? In product mode, this is about *cognitive posture*, not decoration. What state of mind should the UI match?
   - **What good looks like** — references and anti-references. "Like Linear's restraint" / "Not Stripe dashboard. Not a gamified habit tracker."
   - **Constraints** — platform (mobile/web), density (data-heavy vs editorial), accessibility requirements, tech stack.

2. The skill synthesizes the conversation into two artifacts (both live in the project root):
   - **`DESIGN.md`** — follows the [Google DESIGN.md spec](https://github.com/google-labs-code/design.md). YAML frontmatter with typed tokens (colors, typography, spacing, elevation, shapes, components). Markdown body with rationale for *why* those values exist. Sections in fixed order: Overview → Colors → Typography → Layout → Elevation & Depth → Shapes → Components → Do's and Don'ts.
   - **`PRODUCT.md`** — the product's identity: what it is, who it's for, how it should feel, what it's not. This is the persistent soul that every subsequent `/mock` call reads. (See PRODUCT.md format section below.)

3. The skill produces a **preview HTML** file — a single page with two sections:
   - **Style tile** at the top: color swatches, typography specimens at every scale level, button states, a card or two, spacing examples. Abstract — no real content.
   - **Rough composition** below: a representative page layout rendered with the tokens and placeholder-but-realistic content. Enough density to judge whether the palette and type work at actual scale.

**What the user does:** Opens the preview HTML in a browser. Says "colors work" or "accent is too aggressive, try warmer" or "the body text feels too small at this density." The skill revises DESIGN.md and re-renders until it's right.

**What's baked into the skill (the knowledge base):**
- The full DESIGN.md spec — the skill knows how to produce valid, lint-able token files.
- Product-mode design principles (see §Baked-In Knowledge below).
- The 37 slop anti-patterns from Impeccable, reframed as *constraints on generation* (not detection rules). The skill knows these patterns exist and avoids producing them.
- Research foundations: Nielsen heuristics, Gestalt principles, UX laws (Hick's, Fitts's, Miller's, Jakob's), Refactoring UI rules, Don Norman's interaction model.

**Interview depth:** Elaborate but not trivial. The skill should ask 8-12 focused questions, not 30. It should infer what it can and probe only where the answer changes the design. If the user says "internal tool for data analysts," the skill doesn't ask about brand personality — it asks about information density and scan patterns.

---

### Command 2: `/mock`

**Purpose:** Produce a specific screen or page, grounded in the design system, with real hierarchy and real content — not a decorative template.

**What the user provides:** A brief. Can be terse or detailed.

```
/mock "Dashboard for the habit tracker. Main view: today's habits in a compact list, 
       streak counts, a weekly mini-chart. User is scanning, not exploring."
```

```
/mock "Settings page for notification preferences. Toggle groups by channel 
       (email, push, SMS). User comes here to turn things OFF, not on."
```

```
/mock "User profile page"
```

Note how the first two briefs include the user's *job* and *posture* ("scanning, not exploring" / "turn things OFF, not on"). That's the JTBD + pain point context, living where it belongs — at the screen level, not in the product definition. If the brief is terse (third example), the skill infers from PRODUCT.md — **or interviews back** if the brief is too thin to infer from safely (see step 1.5 below).

**What happens:**

1. The skill reads `DESIGN.md` and `PRODUCT.md` from the project root. These are non-negotiable context — every token, every constraint, every anti-reference.

   **The brief IS the task-specific context.** This is where JTBD and pain points live — not in PRODUCT.md, but here, per mock. The user's brief says what the user is trying to accomplish on *this screen*, what goes wrong, what matters most. PRODUCT.md gives the skill the product's soul; the brief gives it the specific job.

1.5. **Interview-back (conditional — only when warranted).** After reading context + brief, the skill assesses whether the brief carries enough signal to produce meaningful iterations without guessing. If the brief is thin, ambiguous, or raises architectural questions that would materially change the output, the skill pauses generation and interviews the user *before* producing any HTML.

   **When interview-back kicks in:**
   - The brief is a bare noun phrase with no job/posture context (`"User profile page"`, `"Settings"`, `"API docs"`) and PRODUCT.md doesn't fill the gap sufficiently.
   - The brief describes a screen with multiple plausible interaction models and none is implied (e.g., `"Data table for logs"` — is this real-time streaming? Paginated? Filterable? Exportable? Each produces a fundamentally different layout).
   - The brief references concepts external to the product that aren't defined anywhere (e.g., `"Integrate with the CI pipeline view"` — what pipeline? What does integration mean here?).
   - The brief implies state or data flow decisions that haven't been specified (e.g., `"Multi-step wizard for onboarding"` — what are the steps? What can users go back and change? What's saved between steps?).

   **When interview-back does NOT kick in:**
   - The brief includes job, posture, and enough domain detail to form a coherent mental model — even if terse.
   - PRODUCT.md + DESIGN.md together provide enough context that reasonable inferences are safe.
   - The user explicitly said "just go" or otherwise signaled they want fast output over precision.

   **How the interview works:**
   - It's a short back-and-forth — **3-6 questions max**, not another `/design-init`. The goal is to unblock generation, not re-establish the product.
   - Questions target **architectural and product decisions**, never surface aesthetics. Examples of good questions:
     - *Data flow:* "Is this data read-only, or does the user mutate it here? If mutation, inline or via a detail drawer/modal?"
     - *State model: "Does this screen need to reflect live updates (WebSocket/polling), or is it snapshot-based? How stale is acceptable?"
     - *Interaction tradeoffs: "For the filter panel — do users typically filter by one dimension at a time (sequential), or do they build complex multi-dimension queries (compositional)? That changes whether we use faceted checkboxes or a query builder."
     - *Edge cases: "What happens when the list is empty versus when it has 10,000 items? Is pagination needed, or virtual scrolling, or does the dataset stay small?"
     - *Real-world usage: "Walk me through who looks at this screen and when. Is it someone deep in a task who needs density, or someone dropping in briefly who needs orientation?"
     - *System constraints: "Does this need to integrate with an existing page/layout (e.g., embed in a sidebar, fit inside a modal), or is it a full standalone page?"
     - *Accessibility: "Are there specific accessibility requirements beyond WCAG AA — keyboard-only navigation patterns, screen reader workflows, high-contrast mode support?"
   - Examples of questions the interview **avoids** (too surface, better answered by tokens or inference):
     - "What color should the primary button be?" → DESIGN.md already has this.
     - "Should it feel modern or classic?" → PRODUCT.md captures emotional register.
     - "Do you want cards or a table?" → This is the skill's job to decide and show as iteration variance.
   - The skill synthesizes answers into an enriched internal brief, then proceeds to step 2. No artifact is written — the enrichment lives only in the generation context for this mock call.

2. The skill produces **2-3 iterations** of a single-page HTML file. Each iteration is a *different approach*, not a random variation:
   - Different layout strategy (e.g., sidebar nav vs top nav, card grid vs table, tabs vs accordion).
   - Different density/whitespace tradeoffs within the token system.
   - Different information hierarchy emphasis — what's above the fold, what's secondary.

   Each iteration is a complete, self-contained HTML file. It renders in a browser. It uses only tokens from DESIGN.md. It includes real (or realistic) content shaped by PRODUCT.md.

3. **What the skill does NOT do (slop prevention baked in):**
   - No purple-blue gradients unless DESIGN.md explicitly specifies purple-blue.
   - No glassmorphism unless the elevation strategy calls for blur-based layering.
   - No side-tab accent borders on cards.
   - No card-in-card-in-card nesting without a hierarchy reason.
   - No Inter/Roboto as default — type comes from DESIGN.md tokens or the skill proposes something intentional.
   - No icon tiles stacked above headings (the universal AI feature-card template).
   - No sparklines as decoration.
   - No gradient text.
   - No monospace-as-"developer" shorthand.
   - No flat type hierarchy (sizes must have ≥1.25 ratio between steps — Refactoring UI).
   - No redundant UX writing (label, sublabel, helper, and hint all saying the same thing).
   - No lorem ipsum — ever. Real content from PRODUCT.md, or realistic placeholder that matches the domain.
   - No decoration without a task justification.

   These aren't checked after the fact. They're part of the skill's generation instructions. The model knows not to produce them the same way a trained designer knows not to.

4. **Product-mode generation rules (always active):**
   - **Start with the feature, not the layout.** (Refactoring UI.) What is the user doing on this screen? Design the primary interaction first, then arrange around it.
   - **Hierarchy is the master skill.** Every element has a clear rank. If you squint, you can still tell what matters most.
   - **Function over form.** Every visual choice must reference a user task or a hierarchy goal. If it doesn't help the user do the thing, it doesn't belong.
   - **Conventions over novelty.** (Jakob's Law.) Nav goes where nav goes. Search goes where search goes. Deviate only with explicit justification from the brief.
   - **Density serves scanning.** In product mode, screens are tools. Users are trying to extract information or complete an action, not have an experience.
   - **States are real.** Every interactive element: default, hover/focus, empty, error, loading. At minimum, the HTML should show the default state with comments or CSS classes indicating other states exist.
   - **Content is the design.** Layout is shaped around real copy and real data, never the inverse.
   - **De-emphasize rather than over-emphasize.** (Refactoring UI.) When the primary element isn't standing out enough, make competing elements quieter — don't make the primary louder.

5. The user reviews the iterations, picks one (or says "merge the layout of 2 with the density of 1"), and the chosen mock is **frozen**.

6. **Frozen mock → production scaffold.** The frozen HTML isn't throwaway reference. It's a starting point for real implementation — semantic HTML, accessible markup, clean token-based styling. The mock *is* the prototype. When it's time to build for real, developers work from the accepted HTML, not from a screenshot.

7. **Pattern extraction (optional, lazy).** Over time, if the user freezes 5+ mocks and clear patterns emerge ("this team always picks the dense table variant, never the card grid"), those patterns can be written back into the skill's knowledge. This is manual/judicious, not automatic. The principle is: update the skill only when it would change a future decision.

---

## File Structure in a Project

```
project/
├── DESIGN.md              # Design system (Google spec format)
├── PRODUCT.md             # Product soul — identity, audience, posture, anti-references
├── previews/
│   └── token-preview.html # Style tile + rough composition from /design-init
├── mocks/
│   ├── dashboard-v1.html  # Iteration 1
│   ├── dashboard-v2.html  # Iteration 2
│   └── dashboard-v3.html  # Iteration 3
└── accepted/
    └── dashboard.html     # Frozen mock → production scaffold
```

---

## DESIGN.md Format — Following Google Spec

The DESIGN.md file follows the [google-labs-code/design.md](https://github.com/google-labs-code/design.md) specification exactly. Key points:

- **YAML frontmatter** with typed design tokens: colors (`#hex`), typography (objects with `fontFamily`, `fontSize`, `fontWeight`, `lineHeight`, `letterSpacing`), spacing (dimensions in `px`/`rem`), rounded corners, and component definitions.
- **Token references** use `{path.to.token}` syntax (e.g., `{colors.accent}` inside component definitions).
- **Markdown body** with 8 sections in fixed order:
  1. Overview (brand personality, emotional register, target audience)
  2. Colors (palettes with semantic roles + rationale)
  3. Typography (scale levels with roles — headline, body, label, caption)
  4. Layout (grid/spacing strategy)
  5. Elevation & Depth (shadow strategy or flat alternatives)
  6. Shapes (corner radii, shape language)
  7. Components (buttons, inputs, cards — styled via token references)
  8. Do's and Don'ts (explicit guardrails)

**Why this format:** It's the emerging standard. Google Stitch, Impeccable, and other tools all consume it. It's both human-readable and machine-parseable. It has a CLI validator (`npx @google/design.md lint`). Using it means your design system is portable — it works with other tools in the ecosystem, not just ours.

---

## PRODUCT.md Format

PRODUCT.md captures the product's **soul** — its identity, purpose, and personality. It is not a task list, not a backlog, and not a spec. It's the persistent context that makes every mock feel like it belongs to the same product.

JTBD and pain points live at the screen level, in `/mock` briefs — because they change per screen. PRODUCT.md captures what *doesn't* change: what this thing is, who it's for, how it should feel.

```markdown
# PRODUCT.md

## What This Is
One sentence. The product's reason for existing.
"A real-time incident management tool for on-call SRE teams."
Not a feature list. Not a pitch. The *thing*.

## Who It's For
Situations and cognitive posture, not demographics.
"An on-call SRE at 3am, one hand on coffee, scanning a dashboard for anomalies.
Not browsing. Not exploring. Scanning."

## How It Should Feel
3-5 attributes describing the cognitive posture of the UI.
- Calm — even when the data says panic
- Clinical — no decoration, no cheerleading
- Dense — information density is a feature, not a bug
- Quietly competent — it works, it doesn't perform

## What It's Not (Anti-References)
"NOT a Stripe clone. NOT a gamified dashboard with streaks and badges.
NOT pastel. NOT playful."

## What Good Looks Like (References)
"Linear's restraint. Grafana's information density. Bloomberg Terminal's disregard for beauty."

## Constraints
- Platform: web, desktop-first
- Density: data-heavy — screens are tools, not experiences
- Accessibility: WCAG AA minimum
- Tech: HTML + Tailwind

## Design Mode
Product. Design serves the task.
```

---

## Baked-In Knowledge — What Lives Inside the Skill

The skill is not a thin wrapper around an LLM. It carries a knowledge base that shapes every generation. This is the cumulative product of our research.

### A. Design Principles (Product Mode)

These are active constraints, not aspirational quotes:

1. **Function before form.** Every visual decision must reference a user task or hierarchy goal.
2. **Hierarchy is the master skill.** Most "bad design" is flat hierarchy, not ugliness. Squint test: can you rank the top 5 elements by importance in 3 seconds?
3. **Start with the feature, not the layout.** (Refactoring UI.) Design the primary interaction, then arrange around it.
4. **De-emphasize, don't over-emphasize.** When the primary element isn't standing out, make competing elements quieter — don't add more visual weight to the primary.
5. **Conventions > novelty.** (Jakob's Law.) Users expect your product to work like others they already know. Deviate only with justification.
6. **Content is the design.** Layout is shaped around real copy and real data. Never the inverse.
7. **Tokens are the only source of visual values.** No hardcoded hex, no magic pixels. If something is needed, add it to DESIGN.md first.
8. **Density serves scanning.** In product mode, users are extracting information or completing actions. Respect their time.

### B. UX Laws (Applied, Not Academic)

| Law | How it shapes generation |
|---|---|
| **Hick's** | Primary screens ≤7 top-level choices. If there are more, group or nest. |
| **Fitts's** | Primary CTA ≥44px touch target. Destructive actions separated from primary actions. |
| **Miller's** | Chunk information into ≤7 groups. Use visual grouping (Gestalt proximity). |
| **Jakob's** | Follow conventions for nav, search, forms, tables. Deviate only when the brief demands it. |
| **Von Restorff** | Use distinctiveness sparingly — for the single most important action on screen. |
| **Tesler's** | Complexity lives somewhere. Don't hide it from the user; reduce it by solving the right problem. |

### C. Gestalt Principles (Visual Grouping Rules)

| Principle | Application |
|---|---|
| **Proximity** | Related items closer together than unrelated. Spacing tokens enforce this. |
| **Similarity** | Same-role items share visual treatment (color, size, weight). |
| **Common Region** | Containers used only when grouping needs reinforcement. Don't box everything. |
| **Figure/Ground** | Primary content separable from background. No camouflage. |
| **Continuation** | Alignment guides the eye. Use it deliberately, don't break it accidentally. |

### D. The Slop Catalog — 37 Anti-Patterns (from Impeccable, reframed as generation constraints)

These are **generation constraints**, not detection rules. The skill knows these patterns and avoids producing them. No separate check step.

**Visual slop:**
- No purple-to-blue gradients unless DESIGN.md specifies that palette.
- No glassmorphism / blur effects unless elevation strategy explicitly calls for it.
- No side-tab accent borders on cards (thick colored stripe on one side).
- No rounded rectangles with generic drop shadows as the only shape language.
- No gradient text on headings.
- No sparklines as decoration (tiny charts conveying no real information).
- No reaching for modals by reflex. If the content needs scrolling and columns, it's a page, not a modal.

**Typography slop:**
- No flat type hierarchy. Font sizes must have ≥1.25 ratio between steps.
- No icon tiles stacked above headings (the universal AI feature-card pattern).
- No monospace as "technical" shorthand.
- No single font for everything. Pair a display/heading face with a body face.
- No overused fonts (Inter, Roboto, Fraunces, Geist, Plus Jakarta Sans, Space Grotesk) unless DESIGN.md explicitly chooses them with awareness that they're common.
- No all-caps body text. Uppercase is for short labels and headings only.

**Layout slop:**
- No card-in-card-in-card nesting (Cardocalypse) without hierarchy justification.
- No copy-paste section layouts where every section uses the same template.
- No massive icon containers larger than the content they introduce.
- No hero-section-metrics-features-CFAQ-footer formulaic page structure as default.

**Content slop:**
- No redundant UX writing (label + sublabel + helper + hint all saying the same thing).
- No lorem ipsum. Ever.
- No generic copy ("The best solution for your needs", "Welcome to our platform").
- No empty state that just says "No data." — it should be actionable.

**Interaction slop:**
- No animation without purpose. Motion is feedback, not decoration.
- No bouncing/wiggling elements as attention-grabbers.

### E. Refactoring UI Rules (Practical Application)

- Design in grayscale first — ensure hierarchy works through size, weight, and spacing before adding color. (The skill applies this mentally during generation; the output is colored, but the hierarchy was established without relying on color.)
- Add color deliberately — color has a job (identify primary actions, indicate state, brand recognition). It's not filler.
- Use font weight for hierarchy — don't go below 400 for body text; use 600-700 for emphasis.
- Align elements to a grid — don't center everything. Left-alignment creates stronger vertical lines.
- Use consistent spacing scale — all values from DESIGN.md tokens.

---

## Implementation Plan — Build Order

### Phase 0: The Knowledge Base (highest leverage, do first)

Before any code, encode the skill's knowledge:

1. **Write the skill's system prompt / instructions file.** This is the core artifact. It contains:
   - The generation constraints (principles + UX laws + Gestalt + slop anti-patterns + Refactoring UI rules) — all §A-E above.
   - The `/design-init` conversation script — what to ask, in what order, how to probe deeper.
   - The DESIGN.md spec — how to produce valid files.
   - The mock generation protocol — how to produce iterations, how to extract JTBD/pain points from the brief.
   - Product-mode rules.

2. **Validate with a dry run.** Manually simulate the `/design-init` conversation with a test product. Check that the DESIGN.md output passes `npx @google/design.md lint`.

### Phase 1: `/design-init` Skill

Build as a slash command (runs once per product):
- Conducts a conversation about the product's soul, audience, posture, and constraints.
- Produces DESIGN.md (Google spec, valid YAML frontmatter + markdown body).
- Produces PRODUCT.md (product identity — soul, not tasks).
- Produces preview HTML (style tile + rough composition).
- Supports iteration: user can say "warmer accent" or "more density" and get revised DESIGN.md + re-rendered preview.

### Phase 2: `/mock` Skill

Build as a slash command (runs many times per product):
- Reads DESIGN.md + PRODUCT.md.
- Takes a brief — this is where JTBD and pain points live, per screen.
- **Interview-back (conditional):** Assesses brief depth after reading context. If thin/ambiguous or implies unresolved architectural choices, asks 3-6 targeted questions about data flow, state model, interaction tradeoffs, edge cases, usage context, system constraints, or accessibility — never surface aesthetics. Skips if brief is sufficient or user signals "just go."
- Extracts the user's job and posture from the brief; supplements with PRODUCT.md when brief is terse.
- Produces 2-3 HTML iterations with different layout approaches.
- All iterations use only DESIGN.md tokens.
- No slop (generation constraints baked in).
- User picks one → moves to `accepted/`.

### Phase 3: Pattern Extraction (lazy, manual)

Over time, if clear patterns emerge from accepted mocks, update the skill's knowledge. This is manual curation, not automation. The bar is: "would this change a future generation decision?" If yes, add it. If not, don't.

---

## What This Is Not

- **Not a component library generator.** It produces mockups, not a shadcn clone. Components in DESIGN.md are style definitions, not code packages.
- **Not a Figma replacement.** There's no canvas, no vector editing, no drag-and-drop. The medium is HTML viewed in a browser.
- **Not brand-mode design.** It's optimized for product UI. If you need a stunning marketing landing page with bespoke illustration and motion design, this workflow will produce competent but not exceptional output.
- **Not multi-step detection.** Slop prevention is baked into generation constraints, not a post-hoc checking pipeline. The skill avoids producing slop; it doesn't catch slop after the fact.
- **Not a code generator.** Accepted mocks are production-viable HTML, but the workflow stops at the mock. Turning them into a real app with routing, state, and data fetching is a separate concern.

---

## Open Questions to Revisit

1. **How terse can the brief be?** *(Mostly resolved by interview-back.)* If someone types `/mock "user profile"` and PRODUCT.md provides enough context to form a safe mental model, the skill generates and notes assumptions. If the brief is thin *and* PRODUCT.md doesn't fill the gap — or if the brief implies unresolved architectural choices — interview-back kicks in (step 1.5). The threshold is: **would guessing produce a mock that's likely wrong in a way the user would catch immediately?** If yes, ask. If no, generate and iterate. Remaining nuance: calibrating the "thinness threshold" so it doesn't over-fire on competent users who prefer terse briefs.

2. **How many iterations?** 2-3 is the default. Should this be configurable? Probably — let the user say `/mock --iterations 5 "..."` if they want more exploration.

3. **Cross-page consistency.** If you generate a dashboard mock and a settings mock separately, how do you ensure they feel like the same product? DESIGN.md + PRODUCT.md should handle this, but it's worth testing.

4. **Mobile variants.** Should each iteration include a responsive/mobile view, or is that a separate `/mock` call with `--mobile`? I'd recommend: include a basic responsive view in the same HTML file (visible on viewport resize), but don't design two separate layouts per iteration.

---

## Relationship to RESEARCH.md

This proposal is the practical distillation of the research in `RESEARCH.md`. The theoretical foundations (Nielsen, Gestalt, UX Laws, Refactoring UI, Atomic Design) are all here — but encoded as generation constraints in the skill's knowledge base, not as separate artifacts in the pipeline. The multi-agent pipeline from the research document has been collapsed into two commands with baked-in expertise. This is simpler, faster, and harder to get wrong.

The key insight from the research that this workflow preserves: **the three artifacts that matter most are the design system (DESIGN.md), the product's soul (PRODUCT.md), and the critique criteria (baked into the skill).** Everything else is orchestration.
