# Agent Workflow for HTML UI/UX Design — Research & Recommendations

A synthesis of how traditional UI/UX designers work, why AI-generated UIs feel like "slop," and a concrete proposal for an agent workflow that produces grounded, non-generic HTML mockups.

---

## 1. How traditional UI/UX designers actually work

Most credible sources describe a five-phase loop: **Discover → Define → Design → Validate → Hand off**. The artifacts produced at each phase are well-codified and they map almost 1:1 to inputs/outputs an agent pipeline can consume.

### 1.1 Phase-by-phase artifacts

| Phase | Purpose | Canonical deliverables | Fidelity |
|---|---|---|---|
| Discover | Understand user + business | Stakeholder/design brief, interviews, competitive audit, jobs-to-be-done (JTBD), personas, empathy maps | text/data |
| Define | Frame the problem | Problem statement, user journey map, information architecture (sitemap), user flows, content model | diagrams |
| Design (UX) | Structure | **Sketches → wireframes** (grayscale, structural, no brand) | low-fi |
| Design (UI) | Surface | **Mockups** (typography, color, imagery, brand) | high-fi |
| Validate | De-risk | **Prototypes** (clickable), usability tests, heuristic evaluation | interactive |
| Hand off | Build | Redlines / specs, **design system / style guide**, tokens, component library | machine-readable |

Sources: Interaction Design Foundation's "14 UX Deliverables", Magic Patterns' "Wireframe vs Mockup vs Prototype", MobiDev's UX process guide, UX Playbook.

### 1.2 Two cross-cutting frameworks

- **Atomic Design (Brad Frost)** — UI as a hierarchy: *atoms → molecules → organisms → templates → pages*. With design tokens added as a "subatomic" layer. This is the single most useful mental model for agents because it gives composable, addressable units.
- **Jobs-to-Be-Done (Clayton Christensen / Tony Ulwick)** — describe what the user is trying to *accomplish*, not who they are demographically. Personas are richer wrappers but the JTBD nucleus is what actually steers design decisions (NN/g).

### 1.3 Inputs that experienced designers refuse to skip
1. **A real problem statement** ("X user, doing Y task, in Z context, fails because…").
2. **Real content** — copy, data, edge-case states, error messages. Designing with lorem ipsum is widely treated as malpractice.
3. **Brand attributes** — voice, mood, references, anti-references ("not this").
4. **Constraints** — platform, accessibility level (WCAG AA/AAA), tech stack, performance budget.

These are exactly the inputs an agent system also needs. Most "slop" comes from skipping them.

---

## 2. Theoretical / philosophical foundations to encode

You want agents that *reason like a designer*, not pattern-match. That means encoding principles, not just examples.

### 2.1 The canon (~7 frameworks)

1. **Nielsen's 10 Usability Heuristics** — visibility of status, match to real world, user control, consistency, error prevention, recognition over recall, flexibility, minimalist aesthetic, error recovery, help. *Universal checklist.*
2. **Gestalt principles** — proximity, similarity, closure, continuation, common region, figure/ground, common fate. *Govern visual grouping.* (NN/g)
3. **Don Norman's interaction model** — affordances, signifiers, mapping, feedback, constraints, conceptual model, discoverability.
4. **UX Laws** (Parallel HQ reference):
   - **Hick's** — choice time grows with options → trim menus.
   - **Fitts's** — target time = f(distance, size) → big primary CTAs.
   - **Miller's** — ~7 chunks of working memory → chunk content.
   - **Jakob's** — users expect your site to work like the ones they already use.
   - **Von Restorff** — distinct item is remembered → use sparingly for primary action.
   - **Tesler's** — irreducible complexity must live somewhere (system or user).
   - **Postel's** — accept liberally, output strictly (forms).
