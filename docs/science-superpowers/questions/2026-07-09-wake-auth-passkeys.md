# Wake authentication and passkeys

**Research question:** What Electron session, popup, permission, entitlement, user-agent, and WebAuthn configuration is required for Google OAuth and macOS Touch ID passkeys to complete reliably in Wake across representative identity providers?

**Background / motivation:** Google sign-in now appears to complete after the earlier tab-serialization fix, but it needs regression proof. GitHub currently reports only partial passkey support, and authentication surfaces feel less integrated than Chrome or Arc. Authentication is a release-blocking browser capability, not an optional integration.

**Hypotheses:**
- H0: The remaining failures are identity-provider limitations that Wake cannot address.
- H1: Correct popup and navigation handling, defensive tab state management, Electron WebAuthn configuration, a matching macOS Keychain access-group entitlement, and explicit account-selection handling will make supported OAuth and passkey flows work without weakening browser security.

**Population & unit of analysis:** One clean and one persisted Wake profile for each test flow: Google account sign-in, Sign in with Google on a third-party site, GitHub password plus Touch ID passkey, passkey cancellation, popup closure, failed authentication, and relaunch persistence.

**Key variables (operationalized):**
- Outcome: authentication completion -> expected provider redirect completes in the initiating tab or approved popup, no uncaught exception occurs, and the authenticated state behaves correctly after relaunch.
- Exposure: runtime configuration -> Electron version, partition, popup policy, permission policy, WebAuthn configuration, entitlement set, and signing identity.
- Failure evidence: console and main-process diagnostics -> navigation decisions, popup lifecycle, WebAuthn account selection, permission checks, and crash-free tab serialization.

**What counts as an answer:** Automated tests for deterministic state and popup logic plus signed local end-to-end evidence for Google OAuth regression, third-party Sign in with Google, GitHub Touch ID passkeys, account selection, cancellation, and failure paths. The result must also identify which visual or behavioral differences come from Wake's shell, Electron/Chromium version, provider policy, or missing native integration.

**Scope & exclusions:** Browser authentication compatibility and macOS platform passkeys are in scope. Automating credentials, reading passwords, bypassing multifactor authentication, or weakening identity-provider checks is excluded.

**Open questions for prior-work survey:** Current Electron WebAuthn support and entitlements; provider restrictions on embedded browsers; partition-level permission behavior; account chooser UX; Chromium/Electron version effects; user-agent effects; and which flows require a real signed build rather than development mode.
