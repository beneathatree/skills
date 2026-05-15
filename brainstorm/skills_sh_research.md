# skills.sh Research Brief

> What it is, how it works, whether it fits our distribution strategy for `/design-init` and `/mock`.

---

## 1. What Is skills.sh?

**skills.sh is Vercel's "npm for Agent Skills"** — an open ecosystem for discovering, installing, and managing AI agent skills. Launched ~Feb 2026, it rapidly became the most visible hub in the agent skills space.

**Three parts:**
| Component | What it does |
|---|---|
| **skills.sh website** | Public catalog / leaderboard of 91K+ skills. Search, browse by topic, see install counts, security audits |
| **`npx skills` CLI** | Install, find, list, update, remove skills from any agent platform |
| **GitHub as registry** | Skills live in GitHub repos. No separate registry server. Any public repo with `SKILL.md` files is automatically discoverable |

**Key stat:** 91,000+ tracked skills across 20+ agent platforms (Claude Code, Cursor, Copilot, Codex, Gemini CLI, Windsurf, Zed, Roo Code, Trae, etc.).

**Who's already there:** Anthropic (203K installs on `frontend-design`), Vercel Labs (305K on `vercel-react-best-practices`), Microsoft/Azure (4.7M total across their collection), shadcn/ui, Supabase, Mattpocock, Browser-use, and hundreds of individual developers.

---

## 2. How Publishing Works (The Surprising Part)

### There is NO separate publish step.

This is the critical architectural detail:

```
Your GitHub repo (public, with SKILL.md files)
        │
        ▼  (auto-discovered via telemetry + GitHub API)
   skills.sh catalog
        │
        ▼  (users run:)
   npx skills add <owner/repo>
```

**To "publish" a skill:**
1. Create a public GitHub repository
2. Put `SKILL.md` files in it (either at root for single-skill, or `skills/<name>/SKILL.md` for multi-skill)
3. Push to GitHub
4. That's it. No registry submission. No approval process. No `npm publish` equivalent.

The skills.sh indexer finds repos through install telemetry and GitHub API crawling. Once people install your skill, it appears on the leaderboard.

**From Vercel's own docs:**
> "There's no special publish command for skills.sh. To 'publish' a skill: push a SKILL.md to a public GitHub repo. That's it."

### Repo Structure (Multi-Skill, Our Case)

```yaml
beneathatree/ui-workflow/
├── skills/
│   ├── design-init/
│   │   └── SKILL.md          # ← /design-init command
│   └── mock/
│       └── SKILL.md          # ← /mock command
├── README.md                  # Required: what the collection is
└── (optional references/, scripts/, assets/)
```

Users install:
```bash
# Both skills:
npx skills add beneathatree/ui-workflow

# Just one:
npx skills add beneathatree/ui-workflow --skill design-init
npx skills add beneathatree/ui-workflow --skill mock
```

---

## 3. Privacy Model: Public Only (With Caveats)

### Public Discovery: ✅ Fully Supported
Public GitHub repos with SKILL.md files are indexed, appear on the leaderboard, show install counts, get security audits (Socket, Snyk, Agent Trust Hub).

### Private Repos: ❌ Not Indexed, ⚠️ Partially Installable

| Capability | Public Repo | Private Repo |
|---|---|---|
| Appears on skills.sh leaderboard | ✅ | ❌ |
| `npx skills add owner/repo` | ✅ | ⚠️ Works if user has Git auth |
| Security audits displayed | ✅ | ❌ |
| Install count tracking | ✅ | ❌ |
| Community discovery | ✅ | ❌ |

