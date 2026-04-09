# Agentic Development Pipeline (Skill-Driven, End-to-End)

This is a full workflow from idea → shipped code → written-down context. Each stage uses a small, focused assistant plus reusable playbooks ([agent-skills](https://skills.sh/addyosmani/agent-skills), [awesome-copilot](https://github.com/github/awesome-copilot)). The same setup works in [Cursor](https://www.cursor.com/), [Claude Code](https://claude.ai/code), [Codex](https://platform.openai.com/docs/guides/codex), or anything that reads the repo’s instruction files.





## Agent Instruction Files (Cross-Tool)

Standard instruction files committed to the repo root. Every major agent runtime reads at least one — same conventions apply whether you're on Cursor, Claude Code, or Codex.

| Path | Purpose | Read by |
|---|---|---|
| `AGENTS.md` | Top-level agent behavior, goals, boundaries | Claude Code, Codex, any agent |
| `.cursorrules` | Cursor-specific coding conventions | Cursor |
| `.github/copilot-instructions.md` | GitHub Copilot workspace instructions | Copilot, awesome-copilot agents |
| `SPEC.md` | Feature spec, acceptance criteria, tasks | All agents (read before coding) |
| `.github/workflows/` | CI/CD pipeline definitions | GitHub Actions |
| `.skills/` | Skills installed via [skills.sh](https://skills.sh/) | Agents (per CLI / tool layout) |
| `docs/` | Raw sources + wiki (long-lived context) | All agents (see step 12) |
| `test-results/` | Test results | e2e testing agent (See step 9) |

For language or framework-specific conventions (React, Next.js, Python, etc.), find and install the relevant community skill from [skills.sh](https://skills.sh/) and reference it in your instruction files alongside the core agent skills.



## MCP servers (tooling bridge).

| Server | Role | Pipeline steps |
|---|---|---|
| [GitHub MCP](https://github.com/github/github-mcp-server) | Branches, PRs, issues, labels, PR comments (e.g. preview URL, E2E report) | 5, 8, 9 |
| [Playwright MCP](https://github.com/microsoft/playwright-mcp) | Drive the browser for real flows (pre-commit QA loop on local/preview, full E2E on staging) | 4, 9 |
| [Chrome DevTools MCP](https://github.com/bjesuiter/mcp-chrome-devtools) | Console, network, runtime inspection on a live page | 9, 10 |
| **Custom / vendor** | Whatever your repo needs (e.g. Vercel, DB, Slack, company tools) | Where relevant |



## The Pipeline

```
Idea → Design → Spec → Code → Review → PR/Issues → CI → Deploy → E2E → Learn → Document

```



### 1) Design
**Agent: Designer**
- Establishes visual direction — branding, UI/UX, design system — before spec or code is written
- Outputs (tokens, component names, screen flows) feed directly into `SPEC.md`

**Tools used:**
- [Superdesign](https://app.superdesign.dev/) — UI/UX + branding exploration; exports React/HTML/CSS, design tokens, and component specs
- [Stitch by Google](https://stitch.withgoogle.com/) — prompt/sketch → production UI; generates a full design system (tokens + component library) with exportable specs

**Skills used:**
- [`frontend-design`](https://github.com/anthropics/anthropic-quickstarts) — Anthropic's base skill for high-quality, accessible UI
- [`impeccable`](https://github.com/pbakaus/impeccable) — builds on `frontend-design` with 7 domain references (typography, color/contrast, spatial, motion, interaction, responsive, UX writing).

**Key commands:** 

- `/critique` (UX review) 
- `/audit` (quality checks like performance, accessibility, etc.) 
- `/normalize` (design system alignment) 

**Anti-patterns it guards against:** overused fonts (Inter/Arial) · gray text on color · pure black/gray neutrals · nested cards · bounce easing


### 2) Define + Plan
**Agent: Planner**
- Refines raw idea → writes spec → breaks into atomic tasks with acceptance criteria

**Skills used:**
- [`idea-refine`](https://skills.sh/addyosmani/agent-skills/idea-refine) — clarify problem, constraints, success criteria
- [`spec-driven-development`](https://skills.sh/addyosmani/agent-skills/spec-driven-development) — produce `SPEC.md` before touching code
- [`planning-and-task-breakdown`](https://skills.sh/addyosmani/agent-skills/planning-and-task-breakdown) — atomic tasks with definition of done
- [`documentation-and-adrs`](https://skills.sh/addyosmani/agent-skills/documentation-and-adrs) — log architecture decisions inline at decision time
- [`context-engineering`](https://skills.sh/addyosmani/agent-skills/context-engineering) — structure context so downstream agents stay coherent



### 3) Build
**Agent: Builder**
- Implements spec in small, testable slices — never one-shot

**Skills used:**
- [`incremental-implementation`](https://skills.sh/addyosmani/agent-skills/incremental-implementation) — thin vertical slices, each commit is a working unit
- [`test-driven-development`](https://skills.sh/addyosmani/agent-skills/test-driven-development) — tests written alongside code, not after
- [`api-and-interface-design`](https://skills.sh/addyosmani/agent-skills/api-and-interface-design) — contract-first APIs, validated at every boundary
- [`frontend-ui-engineering`](https://skills.sh/addyosmani/agent-skills/frontend-ui-engineering) — clean component patterns, accessibility baked in
- Framework/language skill from [skills.sh](https://skills.sh/) — find the relevant one for your stack and reference it in `AGENTS.md`



### 4) Self-Review (Pre-Commit Gate)
**Agent: Reviewer**
- Runs a review pass before anything is staged — CI should never catch what this misses
- For UI-facing work: **Playwright QA loop** on local or preview ([Playwright MCP](https://github.com/microsoft/playwright-mcp); details in `playwright-interactive` below) — inventory from spec + shipped behavior + PR claims; functional then visual pass; one persistent session; proof or documented gaps before commit

**Skills used:**
- [`playwright-interactive`](https://github.com/openai/skills/blob/main/skills/.curated/playwright-interactive/SKILL.md) — persistent Playwright, inventory → functional + visual QA
- [`code-review-and-quality`](https://skills.sh/addyosmani/agent-skills/code-review-and-quality) — correctness, readability, security, perf, spec alignment
- [`security-and-hardening`](https://skills.sh/addyosmani/agent-skills/security-and-hardening) — OWASP basics, input validation, no secrets in code
- [`performance-optimization`](https://skills.sh/addyosmani/agent-skills/performance-optimization) — bundle size, render cost, measure-first



### 5) PR Creation
**Agent: PR Writer** ([GitHub MCP](https://github.com/github/github-mcp-server) + [awesome-copilot](https://github.com/github/awesome-copilot))
- Creates branch, writes PR description from spec, links issues — no manual writing

**Skills used:**
- [`git-workflow-and-versioning`](https://skills.sh/addyosmani/agent-skills/git-workflow-and-versioning) — small commits, clean branch naming, trunk-based
- [awesome-copilot `pr-description`](https://github.com/github/awesome-copilot/tree/main/prompts) — structured title, summary, how-to-test, linked issues
- GitHub MCP — creates branch, opens PR, applies labels, requests reviewers, posts preview URL

**Issue resolution flow:** agent reads open issue via GitHub MCP → generates fix branch → implements fix → opens PR with `fixes #N`



### 6) PR Review Loop
**Agent: PR Reviewer** (AI + CI)
- Reviews like a staff engineer, enforces quality gates before merge

**Skills used:**
- [`code-review-and-quality`](https://skills.sh/addyosmani/agent-skills/code-review-and-quality) — mandatory pre-merge pass
- [`security-and-hardening`](https://skills.sh/addyosmani/agent-skills/security-and-hardening) — catches issues at the PR diff level
- [`debugging-and-error-recovery`](https://skills.sh/addyosmani/agent-skills/debugging-and-error-recovery) — flags risky changes, suggests patches
- [awesome-copilot PR reviewer agent](https://github.com/github/awesome-copilot/tree/main/agents) — GitHub-native review comments



### 7) CI/CD
**Pipeline: [GitHub Actions](https://github.com/features/actions) + [Vercel](https://vercel.com/)**
- Three stages sequential — each must pass before the next runs

**Skills used:**
- [`ci-cd-and-automation`](https://skills.sh/addyosmani/agent-skills/ci-cd-and-automation) — workflow authored and maintained by the skill
- [`shipping-and-launch`](https://skills.sh/addyosmani/agent-skills/shipping-and-launch) — env var validation, health check, rollback trigger

**Stages:**
- **Quality** — lint + typecheck + build
- **Test** — unit + integration
- **Preview smoke** — Playwright against `$VERCEL_PREVIEW_URL`

On failure: agent reads Actions log → diagnoses root cause → pushes patch commit



### 8) Deploy
**Pipeline: [Vercel](https://vercel.com/)**
- Preview on every PR branch · Production on merge to main

**How it works:**
- Agent posts preview URL to PR thread via GitHub MCP after build completes
- [`shipping-and-launch`](https://skills.sh/addyosmani/agent-skills/shipping-and-launch) validates env vars + hits health endpoint post-deploy
- Failed health check → automatic rollback via [Vercel REST API](https://vercel.com/docs/rest-api)



### 9) E2E Testing
**Agent: Runner** ([Playwright MCP](https://github.com/microsoft/playwright-mcp) + [Chrome DevTools MCP](https://github.com/bjesuiter/mcp-chrome-devtools))
- Executes real user flows on live preview · captures traces, logs, screenshots

**Skills used:**
- [`browser-testing-with-devtools`](https://skills.sh/addyosmani/agent-skills/browser-testing-with-devtools) — real browser validation against spec acceptance criteria
- [`debugging-and-error-recovery`](https://skills.sh/addyosmani/agent-skills/debugging-and-error-recovery) — reproduce → isolate → report

**Output:** Structured test report posted back to PR as a review comment via GitHub MCP. Loop closes — acceptance criteria written in step 2, verified here.



### 10) Debugging + Failure Handling
**Agent: Debugger**
- Investigates CI failures, test regressions, deploy issues autonomously

**Skills used:**
- [`debugging-and-error-recovery`](https://skills.sh/addyosmani/agent-skills/debugging-and-error-recovery) — reproduce → isolate → hypothesize → fix → verify
- Chrome DevTools MCP — live console + network inspection on the running preview



### 11) Learning Loop
**Agent: Feedback**
- Converts failures → regression tests · updates agent instruction files

**Skills used:**
- [`context-engineering`](https://skills.sh/addyosmani/agent-skills/context-engineering) — better context = better agent outputs over time

**What gets updated after every incident:**
- New rule added to `AGENTS.md` / `.cursorrules`
- New schema tightening at the failure boundary
- Flaky test → converted to a pinned regression test
- Failed prompt → refined and versioned in `.github/copilot-instructions.md`



### 12) Document
**Agent: Documenter**
- Agents have no memory across sessions — the wiki is their long-term memory. Without it, every session rediscovers the same decisions and risks undoing deliberate choices.

**Skills used:**
- [`documentation-and-adrs`](https://skills.sh/addyosmani/agent-skills/documentation-and-adrs) — decisions, options rejected, reversal cost
- [`context-engineering`](https://skills.sh/addyosmani/agent-skills/context-engineering) — structure knowledge so future agents load the right context without noise

**What gets documented** (lives in `docs/wiki/`): specs + drift · ADRs · incident post-mortems · stable patterns · skills/MCP reference

**Maintenance model:** Inspired by [Karpathy's LLM Wiki](https://gist.github.com/karpathy/b552e21f2a9a452a50b99a7dc7ce8bb0) — raw sources in `docs/raw/` (immutable), agent processes them into wiki pages, rules encoded in `AGENTS.md`. Knowledge compounds instead of resetting.



## Key Design Choices
- **Standard instruction files** — `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `copilot-instructions.md` mean any agent runtime picks up the same conventions without re-prompting
- **Skill-driven** — each step maps to a proven engineering workflow, not a one-off prompt
- **Small agents > one big agent** — each phase has a clear input, output, and failure mode
- **Structured outputs everywhere** — validated contracts at every boundary
- **Read before you write** — every agent invocation starts with `AGENTS.md` + `SPEC.md`