5. **Visual design pillars** — hierarchy, scale, balance, contrast, rhythm.
6. **Atomic Design + Design Tokens** — composition + theming.
7. **Refactoring UI (Wathan/Schoger)** — pragmatic rules:
   - Start with a feature, not a layout.
   - Design in grayscale first; add color last.
   - Establish hierarchy via *size, weight, color contrast, spacing* — not decoration.
   - De-emphasize supporting content rather than over-emphasizing primary.
   - Don't use pure gray text on colored backgrounds; tint towards the background.
   - Work in cycles; build the real thing early.

### 2.2 The philosophical posture

These framings are what separate "design-literate" agents from "image-generator with HTML output":

- **Form follows function.** Every visual choice must justify itself against a user task.
- **Hierarchy is the master skill.** Most "bad design" is not ugliness, it's flat hierarchy.
- **Constraints generate creativity.** A fixed spacing scale, type scale, and 4–6 color ramp produces *more* coherent designs than freedom does. (This is the Tailwind/Refactoring UI thesis.)
- **Content is the design.** Layouts shaped around real copy and real data; never the inverse.
- **Brand is what's *not* generic.** Brand attributes are *constraints that exclude defaults*. Encode anti-patterns as explicitly as patterns.
- **Critique is not optional.** A separate evaluating mind (or agent) catches what the producing mind cannot. Heuristic evaluation is standard practice; replicate it in the loop.
- **The agent is a junior designer; the human is the art director.** (A framing widely shared in current practitioner discourse — see Ashok Naik, Stuti Mazumdar.)

---

## 3. Why AI-generated UI looks like slop (and what causes it)

A clear consensus across recent practitioner write-ups (think.design, Hashbyt, dev.to/a_shokn, Medium/Mageswari):

**Visual tells of AI slop**
- Purple → blue gradients, glassmorphism cards, soft-pastel voids
- Faceless 3D illustrations with floating geometric shapes
- Inter/Roboto everywhere, generous whitespace covering thin content
- Card-grid layouts regardless of content type
- Decorative gradients/shadows replacing real hierarchy
- Hardcoded values rather than tokens; no real component architecture

**Root causes**
1. **Pattern collapse** — LLMs/diffusion models converge on the modal training-set design.
2. **Missing context** — no real brand, no JTBD, no content → model fills with defaults.
3. **No critique loop** — one-shot generation, no evaluation pass.
4. **Decoration over function** — model optimizes for "looks finished," not "works."
5. **No design tokens / no system** — every screen is invented from scratch.

**Implication for your workflow:** the anti-slop machinery is *upstream context* + *downstream critique*, not better prompting alone.

---

## 4. What current leading tools do (and the gap to exploit)

