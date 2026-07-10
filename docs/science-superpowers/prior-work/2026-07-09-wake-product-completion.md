# Prior-work survey: Wake product completion

## Scope

This note grounds four approved investigations: macOS distribution, safe browser migration and tab archives, authentication/passkeys, and the stability/appearance/performance foundation required before Zen-inspired features.

The survey is read-only. It does not inspect environment files, browser profiles, passwords, cookies, session stores, encryption keys, or Keychain items.

## Distribution and Apple credentials

### Established method

Use distinct build lanes instead of making every build depend on release credentials:

1. Local and pull-request builds compile, test, and may package an ad hoc signed artifact without Apple secrets.
2. Release candidates use a Developer ID Application certificate to establish stable publisher identity and validate nested signatures.
3. Public releases use Developer ID signing, hardened runtime, secure timestamps, Apple notarization, stapling, Gatekeeper assessment, DMG integrity checks, and a manual approval gate before publication.

### Findings

- The person installing Wake does not need an Apple Developer account. Publisher credentials are used before the DMG is uploaded.
- Apple documents Developer ID signing and notarization as the normal direct-distribution path. Unsigned and ad hoc signed builds remain useful for development but do not satisfy the approved no-bypass public-install target.
- Open Design follows this same model. Its public release workflow imports a signing certificate, supplies Apple notarization credentials, and packages with `--signed --notarize`. Its downloadable DMG is not evidence that publisher credentials are unnecessary.
- Wake's installed app is currently Developer ID signed, notarized, stapled, and accepted by Gatekeeper. Its signed entitlement set contains Electron hardened-runtime exceptions but no WebAuthn Keychain access group.
- Wake's public repository currently has only `GH_PAT`. PR #1 checks for the certificate and five Apple values before checkout, so it cannot currently run the credential-free build/test portion independently.
- The Apple ID plus app-specific-password notarization lane can potentially be replaced by App Store Connect API-key authentication. The Developer ID signing certificate and its private key remain necessary for trusted public distribution.

### Credential matrix

| Lane | Required private inputs | Publicly usable without Gatekeeper override? |
| --- | --- | --- |
| Local source build | None | No distribution claim |
| Pull-request build/test | Private-source checkout credential only, if source remains private | No |
| Ad hoc packaged DMG | Private-source checkout credential only | No |
| Developer ID signed candidate | Certificate plus certificate password | Not reliably |
| Signed, notarized, stapled public DMG | Certificate pair plus notarization authentication and team identity | Yes |

### Sources

