# Which repo is most likely to get real users? (2026-10-28)

This is an opinion based on what's in the repos: code maturity, notes, market and distribution. I have no usage data, analytics or revenue figures, so treat it as informed judgment. I weighed four things:

- **Pull:** is there a real problem and a real audience?
- **Timing:** is the market moving your way right now?
- **Distribution:** can you reach users cheaply?
- **Readiness:** how close is it to something a stranger could use?

## Ranked list

### 1. verispend — the best shot at a real product, in a crowded space
- **Why:** AI agents that spend money need guardrails. VeriSpend gives them a policy check, human escalation and a tamper-evident audit ledger, all behind an MCP interface agents already speak. The need is growing as agentic payments arrive, and the buyer (finance or compliance) pays for control and audit.
- **Readiness:** it's already deployed with email approvals, a dashboard, hash-chained audit and Phase 4 reports. Few of your other repos got this far on a B2B idea.
- **Competition (added 2026-10-28):** agent payments is a crowded space. At least one funded startup is already in it. Its public homepage describes agent identity tokens, an agentic wallet (tokenized cards plus stablecoins), an automated purchasing feature and per-transaction user mandates (merchant, product, amount). Pricing isn't listed. I read only the homepage, so its docs may show more. Card networks and payment vendors are moving the same way. Expect more entrants, not fewer.
- **Where a wedge could still exist:**
  - **Governance, not payment.** The competitor sits in the payment path. VeriSpend doesn't move money. Agents call it before spending with their own cards or accounts. That makes it complementary: an organization could pay through a competitor and govern with VeriSpend.
  - **Policy depth.** Versioned org policy, per-agent and org budgets, category and vendor rules, and an approver queue go beyond the per-transaction mandates the competitor's homepage describes.
  - **Audit.** A hash-chained ledger that finance can re-verify on demand is a compliance feature, and the competitor's homepage doesn't mention audit.
  - **Self-hosted or open source.** Some teams won't send payment policy to a startup's cloud.
  - **Vendor verification.** The idea in `docs/ideas/vendor-verification-trust-layer.md` is a different angle from spend approval and may be less crowded.
- **Risks:** it has had a production error since July, and the fix is waiting on your go-ahead. The bigger risk is that a competitor or the card networks add policy and audit features, since they have the funding and the rails. Enterprise sales are slow. Distribution is still unsolved.
- **What you'd learn by building it, even if it never finds users:** a working MCP server with a real tool contract; policy evaluation and budget accounting; an append-only hash-chained audit design with on-demand verification; human-in-the-loop approval flows; and secrets and signing key handling. These are the core skills of AI agent infrastructure, which you've said is your career direction. As a portfolio piece it holds up even if the market doesn't. That lowers the cost of continuing.
- **What would help most:** fix prod, then talk to 3–5 teams running purchasing agents before adding features. Ask what they use today and whether a governance layer on top of it would matter to them. Their answers will show whether the wedge is real. Treat the open-source and portfolio angles as the fallback, not a failure.

### 2. mcp-server-mac-os — most likely to be used by strangers
- **Why:** "Talk to your Mac" through Claude is a clear, demoable value proposition. It's open source, well audited (previews and approvals before changes), and has 298 commits, the deepest iteration on any non-kinnoo project. The MCP community shares tools freely, so distribution costs little. It's also the thing you'd use yourself daily.
- **Risks:** it's macOS-only. Apple or Anthropic could ship overlapping features. It's free and AGPL, so it grows users but not revenue.
- **What would help most:** fix the trash-collision bug, record a 60-second demo, and post it where MCP users look.

### 3. duebly — already in use, but a crowded market
- **Why:** it's live in production, you use it daily, and it has sync, auth and a payments hook (the Buy Me a Coffee link). It's the only repo that can already be called a shipped product with at least one user.
- **Risks:** task managers are among the most saturated app categories. "Simple and opinionated" is hard to market against Todoist, Things and TickTick.
- **What would help most:** pick a specific niche instead of the general market. Without one, it stays a personal tool.