- **v0 (Vercel) + shadcn registry** — passes a structured **registry** (JSON describing components, blocks, and tokens) into the model so generations align with a real design system. ([v0 docs](https://v0.app/docs/design-systems)). This is the most promising production pattern.
- **DTCG / W3C Design Tokens spec** — a stable JSON format for tokens, now exportable from Figma natively. Use this as your source of truth.
- **Style Dictionary** — transforms DTCG tokens into CSS variables / Tailwind / SCSS / iOS / Android.
- **Relume, Magic Patterns, Google Stitch, Figma Make** — prompt-to-screen tools; all good for ideation, all weak on systematic consistency and brand fit.
- **Academic work** (arXiv 2412.20071, "Towards Human-AI Synergy in UI Design") — argues the missing piece is *iterative collaboration with intent clarification*, not bigger one-shot models. Aligns with the critique-loop posture.

**The gap:** existing tools either generate one-shot UI without a system, or generate with a system but with no design reasoning. A multi-agent workflow that explicitly does *brief → IA → wireframe → mockup → critique* against an encoded design system has not been productized well — which is your opportunity.

---

## 5. Recommended architecture for your workflow

A pipeline of specialized agents, each producing a typed artifact, with a critique loop. HTML is the final medium, but **intermediate artifacts are structured (YAML/JSON/MD) so each step is reviewable, replayable, and grounded**.

```
┌──────────────┐   ┌──────────┐   ┌────────────┐   ┌──────────┐   ┌─────────┐
│  Intake/     │──▶│ IA &     │──▶│ Content    │──▶│Wireframer│──▶│ Mockup  │
│  Brief Agent │   │ Flow     │   │ Strategist │   │ (lo-fi)  │   │ (hi-fi) │
└──────────────┘   └──────────┘   └────────────┘   └──────────┘   └────┬────┘
        ▲                                                              │
        │                                                              ▼
        │            ┌───────────────┐     ┌────────────────┐    ┌──────────┐
        └────────────│ Revision      │◀────│ Critic +       │◀───│ A11y     │
                     │ Orchestrator  │     │ Slop Detector  │    │ Checker  │
                     └───────────────┘     └────────────────┘    └──────────┘
                                                  │
                                                  ▼ (pass threshold)
                                            ┌──────────┐
                                            │ Handoff  │
                                            │ (HTML +  │
                                            │ spec)    │
                                            └──────────┘
```

**Shared knowledge base (read by all agents):**
- `principles.md` — your house design philosophy + anti-patterns
- `tokens.json` — DTCG-format design tokens
- `components.json` — shadcn-style component registry
- `brand.md` — voice, tone, references, anti-references
- `critique-rubric.yaml` — evaluation criteria

### 5.1 Suggested artifact formats

**`brief.yaml`** (input)
```yaml
product: "Habit tracker for new parents"
primary_user:
  description: "Sleep-deprived first-time parent, 28-38, mobile-first"
  jtbd: "When my baby naps unpredictably, I want to log feeds/sleep in <5s, so I can spot patterns without it becoming another chore."
problem: "Existing apps require too many taps; designed for product managers, not exhausted parents."
success_criteria:
  - "Log a feed in under 5 seconds, one-handed"
  - "Glanceable daily summary"
constraints:
  platform: "mobile web first"
  a11y: "WCAG AA"
  tech: "HTML + Tailwind, no JS framework"
brand:
  attributes: [calm, competent, warm, not-cute]
  references_good: ["Linear's restraint", "Things 3's typography"]
  references_bad: ["pastel baby apps", "gamified streak UIs"]
out_of_scope: ["social features", "AI suggestions"]
```

**`tokens.json`** — DTCG format ([spec](https://tr.designtokens.org/format/))
```json
{
  "color": {
    "ink": { "$value": "#1a1a1a", "$type": "color" },
    "paper": { "$value": "#fafaf7", "$type": "color" },
    "accent": { "$value": "#3a5a40", "$type": "color" }
  },
  "space": {
    "1": { "$value": "4px", "$type": "dimension" },
    "2": { "$value": "8px", "$type": "dimension" }
  },
  "type": {
    "scale": {
      "body": { "$value": { "fontSize": "16px", "lineHeight": "1.5" } }
    }
  }
}
```

**`components.json`** — registry of allowed primitives (subset of shadcn schema). Agents are forbidden from inventing new ones; they may compose.

**`flows.yaml`** — screens, states, transitions.

**`content/<screen>.md`** — real copy, real data, empty/error/loading states explicitly written out.

**`critique-rubric.yaml`** — see §5.3.

### 5.2 What each agent does

| Agent | Reads | Produces | Hard rules |
|---|---|---|---|
| **Intake** | user request | `brief.yaml` | Must extract JTBD + brand anti-references; ask if missing. |
| **IA/Flow** | brief | `flows.yaml`, screen list | Each screen labelled with primary task + secondary tasks. |
| **Content Strategist** | brief, flows, brand voice | `content/*.md` | No lorem. Include empty/error/loading states. Voice matches brand. |
| **Wireframer** | flows, content, components | grayscale HTML per screen | **No color, no images, no gradients.** Hierarchy via type scale, spacing, weight only. (Refactoring UI rule.) |
| **Mockup** | wireframe, tokens, components | final HTML | May only use tokens + registered components. No inline magic values. |
| **A11y Checker** | mockup HTML | report | WCAG AA: contrast ratios, focus order, semantic landmarks, alt text, label/input pairing. |
| **Critic / Slop Detector** | mockup, brief, rubric | scored report + issue list | See §5.3. |
| **Revision Orchestrator** | critic report | routes back to wireframer or mockup | Stops when rubric score ≥ threshold or 3 iterations. |

### 5.3 The critique rubric (the most important file)

This is where your taste lives. Make it explicit so agents can apply it.

```yaml
heuristics:           # Nielsen — block on any failure
  - visibility_of_status
  - match_to_real_world
  - user_control_freedom
  - consistency_standards
  - error_prevention
  - recognition_over_recall
  - flexibility_efficiency
  - minimalist_aesthetic
  - error_recovery
  - help_documentation

gestalt:              # weight 1 each
  proximity: "Related items closer than unrelated. Inspect padding/margin."
  similarity: "Same-role items share visual treatment."
  common_region: "Containers used only when grouping needs reinforcement."
  figure_ground: "Primary action separable from background."

visual_pillars:
  hierarchy:
    test: "Squint test: can a viewer rank importance of top 5 elements in 3 seconds?"
  scale:
    test: "Is the type scale limited to ≤6 steps from tokens?"
  contrast:
    test: "Is the primary CTA the highest-contrast interactive element on screen?"
  spacing:
    test: "All margins/paddings drawn from spacing scale tokens (no magic px)."

ux_laws:
  hicks: "Primary screens ≤7 top-level choices."
  fittss: "Primary CTA ≥44px touch target; not adjacent to destructive actions."
  jakobs: "Conventions for nav, search, cart, etc. only deviated from with explicit justification."

brand_fit:
  attributes_must_be_present: ["from brief.brand.attributes"]
  anti_references_must_be_absent: ["from brief.brand.references_bad"]

slop_detectors:       # block list — fail if any are true without explicit justification
  - "Purple-to-blue gradient on hero"
  - "Glassmorphism / backdrop-blur on >2 surfaces"
  - "Generic 3D human illustration"
  - "More than 3 gradients on the page"
  - "Card grid used when content is not card-like (e.g. an article)"
  - "Default Inter/Roboto with no type personality"
  - "Decorative shadow with no elevation meaning"
  - "Lorem ipsum or placeholder copy present"
  - "Hardcoded hex/px not present in tokens"
  - "Primary CTA contrast < AA"

content_quality:
  - "Empty state present and useful"
  - "Error state present and actionable"
  - "Loading state present"
  - "Microcopy matches brand voice attributes"
```

The critic agent returns scores + a structured issue list. The orchestrator loops until pass or max iterations, surfacing remaining issues to the human.

### 5.4 Operating principles (encoded as system-prompt boilerplate)

Bake these into every agent's system prompt:

1. *Function before form.* Every visual decision must reference a task or hierarchy goal.
2. *Tokens are the only source of color, spacing, and type.* If you need something not in tokens, request it; do not invent.
3. *Wireframe in grayscale.* Color is applied only at the mockup stage.
4. *Show real content.* Never lorem; never `[image]`.
5. *Declare states.* For every interactive screen: default, hover/focus, empty, error, loading.
6. *No defaults.* Justify each pattern by JTBD or convention (Jakob's law). If neither, replace it.
7. *Cite anti-references.* Before finalizing, list 3 things you specifically did *not* do because of brand anti-references.

### 5.5 Why structured intermediate artifacts > one-shot HTML

- **Reviewability** — humans can correct at the cheapest stage (a YAML brief, not a 500-line HTML).
- **Replayability** — fix the brief, regenerate the rest deterministically.
- **Composability** — swap models per agent (cheap for wireframe, strong for critic).
- **Auditable taste** — your rubric is version-controlled and improves over time.

---

## 6. Phased build plan

**Phase 0 — Manual encoding (1–2 days, highest leverage)**
- Write `principles.md`, `critique-rubric.yaml`, `brand.md` starter, `tokens.json` (steal a sensible default and adapt), `components.json` (use shadcn registry as base).
- These three files *are* your design system. Without them, no agent helps.

**Phase 1 — Single-agent critic (1 day)**
- Take any existing HTML mockup, run it through the critic agent against the rubric. Iterate on the rubric until its scores match your gut.
- This validates your taste codification before building the producer pipeline.

**Phase 2 — Linear pipeline (3–5 days)**
- Build Intake → IA → Content → Wireframer → Mockup → Critic, each as a separate prompt + tool, sharing the knowledge base.
- Persist intermediate artifacts to disk; allow human edit between any two steps.

**Phase 3 — Critique loop (2 days)**
- Add Revision Orchestrator; tune threshold.
- Add A11y Checker (can be a deterministic linter — axe-core in CI — not an LLM).

**Phase 4 — Library growth**
- Every accepted output: extract patterns, add new components to registry, refine rubric.
- Over time the system gets more *yours* and less generic — the opposite of pattern collapse.

---

## 7. Specific recommendations for your case

1. **Adopt DTCG + Style Dictionary + a shadcn-style registry as your base.** It's the only standardized format that bridges design tools, code, and AI assistants today.
2. **Treat the rubric as your moat.** Your house critique rubric — especially the anti-slop and brand-fit sections — is the single artifact that prevents you from producing average output.
3. **Force a grayscale wireframe step.** It is the most reliable trick for keeping hierarchy honest, taken directly from Refactoring UI. Skipping it is the #1 cause of pretty-but-flat AI output.
4. **Never let agents write copy and design simultaneously.** Separate content strategist agent first, then design around the actual words. This is how senior designers work and it eliminates a huge slop category.
5. **Make the critic a different model / different system prompt than the producer.** Generating and judging in the same context biases towards self-confirmation.
6. **Keep the human in the loop at three checkpoints**: after the brief, after wireframe, after final mockup. Not in between.
7. **Start narrow.** Pick one product domain (e.g. SaaS dashboards, or marketing pages, or admin tools). The rubric and components for "any UI" are too vague to be useful. Specialization is how you escape the generic.

---

## 8. Further reading (curated)

**Foundational**
- Brad Frost — *Atomic Design* (atomicdesign.bradfrost.com) and his blog post on tokens + atomic design.
- Adam Wathan & Steve Schoger — *Refactoring UI* (refactoringui.com).
- Don Norman — *The Design of Everyday Things*.
- NN/g — Nielsen heuristics, Gestalt videos, Personas vs JTBD article.
- Parallel HQ — *UX Laws & Design Principles* reference guide.

**Agent / AI angle**
- arXiv 2412.20071 — "Towards Human-AI Synergy in UI Design" — intent clarification + iterative refinement framework.
- v0 docs — *Design Systems* page (registry concept).
- DTCG / W3C Design Tokens Community Group spec.
- Style Dictionary docs.

**Critique of current state**
- Ashok Naik — "How to Break the AI-Generated UI Curse" (dev.to).
- Stuti Mazumdar — "When Generated UI Actually Works" (think.design).
- Mageswari — "Why Your AI-Generated UI Looks Like Everyone Else's" (Medium).

---

## TL;DR

The reason AI-generated UI feels generic is not weak models — it's missing upstream context (real brief, real content, real brand, real design system) and missing downstream critique (no evaluating second mind). Traditional designers solve both by working in a phased pipeline of *typed artifacts*: brief → JTBD → IA → wireframe (grayscale, hierarchy-first) → mockup (tokens-only) → critique → handoff. You can mirror that pipeline as a multi-agent workflow with structured YAML/JSON between steps and HTML only as the final medium. The two artifacts that will most differentiate your system from every other AI UI tool are:

1. A **design system** (tokens + component registry + brand voice + anti-references), and
2. A **critique rubric** that encodes Nielsen + Gestalt + Refactoring UI rules + your explicit anti-slop block list.

Build those two first. The agents are mostly orchestration around them.
