# Wake macOS distribution analysis plan

> **For agentic workers:** REQUIRED SUB-SKILL: pre-register this plan with science-superpowers:preregistering-analysis BEFORE execution. Then use science-superpowers:subagent-driven-analysis (recommended) or science-superpowers:executing-analysis to run it step-by-step. Steps use checkbox (`- [ ]`) syntax for tracking.

**Question:** What is the least-privileged build path that produces a frictionless public Wake DMG, and what capabilities change without publisher Apple Developer credentials?

**Design:** Comparative release-engineering experiment across unsigned, ad hoc signed, Developer ID signed, and Developer ID signed plus notarized artifacts.

**Data:** Immutable source SHA, four build manifests, signature and entitlement dumps, notarization records, DMG checksums, Gatekeeper results, and human clean-download observations.

**Primary analysis:** Compare each build lane against the same install, launch, relaunch, signature, entitlement, updater, and platform-capability matrix.

**Decision rule:** Support H1 if the installing user needs no Apple account, only the Developer ID signed plus notarized lane passes the no-bypass public-install gate, and capability differences map to code identity or entitlements rather than notarization itself.

**Execution repositories:** Build and application tests run in the canonical private source repository. Release-lane and promotion changes run in the public release-feed repository.

---

### Task 1: Freeze source and release inputs

**Artifacts:**
- Create: `evidence/macos-release/source-manifest.json`
- Create: `evidence/macos-release/credential-requirements.md`

- [ ] Record the source repository, immutable commit SHA, tag, Node/Electron/electron-builder versions, bundle identifier, minimum macOS version, and workflow SHA.
- [ ] Validate that the source SHA is reachable and that a clean dependency install reproduces the baseline build. Stop if canonical source or a complete approved recovery baseline is unavailable.
- [ ] Record secret names and purposes only. Never record values.

### Task 2: Produce the four build variants

**Artifacts:**
- Create: `evidence/macos-release/build-manifests/<variant>.json`
- Create: `evidence/macos-release/checksums.txt`

- [ ] Build unsigned, ad hoc signed, Developer ID signed, and Developer ID signed plus notarized variants from the same source SHA.
- [ ] Record exact build command, exit status, artifact path, SHA-256, elapsed time, and whether private inputs were present for every variant.
- [ ] Confirm all nested executables are listed in each manifest; do not silently skip missing expected binaries.

### Task 3: Run artifact verification

**Artifacts:**
- Create: `scripts/qa/verify-macos-artifact.sh`
- Create: `evidence/macos-release/results/<variant>.txt`

- [ ] For every DMG run `hdiutil verify`, mount read-only, locate exactly one Wake app, and run `codesign --verify --deep --strict --verbose=2`.
- [ ] Capture `codesign -dv --verbose=4`, signed entitlements, `spctl -a -vvv -t exec`, and `xcrun stapler validate` without suppressing nonzero status.
- [ ] Confirm the Developer ID chain, timestamp, hardened runtime, bundle identifier, Team ID, and expected entitlement differences.
- [ ] Detach every mounted image and verify cleanup.

### Task 4: Run user-visible and capability checks

**Artifacts:**
- Create: `evidence/macos-release/manual-matrix.md`

- [ ] Download each candidate through an HTTP fixture that applies quarantine metadata; copy to Applications, first-launch, quit, and relaunch.
- [ ] Record the exact Gatekeeper dialog and whether any right-click, Privacy & Security, or command-line bypass is required.
- [ ] Run updater-signature, ordinary Keychain persistence, WebAuthn capability, and approved password-manager extension probes on the signed candidates.
- [ ] Have the user perform Touch ID or credential interaction; agents do not enter or inspect credentials.

### Task 5: Split CI build and release gates

**Artifacts:**
- Modify: `.github/workflows/mac-build.yml`
- Create: `docs/qa/macos-release-runbook.md`

- [ ] Make source checkout, install, build, and tests runnable without Apple signing/notarization secrets.
- [ ] Put signing/notarization behind a protected release environment and manual candidate-generation input.
- [ ] Keep publication in a separate approval-controlled promotion job that consumes an immutable verified artifact.
- [ ] Validate workflow YAML, actionlint, unsigned branch execution, signed candidate execution, and the no-publication default.
