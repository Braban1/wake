# Pre-registration: Wake browser migration and archive

**Frozen at commit:** Recorded by the commit that introduces this file.
**Question doc:** `docs/science-superpowers/questions/2026-07-09-wake-browser-migration.md`
**Analysis plan:** `docs/science-superpowers/plans/2026-07-09-wake-browser-migration.md`

## Hypotheses

- H0: Useful migration requires copying password or authenticated session state, or a large searchable archive necessarily increases startup renderer/resource cost without bound.
- H1: Approved browser metadata can import safely without secret stores, and indexed archived-tab metadata remains bounded and renderer-free at one million records.

## Primary analysis (exact)

- Test A: registered synthetic fixtures for bookmarks, sanitized history, open tabs, extension identifiers, and versioned Arc archive formats.
- Test B: generated SQLite archives at 10k, 100k, 500k, and 1M records.
- Outcomes: exact accepted/rejected/deduplicated counts, filesystem/API access log, idempotency, unknown-schema handling, startup delta, private-memory delta, first-page FTS latency, renderer delta, database bytes, and post-checkpoint WAL bytes.
- Inclusion: only manifest-listed fixtures and generated archives with recorded seed and checksum.
- Exclusion: live browser profiles and all password, cookie, storage, token, encryption-key, Keychain, and extension-code sources.

## Prediction

- Direction: approved metadata imports exactly and idempotently; unknown Arc schemas fail closed; one million records add no renderers and stay within fixed resource budgets.

## Decision rule

- Confirm H1 if all fixture assertions pass, excluded stores show zero access, the one-million-record archive adds zero renderers, startup delta is at most 150 ms, private-memory delta is at most 25 MB, every registered first-page FTS query completes within 250 ms, post-checkpoint WAL remains below 64 MB, and database size excluding favicons remains at most 1.5 GB.
- Disconfirm H1 if safe useful metadata cannot be imported without excluded stores, any unknown schema imports silently, any archived record creates a renderer before restoration, or any fixed one-million-record limit is exceeded.

## Sample size & stopping

- Fixed scales: 10k, 100k, 500k, 1M; five cold and five warm runs per scale; fixed fixture manifest.
- Seed: `20260709` for generated archive records and query selection.
- Stop before implementation if canonical source or an approved complete recovery baseline is unavailable.

## Multiplicity

- All scales, fixtures, and registered queries are reported. A failure at any one-million-record hard limit disconfirms the performance part of H1.

## Secondary & exploratory (labeled)

- Chrome Takeout schemas, unregistered Arc versions, Windows paths, and password-manager extension compatibility are exploratory.

## Planned deviations handling

- New adapters, changed sanitization rules, different query sets, or changed scale budgets require a new preregistration. Deviations are labeled exploratory.

