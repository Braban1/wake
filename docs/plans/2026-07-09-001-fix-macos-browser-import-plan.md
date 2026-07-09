---
title: MacOS Browser Import Safety - Plan
type: fix
date: 2026-07-09
artifact_contract: ce-unified-plan/v1
artifact_readiness: implementation-ready
product_contract_source: ce-plan-bootstrap
execution: code
---

# MacOS Browser Import Safety - Plan

## Goal Capsule

- **Objective:** Make the release workflow patch Wake's packaged importer so macOS Chrome, Brave, Edge, and Arc profiles are detected for non-secret bookmark/history import.
- **Authority:** User request prioritizes a working macOS app, Google OAuth stability, and avoiding password/cookie inspection over broader cleanup.
- **Execution profile:** Packaging/config change; prefer workflow validation and local bundle-patch simulation over unit tests because the private source repository is not present in this checkout.
- **Stop condition:** The workflow patch validates, applies cleanly to the extracted packaged bundle, and current installed Wake is observed without credential interaction.

---

## Product Contract

### Summary

Wake should import browser bookmarks and history from local macOS Chrome-family profiles without touching password stores, cookies, Local Storage, or Keychain-backed account data.
Extension import must not be hidden inside the browser import action.

### Problem Frame

The current public checkout owns the GitHub Actions release workflow, not the private Wake source tree.
The bundled importer in the packaged app is Windows-path-specific through `LOCALAPPDATA`, so the macOS UI advertises Chrome, Brave, Edge, and Arc import while detection misses the actual profile roots.
The same import IPC path also discovers and loads browser extensions, which is a surprising side effect for a button that says it imports bookmarks and history.

### Requirements

- R1. The release patch resolves browser profile roots on macOS for Chrome, Brave, Edge, and Arc while preserving the existing Windows roots.
- R2. Browser import remains limited to the existing bookmark and history readers.
- R3. Imported history URLs are sanitized or skipped when they carry auth, token, session, or account-sensitive surfaces.
- R4. The import action does not auto-load extensions from imported browser profiles.
- R5. Previously persisted browser-profile extension paths are removed from startup loading.
- R6. The patch remains fail-fast when expected bundle snippets drift so the workflow does not silently ship an unpatched app.
- R7. Verification avoids reading password, cookie, token, Local Storage, or Keychain-backed browser data.

### Scope Boundaries

- Password import, cookie/session migration, account-token migration, payment/autofill migration, and Keychain access are outside this change.
- Upstream private-source fixes are deferred until `Braban1/wake-src` is available in this workspace.
- Performance optimization beyond identifying follow-up targets is deferred to a separate patch after the importer safety fix.

---

## Planning Contract

### Key Technical Decisions

- KTD1. Patch the built bundle in `.github/workflows/mac-build.yml` because this repository only has the public release workflow and not the private app source.
- KTD2. Add platform-aware root discovery inside the post-build patch instead of editing extracted untracked bundles; CI remains the durable build surface.
- KTD3. Sanitize imported browser history before persistence because Chrome-family history can include auth callback URLs and query tokens.
- KTD4. Remove automatic extension loading from `import:run` and quarantine already-persisted browser-profile extension paths because extension installation is a distinct trust decision from importing bookmarks and history.

### Assumptions

- The packaged bundle shape remains close to the currently extracted `out/main/index.js` and should fail fast if it drifts.
- The local macOS profile roots under `~/Library/Application Support` are representative of the intended macOS release behavior.

---

## Implementation Units

### U1. Patch macOS browser import roots

- **Goal:** Extend the workflow's built-bundle patch to replace Windows-only browser profile discovery with platform-aware roots.
- **Requirements:** R1, R2, R3, R6, R7.
- **Dependencies:** None.
- **Files:** `.github/workflows/mac-build.yml`.
- **Approach:** Replace the bundled `browserDefs()` snippet with a version that includes macOS profile roots for Chrome, Brave, Edge, and Arc, uses the existing Windows paths on Windows, and sanitizes imported history before persistence.
- **Patterns to follow:** Existing `replaceOnce` fail-fast patching in the same workflow step.
- **Test scenarios:** Simulate the patch against the extracted packaged main bundle, verify `detectBrowsers()` can see macOS profile roots that contain `Bookmarks` or `History`, and verify token-bearing history URLs are skipped or stripped without opening secret-bearing files.
- **Verification:** Workflow syntax remains valid and the patch simulation changes only the expected importer code.

### U2. Stop hidden extension import side effects

- **Goal:** Ensure the browser import action does not install unpacked extensions as a side effect and previously persisted browser-profile extension paths do not continue to auto-load.
- **Requirements:** R2, R4, R5, R6, R7.
- **Dependencies:** U1.
- **Files:** `.github/workflows/mac-build.yml`.
- **Approach:** Replace the packaged `import:run` IPC handler with a bookmark/history-only version that still returns an `extensions` field for compatibility, and filter startup extension paths that live under known browser profile `Extensions` directories.
- **Patterns to follow:** Existing explicit extension-loading IPC paths stay intact for user-selected unpacked extension folders.
- **Test scenarios:** Simulate the patch and assert the patched `import:run` block no longer contains `detectBrowserExtensions`, `loadExtension`, or `extensionPaths` updates; assert browser-profile extension paths are filtered from startup loading.
- **Verification:** The renderer can continue to call `import:run`, receive the same result shape, and extension loading remains available only for non-browser-profile paths selected through the explicit extension UI.

### U3. Validate and observe the packaged app surface

- **Goal:** Prove the branch patch is mechanically sound and current installed Wake remains usable.
- **Requirements:** R6, R7.
- **Dependencies:** U1, U2.
- **Files:** `.github/workflows/mac-build.yml`.
- **Approach:** Run YAML/actionlint validation, run the patch body against a copied extracted bundle, and use Computer Use only to inspect the current Wake window state without submitting credentials or reading secrets.
- **Test scenarios:** Validation exits zero; local simulation exits zero; Wake UI is visible and responsive enough for a non-mutating state check.
- **Verification:** Record exact commands and outcomes in the final handoff.

---

## Verification Contract

| Gate | Applies to | Done signal |
|---|---|---|
| Ruby YAML parse | U1-U3 | `.github/workflows/mac-build.yml` parses as YAML |
| `actionlint` | U1-U3 | Workflow schema/action syntax passes |
| Local patch simulation | U1-U2 | Patch applies to an extracted packaged main bundle and static assertions pass |
| Computer Use observation | U3 | Wake's current app window is observed without credential or secret interaction |

---

## Definition of Done

- `.github/workflows/mac-build.yml` contains the macOS importer patch and safe import behavior.
- Validation and local simulation pass.
- The branch is pushed to the existing PR branch.
- Remaining performance issues and non-secret browser migration gaps are reported as follow-up work, not hidden as complete.

---

## Appendix

### Validation Evidence

These checks were run against the workflow patch in this checkout on 2026-07-09.

| Check | Outcome |
|---|---|
| Ruby YAML parse for `.github/workflows/mac-build.yml` | `YAML OK` |
| `actionlint .github/workflows/mac-build.yml` | `actionlint OK` |
| Inline Node patch script syntax check | `inline patch script syntax OK` |
| Targeted import/security patch simulation against extracted packaged main bundle | `targeted import/security patch simulation OK`; fake macOS browser roots detected `arc, chrome` |
| Computer Use observation of installed Wake | Wake opened on `claude.ai/new`, logged in as Bilal, with no credential entry or secret-store interaction by the agent |
