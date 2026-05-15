# UI Workflow Skills

**Agent skills for design-driven frontend workflows. Start with a conversation, end with production-viable HTML mockups -- grounded in real design practice, not slop.**

[![Install with npx skills](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fskills.sh%2Fapi%2Fskills%2Fbeneathatree%2Fskills&query=%24.installs&label=installs&color=blue&cacheSeconds=86400)](https://skills.sh/beneathatree/skills)
[![Skills on skills.sh](https://img.shields.io/badge/skills.sh-20%2B+agents-green)](https://skills.sh)

---

## What This Is

A **multi-session, stateful design workflow** that lives inside your AI coding agent. Each skill composes with the others through shared artifacts:

| Skill | What It Does | When to Run |
|---|---|---|
| `design-init` | Interviews you about your product, then produces a **design system** (`DESIGN.md`), **product identity** (`PRODUCT.md`), and a **preview** you can open in a browser | Once per product |
| `mock` | Reads your design system, takes a brief (e.g., "dashboard"), optionally asks clarifying questions, then generates **2--3 distinct HTML iterations** for you to pick from | Every screen or page |
| *more coming* | | |

The key idea: **`/design-init` captures your product's soul once. `/mock` uses that soul every time.** Every mock feels like it belongs to the same product because they all read from the same source of truth.

## How It Works

```
design-init                           mock
     |                                  |
     v                                  v
 Conversation  -->  DESIGN.md       Brief  -->  2-3 HTML
 with you            PRODUCT.md      +         iterations
                     Preview HTML    DESIGN.md    |
                                      PRODUCT.md   v
                                                    User picks
                                                    one  -->  frozen
                                                              |
                                                     Production scaffold

     (more skills plug into DESIGN.md + PRODUCT.md over time)
```

### Step 1: Establish Your Design System

Run **`design-init`**. The skill asks focused questions about:

- **What your product is** -- its reason for existing, not its feature list
- **Who uses it** -- their situation and mental state, not their demographics
- **How it should feel** -- calm? dense? surgical? warm?
- **What to avoid** -- anti-references are more powerful than references
- **Constraints** -- platform, density, accessibility, tech stack

From this conversation it produces three files in your project root:

| File | Purpose |
|---|---|
| `DESIGN.md` | Your design system in Google-spec format: colors, typography, spacing, elevation, shapes, components |
| `PRODUCT.md` | Your product's persistent identity: what it is, who it's for, how it should feel, what it's not |
| `previews/token-preview.html` | A style tile + rough composition page. Open it in a browser to validate tokens before committing. |

Iterate until it looks right. Say "warmer accent" or "more density" and it revises and re-renders.

### Step 2: Generate Mockups

Run **`mock`** with a brief:

```text
> /mock "Dashboard for the habit tracker. Main view: today's habits
>        in a compact list, streak counts, a weekly mini-chart.
>        User is scanning, not exploring."
```

The skill reads `DESIGN.md` and `PRODUCT.md`, then produces 2--3 HTML iterations -- each a **different approach**, not a random variation:

- Different layout strategy (sidebar vs top nav, cards vs table)
- Different density tradeoffs within your token system
- Different hierarchy emphasis

Pick one (or say "merge layout of v2 with density of v1"). The chosen mock is frozen as a **production scaffold** in `accepted/`.

### The Interview-Back Safety Net

If your brief is thin -- `"User profile page"` with no job or posture context -- `/mock` pauses and asks **3--6 targeted questions** about architecture and product decisions *before* generating anything. Not about aesthetics (tokens handle those). About data flow, state model, interaction patterns, edge cases.

If your brief is rich enough, it skips straight to generation. No unnecessary questions.

## Growing Collection

This repo started with `design-init` and `mock` -- the foundation of interview-driven design system establishment and iterative mockup generation. More skills will be added over time, each reading from and writing to the shared `DESIGN.md` + `PRODUCT.md` artifacts so everything stays consistent.

## Why This Feels Different

Most AI UI tools generate generic output because they skip two things: **upstream context** and **downstream taste**. This workflow builds both in.

**Baked into every generation:**

- **37 slop anti-patterns** -- no purple-blue gradients, no glassmorphism by default, no card-in-card nesting, no lorem ipsum, no flat type hierarchy. Avoided during production, not caught afterward.
- **UX laws applied** -- Hick's (trim choices), Fitts's (big CTAs), Jakob's (follow conventions), Miller's (chunk info), Von Restorff (one standout max).
- **Gestalt principles** -- proximity, similarity, common region, figure/ground, continuation. Visual grouping isn't accidental.
- **Refactoring UI rules** -- grayscale-first hierarchy thinking, color has a job, weight creates emphasis, grid alignment, spacing scale only.
- **Nielsen's 10 heuristics** -- quality floor every iteration must satisfy.

**What you get that single-pass tools don't:**

- A **persistent design system** (DESIGN.md) that makes every mock consistent
- A **product identity** (PRODUCT.md) that keeps anti-references alive across sessions
- **Iterative selection** -- pick from approaches, don't accept whatever comes out
- **Production scaffolds**, not throwaway screenshots -- developers build from the accepted HTML

## Install

### Via skills.sh (recommended)

Works with **20+ agents** automatically:

```bash
npx skills add beneathatree/skills
```

This installs both skills into your agent's skill directory. Supported agents include Claude Code, Cursor, Codex, Copilot, Gemini CLI, Windsurf, Zed, Roo Code, Trae, pi, and more.

Install just one skill:

```bash
npx skills add beneathatree/skills --skill design-init
npx skills add beneathathree/skills --skill mock
```

### Via GitHub (gh skill)

```bash
gh skill install beneathatree/skills design-init
gh skill install beneathatree/skills mock
```

### Manually

Clone or copy `skills/design-init/SKILL.md` and `skills/mock/SKILL.md` into your agent's skills directory.

## What You Need

- An AI coding agent that supports SKILL.md files
- That's it. No build step, no dependencies, no frameworks. Pure markdown instructions.

## What This Is Not

- **Not a component library generator** -- produces mockups, not shadcn clones
- **Not a Figma replacement** -- medium is HTML in a browser, no canvas or drag-and-drop
- **Not brand-mode design** -- optimized for product UI (dashboards, tools, interfaces). Marketing landing pages will be competent but not exceptional
- **Not a code generator** -- accepted mocks are production-viable HTML scaffolds; routing, state, and data fetching are separate concerns
- **Not a post-hoc slop detector** -- prevention is baked into generation constraints

## Structure

```
.
├── README.md                        <-- This file
├── skills/
│   ├── design-init/
│   │   └── SKILL.md                <-- Interview-driven design system establishment
│   ├── mock/
│   │   └── SKILL.md                <-- Iterative HTML mockup generation
│   └── ...                         <-- More skills coming
└── brainstorm/                      <-- Research archive (not needed at runtime)
    ├── proposed_ui_workflow.md      <-- Full workflow specification
    ├── RESEARCH.md                  <-- Research foundations
    └── skills_sh_research.md        <-- Distribution strategy research
```

## License

MIT
