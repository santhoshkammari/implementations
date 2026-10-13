# OpenChamber Deep-Dive Insights

10 parallel agents dissected 1298 TypeScript files across 6 packages. Here are the beautiful things they found.

---

## 1. The Zero-Build Shared Library

The most radical monorepo decision: `@openchamber/ui` has **no compiled output**. Its `build` script is `tsc --noEmit`. Every consumer (web, desktop, VS Code, mobile) imports the raw TypeScript/JSX source directly via Vite aliases and tsconfig paths.

**Why it's beautiful:**
- Zero version skew between UI and consumers
- HMR works seamlessly -- change a component in `ui/src/`, it instantly appears in the dev server
- No build pipeline coordination between packages
- One `App.tsx` renders identically across 5 surfaces

**The tradeoff:** Every consumer must configure Vite aliases pointing at `../ui/src/`. But the simplification is worth it.

---

## 2. RuntimeAPIs -- The One Contract to Rule Five Surfaces

A ~130-line TypeScript interface (`packages/ui/src/lib/api/types.ts`) defines every capability the UI needs: terminal, git, files, settings, permissions, notifications, GitHub, push, diagnostics, editor, VS Code, worktrees.

Each surface injects its own implementation:
- **Web:** Proxies to Express server via HTTP
- **Desktop:** Same (Electron runs the server in-process)
- **VS Code:** `postMessage` bridge to extension host
- **Mobile (hosted):** HTTP to remote server
- **Mobile (Capacitor):** HTTP + Capacitor plugins for native features

The shared UI never imports surface-specific modules. It always consumes through this contract. Adding a new surface = implementing the interface, not modifying shared code.

---

## 3. The VS Code Fetch Bridge

VS Code webviews are sandboxed -- no HTTP server, no direct network. OpenChamber overrides `window.fetch` in the webview and routes ALL API calls through `postMessage` to the extension host. ~50 distinct API routes, each a bridge message.

This means the same React UI works in VS Code without any HTTP server. SSE/WS paths get dedicated bridge channels.

---

## 4. In-Process Electron Server

Electron starts the Express server **inside the main process**, never as a sidecar. No process lifecycle management, no IPC serialization overhead, no port conflicts.

Packaged builds load assets from a custom `openchamber-ui://` protocol handler. Runtime config is injected via HTML `<script>` tags and `executeJavaScript()`.

---

## 5. The Golden Rule of State Cloning

During streaming, `message.part.delta` fires ~60 times/second. The old code cloned ALL state fields on every event, causing every subscriber to re-render 60x/second.

The fix: **only clone the state fields the event will mutate.** Zustand's `Object.is` selector optimization goes from theoretical to practical at 60fps. MessageList renders dropped from ~1972 to ~296 per session.

---

## 6. Event Coalescing with Delta Concatenation

The event pipeline doesn't just deduplicate -- it **concatenates** delta text. Multiple `message.part.delta` events for the same part+field become one event with the concatenated string. This reduces store writes proportionally to streaming burstiness.

Events flush at 30fps (33ms frames), or 200ms under backpressure. Double-buffering avoids allocation during flush.

---

## 7. Two Session Data Scopes

Architecturally distinctive: the system maintains **two distinct session data scopes** with different ownership:
- **Directory-scoped child stores** for live per-directory domain data
- **Global sessions cache** for cold/global coverage

Live activity indicators explicitly must NOT depend on the global cache. This prevents the global cache's refresh cadence from interfering with latency-sensitive streaming.

---

## 8. Bootstrap as Priority-Scheduled State Machine

`ChildStoreManager` treats directory bootstrap as a priority-queue state machine:
- Max 2 concurrent bootstraps
- 5 priority levels (selected > active-project > expanded > visible > background)
- Demand aggregation from multiple owners
- Priority promotion while queued
- Stale generation tracking

This is OS-scheduler-level sophistication for UI state management.

---

## 9. Multi-Run: Title-as-Metadata

Multi-run sessions encode all metadata in the session title using a slash-separated format:

