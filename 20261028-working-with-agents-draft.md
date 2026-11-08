# How I work with coding agents — DRAFT for interview slides and talks

**Status:** first draft, written for you to edit. It's built from what the repos show: session transcripts in `prompts/`, `DECISIONS.md`, `SESSION.md`, `PLAN-*.md`, the agent instruction files, `TASKS.txt`/`EPICS.txt`, the test handoff notes, PR review write-ups and commit history. I haven't read every transcript. Items marked **[verify]** are my inference about *why* you did something, and you'll know better than I do.

---

## Part 1 — The one-pager

### In one sentence
I treat a coding agent as a collaborator whose mode I choose per task: **builder**, **tutor**, **reviewer** or **tester**. I keep the project's memory in files, so the work survives any one session, and I keep a human gate on anything irreversible.

### The five habits that run through every repo
1. **Plan before code, and write the plan down.** Specs and plans come first (`/plan`, `STRATEGY.md`, `PLAN-*.md`, `EPICS → FEATURES → TASKS`). Each task has acceptance criteria and tests.
2. **Memory lives in files, not in the chat.** `CLAUDE.md` sets the rules. `DECISIONS.md` records what was decided and why, including decisions I later reversed. `SESSION.md` is disposable handoff state. Transcripts are exported to `prompts/` so I can recover reasoning.
3. **Choose the agent's role on purpose.** Sometimes it writes the code. Sometimes it writes a *reference build* that I read and then rebuild myself. Sometimes it only explains. Sometimes it reviews.
4. **Verify with something other than the agent's own say-so.** Tests with a fake model, real-browser checks, a secret scan, a second reviewer (Copilot on PRs) and running on a real device.
5. **Humans own irreversible actions.** Production writes, commits and merges wait for my go-ahead. Dangerous tool calls get an approval gate.

### The loop I run
**Spec → plan → small task → build → test and verify → review → record the decision → next slice.**
When the work is new to me, I insert a step: *the agent teaches, then I type it.*

### What I'd say about the evolution
- **Early 2026:** I designed *roles* for agents (tech lead, engineer, git manager) with instruction files and a branching strategy.
- **Spring–summer 2026:** I scaled up to *structured delegation*: numbered tasks, test-agent handoffs and GitHub Copilot opening PRs that I and Copilot reviewed.
- **Late summer–fall 2026:** I moved to *learning-first collaboration*. The agent is a tutor and pair-programmer, I write the core code myself, and decisions are logged as I go.

---

## Part 2 — By repo: how the process differed and what it shows

### waste-app (Jan–Feb 2026) — agents as a team with defined roles
**What's there:** four instruction files: `copilot-instructions.md` (project rules and steps), a **tech lead** agent, a **software engineer** agent and a **git** agent.
- The tech lead must break work into tasks, each with a description, clear acceptance criteria and unit testing, "completed by a separate software engineer".
- The tech lead is told to be critical before major changes and to suggest pitfalls, and to exhaust public options before listing choices for decisions like model selection.
- The engineer must summarize what it did and why after each change, update docs, and *teach me iOS/Swift as we go*.
- The git agent must **analyze and preview a commit** (branch, summary, files) *before* executing it. It is told never to hard-reset history. The branching model is `master` → `build` → `phaseX` → `phaseX/featureY` → `taskZ`, with PRs reviewed by the human and the tech-lead agent.
- `TASKS.txt` is structured: options, pros, cons, criteria, a selected option, a rationale and approvals (for example, the CoreML model decision).

**Takeaway for a talk:** role separation and approval gates, designed before any code. **[verify]** The project stalled after setup, so this repo shows the *process design*, not a shipped result.

### kinnoo (Feb–Oct 2026) — spec-driven delegation at scale
**What's there:** `EPICS.txt` → `FEATURES.txt` → `TASKS.txt` → `TESTS.txt`. Tasks are numbered (task529 and beyond) in YAML with description, files touched, steps, dependencies, tests and status (`needs-review`). Tests and tasks link back to epics. There are about 34 `testNNN-testagent-handoff.md` notes (98 in the public repo), each a short, specific brief for a test-writing agent (for example: "implement this analyzer integration test, cover Python and Node fixtures, assert X, suggested markers…"). About 25 commits mention Copilot, and PR branches named `copilot/...` were merged after review.
- Security process: a gitleaks scan and a written sign-off (2026-04-29) with a conditional go for public release, tied to rotating dev credentials. Import to the public repo is governed by a written runbook.

