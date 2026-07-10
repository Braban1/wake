# Wake authentication and passkeys analysis plan

> **For agentic workers:** REQUIRED SUB-SKILL: pre-register this plan with science-superpowers:preregistering-analysis BEFORE execution. Then use science-superpowers:subagent-driven-analysis (recommended) or science-superpowers:executing-analysis to run it step-by-step. Steps use checkbox (`- [ ]`) syntax for tracking.

**Question:** Does a current Electron runtime plus correct popup, session, permission, WebAuthn, entitlement, and account-selection behavior make Wake's Google and macOS Touch ID authentication reliable?

**Design:** Before/after compatibility analysis using deterministic local identity fixtures followed by signed human verification on real Google and GitHub flows.

**Data:** Electron 34 baseline capability output, Electron 43.1 candidate output, local popup/WebAuthn fixture traces, signed entitlements, sanitized lifecycle logs, and human pass/fail observations.

**Primary analysis:** Compare exact flow completion and failure reasons across runtime, popup type, profile state, blocker state, extension state, and WebAuthn credential count.

**Decision rule:** Support H1 if all local fixture flows pass on Electron 43.1, the signed app exposes the platform authenticator with the authorized Keychain group, Google regression flows complete, and GitHub Touch ID passes registration/authentication/cancellation without credential automation.

**Execution repository:** Canonical private source repository after source-access or recovery approval.

---

### Task 1: Upgrade and characterize the runtime

**Artifacts:**
- Create: `evidence/runtime/baseline.json`
- Create: `evidence/runtime/candidate.json`

- [ ] Resolve canonical source, lockfile, builder configuration, and tests. Stop if canonical source or an approved recovery baseline is unavailable.
- [ ] Record Electron, Chromium, Node, V8, effective user agent, minimum macOS version, bundle identifier, partition names, and extension API support for baseline and candidate.
- [ ] Upgrade to exact Electron 43.1.0 on a dedicated branch and resolve every documented breaking change without user-agent spoofing.
- [ ] Run the complete existing test/build/package suite before authentication changes.

### Task 2: Build deterministic authentication fixtures

**Artifacts:**
- Create: `test/fixtures/auth/server.*`
- Create: `test/auth-popup/*.test.*`

- [ ] Serve fixtures for direct popup, `about:blank` then redirect, intermediary origin, opener messaging, popup close, cancel, nested iframe, and blocked permission.
- [ ] Assert approved popups remain real windows with the initiating partition, context isolation, no Node integration, opener communication, and callback completion.
- [ ] Assert disallowed schemes and unsolicited popups fail closed without converting an expected OAuth popup into a normal tab.
- [ ] Test clean and persisted profiles with blocker disabled, blocker enabled, no extensions, and each approved extension independently.

### Task 3: Add and validate WebAuthn integration

**Artifacts:**
- Create: `test/webauthn/*.test.*`
- Modify: source entitlement and packaging configuration identified in Task 1

- [ ] Configure Touch ID with `U8SK677AL2.com.wake.browser.webauthn` before app readiness.
- [ ] Add the identical `keychain-access-groups` value to the signed entitlement and authorize it through the required Developer ID provisioning path.
- [ ] Implement account selection for zero, one, multiple, canceled, and destroyed-frame cases; assert callback completion exactly once.
- [ ] Package a signed candidate and verify the final entitlement dump, Team ID, provisioning authorization, Developer ID chain, notarization, and staple.

### Task 4: Run signed human authentication verification

**Artifacts:**
- Create: `evidence/auth/manual-matrix.md`

- [ ] User verifies direct Google login and third-party Sign in with Google on clean and persisted profiles.
- [ ] User registers and authenticates a Wake-specific GitHub Touch ID passkey, then tests multiple-account choice, cancellation, failure, and relaunch.
- [ ] Capture only pass/fail, sanitized provider origin, lifecycle state, and error class. Do not capture account names, credential IDs, cookies, tokens, query strings, or screenshots containing secrets.
- [ ] Repeat with blocker and approved password-manager extension toggled independently.

### Task 5: Add regression and diagnostics gates

**Artifacts:**
- Create: `docs/qa/authentication-runbook.md`

- [ ] Add non-sensitive diagnostics for runtime versions, effective user agent, popup lifecycle, permission decisions, renderer termination, and WebAuthn availability.
- [ ] Ensure network logs use metadata-only mode and URL sanitization.
- [ ] Add the local fixture suite to pull-request CI; keep real credentials and Touch ID in the signed human gate.