- [Apple Developer ID certificates](https://developer.apple.com/help/account/certificates/create-developer-id-certificates)
- [Apple notarization requirements](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution)
- [Apple custom notarization workflow](https://developer.apple.com/documentation/security/customizing-the-notarization-workflow)
- [Apple Keychain access-group entitlement](https://developer.apple.com/documentation/BundleResources/Entitlements/keychain-access-groups)
- [Open Design release workflow](https://github.com/nexu-io/open-design/blob/main/.github/workflows/release-stable.yml)
- [Open Design 0.14.0 release](https://github.com/nexu-io/open-design/releases/tag/open-design-v0.14.0)

## Browser migration and archives

### Established method

Prefer user-mediated exports and capability-probed adapters over direct reads of live browser profile databases. Treat undocumented Arc files as versioned experimental inputs selected by the user, reject unknown schemas, and never load copied extension code.

### Findings

- Browser-exported bookmark HTML is the strongest portable first implementation.
- Chromium extension APIs can export sanitized history, currently open tabs, and extension identifiers with explicit permissions, but compatibility must be probed per browser.
- Arc officially provides searchable/restorable auto-archives and defaults to archiving idle unpinned tabs after 12 hours. Arc does not provide an official desktop export for pinned/favorite tabs, and it does not document an archive export contract.
- Community tools identify version-sensitive Arc sidebar and archive files. These are not stable product APIs and must not be read silently from a live profile.
- Zen provides useful workspace, window-state, session-backup, and pressure-based tab-unloading patterns. No Arc-style searchable archive was confirmed in Zen's current public source or documentation.

### Adopted archive design

- Store normalized page metadata, archive events, workspace context, and import provenance in SQLite.
- Use B-tree indexes for state/time/profile/workspace/host and FTS5 for title, host, and sanitized path.
- Use keyset pagination by `(archived_at, id)` rather than deep offsets.
- Create no renderer process for an archived tab; restoration creates a live tab only on demand.
- Provide manual archive/restore, configurable inactivity-based auto-archive, search, pagination, soft-delete grace, capped favicon storage, and age/count retention controls.
- Benchmark 10,000, 100,000, 500,000, and 1,000,000 records for startup cost, private memory, search latency, deep pagination, cancellation, deduplication, WAL growth, and renderer count.

### Security exclusions

Do not import passwords, cookies, browser local/session storage, OAuth tokens, form snapshots, encryption keys, `Local State`, Keychain records, extension storage, or extension code. Allow only sanitized HTTP(S) URLs; reject internal and executable schemes, incognito records, URL credentials, fragments, and authentication/token-bearing URLs.

### Sources

- [Arc Auto Archive](https://resources.arc.net/hc/en-us/articles/19228855311127-Auto-Archive-Clean-as-you-go)
- [Arc export limitation](https://resources.arc.net/hc/en-us/articles/25583851606039-How-Do-I-Export-my-Pinned-and-Favorite-Tabs-in-Arc-to-Another-Browser)
- [Chrome history API](https://developer.chrome.com/docs/extensions/reference/api/history)
- [Chrome tabs API](https://developer.chrome.com/docs/extensions/reference/api/tabs)
- [Chrome management API](https://developer.chrome.com/docs/extensions/reference/api/management)
- [Zen Window Sync and Recovery](https://docs.zen-browser.app/user-manual/window-sync)
- [Firefox tab unloading](https://firefox-source-docs.mozilla.org/browser/tabunloader/)
- [SQLite FTS5](https://www.sqlite.org/fts5.html)
- [SQLite row-value pagination](https://www.sqlite.org/rowvalue.html)

## Authentication, passkeys, and runtime modernization

### Established method

Upgrade the browser engine before adding platform-authenticator code. Preserve real popup semantics with a shared persistent partition, test provider flows without user-agent spoofing, and treat real OAuth and Touch ID as signed-build human verification rather than credential automation.

### Findings

- The installed Wake app embeds Electron 34.5.8 and Chromium 132.0.6834.210.
- Electron added macOS Touch ID WebAuthn through `app.configureWebAuthn()` and `select-webauthn-account` in Electron 41.5.0. Wake's current runtime cannot implement that API.
- Electron 43.1.0 is the current npm release at survey time. Electron 38 and later require macOS 12, so the upgrade requires an explicit macOS 11 support decision.
- A working implementation needs a Team-ID-based Keychain access group, the matching signed entitlement/provisioning authorization, `app.configureWebAuthn`, and account-selection handling that completes its callback exactly once.
- Existing Safari/Chrome/iCloud passkeys are not assumed portable into Electron's device/session-bound authenticator. Wake-specific registration and authentication must be tested.
- Google OAuth appears to work after the prior crash/popup patch, but the hostname allowlist can still convert intermediary popups into ordinary tabs and break `window.opener`, `postMessage`, callback, and close semantics.
- Do not spoof Chrome's user agent. Google documents current browser and accurate-identification expectations, while Electron is not a named guaranteed Google Identity Services browser.

### Sources

- [Electron 41.5.0 WebAuthn release](https://releases.electronjs.org/release/v41.5.0)
- [Electron app API](https://www.electronjs.org/docs/latest/api/app)
- [Electron session API](https://www.electronjs.org/docs/latest/api/session)
- [Electron window-open handling](https://www.electronjs.org/docs/latest/api/window-open)
- [Electron automated testing](https://www.electronjs.org/docs/latest/tutorial/automated-testing)
- [Electron performance guide](https://www.electronjs.org/docs/latest/tutorial/performance)
- [Google OAuth policy](https://developers.google.com/identity/protocols/oauth2/policies)
- [Google Identity Services supported browsers](https://developers.google.com/identity/gsi/web/guides/supported-browsers)

## Dark mode and browser appearance

### Established method

Theme Wake's shell with `system`, `light`, or `dark` and Electron `nativeTheme`. Let sites consume `prefers-color-scheme` without mutation by default. Offer per-origin `Automatic`, `Original`, and `Force dark` behavior, with any forced transformation implemented as DOM-aware and protected by exclusions.

### Findings

- `nativeTheme.themeSource` affects Chromium's color-scheme signal and native controls; it does not guarantee that a website implements a dark design.
- Wake additionally injects a global root inversion plus counter-inversion for media. That strategy explains washed-out pages, double inversions, and inconsistent YouTube media, canvas, SVG, iframe, and control rendering.
- Wake's older appearance is not solely a Chromium problem. Electron provides an engine rather than Chrome's product shell, so Wake owns toolbar, tab, focus, permission, popup, hover, and security presentation. The old engine, custom shell, incomplete extension APIs, and global inversion compound each other.

### Sources

- [Electron nativeTheme](https://www.electronjs.org/docs/latest/api/native-theme)
- [Electron dark-mode guide](https://www.electronjs.org/docs/latest/tutorial/dark-mode)
- [Web color-scheme](https://web.dev/articles/color-scheme)
- [Chrome Auto Dark Theme](https://developer.chrome.com/blog/auto-dark-theme)

## Validation method

- Unit tests for archive storage/query behavior, URL sanitization, popup classification, navigation guards, destroyed-tab serialization, permission decisions, and WebAuthn chooser callback completion.
- Local fixture sites for popup redirects, opener messaging, WebAuthn capability probes, cross-origin permission behavior, and light/dark screenshot assertions.
- Agent-browser for deterministic Electron/browser flows and screenshots; Computer Use for macOS-native dialogs and signed-app interaction.
- Profiling with Electron content tracing, app/process metrics, private memory, event-loop delay, and metadata-only network logs. Never enable sensitive network logging.
- Human-only verification for Google credentials, GitHub passkey registration/authentication, Touch ID, permission prompts, and final release approval.

## Source recovery and Git strategy

### Confirmed repository state

- `Braban1/wake` is a public release feed, not the application source. Its `main` branch contains only the README and release workflow.
- Historical workflow run `28348606425` successfully checked out `Braban1/wake-src` tag `v0.2.2` at source commit `0966e21` and produced a DMG.
- Current GitHub CLI, HTTPS Git, and SSH Git authentication cannot resolve or access `Braban1/wake-src`. Commit `0966e21` is not an object in the public repository.
- The retained Actions artifact and public releases contain packaged DMGs, not a source archive. Actions runner workspaces are ephemeral and the configured npm cache is not a source backup.
- The installed `app.asar` contains executable bundles and dependency metadata but no Wake application source maps. Reconstruction can preserve behavior, but not the original TypeScript/JSX structure, tests, comments, module boundaries, history, or authorship.
- PR #1 is a clean, mergeable draft from `claude/wake-mac-build-debug-gpjrjt` to public-feed `main`. The planning branch is an independent sibling based on `main`.

### Adopted repository boundary

1. Restore access to `Braban1/wake-src` and confirm tag `v0.2.2` first.
2. If the repository was deleted, recover the most complete source backup into a new private source repository and validate its built behavior against the installed app.
3. Reconstruct from packaged bundles only as an explicitly approved emergency project.
4. Keep runtime source and tests in the private source repository. Keep release orchestration, signed artifacts, manifests, and release notes in the public feed.
5. Replace post-build runtime rewrites in the public workflow with source-level fixes once source access is restored.
6. Build immutable source SHAs with publication disabled, then sign/notarize/upload a candidate for automated and human verification. Publish only through a separate approval-controlled promotion step.

### Proposed branch families

Private source repository:

- `recovery/v0.2.2-baseline` only if recovery into a new repository is necessary.
- `infra/electron-43-upgrade` for the runtime and macOS 12 support floor.
- `fix/auth-popup-passkeys` for popup semantics, permissions, WebAuthn, and signed entitlements.
- `fix/site-appearance` for removal of global inversion and per-origin appearance controls.
- `feat/safe-browser-import` for user-mediated browser adapters and extension suggestions.
- `feat/tab-archive` for scalable archive storage, search, retention, and Arc archive adapter.
- `perf/lazy-tabs-and-import` for measured startup, restore, import, and extension work.

Public release-feed repository:

- `release/macos-build-lanes` for credential-free build/test and protected signed/notarized candidate generation.
- `release/manual-promotion` for approval-controlled publication and immutable source/artifact manifests.

Each feature PR remains independently testable and narrowly scoped. Dependent PRs may be stacked temporarily, then retargeted to `main` as predecessors merge.

## Known validity limits

- The canonical `Braban1/wake-src` repository is not accessible through current CLI, HTTPS, or SSH authentication.
- Public workflow history proves that release `v0.2.2` previously checked out source commit `0966e21`, but it does not make that source available now.
- The installed `app.asar` contains built main/renderer output and dependencies but no application source maps. Static bundle findings are strong evidence of packaged behavior, not a maintainable substitute for canonical source.
- Real Google, passkey, Arc adapter, performance, and clean-machine distribution behavior remains unproven until a new source build exists and the registered runtime matrix is executed.

## Relationship to prior work

This work is an extension and repair of an existing Electron browser, not a novel browser-engine implementation. Apple, Electron, Chromium, Arc, Zen, Firefox, and SQLite supply the established distribution, runtime, archive, and validation methods. Wake-specific contributions are the product boundaries, adapters, QA contract, and integration of those established methods.