**Takeaway for a talk:** a lightweight project-management layer that lets agents work in parallel with clear scope. Splitting *implement* from *test* made handoffs small and checkable.
**Honest caveat to prepare for:** 1292 commits and 50+ notes show the process scaled. The product direction did not stay stable. You can speak to what the process caught and what it did not.

### duebly, duebly-core, duedly (spring 2026) — agent-authored PRs with human review
**What's there:** Copilot-authored PRs (`copilot/implement-remote-storage-backend`, `copilot/implement-mvp-feature-set-2`) merged after review. The duebly/duedly repos have brainstorming plans (`brainstorming-plan-*.md`), `MVP-2.md` with "one-shot implementation guardrails" (don't assume a backend, keep toasts lightweight, conflict handling must be non-blocking) and an acceptance-criteria doc (`remote-store-implementation-AC.md`) before the tests.
- Decisions were also made in chat and pasted into the plan. For example, the 5000-task cap came from a Q&A in the brainstorming plan.

**Takeaway:** write guardrails and acceptance criteria *first*, then let an agent implement in one go, then review.

### mcp-server-mac-os (Aug–Sep 2026) — exploration, planning and review loops
**What's there:** 298 commits and about 60 Copilot mentions, so most work went through PRs with Copilot review ("Potential fix for pull request finding"). A session opens with `/plan` to explore another MCP codebase and compare it with this one. Issues are documented in `docs/issues/` (for example the trash name-collision bug).

**Takeaway:** use the agent to survey prior art, decide, then implement in reviewed increments. Keep bugs in writing.

### verispend, openlate-core, webplay (Jul–Sep 2026) — phased plans, review write-ups and gated production actions
- **verispend:** `/plan` against `STRATEGY.md` phases (Phase 4 plan asked to *simulate agent usage and purchases* for tests). The Copilot review resolution is recorded in the repo ("docs: record PR #4 Copilot review resolution"). When a production secret needed changing, the session **stopped and waited for the go-ahead** instead of writing to prod.
- **webplay:** `docs/pr-reviews/pr15.md` documents each review comment and the fix with a commit hash, plus a **credential leak scan** (gitleaks across 49 commits of history and the working tree). It flagged that the build inlines `.env` values. The scan found no leak but surfaced a risk.
- **openlate-core:** session notes finish with what's verified and what is *not* ("I couldn't verify against the live Google API… worth a quick smoke test"). The agent offers to commit but waits.
- **webplay (cafe channel):** iterative creative direction. You rejected a first try ("maybe I didn't prompt correctly"), described the target feel, picked the approved seed, and asked for fades and a rebuild.

**Takeaway:** agents make claims, so the process asks for evidence (hashes, scans, test counts, what was *not* verified). That's the same instinct as an approval gate in an agent.

### gamecafe, rubikx (Jul–Sep 2026) — outcome prompts and device-in-the-loop debugging
- **gamecafe:** a short, outcome-focused prompt (background jazz on the setup page, quiet, looping, on by default). The agent found a hidden requirement (Safari and iOS need HTTP Range support) and verified in headless Chromium.
- **rubikx:** "Keep working and testing until you fix this." When the agent kept failing on the long-press-plus-pan gesture, you **corrected its vocabulary** (what "pan" means to you). It then added an on-screen debug readout and asked you to run it on your iPhone. This is a human-in-the-loop debugging pattern for things only a device can show.

**Takeaway:** when an agent can't observe the thing (a physical gesture), instrument it and let a human be the sensor.

### letcoins (Jul 2026) — scope control and security hygiene
- You pushed back on an overwhelming plan: "As a beginner… let's take this in steps." You also asked for `.env*` in `.gitignore` before anything else. The resulting plan has a **risk table** (token permissions, API budget, key availability, fallbacks).

**Takeaway:** the human sets scope. Security hygiene goes first, not last.

### system-design-load-balancer (Sep–Oct 2026) — the agent as a tutor
- Sessions use `/explain-code` and "next" to walk through code, and probing questions about mental models ("is the actual request logged, then passed to the inner component…?"). At one point: "I'm so confused. I am new to Go and just need to go through the official documentation… myself." That is **knowing when to leave the agent for primary sources**.
- Goroutine and concurrency questions end up as session notes.

