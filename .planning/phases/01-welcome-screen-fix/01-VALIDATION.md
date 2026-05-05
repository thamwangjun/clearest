---
phase: 1
slug: welcome-screen-fix
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-05-05
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | cargo test (Rust built-in) |
| **Config file** | Cargo.toml (workspace) |
| **Quick run command** | `cargo test -p claurst-tui 2>&1 \| tail -10` |
| **Full suite command** | `cargo test --workspace 2>&1 \| tail -20` |
| **Estimated runtime** | ~30 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cargo test -p claurst-tui 2>&1 | tail -10`
- **After every plan wave:** Run `cargo test --workspace 2>&1 | tail -20`
- **Before `/gsd-verify-work`:** Full suite must be green
- **Max feedback latency:** 30 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| 1-01-01 | 01 | 1 | BUG-01 | — | N/A | unit | `cargo test -p claurst-tui onboarding_defaults_hidden` | ✅ | ⬜ pending |
| 1-01-02 | 01 | 1 | BUG-01 | — | N/A | unit | `cargo test -p claurst-tui welcome_enter_transitions` | ❌ W0 | ⬜ pending |
| 1-01-03 | 01 | 1 | BUG-01 | — | N/A | unit | `cargo test -p claurst-tui welcome_no_silent_exit` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `crates/tui/src/app.rs` — D-06 regression test stubs: `welcome_enter_transitions`, `welcome_no_silent_exit`, `welcome_cross_platform`
- [ ] Fix existing failing test `onboarding_defaults_hidden` in `crates/tui/src/onboarding_dialog.rs`

*Existing test infrastructure (cargo test) covers this phase — no new framework install needed.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Visual rendering of welcome screen | BUG-01 | TUI output not easily captured in unit tests | Run `cargo run` on a fresh install and observe welcome screen renders without blank output |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 30s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
