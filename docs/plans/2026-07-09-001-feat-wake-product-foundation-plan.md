---
title: Wake macOS Product Foundation - Plan
type: feat
date: "2026-07-09"
artifact_contract: ce-unified-plan/v1
artifact_readiness: requirements-only
product_contract_source: ce-plan-bootstrap
execution: code
origin: docs/science-superpowers/questions/2026-07-09-wake-product-foundation.md
---

# Wake macOS Product Foundation - Plan

## Goal Capsule

- **Objective:** Ship a public Wake DMG that opens normally on supported Macs with modern authentication, correct site appearance, bounded essential performance, and a maintainable release pipeline; follow it with safe migration and scalable archiving while scoping each Zen-inspired capability independently.
- **Authority:** This Product Contract and its cited validation plans govern implementation; Apple/Electron/provider contracts override inferred behavior; the user's manual release approval remains mandatory.
- **Execution profile:** Source-level, test-first work in the canonical private source repository; release orchestration and artifacts in the public release-feed repository.
- **Stop conditions:** Do not reconstruct source from packaged bundles, copy authentication secrets, publish an unapproved artifact, spoof Chrome identity, or implement against an inaccessible source repository.
- **Current blocker:** `Braban1/wake-src` and source commit `0966e21` are not accessible through current GitHub, HTTPS, SSH, local Git objects, or local worktrees.

---

## Product Contract

### Summary

Wake will use separate development and public-release lanes. Local and pull-request builds remain credential-light, while the public candidate is Developer ID signed, notarized, stapled, verified, manually tested, and promoted only after explicit approval.

The Foundation Release upgrades the obsolete Electron runtime, fixes authentication and dark-mode architecture, removes release-workflow bundle rewriting, and establishes essential performance and QA gates. Safe browser migration and a scalable tab archive follow as separate capabilities that do not block that release. Zen remains a source of interaction patterns for a prioritized follow-up backlog rather than a codebase or product identity.

### Problem Frame

The installed Wake build works well enough to expose product value, but its current release and runtime foundations are not maintainable. The public repository is a binary feed whose workflow rewrites compiled output from a private source repository that is currently inaccessible. Electron 34 cannot support the required macOS Touch ID WebAuthn API. A global page-inversion strategy causes visible appearance defects. Browser import relies on unstable profile details and must not expand into credential or session copying.

### Actors

- A1. Wake user installs the public DMG, browses, authenticates, migrates approved metadata, and manages archived tabs.
- A2. Wake maintainer develops and reviews source-level changes without needing public-release credentials for every branch.
- A3. Release operator generates a signed candidate, reviews automated evidence, coordinates human verification, and promotes the exact approved artifact.
- A4. Automated QA exercises credential-free fixture flows, browser interaction, artifact verification, and performance budgets.

### Requirements

**Distribution and repository ownership**

- R1. Public Wake downloads install without Terminal commands, right-click overrides, or Privacy & Security bypasses.
- R2. The installing user never needs an Apple Developer account; publisher credentials remain isolated to the signed public-release lane.
- R3. Pull-request build, test, and ad hoc packaging run without Apple signing or notarization secrets and produce a credential-free artifact plus authenticated provenance binding its digest to the immutable source SHA, workflow identity, and successful validation run.
- R4. A protected signing job verifies that provenance against an independent trust root, receives only the attested artifact, and executes no repository dependency or lifecycle scripts. It applies Developer ID signing, hardened runtime, notarization, stapling, nested-signature verification, immutable checksums, and Gatekeeper assessment with least-privilege credentials, then rejects any executable or provisioning profile whose effective entitlements differ from a version-controlled, justified allowlist.
- R5. Candidate publication is disabled until the supported macOS/architecture cohort is explicit, the last compatible artifact remains available with upgrade messaging, and the user approves the exact locally tested checksum.
- R6. Application source and tests live in a canonical private source repository with at least two administrators, audited release access, a recoverable backup, and a documented recovery drill; the public repository contains release orchestration, manifests, artifacts, and release notes.
- R7. Public release workflows consume immutable source SHAs and do not rewrite runtime application behavior after compilation.

