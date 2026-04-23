# Agentic Development Workflow

A full workflow from idea → shipped code → written-down context. Each stage uses a small, focused assistant plus reusable playbooks ([agent-skills](https://skills.sh/addyosmani/agent-skills), [awesome-copilot](https://github.com/github/awesome-copilot)). The same setup works in [Cursor](https://www.cursor.com/), [Claude Code](https://claude.ai/code), [Codex](https://platform.openai.com/docs/guides/codex), or anything that reads the repo's instruction files.


## Repository structure + agent configuration

Standard instruction files committed to the repo root. Every major agent runtime reads at least one — same conventions apply whether you're on Cursor, Claude Code, or Codex.

| Path | Purpose |
|---|---|
| `AGENTS.md` | Top-level agent behavior, goals, boundaries |
| `SPEC.md` | Feature spec, acceptance criteria, tasks |
| `.github/workflows/` | CI/CD pipeline definitions |
| `.skills/` | Skills installed via [skills.sh](https://skills.sh/) |
| `docs/` | Raw sources + wiki (long-lived context) |
| `test-results/` | Test results |

For language or framework-specific conventions (React, Next.js, Python, etc.), find and install the relevant community skill from [skills.sh](https://skills.sh/) and reference it in your instruction files alongside the core agent skills.


## MCP servers

| Server | Role |
|---|---|
| [GitHub MCP](https://github.com/github/github-mcp-server) | Branches, PRs, issues, labels, PR comments (e.g. preview URL, E2E report) |
| [Playwright MCP](https://github.com/microsoft/playwright-mcp) | Drive the browser for real flows (pre-commit QA loop on local/preview, full E2E on staging) |
| [Agent Browser MCP](https://github.com/minhlucvan/agent-browser-mcp) | Browser automation via [Vercel agent-browser](https://github.com/vercel-labs/agent-browser) (fast Rust CLI): navigation, forms, a11y snapshots, sessions; install CLI + `npx agent-browser-mcp` in Cursor |
| [Chrome DevTools MCP](https://github.com/bjesuiter/mcp-chrome-devtools) | Console, network, runtime inspection on a live page |
| [DeepWiki MCP](https://mcp.deepwiki.com/) | AI-indexed documentation for GitHub repos — browse wiki structure, read topic pages, and ask grounded questions so agents orient on upstream or dependency code without a full manual read |
| **Custom / vendor** | Whatever your repo needs (e.g. Vercel, DB, Slack, company tools) |


## The pipeline

```
Idea → Design → Spec → Code → Review → PR/Issues → CI/CD → E2E + debugging → Document/Wiki
```

---

### 1) Design
**Agent: Designer**

Establishes visual direction — branding, UI/UX, design system — before spec or code is written. Outputs (tokens, component names, screen flows) feed directly into `SPEC.md`.

**Tools used:**

[Superdesign](https://app.superdesign.dev/) UI/UX + branding exploration; explore multiple UI directions quickly.

- Reference high quality UI interfaces from various design systems and examples.
- Derive branding: color palettes, typography, and layout systems.
- Export directly into an CLI/IDE of your choice as a prompt 

Result: idea → multiple design directions → validated UI in a single session.

**Skills used:**
- [`impeccable`](https://github.com/pbakaus/impeccable) — builds on `frontend-design` with 7 domain references (typography, color/contrast, spatial, motion, interaction, responsive, UX writing)

**Key commands:**
- `/critique` (UX review)
- `/audit` (performance, accessibility)
- `/normalize` (design system alignment)


---

### 2) Define + Plan
**Agent: Planner**

Refines the raw idea into a spec, then breaks it into atomic tasks with clear acceptance criteria.

**Skills used:**
- [`idea-refine`](https://skills.sh/addyosmani/agent-skills/idea-refine) — clarify problem, constraints, success criteria
- [`spec-driven-development`](https://skills.sh/addyosmani/agent-skills/spec-driven-development) — produce `SPEC.md` before touching code
- [`planning-and-task-breakdown`](https://skills.sh/addyosmani/agent-skills/planning-and-task-breakdown) — atomic tasks with definition of done


---

### 3) Build
**Agent: Builder**

Implements the spec in small, testable slices — never one-shot.

**Skills used:**
- [`incremental-implementation`](https://skills.sh/addyosmani/agent-skills/incremental-implementation) — thin vertical slices, each commit is a working unit
- [`test-driven-development`](https://skills.sh/addyosmani/agent-skills/test-driven-development) — tests written alongside code, not after
- [`frontend-ui-engineering`](https://skills.sh/addyosmani/agent-skills/frontend-ui-engineering) — clean component patterns, accessibility baked in
- Framework/language skill from [skills.sh](https://skills.sh/) — find the relevant one for your stack and reference it in `AGENTS.md`


---

### 4) Self-Review (pre-commit gate)
**Agent: Reviewer**

Runs a review pass before anything is staged — CI should never catch what this misses.

For UI-facing work, this includes a **Playwright QA loop** on local or preview ([Playwright MCP](https://github.com/microsoft/playwright-mcp)): build an inventory from the spec + shipped behavior + PR claims, run a functional then visual pass in one persistent session, and document proof or gaps before committing.

**Skills used:**
- [`code-review-and-quality`](https://skills.sh/addyosmani/agent-skills/code-review-and-quality) — correctness, readability, security, perf, spec alignment
- [`security-and-hardening`](https://skills.sh/addyosmani/agent-skills/security-and-hardening) — OWASP basics, input validation, no secrets in code
- [`performance-optimization`](https://skills.sh/addyosmani/agent-skills/performance-optimization) — bundle size, render cost, measure-first
- [`playwright-interactive`](https://github.com/openai/skills/blob/main/skills/.curated/playwright-interactive/SKILL.md) — persistent Playwright session; inventory → functional + visual QA


---

### 5) PR creation
**Agent: PR Writer** ([GitHub MCP](https://github.com/github/github-mcp-server) + [awesome-copilot](https://github.com/github/awesome-copilot))

Creates the branch, writes the PR description from the spec, and links issues — no manual writing.

**Skills used:**
- [`git-workflow-and-versioning`](https://skills.sh/addyosmani/agent-skills/git-workflow-and-versioning) — small commits, clean branch naming, trunk-based
- [`awesome-copilot-pr-description`](https://github.com/github/awesome-copilot/tree/main/prompts) — structured title, summary, how-to-test, linked issues
- GitHub MCP — creates branch, opens PR, applies labels, requests reviewers, posts preview URL

**Issue resolution flow:** agent reads open issue via GitHub MCP → generates fix branch → implements fix → opens PR with `fixes #N`.


---

### 6) PR review loop
**Agent: PR Reviewer** (AI + CI)

Reviews like a staff engineer and enforces quality gates before merge.

**Skills used:**
- [`code-review-and-quality`](https://skills.sh/addyosmani/agent-skills/code-review-and-quality) — mandatory pre-merge pass
- [`security-and-hardening`](https://skills.sh/addyosmani/agent-skills/security-and-hardening) — catches issues at the PR diff level
- [`debugging-and-error-recovery`](https://skills.sh/addyosmani/agent-skills/debugging-and-error-recovery) — flags risky changes, suggests patches
- [`awesome-copilot PR reviewer agent`](https://github.com/github/awesome-copilot/tree/main/agents) — GitHub-native review comments


---

### 7) CI/CD + deploy
**Pipeline: [GitHub Actions](https://github.com/features/actions) + [Vercel](https://vercel.com/)**

**Skills used:**
- [`ci-cd-and-automation`](https://skills.sh/addyosmani/agent-skills/ci-cd-and-automation) — author and maintain workflows, caches, and log-driven fixes
- [`deploy-to-vercel`](https://skills.sh/vercel-labs/agent-skills/deploy-to-vercel) — linked projects, preview vs production, git-push vs CLI deploy
- [`create-github-action-workflow-specification`](https://github.com/github/awesome-copilot/blob/main/skills/create-github-action-workflow-specification/SKILL.md) — formal spec for an existing Actions workflow (good for AI maintenance and onboarding)


---

### 8) E2E testing
**Agent: Runner** ([Playwright MCP](https://github.com/microsoft/playwright-mcp) + [Agent Browser MCP](https://github.com/minhlucvan/agent-browser-mcp) + [Chrome DevTools MCP](https://github.com/bjesuiter/mcp-chrome-devtools))

Executes real user flows on the live preview and captures traces, logs, and screenshots. Use **Playwright MCP** or **Agent Browser MCP** to drive the browser (both fit pre-commit QA on preview and full E2E on staging); **Chrome DevTools MCP** adds live console and network inspection.

**Skills used:**
- [`browser-testing-with-devtools`](https://skills.sh/addyosmani/agent-skills/browser-testing-with-devtools) — real browser validation against spec acceptance criteria
- [`debugging-and-error-recovery`](https://skills.sh/addyosmani/agent-skills/debugging-and-error-recovery) — reproduce → isolate → report

A structured test report is posted back to the PR as a review comment via GitHub MCP. The loop closes: acceptance criteria written in step 2, verified here.


---

### 9) Debugging + failure handling
**Agent: Debugger**

Investigates CI failures, test regressions, and deploy issues autonomously.

**Skills used:**
- [`debugging-and-error-recovery`](https://skills.sh/addyosmani/agent-skills/debugging-and-error-recovery) — reproduce → isolate → hypothesize → fix → verify
- Chrome DevTools MCP — live console + network inspection on the running preview


---

### 10) Document / Wiki
**Agents: Feedback + Documenter**

Same job from two angles: turn what just happened into durable memory so the next session doesn't rediscover or undo it. Agents have no cross-session memory — the repo (`AGENTS.md`, wiki, ADRs) is the substitute.

**Skills used:**
- [`context-engineering`](https://skills.sh/addyosmani/agent-skills/context-engineering) — structure updates so future agents load signal, not noise
- [`documentation-and-adrs`](https://skills.sh/addyosmani/agent-skills/documentation-and-adrs) — decisions, options rejected, reversal cost

**Long-form capture** (lives in `docs/wiki/`): specs + drift · ADRs · incident post-mortems · stable patterns · skills/MCP reference

**Maintenance model:** Inspired by [Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — raw sources in `docs/raw/` (immutable), agent processes them into wiki pages; standing rules stay in `AGENTS.md`. Knowledge compounds instead of resetting.


---

## Key design choices

- **Standard instruction files** — `AGENTS.md` at the repo root so any agent runtime picks up the same conventions without re-prompting
- **Skill-driven** — each step maps to a proven engineering workflow, not a one-off prompt
- **Small agents > one big agent** — each phase has a clear input, output, and failure mode
- **Structured outputs everywhere** — validated contracts at every boundary
- **Read before you write** — every agent invocation starts with `AGENTS.md` + `SPEC.md`