**Private repo installation** works technically (the CLI can clone private repos if the user has GitHub credentials configured), but:
- The skill will **not** appear on the public skills.sh catalog
- There's an [open feature request (#381)](https://github.com/vercel-labs/skills/issues/381) asking for official private skill support — currently unresolved
- A [HN thread](https://news.ycombinator.com/item?id=46972491) confirms this is a known gap; no workaround besides direct git URL

**For private/org-internal distribution**, the recommended path today is:
- GitHub native `gh skill install` (supports private repos with auth)
- Direct git clone into agent skill directories
- Internal documentation with install instructions

---

## 4. Installation Flow — User Experience

### For End Users

```bash
# Discover skills
npx skills find "ui design workflow"

# Install our skills
npx skills add beneathatree/ui-workflow

# Install globally (all projects)
npx skills add -g beneathatree/ui-workflow

# Install specific skill only
npx skills add beneathatree/ui-workflow --skill mock

# List what's available in the repo (without installing)
npx skills add beneathatree/ui-workflow --list

# Update installed skills
npx skills update
```

### What Happens Under the Hood

1. CLI resolves `owner/repo` to a GitHub URL
2. Clones (or copies) the repo
3. Scans for `SKILL.md` files in known locations
4. Copies/symlinks each skill into the **agent-appropriate directory**:

| Agent | Project Path | Global Path |
|---|---|---|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Cursor | `.cursor/skills/` | `~/.cursor/skills/` |
| Codex CLI | `.agents/skills/` | `~/.agents/skills/` |
| GitHub Copilot | `.github/skills/` | `~/.copilot/skills/` |
| Gemini CLI | `.gemini/skills/` | `~/.gemini/skills/` |
| Windsurf | `.windsurf/skills/` | varies |

5. Skill is immediately available — no restart needed

### Alternative: GitHub Native `gh skill` (Newer, More Secure)

GitHub launched `gh skill` in April 2026 (CLI v2.90.0+) as a native alternative:

```bash
gh skill install beneathatree/ui-workflow design-init
gh skill install beneathatree/ui-workflow mock
gh skill install beneathatree/ui-workflow design-init --pin v1.0.0    # Version pinning
gh skill update                                                       # Check for updates
gh skill publish                                                       # Validate + secure your repo
```

**Advantages over `npx skills`:**
- Version pinning (tags, commit SHAs)
- Immutable releases support
- Content-addressed change detection (tree SHA comparison)
- Portable provenance metadata written into SKILL.md frontmatter
- Supply chain security features (tag protection, secret scanning checks)
- Official GitHub integration

**Disadvantages:**
- Newer, less ecosystem momentum than `npx skills`
- Requires GitHub CLI installation
- Smaller community of published skills using it

---

## 5. How skills.sh Compares to Other Distribution Methods

| Method | Discovery | Privacy | Versioning | Cross-Platform | Effort to Publish |
|---|---|---|---|---|---|
| **skills.sh (`npx skills`)** | ✅ Leaderboard + search | Public only | Git-based (latest) | ✅ 20+ agents | Push to GitHub |
| **GitHub `gh skill`** | `gh skill search` | ✅ Public + private | ✅ Tags, SHAs, immutable | ✅ 6+ agents | Push + `gh skill publish` |
| **Direct git clone** | None (manual) | ✅ Any | Git (manual) | ✅ All | Manual docs |
| **npm package** | npmjs.com | ✅ Public + private | ✅ Semver | ❌ Agent-specific | `npm publish` |
| **LobeHub Skills** | Marketplace UI | Public only | Git-based | ✅ Via CLI | PR/submit |
| **Agensi** | Paid marketplace | Public only | ✅ Versions | ✅ Multiple | Application + review |
| **ClawHub** | Plugin marketplace | Public only | ✅ Versions | Primarily Claude Code | PR workflow |

### Key Differentiators

**skills.sh wins on:**
- **Zero-friction publishing** — no registration, no approval, no build step
- **Momentum** — 91K skills, Vercel-backed, de facto standard forming
- **Cross-agent reach** — one install command works for Claude Code, Cursor, Codex, Copilot, etc.
- **Security visibility** — Socket + Snyk audits on leaderboard pages

**skills.sh loses on:**
- **Private skills** — not supported (open issue, no timeline)
- **Version pinning** — `npx skills` always gets latest; use `gh skill` if you need pins
- **Monetization** — free only; no paid/premium tier (vs Agensi which has pricing)
- **Analytics** — basic install counts; no detailed usage analytics dashboard

---

## 6. Competitive Landscape — Other Marketplaces

| Platform | Type | Skills Count | Model | Notable For |
|---|---|---|---|---|
| **skills.sh** (Vercel) | Open index + CLI | 91K+ | Free, open | De facto standard, zero-friction |
| **LobeHub Skills** | Marketplace | 296K+ | Free, open | Largest index, polished UI |
| **Agensi** | Marketplace | ~2.5K | Paid (80/20 split) | Monetization, curated quality |
| **ClawHub** | Plugin store | ~5K+ | Free, PR-based | Claude Code focused |
| **agentskill.sh** | Directory | 110K+ | Free, open | Broad agent support |
| **QASkills.sh** | Niche directory | QA-focused | Free, open | Domain-specific (QA) |
| **GitHub `gh skill`** | Native CLI | Any repo | Free, open | Supply chain security, versioning |
| **SkillsLLM** | Directory | 2.5K+ | Free, open | Aggregated index |

**Takeaway:** The space is fragmenting but coalescing around two leaders: **skills.sh** (for the `npx skills` CLI ecosystem) and **LobeHub** (for the largest index). GitHub's native `gh skill` is the dark horse with enterprise-grade features.

---

## 7. Relevance to Our Skills (`/design-init` + `/mock`)

### Fit Assessment: Excellent 🟢

| Criterion | Status | Notes |
|---|---|---|
| Format compatibility | ✅ Perfect | Pure SKILL.md files, exactly the format skills.sh expects |
| Multi-skill repo | ✅ Supported | Our 2-skill collection maps cleanly to `skills/design-init/SKILL.md` + `skills/mock/SKILL.md` |
| Cross-platform | ✅ Core strength | skills.sh installs to 20+ agent directories automatically |
| No build/bundle step | ✅ Perfect match | We deliberately chose not to bundle the markdown-to-html extension |
| No code execution needed | ✅ Clean fit | Our skills are prompt/instruction-only (no scripts/) |
| Design/UI category | ✅ Proven category | `frontend-design`, `ui-ux-pro-max`, `sleek-design-mobile-apps` all thriving on skills.sh |
| Audience alignment | ✅ Strong | skills.sh users are exactly our target: developers who want better UI from their AI agents |

### Competitive Position on skills.sh

Looking at the **Design & UI** category on skills.sh today:

| Skill | Installs | What It Does |
|---|---|---|
| `frontend-design` (Anthropic) | 404K | Distinctive frontend interfaces, rejects generic AI aesthetics |
| `web-design-guidelines` (Vercel) | 305K | Web interface guidelines review |
| `vercel-composition-patterns` (Vercel) | ~120K | React composition patterns |
| `ui-ux-pro-max` (nextlevelbuilder) | 76K | Advanced UI/UX patterns |
| `sleek-design-mobile-apps` (sleekdotdesign) | 119K | Mobile-first design principles |
| `extract-design-system` (arvindrk) | 94K | Extract design system from existing code |
| **Our `/design-init` + `/mock`** | ? | **Full workflow: interview → design system → production HTML mocks** |

**Our differentiation:** Existing skills are either (a) review/audit oriented, (b) single-pass generation, or (c) component-focused. **None offer the interview-driven design system establishment + iterative mock generation workflow we've designed.** The closest is Anthropic's `frontend-design` (which is single-pass generation with aesthetic guidance, no persistent DESIGN.md, no interview, no iteration protocol).

We'd be the **only skill on skills.sh offering a multi-session, stateful design workflow** with persistent artifacts (DESIGN.md + PRODUCT.md) that persist across mock generations.

---

## 8. Recommendation: Updated Distribution Strategy

### Primary: skills.sh (Public GitHub Repo)

```
github.com/beneathatree/skills                  (public)
  ├── README.md                                  (install instructions, demo)
  ├── skills/
  │   ├── design-init/
  │   │   ├── SKILL.md                           (the full skill)
  │   │   └── references/
  │   │       └── slop-catalog.md                (optional: extracted anti-patterns)
  │   └── mock/
  │       ├── SKILL.md                           (the full skill)
  │       └── references/
  │           └── ux-laws-reference.md           (optional: quick-reference card)
  └── LICENSE                                    (MIT or Apache-2)
```

**Publishing steps:**
1. Create the GitHub repo (public)
2. Structure as multi-skill repo per the layout above
3. Push — you're now on skills.sh
4. Add a `README.md` with install command badge, demo GIF, description
5. Submit to LobeHub Skills index too (parallel submission, minimal effort)

**Install command for users:**
```bash
npx skills add beneathatree/skills
```

### Secondary: GitHub `gh skill` Native Support

Once the repo is public, it automatically works with `gh skill` too:
```bash
gh skill install beneathatree/skills design-init
gh skill install beneathatree/skills mock
```

Run `gh skill publish` to enable supply chain security features (immutable releases, provenance metadata).

### Tertiary: Direct Git (For Power Users / Private Forks)

Document in README:
```bash
# Clone directly into your agent's skills directory
git clone https://github.com/beneathatree/ui-workflow-skills.git ~/.claude/skills/ui-workflow
```

### NOT Recommended (For Now)

| Option | Why Not |
|---|---|
| **Agensi (paid)** | Our skills are niche/technical; paid marketplace limits adoption at this stage |
| **ClawHub** | Claude Code only; we want cross-platform |
| **npm package** | Adds unnecessary tooling; skills.sh is the established convention for SKILL.md distribution |
| **Keeping private-only** | Leaves massive discovery on the table. Our skills gain value from community usage (pattern extraction feedback loop) |

### If Private Distribution Is Needed Later

If there's a future requirement for private/org-internal distribution:
1. Keep the public repo as the "community edition"
2. Create a private fork for internal use
3. Distribute via `gh skill install` (supports private repos with auth)
4. Or document manual clone instructions for internal wikis

---

## 9. Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| skills.sh changes its indexing model | Low | High | Our skills are plain SKILL.md + GitHub; portable to any platform |
| Someone publishes a similar workflow skill | Medium | Medium | First-mover advantage on the interview-to-mock workflow; our knowledge base (slop catalog + UX laws) is differentiated |
| Low initial discoverability | Medium | Medium | Good README + category placement (Design & UI) + cross-post to LobeHub; consider a launch post showing before/after comparisons |
| SKILL.md spec evolves breakingly | Low | Medium | Monitor agentskills.io; our skills are prose-heavy so format changes are less likely to break us |
| `npx skills` CLI has bugs with multi-skill repos | Low | Low | Test thoroughly; fall back to `gh skill` or direct clone |

---

## 10. Action Items

1. ✅ **Research complete** — this document captures findings
2. ⬜ **Create the GitHub repo** `beneathatree/skills` (public)
3. ⬜ **Structure as multi-skill repo** per §8 layout
4. ⬜ **Write SKILL.md files** for both skills (this is the main work — see Phase 0/1/2 in proposed_ui_workflow.md)
5. ⬜ **Write README.md** with install badges, demo, category tags
6. ⬜ **Push to GitHub** — auto-appears on skills.sh
7. ⬜ **Submit to LobeHub Skills** for parallel indexing
8. ⬜ **Run `gh skill publish`** for supply chain security features
9. ⬜ **Test install** across Claude Code, Cursor, and one other agent
10. ⬜ **Launch announcement** — show the workflow in action with a before/after

---

*Research conducted: 2026-05-15*
*Sources: skills.sh website, vercel-labs/skills GitHub repo, Vercel KB guide, InfoQ coverage, DEV.community analysis, GitHub Changelog (gh skill launch), KDnuggets marketplace comparison, HN discussion, open issues #381 & #141*
