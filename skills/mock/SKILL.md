---
name: mock
version: 1.0.0
description: >
  Iterative HTML mockup generation grounded in your design system. Reads DESIGN.md and
  PRODUCT.md (from /design-init), takes a brief, optionally interviews back for clarity,
  then produces 2–3 distinct HTML iterations with real hierarchy and real content.
  User picks one → frozen mock → production scaffold. No slop.
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
  - mockup
  - html
  - frontend
  - prototype
  - design-system
requires:
  - DESIGN.md (produced by /design-init)
  - PRODUCT.md (produced by /design-init)
produces:
  - mocks/<name>-v1.html
  - mocks/<name>-v2.html
  - mocks/<name>-v3.html
  - accepted/<name>.html (after selection)
---

# `/mock` — Iterative HTML Mockup Generation

> **Requires `DESIGN.md` and `PRODUCT.md` in the project root** (produced by `/design-init`). Run `/design-init` first if they don't exist.

---

## What This Skill Does

1. **Reads context** — loads `DESIGN.md` and `PRODUCT.md` as non-negotiable ground truth.
2. **Accepts a brief** — the user describes what screen or page they need.
3. **Conditionally interviews back** — if the brief is thin/ambiguous, asks 3–6 targeted questions *before* generating anything.
4. **Produces 2–3 HTML iterations** — each a different approach (layout strategy, density, hierarchy emphasis). All use only tokens from DESIGN.md.
5. **User picks one** — chosen iteration is frozen as a production-viable scaffold in `accepted/`.

---

## Prerequisites Check

Before doing anything:

1. **Verify `DESIGN.md` exists** at the project root. If missing, stop and tell the user: > "No `DESIGN.md` found. Run `/design-init` first to establish your design system and product context."
2. **Verify `PRODUCT.md` exists** at the project root. Same message if missing.
3. **Read both files fully.** Every token, every constraint, every anti-reference is binding. You do not deviate from them without explicit user instruction in the brief.

---

## Step 1: Receive and Parse the Brief

The brief is the user's input when they invoke this skill. It can be terse or detailed.

### Examples of Good Briefs (Rich with Signal)

```
"Dashboard for the habit tracker. Main view: today's habits in a compact list,
streak counts, a weekly mini-chart. User is scanning, not exploring."
```

```
"Settings page for notification preferences. Toggle groups by channel
(email, push, SMS). User comes here to turn things OFF, not on."
```

These include **job** ("scanning", "turn things OFF") and **posture** — exactly what you need to make layout decisions.

### Examples of Thin Briefs (May Need Interview-Back)

```
"User profile page"
```

```
"Settings"
```

```
"API docs"
```

These are noun phrases with no job/posture context. They may be fine if PRODUCT.md fills the gap — or they may trigger interview-back (Step 1.5).

### What You Extract From Every Brief

1. **What screen/page** is being requested
2. **The user's job on this screen** — what are they trying to accomplish?
3. **Cognitive posture** — scanning? exploring? filling out a form? monitoring? comparing?
4. **Key content types** — data tables? forms? charts? lists? text? media?
5. **Pain points** — what goes wrong? what's frustrating about current approaches?
6. **Priority ranking** — what matters most on this specific screen?

If the brief doesn't provide some of these, infer from PRODUCT.md before asking.

---

## Step 1.5: Interview-Back (Conditional — Only When Warranted)

> **This is NOT an automatic step.** It fires only when generation would likely produce something wrong in a way the user would catch immediately.

### When to Interview-Back

Trigger interview-back when **any** of these are true:

1. **Bare noun phrase + no PRODUCT.md rescue.**
   - Brief is `"User profile page"` or `"Settings"` or `"API docs"`
   - AND PRODUCT.md doesn't provide enough domain/posture context to form a safe mental model

2. **Multiple plausible interaction models, none implied.**
   - Brief: `"Data table for logs"` — Is it real-time streaming? Paginated? Filterable? Exportable? Each = fundamentally different layout.

3. **Undefined external concepts.**
   - Brief: `"Integrate with the CI pipeline view"` — What pipeline? What does "integrate" mean here?

