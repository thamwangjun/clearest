# Roadmap: claurst

**Milestone:** v1
**Defined:** 2026-05-04
**Granularity:** Standard
**Coverage:** 1/1 requirements mapped

## Phases

- [ ] **Phase 1: Welcome Screen Fix** - Fix silent exit on Enter keypress at first-launch welcome screen

## Phase Details

### Phase 1: Welcome Screen Fix
**Goal**: Users can complete the first-launch welcome screen without claurst exiting silently
**Depends on**: Nothing (first phase)
**Requirements**: BUG-01
**Success Criteria** (what must be TRUE):
  1. User presses Enter on the welcome screen and claurst proceeds to the main TUI session instead of exiting
  2. User who encounters the welcome screen for the first time sees no unexpected termination or blank output
  3. Pressing Enter on the welcome screen produces the same result across platforms (macOS, Linux)
**Plans**: 1 plan

Plans:
- [x] 01-01-PLAN.md — Fix failing onboarding_defaults_hidden test and add D-06 regression tests (Enter/Esc on Welcome/KeyBindings pages; assert should_quit=false)

## Progress Table

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Welcome Screen Fix | 0/1 | Not started | - |

---
*Roadmap defined: 2026-05-04*
*Last updated: 2026-05-05 after phase 1 planning*
