<!-- GSD:project-start source:PROJECT.md -->
## Project

**claurst**

claurst is a full Rust rewrite of Anthropic's Claude Code CLI, reimplementing its multi-provider LLM query loop, ratatui TUI, MCP client, slash commands, tool suite, plugin system, and bridge/remote-control protocol. It targets developers who want the Claude Code experience in a compiled, dependency-light binary — and serves as the upstream Rust reference implementation for the kuberwastaken/claurst community.

**Core Value:** A reliable, feature-complete Rust alternative to Claude Code that stays current with upstream changes and fixes bugs fast enough that contributors trust it for daily use.

### Constraints

- **Tech Stack:** Rust only. No new language runtimes. Dependencies must be compatible with workspace resolver v2.
- **Compatibility:** Must maintain CLI and TUI UX continuity across releases — no breaking changes to settings.json schema without migration.
- **Milestones:** New Claude Code features are discovered by the owner and brought in as new milestones — not continuously tracked.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Existing Stack (Do Not Change)
| Category | Crate | Version |
|----------|-------|---------|
| Async runtime | tokio (full) | 1.44 |
| HTTP client | reqwest (stream, native-tls) | 0.13 |
| SSE raw streaming | hand-rolled `sse_parser` in `crates/api/src/lib.rs` | — |
| TUI | ratatui | 0.29 |
| Terminal backend | crossterm (event-stream) | 0.28 |
| MCP SDK | rmcp | 1.4.0 |
| Concurrency | parking_lot, dashmap | 0.12 / 6 |
| Storage | rusqlite (bundled) | 0.31 |
## Domain 1: Async SSE / Streaming LLM Responses
### Current state
### Recommendation: Do NOT add eventsource-stream or reqwest-eventsource
### What actually needs doing
- Implement `SseStreamParser::parse()` for Anthropic/Google (Phase 2A marker).
- The Minimax provider uses `AnthropicClient` with `use_bearer_auth: true` which
- Ollama provider already reads `OLLAMA_HOST` env var from
- Custom OpenAI base URL (#106): `OPENAI_BASE_URL` env var is already wired in
## Domain 2: Terminal UI Patterns
### Current state
- `virtual_list.rs` — a `VirtualList<T: VirtualItem>` with `scroll_offset` and
- `app.rs` — full `handle_mouse_event()` implementing scroll, text selection,
- All major dialogs exist as separate files.
### Mouse capture issue (#104)
### Recommendation: Do NOT add tui-input or tui-textarea
### For missing UI components (agents view, settings parity)
- State machine with a `Mode` enum (list / detail / edit / confirm-delete)
- `StatefulWidget` with `ListState` for navigable lists
- Wrap in an overlay using the existing `overlays.rs` machinery
## Domain 3: MCP Sandbox / Permission Enforcement (Security Issue #123)
### Current state
- `PermissionLevel` (Read / Write / Execute / Network)
- `PermissionRule` with tool name + glob path pattern matching
- `PermissionManager` with session and persistent scopes
- `InteractivePermissionHandler` that sends a `PermissionRequest` over a
### What to add for OS-level sandboxing (optional, Linux-only)
| Crate | Version | What it does |
|-------|---------|--------------|
| `landlock` | 0.4.4 | Linux Landlock LSM — restricts filesystem access for a process without root. Requires Linux 5.13+. |
## Domain 4: Cross-Provider Model Routing
### Current state
### Recommendation: No new crates needed for routing
### For Managed Agents (plan.md)
## Domain 5: Voice / ALSA (#88)
### Current state
## Domain 6: YAML Frontmatter for Agent Files
### Current state
### Recommendation: Add `serde_yml` for agent file parsing
## Gaps Summary: What to Add vs What to Leave
### Add (with justification)
| Crate | Version | Crate for | Confidence | Condition |
|-------|---------|-----------|------------|-----------|
| `landlock` | 0.4.4 | MCP child process filesystem restriction | MEDIUM | Linux only, only if deeper MCP sandbox desired beyond allowlist fix |
| `serde_yml` | 0.0.12 | Agent YAML frontmatter (spec/05 agents subsystem) | MEDIUM | Only when implementing full agents CRUD from spec |
### Do NOT Add
| Crate | Reason |
|-------|--------|
| `eventsource-stream` / `reqwest-eventsource` | Hand-rolled SSE parser in lib.rs is sufficient; adding this creates conflicting abstraction layers |
| `tui-textarea` | Conflicts with existing hand-rolled prompt_input.rs; more work to integrate than to fix |
| `tui-input` | Same reason as tui-textarea |
| `seccompiler` | Wrong tool for the MCP security problem; the fix is a path allowlist, not syscall filtering |
| `async-openai` | Would duplicate all of crates/api; the LlmProvider trait is the abstraction layer |
| `whisper-rs` / `sherpa-onnx` | Explicitly out of scope per PROJECT.md |
| `serde_yaml` | Deprecated as of v0.9.34+; use serde_yml if YAML is needed |
| `tower-lsp` / `lsp-types` | claurst-core already has a hand-rolled LSP client over JSON-RPC; adding tower-lsp would require a complete rewrite of lsp.rs for no behavioral gain |
| `keyring` | Auth is stored in `~/.claurst/auth.json` (plaintext JSON). A keyring would improve security but requires platform-specific library linking (libsecret on Linux) and is not required for parity |
## Pattern Recommendations
### Pattern 1: Per-Provider SSE Implementation (HIGH confidence)
### Pattern 2: MCP Security Fix Without New Crates (HIGH confidence)
### Pattern 3: Provider Registry Extension for Custom Base URL (HIGH confidence)
### Pattern 4: Ratatui Upgrade Path (MEDIUM confidence)
## Confidence Assessment
| Area | Confidence | Reasoning |
|------|------------|-----------|
| SSE streaming patterns | HIGH | Directly read the implementation; hand-rolled parser is proven |
| MCP permission fix approach | HIGH | Permission infrastructure fully exists; issue is application logic |
| Ollama/minimax/custom-URL bugs | HIGH | Root causes confirmed in source; no crate changes needed |
| landlock for MCP sandbox | MEDIUM | Crate is well-maintained official bindings; Linux-only constraint limits scope |
| serde_yml for agent YAML | MEDIUM | serde_yml is young (0.0.x); API stable but not battle-tested at scale |
| ratatui 0.30 upgrade | MEDIUM | API is mostly additive; risk is in the 12 crates that use ratatui |
| Voice/ALSA fix | HIGH | Bug is in configuration/state management, not audio stack |
## Sources
- Codebase: `/Users/thamw/development/local/claurst/src-rust/crates/` (directly read)
- [eventsource-stream on crates.io](https://crates.io/crates/eventsource-stream) — v0.2.3
- [reqwest-eventsource on crates.io](https://crates.io/crates/reqwest-eventsource) — v0.6.0
- [landlock on crates.io](https://crates.io/crates/landlock/0.4.1) — v0.4.4
- [rust-landlock GitHub](https://github.com/landlock-lsm/rust-landlock) — official bindings
- [serde_yml on crates.io](https://crates.io/crates/serde_yml) — v0.0.12
- [ratatui v0.29 highlights](https://ratatui.rs/highlights/v029/)
- [ratatui on crates.io](https://crates.io/crates/ratatui) — v0.30.0 available
- [seccompiler on crates.io](https://crates.io/crates/seccompiler) — v0.5.0
- [How to Run Rust Binaries Without Root Using Sandboxing](https://oneuptime.com/blog/post/2026-01-07-rust-sandboxing-seccomp-landlock/view)
- ratatui Context7 docs (`/ratatui/ratatui`) — StatefulWidget / VirtualList patterns
- rmcp Context7 docs (`/websites/rs_rmcp_rmcp`) — MCP SDK patterns
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
