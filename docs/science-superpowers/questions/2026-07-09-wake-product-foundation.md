# Wake product foundation and Zen-inspired roadmap

**Research question:** Which stability, appearance, and performance fixes must form Wake's release baseline, and which Zen-inspired browser capabilities can then be added incrementally without changing Wake into a Firefox fork or degrading authentication, privacy, and startup performance?

**Background / motivation:** Wake has a release-blocking site-appearance defect: its dark shell can leave sites such as YouTube bright or visually distorted. It also has eager runtime work, incomplete browser primitives, and visual/interaction behavior that feels older than Chrome or Arc. The approved direction is a focused productivity browser that adopts selected Zen interaction patterns while retaining Wake's Electron architecture and product identity.

**Hypotheses:**
- H0: Feature breadth is the fastest path to a useful Wake release.
- H1: A measured foundation of crash resistance, site-aware appearance without global inversion artifacts, lazy tab restoration, bounded import and extension work, modern browser interaction states, and repeatable QA must precede new browser capabilities; Zen-inspired features should then ship as independently testable slices.

**Population & unit of analysis:** Representative cold start, warm start, one-tab, many-tab, workspace restore, media, OAuth, passkey, import, dark-site, light-site, and extension scenarios; each proposed Zen-inspired capability is a separate product and implementation unit.

**Key variables (operationalized):**
- Outcome: release baseline -> no known P0 authentication or crash defect, correct dark/light appearance on representative sites including YouTube, modern and consistent browser interaction states, bounded startup process and memory growth, and passing smoke scenarios on supported macOS versions.
- Performance measures: startup-to-interactive time, renderer count, resident memory, event-loop delay, imported-record throughput, extension-load timing, and hidden-tab resource use.
- Product candidates: vertical tabs, workspaces, split views, compact mode, profiles or containers, keyboard navigation, command palette, tab unloading, and site-specific appearance controls.

**What counts as an answer:** A measured baseline with explicit budgets, a root-cause account of Wake's dated or inconsistent browser surfaces, a ranked capability matrix comparing Wake and Zen, and a staged PR roadmap in which every slice has user-visible acceptance criteria, automated coverage, agent-browser and computer-use scenarios, profiling evidence where relevant, a human verification gate, and a rollback boundary.

**Scope & exclusions:** Stabilization, performance, appearance, browser ergonomics, and selected Zen-inspired features are in scope. Forking Zen or Firefox, a full visual redesign, password/session cloning, and bundling broad feature work into the release-foundation PR are excluded.

**Open questions for prior-work survey:** Why Wake's content and authentication surfaces differ from current Chrome or Arc; which Zen features deliver the most value without Gecko-specific dependencies; which Wake behaviors are identity-defining; what performance budgets are realistic for Electron on Apple Silicon; and which browser QA flows can be automated locally without handling credentials.
