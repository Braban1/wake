# Pre-registration: Wake product foundation

**Frozen at commit:** Recorded by the commit that introduces this file.
**Question doc:** `docs/science-superpowers/questions/2026-07-09-wake-product-foundation.md`
**Analysis plan:** `docs/science-superpowers/plans/2026-07-09-wake-product-foundation.md`

## Hypotheses

- H0: Adding feature breadth before foundational appearance/runtime work produces an equally stable and usable Wake release.
- H1: Removing destructive appearance behavior and measured eager work first produces a stable baseline that supports independently shippable Zen-inspired features.

## Primary analysis (exact)

- Scenarios: cold/warm start, one tab, 25-tab restore, workspace restore, media, popup, light site, dark site, mixed-theme site, and extension restore.
- Conditions: immutable baseline app versus candidate after each foundation PR; identical macOS/hardware, fixture dataset, settings, blocker, extensions, and viewport.
- Outcomes: deterministic fixture screenshot result, accessibility/interaction smoke result, window-visible time, first-tab-interactive time, total private memory, live renderer count, main-process event-loop stalls, request count, and feature acceptance result.
- Feature ranking: fixed weighted score of user value 35%, dependency feasibility 25%, regression/privacy risk 20%, performance impact 10%, and independent rollback 10%.

## Prediction

- Direction: the foundation candidate removes global-inversion defects and meets every fixed performance and interaction threshold; Zen-inspired candidates decompose into narrow Electron-feasible PRs.

## Decision rule

- Confirm H1 if deterministic appearance fixtures pass, cold window-visible median is at most 2.5 seconds with no run above 4 seconds, cold first-tab-interactive median is at most 4 seconds with no run above 6 seconds, warm window-visible median is at most 1.5 seconds with no run above 2.5 seconds, 25-tab restore creates at most three live content renderers before user activation, total private memory remains below 750 MB, no main-process event-loop stall exceeds 200 ms, and the smoke matrix has zero release-blocking failure.
- Disconfirm H1 if any appearance fixture retains root inversion artifacts, any hard resource threshold is exceeded, a baseline browser flow regresses, or the proposed Zen work cannot be decomposed without a Gecko fork.

## Sample size & stopping

- Five fixed runs per automated performance scenario on one recorded Apple Silicon machine; one human appearance/interaction approval pass after automated checks.
- Seed: `20260709` for generated tab/workspace fixture data.
- Stop before source changes if canonical source or an approved complete recovery baseline is unavailable.

## Multiplicity

- Every scenario and run is reported. Hard-threshold failure is not averaged away. The Zen ranking reports all candidates, not only selected features.

## Secondary & exploratory (labeled)

- Intel performance, macOS versions beyond the registered minimum/latest pair, additional websites, and unplanned Zen features are exploratory.

## Planned deviations handling

- Hardware, runtime, fixture, viewport, scenario, threshold, or scoring-weight changes require a new freeze. Deviations are labeled exploratory.

