# Pre-registration: Wake macOS distribution

**Frozen at commit:** Recorded by the commit that introduces this file.
**Question doc:** `docs/science-superpowers/questions/2026-07-09-wake-macos-distribution.md`
**Analysis plan:** `docs/science-superpowers/plans/2026-07-09-wake-macos-distribution.md`

## Hypotheses

- H0: A publisher without Apple Developer Program membership can meet Wake's frictionless public-install target without limiting trusted update, entitlement, or code-identity behavior.
- H1: The installing user needs no Apple account, but Wake's publisher needs Developer ID signing and notarization for the approved public lane; development lanes can omit Apple credentials with explicit distribution limitations.

## Primary analysis (exact)

- Test: four-variant artifact matrix from one immutable source SHA.
- Variants: unsigned, ad hoc signed, Developer ID signed, Developer ID signed plus notarized/stapled.
- Outcomes: DMG integrity, nested signature validity, Gatekeeper assessment, notarization ticket, first launch from quarantined download, relaunch, updater identity, ordinary Keychain persistence, final entitlements, and WebAuthn capability.
- Inclusion: Apple Silicon macOS 12 or later builds produced by the registered workflow from the same source SHA.
- Exclusion: artifacts with mismatched source SHA, dependency lock, bundle identifier, or unrecorded post-build mutation.

## Prediction

- Direction: only the Developer ID signed plus notarized/stapled variant will pass the no-bypass public-install gate.
- Capability expectation: notarization changes Gatekeeper trust, while Keychain sharing/WebAuthn behavior follows stable code identity and authorized signed entitlements.

## Decision rule

- Confirm H1 if the installing user supplies no Apple account, the fully signed/notarized variant passes all public-install checks, every unsigned/ad hoc/unnotarized variant requires a bypass or fails the public gate, and capability differences match the signed entitlement matrix.
- Disconfirm H1 if a publisher-without-membership variant passes the same quarantined no-bypass matrix and retains the same trusted identity/entitlement behavior.

## Sample size & stopping

- Four fixed build variants; one immutable source SHA; five automated verification repetitions per variant; one human first-launch/relaunch sequence per variant.
- Stop before building if canonical source or an approved complete recovery baseline is unavailable.
- No variant substitution or post-result credential changes.

## Multiplicity

- One confirmatory matrix. Every variant and outcome is reported; no selection of only successful checks.

## Secondary & exploratory (labeled)

- App Store Connect API-key notarization, Intel builds, additional macOS versions, and updater migration are exploratory until separately registered.

## Planned deviations handling

- Any source, runtime, certificate, entitlement, or OS change invalidates cross-variant comparison and requires a new frozen matrix. Deviations are reported as exploratory.