4. **Unspecified state/data flow decisions.**
   - Brief: `"Multi-step wizard for onboarding"` — What steps? Can users go back? What's saved between?

5. **Ambiguous scope/structure.**
   - Brief: `"The main page"` — For a complex product, "main" could mean dashboard, feed, home, landing, or workspace.

### When to SKIP Interview-Back

Skip when **any** of these are true:

1. **Brief includes job + posture + enough domain detail** — even if terse. The signal is sufficient.
2. **PRODUCT.md + DESIGN.md together fill the gaps** — reasonable inferences are safe.
3. **User explicitly signals speed over precision** — e.g., "just go", "quick mock", "don't ask me questions".

### How the Interview Works

- **3–6 questions maximum.** Not another `/design-init`. Unblock generation, don't re-establish the product.
- **Target architectural and product decisions ONLY.** Never surface aesthetics.
- **Synthesize answers into an enriched internal brief.** Write nothing to disk — enrichment lives only in this generation context.

#### Good Questions (Ask These)

| Category | Example Questions |
|---|---|
| **Data flow** | "Is this data read-only, or does the user mutate it here? If mutation: inline editing or via a detail drawer/modal?" |
| **State model** | "Does this screen reflect live updates (WebSocket/polling), or is it snapshot-based? How stale is acceptable?" |
| **Interaction tradeoffs** | "For filters — do users typically filter by one dimension at a time (sequential), or build complex multi-dimension queries (compositional)?" |
| **Edge cases** | "What happens when the list is empty vs. 10,000 items? Pagination, virtual scrolling, or small-and-stays-small?" |
| **Usage context** | "Who looks at this screen and when? Someone deep in a task needing density, or someone dropping in briefly needing orientation?" |
| **System constraints** | "Does this embed in an existing layout (sidebar, modal), or is it a full standalone page?" |
| **Accessibility** | "Specific a11y needs beyond WCAG AA? Keyboard-only patterns? Screen reader workflows? High-contrast mode?" |

#### Bad Questions (Never Ask These)

| Question | Why Not |
|---|---|
| "What color should the button be?" | DESIGN.md already defines this. |
| "Should it feel modern or classic?" | PRODUCT.md captures emotional register. |
| "Do you want cards or a table?" | This is YOUR job — show it as iteration variance. |
| "How many columns?" | Decide based on content type + density from tokens. Show alternatives. |

After the interview, proceed to Step 2 with your enriched understanding.

---

## Step 2: Generate 2–3 HTML Iterations

Produce **2–3 complete, self-contained HTML files** in `mocks/`. Each iteration is a **different approach**, not a random variation of the same layout.

### Naming Convention

```
mocks/<screen-slug>-v1.html
mocks/<screen-slug>-v2.html
mocks/<screen-slug>-v3.html
```

Derive `<screen-slug>` from the brief (e.g., `"dashboard"`, `"settings-notifications"`, `"user-profile"`).

### What Makes Iterations Distinct

Each iteration should differ in **at least two** of these dimensions:

| Dimension | Example Variants |
|---|---|
| **Layout strategy** | Sidebar nav vs. top nav; card grid vs. data table; tabs vs. accordion; single-column vs. two-column |
| **Density/whitespace** | Compact (tight spacing, smaller type within token bounds) vs. comfortable (generous spacing, larger type) |
| **Information hierarchy** | What's above the fold vs. below; what's primary vs. secondary vs. tertiary |
| **Interaction pattern** | Inline editing vs. modal/drawer; expandable rows vs. dedicated detail page; instant apply vs. save button |
| **Content grouping** | Grouped by category vs. grouped by frequency vs. grouped by recency |

**Important:** All variations must stay within DESIGN.md token bounds. "Denser" means using tighter spacing tokens (xs/sm instead of lg/xl), not inventing new values.

### HTML Quality Requirements

Every iteration MUST satisfy all of these:

