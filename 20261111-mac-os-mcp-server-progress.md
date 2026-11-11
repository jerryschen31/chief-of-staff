# macOS MCP Server: Progress and Next Steps

Repo: `~/gh/public/mcp-server-mac-os` (GitHub org `kinnoo-project`)
Snapshot taken 2026-11-11 from git history, `docs/`, `notes/` and `prompts/` session logs.

---

## TL;DR

- **What it is:** a Go program that lets an AI client (Claude Code, Claude Desktop, or your own Go code) operate your Mac in plain language: files, Mail, Calendar, Messages, Photos, Wi-Fi, screenshots, backups, and more. The client starts it as a subprocess and talks to it over stdin/stdout. There is no port or URL.
- **Size today:** **174 operations in 23 domains**, exposed to the model as **26 tools**. Tagged up to **v0.4.1**. 71 PRs merged.
- **Built:** a safe execution engine (no shell, allowlisted binaries, preview → confirm → undo for anything that changes state), then three waves of features (May–July), Maps (Aug), and one small fix (plain `.tar`, Aug 29).
- **Since then (Sept):** no code changes. You compared it against `macos-automator-mcp` and got a ranked improvement plan, which you **did not approve**. You also found a **real bug in batch file deletion** that is still unfixed in code.
- **Where you are:** branch `feat/explore-improvements`, in sync with origin. Its only commit is the bug write-up.
- **Suggested first task:** fix the Trash-collision bug (below), then decide on the September improvement plan.

---

## 1. How it works, in plain terms

You need these ideas to follow the rest of the document.

**Capability / operation / domain.**
- A **capability** is one JSON entry in `internal/registry/manifests/*.json`, for example "list a directory" or "send an iMessage". It records which program to run, what parameters it takes, how risky it is, and how to undo it.
- **Operation** is the name the model uses for that same entry when it calls it: `filesystem(operation: "list_directory", ...)`.
- A **domain** is a group of operations, such as `filesystem`, `application-mail` or `network`. Each domain is **one MCP tool**. Its description lists every operation in the group.
- Adding an operation means adding a JSON entry. That makes one tool's description longer, but the number of tools stays the same.

**The 26 tools** are the 23 domain tools plus three shared tools:
- `execute` runs a change that was previewed earlier.
- `undo` reverses a change that already ran.
- `pipeline` chains several read-only steps on the server, like a safe `cmd1 | cmd2` with no shell.

**Reads vs. changes.**
- **Reads** run immediately.
- **Changes** are previewed first. The server works out the exact command it will run and the exact command that would undo it. It returns a preview and a single-use token. Nothing happens until the client calls `execute` with that token. If the change can be undone, `execute` returns an undo token.
- A few low-risk changes, such as opening a URL or posting a notification, skip the preview ("auto-commit").
- Deletes always go to the Trash, so they can be undone.

**Safety rules the code enforces:**
- Programs run directly with argument lists. Nothing goes through `sh -c`.
- Only binaries in `/usr/bin`, `/usr/sbin`, `/bin` and `/sbin` can run. Risky ones (`diskutil`, `tmutil`, and similar) are limited to specific subcommands.
- AppleScript gets data through `--`, so text from the model can never be read as a command-line option.
- Command output is capped at 32 KB, and every command times out after 2 minutes.
- Anything that needs admin/root is **out of scope by design**. In those cases the server opens the right System Settings pane so you can click through it yourself.

**Platform:** macOS 13 Ventura or later, Universal 2 binary (arm64 + amd64), stdio transport only.

**Quality process:**
- Every feature is one PR into `build`, reviewed by Copilot, with the review written up in `docs/pr-reviews/prNN.md`.
- Every feature gets a design note in `docs/issues/`.
- An **eval harness** (`go run ./cmd/runevals`) checks that a real model picks the right tool and operation. It has about 167 cases: 114 automated and 53 manual ones that need permissions granted.

---

## 2. What has been built, in order

