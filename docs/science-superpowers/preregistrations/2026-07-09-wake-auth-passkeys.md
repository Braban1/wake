# Pre-registration: Wake authentication and passkeys

**Frozen at commit:** Recorded by the commit that introduces this file.
**Question doc:** `docs/science-superpowers/questions/2026-07-09-wake-auth-passkeys.md`
**Analysis plan:** `docs/science-superpowers/plans/2026-07-09-wake-auth-passkeys.md`

## Hypotheses

- H0: Remaining Google and Touch ID failures are provider limitations that persist after a current runtime and correct integration.
- H1: Electron 43.1 plus real popup semantics, shared persistent sessions, origin-aware permissions, authorized WebAuthn entitlements, and account-selection handling makes the registered flows reliable.

## Primary analysis (exact)

- Runtime comparison: installed Electron 34.5.8 baseline versus exact Electron 43.1.0 candidate.
- Local flows: direct popup, blank-then-redirect, intermediary origin, opener messaging, popup close, cancel, nested iframe, blocked permission, and zero/one/multiple/destroyed WebAuthn account selection.
- Human flows: direct Google login, third-party Sign in with Google, Wake-specific GitHub Touch ID passkey registration, authentication, multiple-account choice, cancellation, failure, and relaunch.
- Conditions: clean/persisted profile; blocker off/on; no extension/each approved extension independently.
- Outcomes: exact completion state, callback count, popup/window lifecycle, sanitized error class, platform-authenticator availability, and final signed entitlement evidence.

## Prediction

- Direction: Electron 34 lacks the platform authenticator; the fully configured signed Electron 43.1 candidate passes every local flow and the human Google/GitHub matrix without user-agent spoofing.

## Decision rule

- Confirm H1 if every local fixture assertion passes, the signed candidate exposes the platform authenticator with `U8SK677AL2.com.wake.browser.webauthn`, account callbacks complete exactly once, and every human Google/GitHub flow reaches its expected success or cancellation state without an uncaught exception.
- Disconfirm H1 if the correctly signed/configured candidate still reports no platform authenticator, loses required popup semantics, or a registered provider flow fails under the baseline no-extension/no-blocker condition.

## Sample size & stopping

- Each local combination runs three times. Each human provider flow runs once per registered clean/persisted condition because credentials and Touch ID are human-gated.
- Stop before the runtime upgrade if canonical source or an approved complete recovery baseline is unavailable.
- Do not change the user agent or provider allowlist based on observed outcomes.

## Multiplicity

- Every registered flow and condition is reported. No provider/condition is dropped after failure.

## Secondary & exploratory (labeled)

- FedCM/One Tap on unregistered third-party sites, existing iCloud/Safari/Chrome credential portability, and additional password-manager extensions are exploratory.

## Planned deviations handling

- Runtime, entitlement-group, popup-policy, fixture, or provider-matrix changes require a new freeze. Deviations are reported separately as exploratory.

