# Wake browser migration

**Research question:** Which Chrome, Arc, Brave, and compatible Chromium profile data can Wake import through explicit user action, including Arc's archived-tab history, and how should Wake provide durable manual and automatic tab archiving without reading authentication secrets or degrading as the archive grows?

**Background / motivation:** Users need a low-friction path into Wake, but copying authenticated browser state would create a high-value secret-handling surface. The approved product boundary is metadata migration plus trusted password-manager integration, not silent credential or session cloning.

**Hypotheses:**
- H0: Useful browser migration requires Wake to copy passwords or authenticated session state.
- H1: Bookmarks, sanitized history, open tabs, Arc archived tabs, search preferences, and an extension reinstall list provide useful migration without copying authentication secrets; Wake can store archived tabs as lightweight indexed metadata, archive inactive tabs after a user-selected duration, and restore them on demand without keeping live browser processes.

**Population & unit of analysis:** One user-selected profile from each supported browser; each supported data category is a separate import unit with preview, consent, execution, and rollback or deduplication behavior. Each Wake archive record is an independent metadata record with a lifecycle of archive, search, restore, re-archive, and delete.

**Key variables (operationalized):**
- Outcome: safe useful migration -> selected metadata appears correctly in Wake, Arc archive history remains discoverable, duplicates are bounded, sensitive URLs are excluded, and no credential-bearing stores or Keychain APIs are accessed.
- Exposure: browser and data category -> Chrome, Arc, Brave, compatible Chromium; bookmarks, history, open tabs, search preferences, and extension identifiers.
- Archive scalability: retained records -> search latency, database growth, startup cost, memory use, pagination behavior, retention controls, and live renderer count as the archive reaches representative large sizes.
- Safety signals: filesystem and runtime access -> exact paths opened, schemas queried, URL fields retained, extensions proposed, and settings mutated.

**What counts as an answer:** A source-backed capability matrix and import/archive contract, followed by fixture-based tests proving data fidelity, sensitive-URL filtering, idempotency, cancellation behavior, zero access to excluded secret stores, indexed archive retrieval, bounded startup work, and no live renderer allocation for archived tabs.

**Scope & exclusions:** Bookmarks, sanitized history, open tabs where recoverable, Arc archived-tab history, search preferences, extension reinstall suggestions, manual archive/restore, configurable inactivity-based auto-archive, search, pagination, and retention controls are in scope. Password databases, cookies, local/session storage, OAuth tokens, browser encryption keys, Keychain entries, and silent extension loading are excluded.

**Open questions for prior-work survey:** Arc profile and archive compatibility across versions; Zen's tab unloading and archive behavior; reliable recovery of open tabs; browser-lock handling; archive indexing and compaction; supported password-manager extensions under Electron; and whether browser-native export formats provide a more stable contract than direct profile reads.