**Runtime, authentication, and appearance**

- R8. Electron 43.1.0 is the validated planning baseline. When source work begins, Wake selects the latest stable Electron release that supports native WebAuthn and the tested macOS target, records its support window and minimum macOS version, then locks that exact version reproducibly.
- R9. Google login and third-party Sign in with Google preserve real popup, opener, callback, close, session-partition, and cancellation behavior without Chrome user-agent spoofing.
- R10. macOS Touch ID passkeys use Electron WebAuthn configuration, an authorized Team-ID Keychain access group, signed entitlements, and callback-once account selection.
- R11. Wake's shell follows `system`, `light`, and `dark` modes while normal web content remains unmodified by default.
- R12. Per-origin appearance offers `Automatic`, `Original`, and `Force dark`. `Automatic` advertises the Wake shell preference and relies on native site support; `Original` leaves the site's OS-derived preference untouched; `Force dark` is an explicit site override. Overrides persist per origin, can be reset, and no default path globally inverts page roots or protected media, authentication, or payment surfaces.
- R13. Diagnostics expose non-sensitive runtime, popup, permission, crash, and WebAuthn capability information without recording credentials, tokens, sensitive URLs, or account identifiers.
- R14. Every untrusted web-content and popup renderer uses Chromium sandboxing and context isolation without Node or Electron privileges; IPC, bridge, navigation, download, and permission capabilities are minimal, typed, sender-origin and frame validated, and denied by default.

**Migration and tab archive**

- R15. Wake imports user-selected bookmarks, sanitized history, open tabs, search-provider choices, untrusted extension identifiers for reinstall suggestions, and experimental versioned Arc archive data.
- R16. Import never reads or copies passwords, cookies, local/session storage, form data, tokens, encryption keys, Keychain records, extension storage, or extension code.
- R17. Imports are hostile input: a no-network, no-code-execution worker enforces size, nesting, record-count, elapsed-time, scheme, and versioned schema limits before mutation. One versioned URL-storage policy applies before import or archive persistence: allowlisted schemes only, no user information or authentication callbacks, no secret-like parameters, query/fragment values only on an explicit non-sensitive allowlist, and indexing of only the sanitized result. Every import supports per-category/item preview, cancellation with zero mutation, provenance, transactional commit, idempotency, deduplication, and actionable errors.
- R18. Wake provides manual archive/restore and profile-scoped inactivity auto-archive that is disabled until configured with an hours-or-days threshold. Actual tab activation resets inactivity; pinned, active, audible, WebRTC, downloading, and unsaved-form tabs are exempt; each archive action supports immediate undo.
- R19. A dedicated profile-scoped Archive view supports search, workspace/host/time filters, keyset pagination, retention, soft-delete grace, empty/loading/error states, and zero live renderers until restoration. Restore reuses an already-open duplicate, otherwise returns to the recorded workspace or the current workspace when the original no longer exists.
- R20. The one-million-record archive meets the fixed startup, memory, search, WAL, storage, and renderer budgets in the registered validation plan.
- R21. Imported and archived browsing metadata remains local to the macOS user and Wake profile, is excluded from telemetry, uses redacted diagnostics, and has a permanent-delete path that removes primary rows, search indexes, favicons, provenance, managed caches, and checkpointed WAL content within a documented window. Wake relies on macOS user filesystem protections, including FileVault when enabled, rather than custom archive encryption, and release documentation states that boundary explicitly.
- R22. Extension suggestions resolve only normalized identifiers through an allowlisted official marketplace, show marketplace-supplied publisher identity, and require explicit confirmation; imported names and links are never trusted as install sources.

**Performance, QA, and product evolution**