```
{group}/{provider}/{model}                    # single run
{group}/{provider}/{model}/{index}            # duplicate model
{group}/{provider}/{model}/fusion             # fusion session
```

No schema changes, no new database fields. The title is human-readable AND machine-parseable. The existing session infrastructure handles everything.

---

## 10. Worktree-Based Run Isolation

Each multi-run session gets its own git worktree and branch. File changes don't conflict, each branch can be merged independently, and setup commands (like `bun install`) run per-worktree.

Two modes: fast path (create directory, start session immediately) and bootstrap wait (poll until deps installed). The bootstrap system polls every 250ms with a 5-minute timeout.

---

## 11. Fusion: LLM-Powered Result Synthesis

Fusion sends all multi-run outputs to another model with structured instructions to synthesize the best answer. Source outputs are wrapped in delimited blocks (`--- RESULT N: ... ---`). The user sees only a clean prompt; technical plumbing uses `synthetic: true` parts.

Fusion instructions are editable through Magic Prompts settings. The system relies on the LLM for conflict resolution: "prefer the most specific, well-supported, and internally consistent answer."

---

## 12. Session Goals: The Auditor Pattern

The most striking design choice: **the agent doing the work cannot declare itself done.** A separate small model audits progress after each turn and returns `{"verdict": "continue" | "complete" | "blocked"}`.

The goal loop is purely server-side. UI can close, browser can disconnect, laptop can sleep -- the server keeps running. Genuine "set it and forget it" autonomous work.

**Safety limits:**
- 20 auto-continuations max
- 3 consecutive "blocked" verdicts before settling
- 2 consecutive audit failures before stopping
- User abort = immediate pause before any continuation
- Token budget with snapshot-based accounting that survives compaction

---

## 13. Ambiguous Transport Failure Handling

When a prompt send fails after leaving the client, the system recognizes this as **ambiguous** (server may already be answering). Instead of rolling back (which would produce two AI responses), it waits for reconnection, refetches recent messages, and confirms in place.

This is a genuine distributed systems insight: distinguishing "definitely failed" from "maybe succeeded."

---

## 14. The E2E Encrypted Relay

The Private Relay is a three-layer architecture:

1. **Relay routing** via Cloudflare Worker
2. **E2E encryption** using standard WebCrypto (ECDH P-256 + HKDF-SHA-256 + AES-256-GCM)
3. **Tunnel multiplexing** with a custom frame protocol (12 frame types, 64KB max, fragmentation, batching)

Frame batching with leading-edge flush cuts WebSocket messages by ~33% without visible latency. Body buffering (512KB) prevents silent corruption from dropped relay frames.

**The relay is invisible to features.** Every HTTP request goes through `runtimeFetch()`, every WebSocket through `openRuntimeWebSocket()`. A feature developer never needs to think about relay mode.

---

## 15. Electron IPC Security: Two-Tier Command Classification

IPC commands are classified by capability:
- **Tier 1 (safe for remote):** Window ops, theme, tray, version
- **Tier 2 (local-only):** Filesystem, SSH, updates, dialogs, app restart

Every IPC handler checks origin before dispatching. Even a compromised local renderer is constrained. A remote renderer is blocked from all privileged operations.

File read security adds defense-in-depth: denied paths (`.ssh`, `.aws`, `.gnupg`), denied filenames (`.env`, `*.pem`), 50MB size limit, explicit MIME mapping.

---

## 16. Mobile: Size-Class Over Device-Class

`useTabletLayout()` uses `Math.min(width, height) >= 600px` -- not userAgent sniffing. Foldables naturally transition between phone and tablet layouts as they fold/unfold. Android `configChanges` prevents WebView recreation on fold state changes.

Keyboard choreography: iOS uses `resize: 'none'` and manually animates UI via CSS transforms during keyboard events, then snaps layout after animation completes. Avoids 1.5s lag of native `adjustResize`.

---

## 17. Presence-Aware Push Suppression

The server tracks which clients are visible (desktop/web/mobile). Mobile push is skipped when an interactive client is already showing the notification. Reduces notification fatigue without user configuration.

---

## 18. Widget Data Bridge via Window Globals