**Structure:**
- Semantic HTML5 (`<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>`, `<button>`, `<input>`, `<label>`, `<table>`, etc.)
- Proper heading hierarchy (single `<h1>`, logical `<h2>`→`<h3>` progression)
- `<label>` properly associated with every `<input>`, `<select>`, `<textarea>`
- Meaningful `alt` text on images (or `role="presentation"` for decorative ones)
- ARIA landmarks where they aid navigation (`role="navigation"`, `role="main"`, etc.)

**Styling:**
- Single inline `<style>` block (no external stylesheets, no CDN CSS frameworks)
- CSS custom properties mapped from DESIGN.md tokens (e.g., `--color-accent: #3a5a40`)
- ALL color values, font sizes, spacing, radii, shadows referenced from these custom properties — NO hardcoded hex or magic pixels anywhere
- Responsive: works at 375px (mobile) through 1440px (desktop) via media queries
- `prefers-reduced-motion` respected (no animation if set)

**Content:**
- Real or realistic placeholder copy matching the product domain (from PRODUCT.md)
- NO lorem ipsum — ever
- NO generic marketing fluff ("Welcome to our platform", "The best solution for your needs")
- Data that looks like real product data (realistic names, numbers, dates, statuses)
- Empty state visible OR indicated via comment (e.g., `<!-- Empty state: [actionable message] -->`)
- Error state indicated via comment (e.g., `<!-- Error state: [specific error + recovery] -->`)
- Loading state indicated via comment (e.g., `<!-- Loading state: skeleton/skeleton -->`)

**States (per interactive element):**
- Default state is rendered visually
- Hover/focus, disabled, error, loading states indicated via CSS comments or class names in the `<style>` block

### Self-Check Before Presenting

Before showing iterations to the user, verify every item on this checklist:

- [ ] Squint test: can I rank the top 5 elements by importance in 3 seconds?
- [ ] Primary CTA is the highest-contrast interactive element on screen
- [ ] All colors come from DESIGN.md tokens (zero hardcoded hex)
- [ ] All spacing comes from DESIGN.md spacing scale (zero magic pixels)
- [ ] Type scale has ≥1.25 ratio between adjacent steps
- [ ] No slop pattern present (see Slop Catalog below)
- [ ] Content is realistic, not lorem ipsum
- [ ] Semantic HTML used throughout
- [ ] Works at mobile viewport (375px) AND desktop viewport (1440px)
- [ ] Each iteration is meaningfully different from the others

---

## Step 3: Slop Prevention (Baked Into Generation)

These are **generation constraints**, not post-hoc checks. The skill avoids producing these patterns the same way a trained designer knows not to.

### Visual Slop — DO NOT Produce

