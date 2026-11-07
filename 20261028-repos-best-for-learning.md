# Which repo teaches the most for an AI engineering role? (2026-10-28)

This is a judgment from the READMEs, session notes, plans and git history. I haven't read every line of code in every repo, so the less documented ones are ranked on what the notes show. I scored each repo on four things:

- **Agent engineering:** building and operating agents, tools, MCP, safety and evaluation.
- **System design:** concurrency, data flow, consistency, failure handling and scale.
- **Production engineering:** deployment, infrastructure, CI, security and cost.
- **Working with AI agents:** how much the repo builds the habit of directing a coding agent well (specs, decision logs, review loops).

## Ranked list

### 1. minimal-agent — the best match for the job you want
- **What it teaches:** how an agent works inside. That means the ReAct loop, tool calling, sessions, MCP client setup, tracing, graceful shutdown with process groups, and an **approval gate** (human-in-the-loop). Interviews for agent roles test exactly this.
- **Why it ranks first:** it's in Go, so it teaches concurrency along the way. The `DECISIONS.md` and `PLAN-*.md` files show real design practice: open questions settled, deferred items tracked, trade-offs written down. It's also your most active repo (150 commits, recent work).
- **Gaps to fill:** there's no evaluation harness yet. Building one (a task suite that scores whether a run succeeded) is the most valuable next thing for hiring signal.
- **How to get the most from it:** be able to explain, without notes, why the approval gate uses a deny list, how tracing avoids leaking secrets, and what you deferred and why.

### 2. verispend — agent safety and a real product slice
- **What it teaches:** designing an MCP tool contract (`request_purchase`, `check_approval`, `record_outcome`), policy evaluation, budget accounting, human escalation, a hash-chained append-only ledger, signed tokens, and Cloudflare Workers deployment. It's the most direct "AI safety and human-at-the-helm" example you have.
- **Why it ranks second:** it covers more of a product's life than minimal-agent: production incidents (the signing-key error), security review by Copilot, and phased delivery (Phases 1–4). Debugging a real prod failure is worth a lot in an interview story.
- **Gaps to fill:** idempotency and race conditions in budget accounting, plus tests that simulate a misbehaving agent.
- **How to get the most from it:** write a short design doc covering threat model, ledger design and failure modes. It doubles as a portfolio piece.

### 3. kinnoo — the biggest end-to-end system, and a lesson in scope
- **What it teaches:** a CLI, a registry service on AWS (Lambda, RDS), a Next.js app on Cloudflare, archive signing and integrity checks, CI/CD, Terraform, and a secret-scanning and public-release process. With 1292 commits and dozens of "understanding-*" notes, it's the deepest engineering history you have.
- **Why it ranks third and not higher:** much of it isn't about agents, and the product direction has been unsettled. But the infrastructure and release discipline are real experience that few candidates can show.
- **The lesson:** why scope grew faster than users, and what the pivot to compliance-as-code is meant to fix. Being able to tell that story honestly is useful in interviews.
- **How to get the most from it:** pick one subsystem (signing and verification is the best candidate) and be able to explain it end to end.

### 4. mcp-server-mac-os — protocol discipline and safe tool design
- **What it teaches:** MCP over stdio (why stray stdout breaks the protocol), a declarative manifest registry that builds argv, parameter validation, and preview-then-approve flows for dangerous actions. It also has 298 commits and a documented bug trail.
- **Why it matters for the role:** MCP servers are a common integration point for agents, and the safe-by-default design (previews, approvals, audit) is what employers want to see.
- **Gaps to fill:** the trash name-collision bug is a good case study in batch operations and staging. Fixing it and writing the post-mortem is a small, concrete learning task.

### 5. system-design-load-balancer — fundamentals that interviews test
- **What it teaches:** goroutine safety, health checks, target groups, concurrency limits, and failure modes such as a backend that answers `/health` but times out on real traffic.
- **Why it ranks fifth and not higher:** it's a learning exercise by design. But system design is a direct gate for the roles you're targeting, and this is your best preparation for it.
- **How to get the most from it:** run the health-check-versus-real-traffic experiment, record the result, and extend it with retries, backoff and circuit breaking. Those are the same patterns agents need when calling flaky tools.

