# Wake macOS distribution

**Research question:** What is the least-privileged, reproducible build and release path that produces a Wake DMG which an arbitrary macOS user can download from GitHub and open without Terminal commands or a Gatekeeper bypass, and what functionality is lost if the publisher does not use an Apple Developer account?

**Background / motivation:** Wake currently has a locally usable notarized artifact and a public release-feed repository, but the branch workflow stops before building when its signing inputs are unavailable. Other projects, including `nexu-io/open-design`, distribute downloadable DMGs without asking the installing user for an Apple Developer account. The answer must distinguish credentials required from the publisher from anything required from the person installing the app, and determine whether Apple Developer Program membership and each current GitHub secret are necessary for local builds, pull-request validation, notarization, public releases, and platform features.

**Hypotheses:**
- H0: A publisher without Apple Developer Program membership can meet the frictionless public-install target without limiting Wake's password-manager, Keychain, passkey, update, or hardened-runtime capabilities.
- H1: The installing user never needs an Apple Developer account, but the publisher needs Developer ID signing and Apple notarization for a frictionless public DMG; unsigned or ad hoc signed builds remain useful for local and controlled testing with measurable distribution or capability limitations.

**Population & unit of analysis:** Supported Apple Silicon macOS versions; one clean download, mount, copy, first launch, quit, and relaunch sequence per build variant.

**Key variables (operationalized):**
- Outcome: frictionless installation -> Gatekeeper accepts the downloaded app without override instructions, and signing, notarization, stapling, DMG integrity, launch, quit, and relaunch checks pass.
- Exposure: build variant -> unsigned, ad hoc signed, Developer ID signed but not notarized, and Developer ID signed plus notarized.
- Release inputs: credential or secret -> certificate material, certificate password, Apple account identifier, app-specific password or supported notarization credential, and team identifier.
- Capability impact: platform feature -> Keychain access, password-manager extension behavior, Touch ID passkeys, WebAuthn entitlements, hardened runtime, update trust, and first-launch UX under each signing lane.

**What counts as an answer:** An evidence matrix grounded in current Apple documentation, comparable open-source DMG projects, signature inspection, and clean-machine tests that identifies which credentials are mandatory for each build lane and which app capabilities change. The implementation outcome is the simplest viable local/CI/public release split with an automated release gate that fails early with actionable diagnostics.

**Scope & exclusions:** Public direct-download DMG distribution through GitHub is in scope. Mac App Store submission, automatic updates, universal binaries, and Windows/Linux distribution are excluded from the first release lane. No environment files or secret values will be read or recorded.

**Open questions for prior-work survey:** How `nexu-io/open-design` and comparable projects sign and distribute their DMGs; which Gatekeeper behaviors apply to unsigned, ad hoc signed, and quarantined downloads; whether password-manager and passkey features require signing or entitlements; whether current notarization supports a safer App Store Connect API-key lane; whether GitHub environments can isolate release secrets; and which clean macOS versions should form the minimum smoke matrix.
