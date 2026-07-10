# Wake product foundation and Zen-inspired roadmap analysis plan

> **For agentic workers:** REQUIRED SUB-SKILL: pre-register this plan with science-superpowers:preregistering-analysis BEFORE execution. Then use science-superpowers:subagent-driven-analysis (recommended) or science-superpowers:executing-analysis to run it step-by-step. Steps use checkbox (`- [ ]`) syntax for tracking.

**Question:** Does fixing appearance, startup/runtime work, and browser interaction foundations before adding Zen-inspired features produce a stable and measurably modern Wake baseline?

**Design:** Scenario-based before/after visual, behavioral, and performance analysis with fixed fixtures and budgets, followed by ranked feature slicing.

**Data:** Baseline and candidate screenshots, accessibility trees, interaction traces, content traces, process metrics, private memory, event-loop delay, renderer counts, network request counts, and source-backed Zen capability records.

**Primary analysis:** Compare the fixed scenario matrix before and after each foundation PR; rank Zen-inspired features by user value, dependency cost, regression risk, and measured performance impact.

**Decision rule:** Support H1 if the foundation removes global-inversion defects, passes deterministic light/dark visual fixtures, reaches cold window-visible median at or below 2.5 seconds with no run above 4 seconds, cold first-tab-interactive median at or below 4 seconds with no run above 6 seconds, warm window-visible median at or below 1.5 seconds with no run above 2.5 seconds, restores 25 tabs with at most three live content renderers before user activation, stays below 750 MB total private memory in that scenario, records no main-process event-loop stall above 200 ms, passes the browser interaction smoke matrix, and yields independently shippable Zen-inspired slices rather than a broad fork.

**Execution repository:** Canonical private source repository after source-access or recovery approval.

---

### Task 1: Capture a reproducible baseline

**Artifacts:**
- Create: `evidence/product-baseline/manifest.json`
- Create: `qa/flows/foundation-smoke.md`

- [ ] Resolve canonical source and exact test/build commands. Stop if canonical source or an approved recovery baseline is unavailable.
- [ ] Record immutable source/app SHA, runtime versions, macOS/hardware, settings, extension/blocker state, viewport, and data-fixture checksum.
- [ ] Capture cold/warm start, one-tab, 25-tab, workspace restore, media, popup, dark site, light site, and mixed-theme site scenarios.
- [ ] Record five runs per performance scenario and preserve all results, including regressions.

### Task 2: Fix and validate appearance behavior

**Artifacts:**
- Create: `test/appearance/*.test.*`
- Create: `qa/flows/site-appearance.md`

- [ ] Remove root-level CSS inversion from the default content path.
- [ ] Keep shell theme `system | light | dark`; add per-origin `Automatic | Original | Force dark` content behavior.
- [ ] Protect images, video, canvas, SVG, iframe, authentication, payment, and document surfaces from forced transformations unless explicitly supported.
- [ ] Use local light/dark/mixed fixtures for deterministic screenshot assertions, then verify YouTube and representative sites through agent-browser and Computer Use.
- [ ] Require human approval of the final YouTube light/dark appearance before merging the appearance PR.

### Task 3: Measure and reduce eager runtime work

**Artifacts:**
- Create: `scripts/bench/foundation.*`
- Create: `evidence/performance/results.json`

- [ ] Instrument app ready, window visible, first interactive tab, workspace restore, extension restore, import, and archive readiness.
- [ ] Record `app.getAppMetrics`, per-process private memory, renderer count, event-loop delay, content tracing, and metadata-only network counts.
- [ ] Restore inactive tabs as metadata/sleeping state, defer nonessential extension work, batch import writes, and replace request-path linear scans where measurements justify each change.
- [ ] Re-run the fixed five-run matrix after every performance PR and reject changes that improve one metric by silently dropping behavior.

### Task 4: Audit browser shell modernization

**Artifacts:**
- Create: `docs/product/browser-shell-gap-matrix.md`

- [ ] Compare Wake with current Chrome, Arc, and Zen for toolbar states, tab states, focus, hover, permission prompts, popup ownership, security origin display, keyboard navigation, command access, error states, and recovery.
- [ ] Attribute every gap to engine age, Electron limitation, Wake shell implementation, extension behavior, or product choice.
- [ ] Rank fixes by release blocker, frequent friction, polish, and intentional differentiation; do not copy visual treatment without a Wake use case.

### Task 5: Slice Zen-inspired features

**Artifacts:**
- Create: `docs/product/zen-capability-roadmap.md`

- [ ] Evaluate workspaces, split views, compact mode, Essentials, profiles/containers, keyboard navigation, command palette, tab unloading, window mirroring, session recovery, and site-specific appearance.
- [ ] Give each candidate an independent user outcome, source dependency, API feasibility, persistence model, performance budget, QA flow, rollback boundary, and proposed PR.
- [ ] Defer any candidate that requires a Firefox/Gecko subsystem, weakens authentication/privacy, or cannot remain independently testable in Electron.
- [ ] Preserve Wake as a focused productivity browser; do not fork Zen or Firefox.