iOS WidgetKit reads live data from the app via `window.__OPENCHAMBER_WIDGET_SNAPSHOT__()`. The native shell calls `evaluateJavaScript` to invoke it, stores JSON in App Group UserDefaults, and reloads widget timelines. No IPC, no plugin -- just a synchronous function call through the WebView bridge.

---

## 19. CLI: Three-Mode Output Contract

Every command preserves identical information fidelity across human (TTY), `--quiet`, and `--json` modes. This is enforced by documentation placement rules and tested.

- `--json`: Valid JSON with `status` field, structured data, `messages` array
- `--quiet`: Space-separated fields, one record per line, errors to stderr
- Human: Clack banners, colored status, spinners, progress bars

The hand-rolled argument parser includes Levenshtein distance typo suggestions and deprecated flag migration guidance.

---

## 20. Self-Healing Instance Discovery

The lifecycle system:
- Reads PID files from `~/.config/openchamber/run/`
- Verifies each PID is alive AND is actually OpenChamber (cmdline verification)
- Proves the port is serving via HTTP health check
- Detects stale PIDs, PID recycling, desktop vs CLI runtime
- Multi-strategy stop: HTTP shutdown, then SIGTERM, then SIGKILL

---

## 21. Performance Measurement That Fails Loudly

The idle probe wraps every scheduling API (`setTimeout`, `setInterval`, `requestAnimationFrame`, observers, `fetch`, `postMessage`) to attribute wall time to call sites. Missing trace data, throttled renderers, or scenarios that never ran are all detected and reported.

**The principle:** "A metric reading zero, absent, or perfectly quiet is a claim that requires evidence, because a disabled instrument reports exactly the same thing."

---

## 22. Vite Chunk Splitting Against Bun's Nesting

The `manualChunks` function navigates Bun's nested `node_modules/.bun/<pkg>@<ver>/node_modules/<pkg>/` structure to find real package names. Without this, the first Shiki language request downloads 7.4MB of grammars. The Vite preload helper is pinned to its own chunk to prevent it from dragging Shiki into the eager graph.

---

## 23. The Skills System: Canonical Ownership + Companion Pointers

14 specialized skills with a **canonical ownership model**: each cross-cutting concern has one owner skill. Other skills reference it via pointer + local consequence, never copying the rule.

This prevents rule drift, reduces maintenance burden, and makes ownership structure explicit. The anti-duplication rule is itself enforced by the `writing-for-agents` skill.

---

## 24. Agent Self-Discovery of Applicable Guidance

Skills are matched by the **character of the change**, not file paths. The same file modification might trigger different skills depending on what is being changed. The PR review agent independently determines skill applicability from the diff, creating a closed compliance loop.

---

## 25. DOCUMENTATION.md as Distributed Architecture Reference

~40+ DOCUMENTATION.md files co-located with modules. The sync documentation alone is 440 lines of extremely detailed architecture reference. Agents are instructed to dynamically discover the nearest one before editing, not rely on a static map.

---

## The Meta-Patterns

### What Makes OpenChamber's Architecture Beautiful

1. **One React tree, five surfaces** -- Differentiation at the entry point, not the component level
2. **Contracts over implementations** -- RuntimeAPIs interface is the only boundary
3. **Authoritative state over heuristics** -- Every design decision favors live data over cached inference
4. **Failure is not empty** -- Distinguishing "no data" from "failed to get data" as an architectural invariant
5. **Process through structure, not volume** -- Mandatory loading, completion criteria, automated review
6. **Measurement-first optimization** -- Every change must prove its metric before shipping
7. **Distributed ownership, centralized routing** -- One concern, one owner, one source of truth
8. **Transparent complexity** -- E2E encryption, relay tunneling, multi-run orchestration are invisible to feature code
9. **Graceful degradation everywhere** -- Non-git projects still get multi-run, failed runs don't block others, offline mode with cache
10. **Context as a budget** -- The writing-for-agents skill treats every token of agent context as a cost decision

---

*10 agents, 1298 files, 1 insight: great architecture is not about clever tricks -- it's about clean contracts, clear ownership, and measurement over intuition.*