- R23. Wake measures startup, first interactive tab, workspace restore, restoration of already-installed Wake extensions, import, archive readiness, private memory, renderer count, event-loop stalls, and metadata-only network behavior before optimizing.
- R24. Inactive tabs restore as metadata or sleeping state; nonessential extension/import work does not block first interaction.
- R25. Automated tests cover deterministic state, fixtures, storage, permissions, popups, appearance, accessibility, artifact integrity, and performance budgets.
- R26. Agent-browser validates deterministic Electron/browser flows; Computer Use validates macOS-native dialogs and signed-app behavior; humans alone perform credential and Touch ID interactions.
- R27. The foundation produces a prioritized Zen-inspired backlog only. Each selected capability requires its own Product Contract and ships after the foundation gates as an independent PR with a user outcome, persistence boundary, performance budget, QA flow, and rollback boundary.
- R28. Wake remains an Electron productivity browser and does not fork Zen or Firefox.
- R29. Every new workflow supports complete keyboard operation, visible focus, semantic labels and announcements, sufficient contrast, and reduced-motion preferences, with automated checks and human macOS accessibility QA.
- R30. Before Foundation promotion, Wake publishes and automates an Electron/Chromium maintenance policy that tracks upstream support, blocks unsupported runtime releases, defines regular upgrade cadence and critical-vulnerability response windows, and exercises an emergency signed-release path.

### Delivery Milestones

1. **Foundation Release:** source continuity, credential-free validation, protected signing/promotion, supported-platform decision, current Electron, Google/GitHub authentication regression, Touch ID WebAuthn, site-aware appearance, essential startup/memory budgets, accessibility, and automated plus human QA.
2. **Migration Capability:** Chrome/Brave portable exports and explicit extension suggestions first; experimental Arc archive adapters only for named, fixture-backed schemas. Migration does not gate the Foundation Release.
3. **Archive Capability:** manual archive, profile-scoped auto-archive, indexed recovery, privacy lifecycle, and one-million-record budgets. Archive does not gate the Foundation Release.
4. **Product Evolution:** prioritize Zen-inspired outcomes, then create a separate Product Contract before implementing each one.

### Key Flows

- F1. **Public installation:** A3 builds an immutable candidate; A4 verifies it; A1 installs and relaunches it normally; the user approves; A3 promotes the same bytes.
- F2. **Safe migration:** A1 selects a supported export or experimental Arc archive file; Wake previews accepted/rejected items; A1 confirms; Wake records sanitized metadata and provenance without opening excluded stores.
- F3. **Authentication:** A1 initiates OAuth or passkey login; Wake preserves provider window/session semantics; A1 completes credentials or Touch ID; Wake records only sanitized lifecycle evidence.
- F4. **Tab archive:** A1 manually archives or reaches the configured inactivity threshold; Wake stores metadata and destroys live browsing state; search and restore recreate a tab on demand.
- F5. **Feature delivery:** A2 lands foundation PRs in dependency order; A4 runs automated and profiling gates; A1 performs the defined human checks; only then does the next feature slice begin.

### Acceptance Examples

- AE1. Given a quarantined public DMG, when A1 mounts, copies, and opens Wake, then Gatekeeper accepts it without an override and relaunch succeeds.
- AE2. Given a pull request without Apple secrets, when CI runs, then source checkout, install, tests, build, and non-public packaging complete while signing/promotion remain skipped.
- AE3. Given YouTube in dark mode, when content appearance is `Automatic`, then YouTube controls, video, canvas, logos, and text use the site's own supported appearance without global inversion artifacts.
- AE4. Given a third-party Sign in with Google popup, when it redirects through an intermediary origin, then opener messaging, callback, close, and persistent partition behavior remain intact.
- AE5. Given a signed and entitled candidate on the selected locked Electron release, when A1 starts GitHub passkey authentication, then Wake offers the macOS platform authenticator and completes success or cancellation without an uncaught exception.
- AE6. Given an Arc archive fixture with an unknown schema, when A1 previews import, then Wake imports nothing and explains that the format is unsupported.
- AE7. Given an import containing a token-bearing callback URL, when sanitization runs, then the record is rejected and no token value enters logs or storage.
- AE8. Given one million archived records, when Wake launches and searches, then it creates no archived-tab renderers and meets every registered archive budget.
- AE9. Given 25 saved tabs, when Wake starts, then at most three content renderers exist before A1 activates additional tabs and the registered startup/memory budgets hold.
- AE10. Given a verified candidate awaiting publication, when automated QA passes but the user has not approved, then no GitHub release asset is promoted.
- AE11. Given a representative fixture for each declared Chrome, Brave, and Arc format, when A1 imports it twice, then accepted/rejected counts and allowed fields match the fixture exactly, archive timestamps and provenance are preserved, duplicates are not created, and excluded stores are never opened.
- AE12. Given an archived tab whose original workspace was deleted, when A1 restores it, then Wake opens it in the current workspace and preserves the archive record until restoration succeeds.