### Phase 0: Scaffold and first read-only server (May 26–30, 2026)
- Generated the initial code with kinnoo.ai and set up the Claude agent files.
- Wrote notes listing every macOS command-line tool available on Sequoia, plus a note on why a Mac MCP server is worth building (`notes/2026052*`).
- Built the first server: read-only filesystem tools (`find`, `stat`, `wc`, `du`, `grep`, `pwd`, and others).
- Fixed tools returning empty output and hardened `find` path parsing. Merged as **PR #1**.

### Phase 1: The "read-only spine", the engine rewrite (June 16–17)
- Replaced the one-tool-per-command design with a **registry + engine**:
  - **Slice A:** capability types, the loader, and validation.
  - **Slice B:** the engine, the allowlist policy, and a working read path.
  - **Slice C:** moved the existing filesystem commands onto the new design.
  - **Slice D:** the real entry point; deleted the old code.
- Spec: `docs/specs/capability-engine-readonly-spine.md`.
- **Domain-tool surface:** one tool per domain (described in §1) replaced the earlier generic verbs (`list_capabilities`, `plan_action`, ...).
- Added `largest_files`, which ranks the top-N files by size in one call.

### Phase 2: Making changes safely (June 17–18)
- Built the preview → execute → undo flow with single-use tokens that expire. It was first tested on `mkdir` and `write_setting` (a curated `defaults write` in a new `preferences` domain).
- Built the **eval harness** for checking tool selection.
- Added the read-only **`pipeline`** tool: up to 5 stages, 1 MiB passed between stages.

### Phase 3: Apple apps and system domains, "impl4–impl18" (June 18–23, PRs #7–#31)
In order:
1. **Mail:** Spotlight search and `send_mail` with attachments. Added protection against AppleScript option injection.
2. **Calendar and Reminders:** create, move and cancel events and reminders, all undoable.
3. **Phone:** `find_contact` and `call` (cellular or FaceTime).
4. **Messages:** read and search conversations, send iMessage. Recovers message text from `attributedBody` when the plain text field is empty.
5. **Apps, printers and system:** launch, focus and quit apps; print; read Wi-Fi, Bluetooth and battery state; open System Settings panes. Added the "auto-commit" path for low-risk changes.
6. **Open file** and **open website**. Hardened URL checks: allows `host:port`, rejects embedded `user:pass@`.
7. **Notes.**
8. **Screenshot:** `capture_screen`, saved to `~/Pictures/Screenshots`.
9. **Network diagnostics:** IP, DNS, gateway, ping and DNS lookup, all read-only.
10. **Processes:** CPU, memory and GPU load, launchd agents, and graceful quit/terminate. There is deliberately no force-kill, and it refuses to terminate GUI apps by PID.
11. **File move, copy and remove**, all undoable through the Trash. Raised the output cap to 32 KB.
12. **Security test gate:** an injection sweep test and a security-invariants test. Added the wall-clock timeout and a free-space check before copying.
13. **iMessage attachments fix:** files are now copied into Messages' own sandbox before sending.
14. **Photos:** search, export, favorite, rename, keywords and albums. There is no delete, because Photos automation can't do it.
15. **Shorter tool menus** so long tool descriptions don't get cut off, plus **batch move** via `source_glob`.

### Phase 4: Eval "Layer B" (July 1, PRs #32–#34)
- Evals now also check outcomes. They use fixtures and verify the resulting state, not only which tool was picked.
- Added the automated (CI-safe) and manual corpora, live progress output (`-verbose`), and the `/ship-pr` and `/runevals` skills.

