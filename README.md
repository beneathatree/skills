# Agent Workflow Skills

**Skills for building products with AI agents -- starting with design, expanding into engineering. Each skill composes with others through shared artifacts. Grounded in real practice, not generic defaults.**

[![Install with npx skills](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fskills.sh%2Fapi%2Fskills%2Fbeneathatree%2Fskills&query=%24.installs&label=installs&color=blue&cacheSeconds=86400)](https://skills.sh/beneathatree/skills)
[![Skills on skills.sh](https://img.shields.io/badge/skills.sh-20%2B+agents-green)](https://skills.sh)

---

## What This Is

A **multi-session, stateful workflow** for building products inside your AI coding agent. Skills compose with each other through shared artifacts -- so output from one skill becomes input for another, and everything stays consistent across sessions.

### Available Now (Design)

| Skill | What It Does | When to Run |
|---|---|---|
| `design-init` | Interviews you about your product, then produces a **design system** (`DESIGN.md`), **product identity** (`PRODUCT.md`), and a **preview** you can open in a browser | Once per product |
| `mock` | Reads your design system, takes a brief, optionally asks clarifying questions, then generates **2--3 distinct HTML iterations** for you to pick from | Every screen or page |

### Coming Next (Engineering)

More skills will plug into the same `DESIGN.md` + `PRODUCT.md` artifacts to cover backend architecture, API design, data modeling, testing strategies, and beyond. The shared artifact model means an engineering skill will know your product's soul before it writes a single line of code.

The key idea: **capture your product's identity once, then let every skill -- design or engineering -- build from that same source of truth.**

## How It Works (Design Workflow)

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

     (engineering skills will read DESIGN.md + PRODUCT.md too)
```

### Step 1: Establish Your Product's Foundation

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

If your brief is thin -- `"User profile page"` with no job or posture context -- `/mock` pauses and asks **3--6 targeted questions** about architecture and product decisions *before* generating anything. About data flow, state model, interaction patterns, edge cases.

If your brief is rich enough, it skips straight to generation. No unnecessary questions.

## Growing Collection

This repo starts with design skills because that's where product clarity begins -- if you don't know what you're building and who it's for, no amount of engineering skill matters. But the collection will expand into engineering: backend architecture, API design, data modeling, testing, deployment. Every new skill reads from the same `DESIGN.md` + `PRODUCT.md` foundation, so a backend skill knows your product's constraints before it proposes a database schema.

## Why This Feels Different

Most AI tools generate generic output because they skip two things: **upstream context** and **downstream taste**. This workflow builds both in.

**What's baked into every generation:**

- **Anti-pattern knowledge** -- each domain has its own catalog of common mistakes. Design has slop patterns; engineering will have its own equivalent. Avoided during production, not caught afterward.
- **Principles applied, not just listed** -- UX laws, Gestalt principles, Refactoring UI rules for design. SOLID principles, testing pyramids, security checklists for engineering (coming).
- **Shared artifacts across sessions** -- `DESIGN.md` and `PRODUCT.md` persist. Switch agents, switch machines, come back next week -- the context is still there.
- **Iterative selection** -- pick from approaches, don't accept whatever comes out.
- **Production scaffolds** -- output you can build from, not throwaway reference material.

## Install

### Via skills.sh (recommended)

Works with **20+ agents** automatically:

```bash
npx skills add beneathatree/skills
```

Supported agents include Claude Code, Cursor, Codex, Copilot, Gemini CLI, Windsurf, Zed, Roo Code, Trae, pi, and more.

Install just one skill:

```bash
npx skills add beneathatree/skills --skill design-init
npx skills add beneathatree/skills --skill mock
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

- **Not a template library** -- these skills produce output tailored to *your* product, not generic starters
- **Not a replacement for judgment** -- the agent is a junior designer/engineer; you're the art director/tech lead
- **Not framework-specific** -- pure markdown instructions work across any stack
- **Not a closed set** -- starts with design, grows into engineering. The artifact model is extensible by design

## Structure

```
.
├── README.md                        <-- This file
├── skills/
│   ├── design-init/
│   │   └── SKILL.md                <-- Interview-driven product foundation
│   ├── mock/
│   │   └── SKILL.md                <-- Iterative mockup generation
│   └── ...                         <-- More skills coming (design + engineering)
└── brainstorm/                      <-- Research archive (not needed at runtime)
    ├── proposed_ui_workflow.md      <-- Design workflow specification
    ├── RESEARCH.md                  <-- Research foundations
    └── skills_sh_research.md        <-- Distribution strategy research
```

## License

MIT