### 4. kinnoo — big ambition, uncertain pivot
- **Why:** it's the most developed codebase (1292 commits), with a registry, signing and a web app. The 2026-05-29 pivot to compliance-as-code for platform teams is a real B2B angle.
- **Risks:** you've decided to wind down the private repo, the original "GitHub for agents" idea has a chicken-and-egg registry problem, and you've been rethinking the direction for months.
- **What would help most:** decide whether the pivot is real before spending more time on the import. If it is, build the `kinnoo check` command and test it on your own agents first.

### 5. minimal-agent — strong learning project, weak market position
- **Why:** the approval gate, tracing and safe MCP handling reflect real thought about agent safety, and that aligns with your stated career direction. It's the most active repo right now.
- **Risks:** agent harnesses are a very crowded space with big players. Its best use is as a portfolio piece and a base for your other projects, including VeriSpend.
- **What would help most:** keep it going, but don't count on it for users.

### 6. webplay — fun, with a wild-card audience
- **Why:** it's live, running 24/7, and cheap to try. A distinct channel such as "Cafe for Reading" could attract a small audience. Streams like this can build listeners without any marketing.
- **Risks:** lofi radio already has big, free competitors. Generating the music depends on a GPU at home. There's no revenue model.
- **What would help most:** merge the pending PRs, then watch listener numbers for a month before investing more.

### 7. letcoins — finished, narrow
- **Why:** v1 shipped with a polished feature set, and "learn to read candles" has an audience.
- **Risks:** crypto-education is a hard niche to monetize, API limits (CoinGecko 10k a month) cap growth, and "not financial advice" apps tend to stall.
- **What would help most:** set the Anthropic key so the Explain feature works, and share it once.

### 8. openlate — good idea, hard cold start
- **Why:** finding late-night food is a real, repeated need and the search logic has been tuned.
- **Risks:** it competes with Google Maps on Google's own data, which also drives your costs. A local-discovery app needs users in each city to be useful.
- **What would help most:** pick one city, deploy the pending search work and test it there.

### 9. gamecafe — delightful, hard to distribute
- **Why:** the "tablet board plus phone hands" idea is a good in-person experience, and three games already work.
- **Risks:** Land Conquest and Taco Truck closely mirror Catan and Sushi Go!, which creates IP risk if it goes public (you've renamed things, which helps). Players must install nothing, but they need someone to host it locally.
- **What would help most:** keep it as a family or friends tool unless you build original games.

### 10. sodigames — shipped, no business model
- **Why:** it's finished, ad-free and instantly playable. It's pleasant but there's nothing to hold users.
- **What would help most:** nothing needed. Leave it live.

### 11. rubikx — niche toy
- **Why:** a well-built Rubik's cube simulator, but the gesture bug is unresolved and the audience is small.
- **What would help most:** fix the gesture bug first.

### 12. boxmail — a showcase, by its own plan
- It exists to support a Box job application, and the plan says to stop absent external signal. Its job is to impress one reader, not to gain users.

## Not realistic as products
These are learning projects, archived work or placeholders, so they aren't ranked:

- **learn**, **system-design-load-balancer** and **system-design-resilient-mesh**: learning and interview prep.
- **mcp-os**, **openlate** (public mirror) and **kinnoo** (public mirror): early versions or mirrors of items above.
- **hubseq_frontend** and **hubseq_terraform**: a 2022 product with its backend already shut down.
- **waste-app**, **freelance** and **bitploy**: plans or placeholders with no code.

## Bottom line

- **Best bet for a business:** verispend, with the caveat that the agent-payments space is crowded (see its entry) and the wedge is governance, policy depth and audit, not payments.
- **Best bet for people actually using it:** mcp-server-mac-os.
- **Biggest wildcard:** webplay, because it's cheap to run and you'd learn quickly whether anyone listens.
- **The common problem:** none of these projects has an owner-defined target user or a launch plan. Picking the first 5 users for one or two of these would help more than adding features.