### Phase 5: "U units" U0–U17, filling everyday gaps (July 1–3, PRs #35–#56)
| Unit | What it added |
|---|---|
| U0 | Engine can pass input through stdin (needed for clipboard writes) |
| U1 | `write_file`, `append_to_file` |
| U2 | About This Mac, uptime, disk usage, software update check |
| U3 | More System Settings links (Focus, Keyboard, Apple ID) |
| U4 | `spotlight_search` (`mdfind`) |
| U5 | New **clipboard** tool: read and write |
| U6 | `notify` and `speak`, so the agent can get your attention |
| U7 | AirPlay device discovery, keyboard input-source reads |
| U8 | `read_setting`, `set_appearance` (light/dark) |
| U9 | `compress` and `extract` (via `bsdtar`) |
| U10 | Mail `list_inbox` and `read_message` |
| U11 | Window management: list, move, resize, minimize |
| U12 | Screenshot of a region or a single window |
| U13 | New **Safari** domain: list tabs, current tab (read-only) |
| U14 | New **Contacts** domain: read a full card, create a contact |
| U15 | `display_sleep`, `wifi_set_power` (opens Settings when admin is required) |
| U16 | New **Music** domain: now playing, play/pause/skip |
| U17 | App Store search and open a product page |

After U17:
- A full eval run found 2 bugs, both fixed (PR #54).
- `wifi_status` was reporting "not connected" when Wi-Fi was connected. It now reports the network actually joined and the signal strength (PR #56).
- The Stocks app was investigated and judged **infeasible** (`docs/ideas/stocks-capability-deferred.md`).

### Phase 6: "V units" V0–V10, more of the built-in macOS binaries (July 4–5, PRs #57–#68)
An audit found the server used only **49 of 1,249** binaries in the trusted directories (`docs/ideas/darwin-binary-coverage-audit.html`). You chose 11 groups to add. Decisions you made along the way:
- Keychain access returns metadata only, never secrets.
- SSH opens Terminal.app.
- Destructive disk operations such as eject are **never executed**. The server returns the exact command for you to run yourself.
- Key remapping only allows a fixed list of common remaps.

| Unit | What it added |
|---|---|
| V0 | The audit and the roadmap (`docs/ideas/capability-expansion-phase2.md`) |
| V1 | Network: traceroute, whois, route table, interface stats, DNS cache lookup and flush |
| V2 | Files: image info, convert, resize (`sips`), document conversion (`textutil`), Quick Look thumbnails |
| V3 | Keep-awake via `caffeinate`: `keep_awake`, `allow_sleep`, `sleep_assertions` (the engine can now run detached processes) |
| V4 | `system_log` (`log show`), `top_processes`, `thermal_state` |
| V5 | New **security** domain: code-signature check, Gatekeeper check, SIP status, quarantine info |
| V6 | Keychain metadata ("do I have a saved password for X?", without revealing it) |
| V7 | New **storage** domain: Time Machine status and backups, volumes, mount/attach disk images; eject returns the command to run |
| V8 | New **shortcuts** domain: list your Shortcuts, run one by name (previewed first) |
| V9 | Key remapping (`hidutil`, e.g. Caps Lock → Escape), status of sharing services |
| V10 | SSH: list key fingerprints and `~/.ssh/config` hosts, staged "open Terminal and ssh to X" |

### Phase 7: Maps (August 1, PR #69)
- New `application-maps` domain: `directions`, `search_locations`, `show_location`.
- These **open Maps.app with the answer on screen**. They cannot return distances or ETAs into the chat, because Maps has no scripting interface.
- A design for getting that data back through Shortcuts was written up and **deferred** (`docs/ideas/maps-data-tier-deferred.md`).

### Phase 8: Small fixes and exploration (Aug 29 – Sept 25)
- **Aug 29:** `extract` can now unpack plain `.tar` files (PR #71).
- **Sept 12:** you compared this repo with steipete's `macos-automator-mcp` and got a ranked improvement plan. You **rejected the plan when it was presented** and nothing from it was built (see §3.2).
- **Sept 20:** you used the server to find what was using disk space. That session uncovered the Trash bug in §3.1.
- **Sept 25:** you asked how to call the server from another Go codebase. The answer: launch `bin/macos-darwin-mcp` with the SDK's `mcp.CommandTransport`. No URL is needed.

### What it covers today, by domain
`filesystem` · `application` (apps, windows, App Store) · `application-mail` · `application-calendar` · `application-reminders` · `application-messages` · `application-phone` · `application-notes` · `application-photos` · `application-contacts` · `application-music` · `application-safari` · `application-maps` · `screenshot` · `clipboard` · `printer` · `system` · `network` · `process` · `security` · `storage` · `shortcuts` · `preferences`
Plus `execute`, `undo` and `pipeline`.

---

## 3. Planned next steps, in suggested order

### 3.1 Fix the batch-delete bug first (correctness and safety)
Write-up: `docs/issues/bug-remove-trash-collision-batch-staging.md`.

**Status:** the write-up ends with `**fixed**`, but no code under `internal/` has changed since Aug 29, so it is **not actually fixed**. Correct that label, or fix the bug.

There are two problems:
1. **The Trash name is chosen when the change is previewed, not when it runs.**
   - Example: you preview 18 removals of folders all named `.venv`. Nothing has run yet, so all 18 previews pick `~/.Trash/.venv`.
   - When they run, the first move works, the second lands inside the first, and the rest fail.
   - The same helper (`trashPathFor` in `internal/engine/mutate_filesystem.go`) is used by several other undo paths, so those are affected too.
2. **A failed command is still reported as "Applied" and returns an undo token.**
   - Using that token could move the wrong item back.
   - Check whether this happens for every staged operation (the shared execute path) or only for `remove`.

Fix:
- Choose (or re-check) the Trash name when the command runs.
- Treat a non-zero exit code as a failure: no "Applied" message and no undo token.
- Add the two regression tests described in the write-up.

### 3.2 Decide on the September improvement plan (not yet approved)
From the `macos-automator-mcp` comparison (`prompts/20260912-session-1.md`). The plan's conclusion: keep this repo's safe design, and borrow the other project's polish and app coverage ideas.

**Tier 1: small infrastructure wins, no behavior changes**
1. **MCP tool annotations.** Mark read-only domains (`security`, `screenshot`, `application-safari`) with `readOnlyHint`, so Claude Code stops asking for approval on those reads.
2. **End-to-end stdio test.** Build the real binary and talk to it through the SDK client. This catches stray writes to stdout that would break the protocol.
3. **`--version` flag** with the version set at build time. Today the code hard-codes `0.4.0` while the latest tag is `v0.4.1`.
4. **`CHANGELOG.md`** and **Dependabot**.

**Tier 2: user-experience and coverage**

5. **A one-line purpose and required-permission blurb per domain.** Every domain description currently starts with "Perform a macOS X operation."
6. **One shared classifier for AppleScript permission errors.** Today 11 separate helpers each map errors like -1743 or -25211 to the right Settings pane.
7. **Prebuilt Universal 2 release binary** (unsigned, with checksums), plus a note on removing the quarantine flag. This needs your OK because it publishes a downloadable file.
8. **New operations:**
   - `get_volume`, `set_volume`, `mute`
   - Finder: the current selection, and "reveal in Finder"
   - Chrome: list tabs and current tab (read-only)

**Tier 3: optional**

9. Use JXA (JavaScript for Automation) with JSON output for *new* AppleScript-based reads.
10. Enforce the binary deny-list when manifests load, not only in tests.

### 3.3 Standing roadmap (README "Roadmap" section)
- **Wider eval coverage:** test with more models (currently only `claude-sonnet-4-6`), and add cases for each new domain.
- **More file operations:** batch `copy` and `remove` with `source_glob`, and overwrite-with-backup. Batch remove depends on the §3.1 fix.
- **Multi-step change plans:** preview and run several changes as one unit. On failure, stop and report what was completed, and let you undo those steps. This is the biggest open design problem (`docs/issues/issue-composition-and-transactional-rollback-limitations.md`).
- **Force mode:** an opt-in to skip the preview for low-risk, undoable changes. It would never apply to changes that can't be undone.
- **More app depth:** more `preferences` settings, more `application-*` operations, and changes (not just reads) in domains like network.

### 3.4 Parked or deferred (each has a note in `docs/ideas/` or `docs/issues/`)
- **Maps data through Shortcuts:** use Shortcuts' "Get travel time" and nearby-places actions to return real distances and ETAs.
- **Stocks:** judged infeasible.
- **Signed and notarized binary:** waiting on an Apple Developer account (`note-prebuilt-binary-distribution.md`).
- **Settings that need admin rights:** toggling Bluetooth, Low Power Mode, printer enable, and similar. These deliberately stay as "opens the Settings pane" (`issue-system-control-admin-gated-gaps.md`).
- **Mail search through AppleScript, and listing vs. searching Photos:** known limitations, documented.

### 3.5 Housekeeping
- **Branches:** you're on `feat/explore-improvements`. `build` and `main` are in sync. About 35 merged feature branches remain locally and could be deleted.
- **Stale counts:** domain, operation and test counts have drifted across README, `PROJECT_SUMMARY.md` and `notes/20260703-capabilities-summary.md` (which says 22 domains / 171 operations). There is a note about this: `docs/issues/note-doc-counts-drift.md`.
- **Your process:** one PR per unit via `/ship-pr`, Copilot review, a review write-up in `docs/pr-reviews/`, a design note, and new eval cases. `/runevals` before tagging a release.

---

## 4. Where to look first when you pick it back up
1. `PROJECT_SUMMARY.md` and `CLAUDE.md`: the goals and the rules every change must follow.
2. `docs/architecture.md`: the full list of operations, how changes flow, the security model, and a map of the code.
3. `internal/server/tools.go` (`Register`) → `internal/server/menu.go` → `internal/engine/`: how a tool call becomes a command.
4. `docs/issues/bug-remove-trash-collision-batch-staging.md`: the first fix to make.
5. `prompts/20260912-session-1.md`: the improvement plan to accept, trim or drop.

## Summary
What's built: 174 operations in 23 groups, which the AI client sees as 26 tools. The latest release tag is v0.4.1. The work came in eight phases:
- May: a first server that could only read files.
- June: a rewrite into a central engine driven by JSON files, plus a way to make changes safely: preview, confirm, then undo if needed.
- June: support for the Apple apps (Mail, Calendar, Reminders, Phone, Messages, Notes, Photos), plus system, network and process tools.
- July 1–3: 18 smaller units (U0–U17) filling everyday gaps, such as clipboard, windows, Safari, Contacts and Music.
- July 4–5: 11 units (V0–V10) adding more of the Mac's built-in command-line tools, such as security checks, Time Machine and disks, Shortcuts, and SSH.
- August: Maps, which opens Maps.app with the answer on screen but can't return distances or travel times to the chat.
- August–September: a fix for unpacking plain .tar files, then some exploring.

There have been no code changes since August 29.

Two things to know before you start:

- There's a real, unfixed bug in deleting several items at once. The write-up in docs/issues/bug-remove-trash-collision-batch-staging.md is labelled "fixed", but no code has changed since it was written.
  - If you queue up several deletions of items with the same name (for example, many .venv folders), they all try to go to the same place in the Trash, so most fail.
  - Those failed deletions still report "Applied" and hand back an undo token, and using that token could move the wrong item back.
  - I put this first in the next steps.
- You rejected the improvement plan from September 12. It came out of comparing your server with macos-automator-mcp, and nothing from it has been built. The file lists it as a ranked set of items for you to accept, trim or drop. The top ones are labels that stop Claude Code asking for approval on read-only tools, an end-to-end test of the real program, a --version flag, a downloadable prebuilt program, and volume, Finder and Chrome-tab operations.

The rest of the next steps come from the project's README roadmap: test against more models, copy or remove many files at once, run several changes as one unit with rollback, and an opt-in "force" mode that skips the preview. The file also lists what's been parked: getting Maps data back through Shortcuts, Stocks (judged not possible), and a signed release.