### 6. learn — core language and distributed-systems basics
- **What it teaches:** Go channels (blocking and unblocking), slices, and Consul first principles, plus the clean-code skill.
- **Why it matters:** this is where depth gets built. It's low in the list because it's notes, not systems.

### 7. gamecafe — real-time state synchronization
- **What it teaches:** a server-authoritative game state shared by a board and several phones, hidden hands, turn handling, reconnection, local DNS and media serving (HTTP Range for Safari).
- **Why it matters:** multi-client state consistency is a real system-design topic, and rules engines are a good testbed for agent-written code with tests.

### 8. openlate-core — third-party API integration and cost control
- **What it teaches:** fan-out queries, result filtering, fallbacks, response caching (fresh and stale), a daily budget counter that refunds on failure, and tests for all of it.
- **Why it matters:** cost control and caching around expensive APIs is exactly what LLM-backed applications need.

### 9. duebly and duebly-core — sync and auth
- **What it teaches:** last-write-wins sync with tombstones, offline-first design, JWT verification against JWKS, and R2 snapshot storage.
- **Why it matters:** consistency trade-offs are a standard system-design topic, and you have a working example.

### 10. webplay — generative AI in production
- **What it teaches:** an AI music pipeline (generation, selection by seed, fades, segmenting) feeding a live stream. It's the only repo with a real generative-AI production pipeline.
- **Why it ranks tenth:** it's media generation, not agents. It also has an operational lesson: build tooling inlined a deployment credential, which is the kind of mistake worth learning from.

### 11. letcoins — LLM-with-tools in a small package
- **What it teaches:** an LLM call with web search, permanent caching in D1, a budget on API calls (CoinGecko limits), and graceful degradation when the key is missing.
- **Why it ranks eleventh:** it's small, but it's a clean example of LLM cost and fallback design.

### 12. mcp-os and boxmail — design thinking without much code
- **mcp-os:** its code review of a guarded script-execution design (generate, validate, approve, execute) is worth rereading as threat-modeling practice.
- **boxmail:** a plan-heavy repo about compliance-aware architecture. Good for product and requirements thinking, but there's little to run.

### Lower value for this goal
**rubikx** (Swift and RealityKit), **sodigames**, **hubseq_frontend**, **hubseq_terraform**, **waste-app**, **freelance**, **bitploy**, **openlate** (public mirror) and **system-design-resilient-mesh** (README only, not started) don't build agent or system-design skills, though some have other value.

## Thoughts

**1. Your strongest signal is in agent safety.** minimal-agent, verispend and mcp-server-mac-os all share a theme: human-in-the-loop approval, audit and safe defaults. That's coherent and matches the career direction in your load-balancer README. Make it the thread of your story, not three unrelated projects.

**2. You have breadth. Depth and evidence are what's missing.** Employers will ask how you know your agent works. None of the repos has an evaluation suite. Adding one to minimal-agent would be the single most valuable thing for hiring.

**3. Other gaps against typical AI engineering job descriptions:**
- Retrieval (RAG) and embeddings.
- Multi-agent orchestration, or at least one agent delegating to another.
- Observability beyond logs: dashboards or trace viewing.
- Prompt and model evaluation, including cost and latency tradeoffs.
- A deployed agent that real people use.

**4. Working with AI agents is already a visible strength.** Many repos have `prompts/` session logs, `DECISIONS.md`, plans, Copilot PR-review loops and recorded review resolutions. Few candidates leave a paper trail like this. Turn it into evidence: a short write-up of how you direct coding agents (spec first, small PRs, review loops, what you don't delegate) is likely worth more than another project.

**5. Finishing beats starting.** Several repos have finished work that is uncommitted (minimal-agent, gamecafe, openlate-core, rubikx, verispend). A public repo with a clean history, a README and a demo reads better than a larger private one.

## Suggested next 90 days

1. Commit and merge minimal-agent's approval gate, then add a small evaluation harness (even 10 tasks with pass or fail).
2. Fix verispend's production error, then write its threat model and design doc.
3. Run the load-balancer health-check experiment and write up the results.
4. Write one page on how you work with coding agents, and link it from your READMEs.
5. Add one missing area, such as RAG or a second agent, to a project you already have, instead of starting a new one.
