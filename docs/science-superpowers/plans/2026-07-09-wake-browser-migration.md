# Wake browser migration and archive analysis plan

> **For agentic workers:** REQUIRED SUB-SKILL: pre-register this plan with science-superpowers:preregistering-analysis BEFORE execution. Then use science-superpowers:subagent-driven-analysis (recommended) or science-superpowers:executing-analysis to run it step-by-step. Steps use checkbox (`- [ ]`) syntax for tracking.

**Question:** Which browser metadata can Wake import safely, including Arc archived tabs, and can Wake's archive remain fast and renderer-free at one million records?

**Design:** Fixture-driven compatibility and security analysis plus deterministic SQLite scale benchmarks.

**Data:** Synthetic Chrome/Brave/Arc export fixtures, versioned Arc community-format fixtures with all personal data replaced, generated archives of 10k/100k/500k/1M records, and instrumented filesystem-access logs.

**Primary analysis:** Verify data fidelity, excluded-store non-access, archive query latency, startup cost, private memory, database/WAL growth, and live renderer count at each fixed scale.

**Decision rule:** Support H1 if approved metadata imports are idempotent and sanitized, excluded secret stores are never opened, unknown Arc schemas fail closed, and the one-million-record archive adds zero renderers, no more than 150 ms startup time and 25 MB private memory over the empty archive, returns first-page FTS results within 250 ms in every registered run, keeps post-checkpoint WAL below 64 MB, and stores no more than 1.5 GB excluding optional favicons.

**Execution repository:** Canonical private source repository after source-access or recovery approval.

---

### Task 1: Establish source and fixture boundaries

**Artifacts:**
- Create: `test/fixtures/browser-import/README.md`
- Create: `test/fixtures/browser-import/manifest.json`

- [ ] Resolve canonical source paths and existing import/archive tests. Stop if canonical source or an approved recovery baseline is unavailable.
- [ ] Generate fixtures without reading local browser profiles. Include normal, duplicate, malformed, authentication-sensitive, internal-scheme, unknown-schema, canceled, and locked-source cases.
- [ ] Record fixture provenance, schema version, expected imported count, expected rejected count, and SHA-256.

### Task 2: Lock the import contract

**Artifacts:**
- Create: `docs/contracts/browser-import.md`

- [ ] Define supported inputs: bookmark HTML, capability-probed local extension export for sanitized history/open tabs/extension IDs, and explicitly selected Arc archive files.
- [ ] Define excluded inputs: passwords, cookies, local/session storage, form data, tokens, encryption keys, `Local State`, Keychain records, extension storage, and extension code.
- [ ] Define URL acceptance, sensitive-surface quarantine, preview, cancellation, deduplication, import provenance, and rollback behavior.

### Task 3: Validate adapters

**Artifacts:**
- Create: `test/browser-import/*.test.*`
- Create: `evidence/browser-import/results.json`

- [ ] Run each adapter against every registered fixture and assert exact accepted/rejected/deduplicated counts.
- [ ] Instrument filesystem access and fail the test if any excluded filename, directory class, or Keychain API is touched.
- [ ] Re-run every successful import and assert idempotent record counts and stable provenance.
- [ ] Assert unknown Arc schema versions fail closed with an actionable user message.

### Task 4: Implement and benchmark Wake archive storage

**Artifacts:**
- Create: `scripts/bench/archive-scale.*`
- Create: `evidence/archive-scale/results.json`

- [ ] Store normalized pages, archive events, workspace context, and import provenance with indexed state/time/profile/workspace/host fields and FTS5 search fields.
- [ ] Implement keyset pagination on `(archived_at, id)`, soft-delete grace, age/count retention, and capped favicon storage.
- [ ] Generate fixed 10k, 100k, 500k, and 1M datasets and run five cold and five warm measurements per scale.
- [ ] Record median and worst startup overhead, query latency, private memory, database/WAL bytes, cancellation latency, deduplication time, and renderer count.
- [ ] Confirm archive search and startup never hydrate archived tabs or create renderer processes.

### Task 5: Validate archive user flows

**Artifacts:**
- Create: `qa/flows/tab-archive.md`

- [ ] Test manual archive, restore, re-archive, search, pagination, retention, clear, undo within grace, and expired deletion.
- [ ] Test configurable inactivity windows with a fake clock; active, pinned, audible, WebRTC, download, form-dirty, and recently viewed tabs must not archive unexpectedly.
- [ ] Run agent-browser flows against fixtures and reserve user browser-profile selection for human verification.