**Takeaway:** the goal is understanding, not output. The agent is evaluated by whether *I* can now explain it.

### minimal-agent (Sep–Oct 2026) — the most developed version of the process
**What's there:**
- **Three build modes, stated in `CLAUDE.md`:** `mvp1/` was written by the agent as a readable reference. `mvp2/` and the root code are written by *me*, "tutored step by step": "explain the concept, show the smallest next slice, and wait for him to write/confirm it." `mvp3/` and `mvp4/` (autonomous and orchestrator modes) were written by the agent as templates *before* I built my own, the way mvp1 preceded mvp2.
- **Durable memory with rules:** `DECISIONS.md` (about 2,900 lines) is append-only and chronological. Reversed decisions get a new entry plus a "superseded by" note on the old one, keeping both lines of reasoning. `SESSION.md` is explicitly disposable. Open questions stay in the session file until they become decisions.
- **Decision hygiene:** items are marked "decided", "deferred" or "open". Examples: "Hash chain deferred; threat model first"; "cut to v1. The goal is to learn how hooks are built… not a complete approval system"; a build-order decision to do tracing and approval before the autonomous port.
- **Design interrogation:** many prompts are *questions about trade-offs*: "Is `-maxspend` something that can actually be coded as a constraint?", "Do I need a subagents flag?", "is a slice still efficient here?" Debates with another chatbot are brought in as second opinions. The agent is asked to justify choices with a written scorecard (Go vs TS/Python/Rust).
- **Verification:** tests drive the loop with a scripted fake provider (no network), `-race` after touching concurrency, and real run logs. PR review fixes are logged in `DECISIONS.md` (PR #19, PR #20).
- **Process automation:** `/export` saves transcripts into `prompts/` (gitignored) so earlier reasoning is recoverable.

**Takeaway:** the strongest example. Reference build, then hand-build, with a written decision trail, shows I use agents to *learn faster* without outsourcing understanding.

---

## Part 3 — Material for slides and interview stories

**Candidate story 1 — "Reference build, then rebuild."** For autonomous and orchestrator modes I had the agent write a reference implementation first, read it, then built my own. *Why it works:* it separates getting a design from owning it. **[verify]** Add what you changed from the reference.

**Candidate story 2 — "The decision log caught a reversal."** `DECISIONS.md` mentions "superseded" or "reversed" about 22 times (I haven't checked each one) (for example `model_request` bookkeeping simplified; `command_start`/`command_end` dropped). Pick one and walk through: first design, what the run showed, the revised design.

**Candidate story 3 — "Don't trust, verify."** The webplay credential scan and PR write-up: what the agent claimed, what the scan found, and the risk it surfaced (token inlined in the build bundle).

**Candidate story 4 — "Human as sensor."** rubikx gesture debugging via an on-device readout.

**Candidate story 5 — "Where I stop the agent."** Production secret changes, commits, merges, and the approval gate you built into minimal-agent. The theme connects how you *work with* agents to how you *build* them.

**Slide skeleton (8 slides):**
1. The loop (spec → plan → task → build → verify → review → record).
2. Choosing the agent's role (builder, tutor, reviewer, tester).
3. Memory in files (`CLAUDE.md`, `DECISIONS.md`, `SESSION.md`, transcripts).
4. Structured delegation (EPICS → TASKS, test handoffs).
5. Verification beyond the agent (fake-provider tests, real browser, scans, Copilot review).
6. Learning mode (reference build, then rebuild).
7. Human gates (prod, commits, approval gate).
8. What I'd do differently / what's next.

## Part 4 — What I can't tell from the repos (fill in)
- **Why you switched models and effort levels.** The logs show `/model` changes (Opus, Sonnet, Fable) and `/effort` medium/high from session to session, but not your reasons.
- **What went wrong.** The repos show process, not failures. Interviewers will ask for a time an agent misled you or you wasted time. Candidates: the rubikx gesture bug, the kinnoo scope growth, the webplay token inlining.
- **Quantified outcomes.** Counts like commits, tests, PRs, and time saved would strengthen every section, and I don't have time-based data.
- **Which habits are deliberate and which just happened.** Be ready to say what you'd keep and drop.