- Purple-to-blue gradients (unless DESIGN.md explicitly specifies purple-blue palette)
- Glassmorphism / backdrop-blur (unless elevation strategy calls for blur-based layering)
- Side-tab accent borders on cards (thick colored stripe on left edge)
- Rounded rectangles with generic drop shadows as the only shape language
- Gradient text on headings
- Sparklines as decoration (tiny charts conveying no real information)
- Modals by reflex (if content scrolls and has columns → it's a page, not a modal)
- More than 3 gradients on a single page
- Decorative shadows with no elevation meaning

### Typography Slop — DO NOT Produce

- Flat type hierarchy (sizes MUST have ≥1.25 ratio between steps)
- Icon tiles stacked above headings (universal AI feature-card template)
- Monospace as "developer/technical" shorthand (unless product IS a code editor)
- Single font for everything (always pair display + body faces)
- Overused defaults without deliberate choice: Inter, Roboto, Fraunces, Geist, Plus Jakarta Sans, Space Grotesk
- All-caps body text (uppercase for short labels/headings only)

### Layout Slop — DO NOT Produce

- Card-in-card-in-card nesting ("Cardocalypse") without clear hierarchy justification
- Copy-paste section layouts (every section uses identical template)
- Massive icon containers larger than the content they introduce
- Hero→metrics→features→CTA→FAQ→footer formulaic structure as default
- Centered-everything layouts without justification (left-align creates stronger scan lines)

### Content Slop — DO NOT Produce

- Redundant UX writing (label + sublabel + helper + hint all saying the same thing)
- Lorem ipsum — ever
- Generic marketing fluff
- Empty states that just say "No data" (must be actionable)
- Placeholder images as solid-color boxes without alt text describing what belongs there

### Interaction Slop — DO NOT Produce

- Animation without purpose (motion is feedback, not decoration)
- Bouncing/wiggling elements as attention-grabbers
- Hover effects that only change color (combine with subtle transform or shadow shift at minimum)

---

## Step 4: Product-Mode Generation Rules (Always Active)

These rules govern every iteration you produce:

1. **Start with the feature, not the layout.** (Refactoring UI.) What is the user's primary job on this screen? Design that interaction first. Arrange everything else around it.

2. **Hierarchy is the master skill.** Every element has a clear rank. Apply the squint test: blur your eyes (mentally) — can you still tell what matters most? If not, fix the hierarchy before polishing anything else.

3. **Function over form.** Every visual choice must reference a user task or a hierarchy goal. If it doesn't help the user accomplish their job, remove it.

4. **Conventions over novelty.** (Jakob's Law.) Nav goes where nav goes. Search goes where search goes. Tables have headers. Forms have submit buttons. Deviate only when the brief gives explicit justification.

5. **Density serves scanning.** In product mode, screens are tools. Users extract information or complete actions. Respect their time with appropriate information density (from PRODUCT.md constraints).

6. **States are real.** Every interactive element has: default, hover/focus, disabled, error, loading. Render default; indicate others via comments/classes.

7. **Content is the design.** Layout is shaped around real copy and real data. Never start with an empty grid and "fill it in." Start with the content and wrap the layout around it.

8. **De-emphasize, don't over-emphasize.** (Refactoring UI.) When the primary element isn't standing out enough, make competing elements quieter — don't add more visual weight to the primary. Reduce noise, don't increase signal volume.

9. **Anti-references are sacred.** Before finalizing any iteration, mentally check: "Would the anti-references in PRODUCT.md hate this?" If yes, revise. Cite 3 things you specifically did NOT do because of anti-references.

---

## Step 5: Present and Select

After generating all iterations:

1. **Tell the user** where the files are: `mocks/<slug>-v1.html`, `mocks/<slug>-v2.html`, etc.
2. **Describe what makes each iteration distinct** in one sentence each. Focus on the strategic difference, not visual description:
   > "- **v1**: Sidebar navigation with dense data table. Best for power users who know what they're looking for.
   > - **v2**: Top navigation with card-based layout. Better for users who benefit from visual grouping.
   > - **v3**: Collapsible sections with progressive disclosure. Balances density with scannability."

3. **Wait for the user's decision.** Handle these responses:

| User says | You do |
|---|---|
| "I'll take v2" | Freeze v2 → `accepted/<slug>.html`. Confirm location. Done. |
| "Merge v1's layout with v2's density" | Produce a new iteration incorporating both. Place in `mocks/`. Present for approval. |
| "None of these — try [specific direction]" | Generate new iteration(s) following the feedback. Stay in bounds of DESIGN.md tokens. |
| "Make v1 darker/more dense/lighter/etc." | Adjust within DESIGN.md token bounds. Re-render. Note: if adjustment needs new token values, suggest updating DESIGN.md instead of hacking it here. |
| "Can I see more variations?" | Produce additional iterations (up to 5 total). Keep them distinct. |

### Freezing the Accepted Mock

When the user picks an iteration:

1. Copy the chosen file to `accepted/<slug>.html`
2. Ensure the file is production-viable:
   - Clean up any TODO/experimental comments
   - Verify all semantic HTML is correct
   - Verify all token references are consistent
   - Add a header comment noting: source iteration, date, and that this is the accepted scaffold
3. **Confirm to the user** that the frozen mock is ready for development handoff

```html
<!--
  Accepted mock: <screen-name>
  Source: mocks/<slug>-vN.html
  Generated: <date>
  Design system: DESIGN.md (this project)
  Status: FROZEN — production scaffold
  To modify: edit DESIGN.md tokens, then regenerate via /mock
-->
```

---

## Knowledge Base — Baked Into Every Generation

### A. UX Laws (Applied)

| Law | Rule |
|---|---|
| **Hick's** | ≤7 top-level choices per screen. Group/nest if more. |
| **Fitts's** | Primary CTA ≥44px touch target. Destructive actions separated from primary. |
| **Miller's** | Chunk into ≤7 groups. Use Gestalt proximity for visual grouping. |
| **Jakob's** | Follow conventions. Deviate only with brief justification. |
| **Von Restorff** | One standout element max. Use for the primary action only. |
| **Tesler's** | Complexity lives somewhere. Don't hide it behind minimal UI. |

### B. Gestalt Principles (Visual Grouping)

| Principle | How You Apply It |
|---|---|
| **Proximity** | Related items get tighter spacing (sm/md) than unrelated groups (lg/xl). |
| **Similarity** | Same-role items share identical styling (color, size, weight). |
| **Common Region** | Use containers/borders only when grouping needs reinforcement. Don't box everything. |
| **Figure/Ground** | Primary content pops from background via contrast (DESIGN.md ink/paper tokens). |
| **Continuation** | Align elements to guide the eye through the intended reading/scanning order. |

### C. Refactoring UI Rules (Practical)

- **Grayscale-first mindset:** Hierarchy was proven via size/weight/spacing before color was applied. If removing color would flatten the hierarchy, fix the hierarchy — don't add more color.
- **Color has a job:** Primary actions, status indication, brand recognition. Never decorative.
- **Weight for hierarchy:** Body ≥400, emphasis 600–700, never ≥700 for body copy.
- **Grid alignment:** Left-align content blocks. Reserve centering for specific focal elements (hero text, empty states).
- **Spacing scale only:** Every gap is a DESIGN.md token value. No arbitrary padding/margin.

### D. Nielsen Heuristics (Quality Floor)

Every iteration must satisfy all 10 at minimum:

1. Visibility of system status
2. Match between system and real world
3. User control and freedom
4. Consistency and standards
5. Error prevention
6. Recognition rather than recall
7. Flexibility and efficiency of use
8. Aesthetic and minimalist design
9. Help users recognize, diagnose, and recover from errors
10. Help and documentation

---

## File Structure (in the user's project)

```
project/
├── DESIGN.md              ← Required input (from /design-init)
├── PRODUCT.md             ← Required input (from /design-init)
├── previews/
│   └── token-preview.html ← From /design-init
├── mocks/
│   ├── <screen>-v1.html   ← Your output (iteration 1)
│   ├── <screen>-v2.html   ← Your output (iteration 2)
│   └── <screen>-v3.html   ← Your output (iteration 3)
└── accepted/
    └── <screen>.html      ← Frozen mock (after user picks one)
```

---

## What This Skill Is NOT

- **Not a code generator.** Accepted mocks are production-viable HTML scaffolds, but routing/state management/data fetching are separate concerns.
- **Not a component library.** It produces page-level mockups, not reusable component packages.
- **Not a Figma replacement.** Medium is HTML in browser. No canvas, no drag-and-drop, no vector tools.
- **Not brand-mode design.** Optimized for product UI (tools, dashboards, interfaces). Marketing pages will be competent but not exceptional.
- **Not a post-hoc slop detector.** Prevention is baked in. The skill doesn't produce slop — it doesn't need to catch it afterward.
- **Not a replacement for `/design-init`.** It consumes DESIGN.md and PRODUCT.md; it cannot create them.

---

## Relationship to `/design-init`

`/mock` **depends on** `/design-init`. The workflow is intentionally split:

| | `/design-init` | `/mock` |
|---|---|---|
| **Runs** | Once per product | Many times per product |
| **Input** | Conversation with human | Brief + DESIGN.md + PRODUCT.md |
| **Output** | Design system + product identity | HTML iterations → frozen mock |
| **Focus** | "Who are we?" | "What does this screen look like?" |
| **Changes when** | Product pivots direction | Every new screen or revision |

Running `/mock` without DESIGN.md + PRODUCT.md will fail — there are no tokens or product identity to ground the generation in. That grounding is what separates this workflow from generic AI UI generators.