### Success Criteria

- The release workflow has separate credential-free validation, protected candidate generation, and approval-controlled promotion.
- The Foundation Release candidate passes automated artifact checks and the user's install/auth/appearance smoke test independently of migration and archive delivery.
- The selected locked Electron release, Google regression, Touch ID WebAuthn, and site-aware appearance pass their registered matrices.
- Browser migration proves positive fidelity and zero access to excluded stores, and the archive passes the one-million-record budget in their later capability milestones.
- Every implementation change is source-level, reviewed, tested, and represented by a narrowly scoped non-`main` PR.

### Scope Boundaries

**In scope**

- macOS Apple Silicon public distribution, current Electron modernization, Google/GitHub authentication, Touch ID passkeys, site appearance, Chrome/Arc/Brave metadata migration, tab archive, performance instrumentation, QA, and a prioritized Zen-inspired backlog.

**Deferred to follow-up work**

- Intel validation, Windows/Linux parity, multi-device archive sync, full profile/container implementation, automatic updates beyond signature trust, and additional Zen-inspired features after the foundation.

**Outside this product's identity**

- Password/cookie/session-token cloning, silent browser-profile reads, automatic copied-extension loading, Chrome user-agent impersonation, and a Zen/Firefox fork.

### Dependencies and Blocking Question

- **Blocking:** Grant the authenticated `bilal-ghafoor` account read access to `Braban1/wake-src` and confirm tag `v0.2.2` at source commit `0966e21`, or provide the most complete source backup and approve recovery into a new private repository.
- **Requires explicit scope change:** If neither original source nor a full backup exists, decide whether to fund a separate reconstruction project from packaged bundles. Reconstruction is not authorized by this plan.
- **Available locally:** Two valid Developer ID Application identities, ASC CLI 2.2.0, and an installed Wake app that is signed, notarized, stapled, and Gatekeeper accepted.
- **Missing in GitHub:** Signing/notarization secrets and a protected release environment; these block public CI candidate generation but do not need to block pull-request builds after workflow separation.

### Sources and Validation Contracts

- `docs/science-superpowers/prior-work/2026-07-09-wake-product-completion.md`
- `docs/science-superpowers/plans/2026-07-09-wake-macos-distribution.md`
- `docs/science-superpowers/plans/2026-07-09-wake-browser-migration.md`
- `docs/science-superpowers/plans/2026-07-09-wake-auth-passkeys.md`
- `docs/science-superpowers/plans/2026-07-09-wake-product-foundation.md`
- [Apple Developer ID certificates](https://developer.apple.com/help/account/certificates/create-developer-id-certificates)
- [Apple notarization requirements](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution)
- [Open Design signed release workflow](https://github.com/nexu-io/open-design/blob/main/.github/workflows/release-stable.yml)
- [Electron 41.5.0 WebAuthn release](https://releases.electronjs.org/release/v41.5.0)
- [Arc Auto Archive](https://resources.arc.net/hc/en-us/articles/19228855311127-Auto-Archive-Clean-as-you-go)
- [Zen Browser source](https://github.com/zen-browser/desktop)
