# DeepSeek Harness (dsh) — Complete Deep Dive, Caveman Mode

Repo: https://github.com/deepseek-ai/deepseek-harness · v0.1.0-rc.5 · cloned at /tmp/opencode/deepseek-harness
Everything below verified by reading source (file:line cited) AND by running the machine (see Verification Ledger at end).

---

## 1. WHAT THIS IS

DeepSeek Harness = **dsh**. Open-source agent harness from DeepSeek AI. Monorepo: ~456K lines TS+Python, 1981 TS files, ~50 npm packages, pnpm workspaces. Developer preview, breaking changes expected.

One rule drives everything:

> **Everything is a plugin.** Model adapter, tool registry, session log, sandbox, HTTP server, browser UI, agent loop itself — all plugins. No privileged core to patch.

Powered by **Cordis** (vendored, `vendor/cordis`) — plugin framework. Running dsh = stack of YAML patch layers booted into a plugin tree.

---

## 2. CORDIS — THE PLUGIN FRAMEWORK

Toy example:

```js
ctx.provide('sessions', new SessionStore(ctx))   // ctx.sessions now exists
ctx.on('session/event', (session, event) => log(session.id, event.type))
ctx.effect(() => () => cleanup())                // disposer runs on unload
```

Mechanics (`vendor/cordis/src/*`):

- **Context** (`context.ts:42`) = Proxy over service store. Plugin claims service via `ctx.<key>`.
- **Plugin = fiber** (`fiber.ts:147`). States PENDING→LOADING→ACTIVE→UNLOADING→DISPOSED. Loads only when all `inject` deps present (`fiber.ts:611`). Unload → all registrations unwind. **HMR = unload + reload.**
- **Effects** (`fiber.ts:415`): `ctx.effect()` = registration + disposer, reverse-order cleanup. Every API is an effect: `ctx.on`, `ctx.provide`, `ctx.plugin`.
- **5 event modes** (`events.ts:183-243`): `emit` (fire-forget), `parallel` (allSettled), `serial` (first non-null bails), `bail` (sync serial), `waterfall` (around-middleware — **last arg = `next()`; not calling `next()` vetoes the rest of the chain**). Waterfall law: a listener that observes must call `next()`, else it silently eats default behavior for everyone downstream.
- **Dependency re-tracking** (`fiber.ts:611-623`): service replaced → epoch changes → consumers `_unload`+`_reload`. Swap `dsh-bash-local` for `dsh-bash-sandbox` in config → every shell consumer restarts on the new provider.
- **Config** = `cordis.yml` rows (`id`, `name`, `config`, `inject`, `disabled`, `group`). Loader (`vendor/loader`) imports module, validates, mounts fiber. `!!js` expressions evaluate inside plugin `config` only (postmortem 0002: putting `!!js` in `disabled` stays truthy → feature silently off).
- **`isolate` realms** (`loader/src/config/isolate.ts`): rewrite which service a row resolves — two sessions each see own `shell` provider. How agent presets isolate per-session plugin mounts.
- **Scope** (`packages/core/scope`): per-agent registration primitive. Scope key = live Agent object. Registration through `agent.ctx` = visible to that agent only AND unwinds on agent disposal. Events flow UP the scope chain (parent sees child; child never sees parent's).

---

## 3. BOOT — PROFILES, BUNDLES, PATCHES

```
bundle layers (profile's dsh.profile.bundles, in order)
→ profile's cordis.patch.yml
→ $DSH_HOME/cordis.patch.yml   (machine-local, outranks)
→ --patch overlays (argv order)
→ flag-derived patches (telemetry disable)
```

- **Profile** = named composition in Harness home. Ships templates: `web`, `headless`.
- **Bundle** = npm package declaring `"dsh": {"bundle": {"patch": "./cordis.patch.yml"}}`. Config layer, never code the launcher imports.
- **Patch** = target row **by id**, replace whole `config` (never deep-merge), or `insert:` rows.
- `--dump-config` prints composed tree. Three bundles: **dsh-base** (first layer: model, tools, sandbox, approval, persistence, settings, credentials, telemetry, agent spine — ~90 rows), **dsh-headless** (one-shot runner, no server), **dsh-web-app** (browser app + host BFF; moves agent plane behind per-session presets).
- `dsh web` = profile `web`, default port 3080, loopback only. `--host 0.0.0.0` rejected. `dsh --profile headless "task"` = one agent, run task, print last assistant text, exit 0/1.

---

## 4. THE HEART — SESSION LOG

**Session = append-only log of typed `SessionEvent`s. Single source of truth.** LLM history is DERIVED from the log, never stored separately. Invariant: **model-visible means logged** — anything reaching a model request must be reconstructable from the log; a runtime invariant asserts it.

Events (`packages/core/session/src/types.ts:236-333`, merge-extensible via declaration merging): `turn/start|end`, `step/start|end`, `user/message`, `assistant/chunk` (**raw** stream chunks — replay fidelity), `assistant/message` (cites `sourceEventSeqs`), `tool/call` (raw unparsed JSON arguments), `tool/result`, `request/header` (full request envelope snapshot), `request/context`, `todo/write`, `session/end-seed`. Plugins extend the map; log-only events carry no `surfaceOp`.

Envelope: `{type, seq (contiguous, seq=log.length), time, data, ignorable?, surfaceOp?}`. **`ignorable` absent = required** — unknown type without the marker → refuse reconstruction (over-refuse, never silently resume a gutted session).

Write path (`session/src/index.ts:604`): lossless-JSON snapshot → validate surface → deep-freeze → push → sync `session/event` broadcast (contained listeners). Durability = plugin concern; `ctx.sessions.flush()` = the one durability barrier.

Verified transcript (my headless run, decompressed `session.jsonl.zstd`): session header, permission/preset, sandbox/mode, approval/policy (boot defaults), inbox splices, turn/start→step/start, 4 user/message (task + instructions reminder + runtime-context snapshot + skills catalog), request/header+context, 4 assistant/chunk + packed text-chunks row, assistant/message, step/end→turn/end, session/title ×2 + title-llm-request.

---

## 5. THE AGENT LOOP — TURNS AND STEPS

- **step** = one model request + tools it calls. **turn** = zero+ steps.
- Driver `ReactLoopAgent` (`agent-loop/src/agent.ts`). `followup()` → inbox → wake → `kick()` loops `while(turn())`.
- One turn: `turn/start` → `preStep`: claim inbox batch (all next-step + one next-turn) → **assemble system prompt + tool schemas** (agent-scoped) → runtime-context snapshot if changed → **`agent/pre-step` waterfall** (rewrite or reject; reject/empty first claim closes durable turn with NO step) → append messages as `user/message` → `step/start` → build request → **`agent/request` waterfall** → log `request/header` → **`llm/stream` waterfall** → raw chunks appended one-by-one as `assistant/chunk`, fed to `BlockAssembler` → `assistant/message` → tool calls → `step/end` → **`agent/turn-stopping`** (serial, no next()) → `turn/end` reason: `completed|aborted|blocked|error|max-tokens`.
- Errors: failed request → **`agent/request-error` waterfall** (llm-retry → durable retry with backoff; compaction-basic → overflow recovery). Cancellation: `cancel(cause, {keepInbox})`; started tool bodies drain (outcome ABORTED), never abandoned mid-body.
- **One-send-one-turn**: each followup = sole ordinary message of its own turn. Inbox = replay-once projection of durable `agent/inbox/spliced` events — fork/resume restore pending work from the log.

---

## 6. LLM LAYER

- **`ctx.llm`** = adapter registry + streaming API. Message `{role, content: ContentBlock[], source}`; blocks merge-extensible: `text|reasoning|image|tool-call|tool-result`. **`StreamChunk` = CLOSED union** (new variant breaks all consumers at compile time): `block-start|text-delta|reasoning-delta|tool-call-delta|block-end|usage|finish`.
- **Adapter seam** (`LlmAdapter`): required `stream(options)`. Contract: usage before finish, nothing after; tool-call arguments raw JSON strings end-to-end; one call = one provider attempt; abort→ABORTED, idle watchdog (5min)→TIMEOUT, overflow→CONTEXT_WINDOW_EXCEEDED, empty→EMPTY_RESPONSE (retryable).
- **Twin adapters** (design-verification pair): `dsh-llm-deepseek` (direct fetch+SSE to `/chat/completions`, route `deepseek-official`, catalog `deepseek-v4-flash`/`deepseek-v4-pro`, thinking modes, text-only) and `dsh-llm-pi-ai` (generic multi-provider: openai-completions/openai-responses/anthropic-messages + installed catalog routes; dormant until configured).
- **Default model** (`agent-default-model`): `deepseek-official` / `deepseek-v4-flash` (base bundle). Switch = settings change, no restart.
- **`dsh-llm-retry`**: listens `agent/request-error`; retry policy captured at registration (immutable); retries durable BEFORE cancellable wait; count derived from log (crash-safe). Codes EMPTY_RESPONSE|RATE_LIMIT|SERVER|TIMEOUT|TRANSPORT, backoff 500ms→10s.
- **`dsh-token-meter`**: measures by REPLAYING the durable log. Reuses provider `usage` when envelope matches; else heuristic 4 chars/token.

---

## 7. TOOLS — REGISTRY + GUARDED PIPELINE

`ToolDefinition` = model schema + **`output`** (schema+render+presentationMeta) + `execute(args, exec)` + optional `timeoutMs` (never sent to model) + `presentCall/presentResult` (UI render intent: generic/terminal/diff).

Pipeline (`tools/src/index.ts:1342`):
1. mint opaque `ToolExecutionToken`, snapshot+freeze args
2. **`tools/pre-execute` waterfall** → `allow|deny|ask` (ask → `ctx.approval`; degrade to deny if none)
3. **monotonic guards** (no allow result — listener ordering can't re-permit)
4. **`tools/execute` waterfall** (timeout/retry/metrics wrappers; registry re-fuses caller signal before body)
5. body runs
6. **`tools/post-execute` waterfall** → `accept|block` (block = corrective feedback becomes error)
7. `finalizeContent` once, then `tools/result` emit (frozen authoritative outcome)

Cancellation never abandons a started body: drains → ABORTED; not-yet-started → ABORTED_BEFORE_DISPATCH (synthetic results keep replay valid). Parallel calls run bounded scheduler, results committed in model order.

---

## 8. THE EXECUTION WORLD — FS / SHELL / TERMINAL / SUBPROCESS / SANDBOX / CODE-RUNTIME / LSP

Governing idea — **one execution world** (`docs/architecture.md:102`): `ctx.fs` and `ctx.subprocess` implemented by PAIRED providers (`fs-local`+`subprocess-local`, or `fs-e2b`+`subprocess-e2b`). Everything else (bash, PTY, LSP, glob/grep) builds on those two seams. Point fs+subprocess at a remote sandbox → bash+PTY+LSP follow, zero provider forks.

- **fs seam** (`ctx.fs`): target-identity based (`resolve(path)` → opaque branded `FsTarget`). Ops read/write/edit/list/stat with **CAS guards** (`createIfAbsent`, `replaceIfVersion` — backend-enforced). Atomic writes: 0700 staging, fsync, rename. **No timeouts on file I/O** (can't kill in-flight fsync). Policy: `fs/write-intent|edit-intent` decision waterfalls + `fs/observed`; fs-sandbox fence re-canonicalizes + checks writable roots (**containment, not security boundary**). Tools: `read|write|edit|read_image`, `glob|grep` (packaged ripgrep via subprocess, never shell), `str_replace_editor`.
- **shell seam** (`ctx.shell`): `resolve(request)→Spec` split — **no hidden defaults**. Providers: `bash-local` (`bash -c` via subprocess, env scrubbing + ENV_OVERRIDES), `bash-sandbox` (wraps argv via `ctx.sandbox.confine`), pwsh twins (Windows). `shell-env`: trusted `DSH_*` registry; subprocess seam discards ambient DSH_*/credential-shaped env from children. Tools: `bash` (one-shot) vs `bash` (persistent PTY — same name; deployment mounts one).
- **terminal seam** (`ctx.terminals`): persistent PTYs, owner-scoped (exact Agent identity), one exclusive send. Provider `terminal-bash`: `/bin/bash --noprofile --norc -i` via `spawnTerminal` (**node-pty**), readiness via prompt-marker + process-group inspection + /proc syscall probes. Tools: `terminal_open/close/list/read/send/signal`.
- **subprocess seam** (`ctx.subprocess`): fully explicit spec — "this seam applies no defaults". Detached process groups; `terminate()` = SIGTERM→grace→SIGKILL gated on whole-tree liveness (PID-reuse protection); Windows `taskkill /T`. Output: tail-keep + spill files (0600 'wx' in 0700 dir).
- **sandbox seam** (`ctx.sandbox`): `confine(argv, policy)` wraps EXACT argv. **Silent unconfined passthrough forbidden** — no backend → `SANDBOX_UNAVAILABLE` to model. Modes `read-only|workspace-write|danger-full-access`. Runners: **Linux bwrap→Landlock**, **macOS Seatbelt**, **Windows ACL WRITE_RESTRICTED** (partial by construction). `sandbox-policy`: override > session's `sandbox/mode` event > default (`read-only`, fail-safe). Escalation = strictly-wider table checked at execution + `approveEscalation` fail-closed before anything executes.
- **landlock-run** (`native/`): ~300 lines C11, self-restrict-then-exec. Ruleset on ITSELF (no_new_privs + `landlock_restrict_self`) then `execvp` — ruleset survives execve, all descendants confined, host free. Kernel can't enforce → **exit 125, no exec**. ABI 1→5 negotiated. Per-platform npm optional deps, no install-time build fallback.
- **code-runtime** (`ctx.codeRuntime`): fresh Worker per run, `env: {}`, heap capped, hostile inbound messages re-validated, computeMs metered on worker event-loop utilization. **Containment, not security boundary** — model code has bash-equivalent trust. Consumer: Code Mode (`presentAs('code')`).
- **e2b** = remote-sandbox POC (cloud Linux VMs): full fs + subprocess providers pointing at remote sandbox — proves one-execution-world.
- **lsp seam** (`ctx.lsp`): closed union `goToDefinition|findReferences|goToImplementation|hover`. Provider pools one server process per workspace, reads through `ctx.fs`, launches through `ctx.subprocess` — inherits the execution world.

---

## 9. MODEL-FACING CAPABILITIES (SD / Provider / Consumer triples)

- **web** (`ctx.web`): provider registry, resolved at execution (tool never knows provider). Fetch `http`: same-origin redirects only, size caps, turndown HTML→md (512 nesting guard). **No SSRF/private-network blocking — documented**; `web_fetch` off by default. Search: `deepseek-official` (Anthropic-compatible API + native web_search tool), `exa`, `perplexity` (sonar). Tools: `web_search` (maxResults 8), `web_fetch`.
- **skill** (`ctx.skills`): layered registry. Filesystem provider scans 6 roots in rank order (project `.dsh/skills` < project `.agents/skills` < custom < `$DSH_HOME/skills` < `$AGENTS_HOME/skills` < bundled). SKILL.md = YAML frontmatter + body. Tool `skill({name})`; prepended pre-step listener injects durable `<system-reminder>` catalog (sha256-digest gated); `/name` gesture loads skill.
- **spill** (`ctx.spillStore`): oversized tool text escapes context → session-scoped artifact + opaque locator + hint ("Use read with offset/limit, or grep this path"). Consumer = post-execute transformer: >maxInlineBytes → save full, replace model result with head/tail preview + locator. Best-effort (save failure keeps inline).
- **todo**: `todo_write` — whole-list replacement, validation (≤1 in_progress unless configured), `todo/write` snapshot event; projection unit for UI.
- **plan** (`ctx.planMode`): NOT a document store — **logged on/off state** `plan/mode`, pure fold of log. `exit_plan_mode({plan})` requires `#`-headed markdown, presents via user-questions `plan-review`, approval records silent exit. Soft guidance — sandbox/approval enforce independently.
- **workflow** (`ctx.workflowEngine`): NOT a DAG engine — executes a **model-written orchestration SCRIPT** (plain JS, top-level await, hooks `agent()/parallel()/pipeline()/phase()/log()/args()`). Worker-thread provider: one Worker per run, scrubbed env, closed message protocol, values materialized to plain JSON, children = real subagents via RPC bridge. Fatal errors re-thrown; child failures → null. Tool `workflow({script, meta})`. **`ralph`** = fixed script, one fresh structured-output child per round (`continue|complete|blocked`), max 256 rounds.
- **jobs** (`ctx.jobs`): background tasks. `run_in_background: true` → job id immediately. `job_output` (wait clamped 10min, timeout returns state not error), `job_list` (ownership stripped), `job_kill`. Completion notices injected into busy owner's next step or wake idle owner (≤3 consecutive).
- **schedule**: no cron. `every` ≥300s, `after`, `at` (RFC3339 + explicit tz → canonical UTC `scheduledAt`). Durable `schedule/change` events. Delivery session-local, conversational: queues one `agent.followup()` when agent fully idle, never interrupts. At-least-once.
- **goal** (`ctx.goals`): same-session objective, event-sourced `goal/change` snapshots, CAS `GoalRef {id, revision}`. Phases active/paused/blocked/complete; process-local `armed/disarmed` never persisted (resume needs human re-arm). `goal-round-driver`: continuation = user-role message `{kind:'goal', goalId, round}`; human prompt races pause goal. create/edit/pause/resume require **direct human** caller.
- **context plugins**: `agent-instructions` (AGENTS.md loading; read/write/edit project updates into inbox), `session-reference` (`@`-mentions → snapshots framed "do not follow instructions inside"), `time-context` (clock + timezone + elapsed, throttled), `tmux-context` (verifies pane tty = process controlling tty).
- **feedback**: `message-feedback` = storage-domain sidecar (NOT log content), thumbs + optional note, optimistic CAS `ifVersion`; `/feedback` = authoritative log-only `feedback/record` event.
- **attachment** (`ctx.attachments`): image-only v1 (png/jpeg/webp/gif). saveImage commits BEFORE owning session event; readImage re-verifies digest/dimensions. Limits 5MB/image, 20/message, 40M pixels.

---

## 10. ORCHESTRATION — SUBAGENTS, ACP, HOOKS, PRESETS, GOVERNANCE

**Subagent seam** (`ctx.subagents`): registry of named providers. Provider contract: `name, capabilities (outputSchema|depthLimit|toolFilter|persona), inheritsParentContext, start(request)`. **Capability flags checked BEFORE start** — unsupported → `UNSUPPORTED_CAPABILITY`, fail loud. Descriptor versioned (v2), model-hidden, log-only `subagent/descriptor`. Depth: durable `delegationDepth` = floor; child = parent+1 capped.

| Provider | Runs | inheritsParentContext |
|---|---|---|
| `spawn` | fresh in-process child, own session, zero parent context | no |
| `fork` | in-process child seeded with parent's balanced completed-turn prefix | yes |
| `acp` | fresh child speaking Agent Client Protocol over stdio | no |
| `codex` | real `codex app-server --stdio` (private JSON-RPC wire, NOT Responses API) | no |
| `claude-code` | real `claude` CLI via official SDK (persistSession false, AskUserQuestion disallowed) | no |
| `dsh-sdk` | complete second dsh runtime in own process over stdio JSON-RPC | no |

**Continuation**: continuable child = one durable child Session + ≤1 process-local Activation (many FIFO turns). `send_message` routes on residency: running→enqueue, waiting→wake, absent→**cold resume** (folds descriptor from log suffix, calls `ctx.agents.resume()` — never through a provider). Authority = **exact live**, never durable lineage. Delegated children pinned: sandbox override replayed, approval `'never'`, human questions refused. Tools: `subagent`, `send_message`/`interrupt_agent`/`list_agents`, `report` (per-continuable-child, `wakeup`|`quiet`).

**ACP** (`packages/acp`): dsh's ACP **server** (automation-only): `initialize/newSession/session/prompt/session/update/cancel`. Streams only committed assistant text. Answers approval by forwarding one-shot request_permission.

**Hooks**: `hook-protocol` = dialect-neutral wire (JSON on stdin, exit 0 ok / exit 2 block / other = non-blocking error; decisions merge most-restrictive deny>ask>allow). `hooks-claude-code` consumes unmodified Claude Code `hooks.json` (SessionStart, UserPromptSubmit→pre-step, PreToolUse, PostToolUse, Stop→turn-stopping which steers, SubagentStart/Stop). `hooks-codex` = snake_case, regex matchers, blocking-only.

**Presets** (`ctx.agentPresets`): per-session composition from preset `cordis.yml`, mounted ONCE per preset under a standing scope, joined by every agent naming it. Guards: inactive rows rejected; root-realm service leaks reject the mount — preset services MUST sit behind `isolate` realms.

**Guards**: `repeat-tool-reminder` (advisory consecutive-repeat detector, thresholds [3,5,8]), `timeout-policy` (cooperative tool timeout: swaps signal during dispatch, restores after, own-timer-only → `TOOL_TIMEOUT`).

**Interaction**: `approval` — fail-closed `allowed-once|rejected|cancelled|unavailable`; `'never'` enforced INSIDE service (listener order can't bypass); requires open turn. `permission-presets` — sandbox mode + approval bundled: `workspace-write` (workspace-write+ask), `danger-full-access` (danger+never). `commands` — human slash commands without model message. `user-questions` — pause tool call for human; only exact live runtime root may ask.

**Credentials** (`ctx.credentials`): config carries **references** (branded env-var names); providers own values. `credentials-local`: `$DSH_HOME/.credentials.yaml` (owner-only 0600 enforced), layer stack process env (wins) > managed file > `.env` > home `.env`. Empty stored value = absent everywhere. Never materialized into environment.

**Settings** (`ctx.settings`): one user-owned raw document, per-namespace sections; resolution = defaults → registrant base → user section. `describe({redactSecrets:true})` mandatory on wire. Comment-preserving leaf-level diff patching, cross-process writer lock, hot-publish.

**Identity**: anonymous per-harness-home UUID, never derived from hostname/network/git. Shared by telemetry + feedback.

**Workspace** (`ctx.workspaceRegistry`): durable entities — stable UUID over canonical `fs.realpath`, ordered sessionIds account. One-route-to-add (two-write recovery). Model-invisible.

**Extensions** (`tool-cordis` + runners): agent inspects/mounts its OWN plugins: `cordis_inspect_list/query/self`, `cordis_define` (plain JS body, validate-only), `cordis_run` (run/update, rollback), `cordis_stop`, `cordis_undefine`. Host halves in vm sandbox; client-bearing activation may need approval. **Never shipped** (security).

**Invariants** (`ctx.invariants`): package-owned runtime checks in dedicated child fibers; `fail()` → `InvariantError {code:'INVARIANT', packageName}`. Assert authoritative event/data relations only.

---

## 11. DURABILITY LAYER — PERSISTENCE / STORAGE / QUERY / TELEMETRY / TITLES / COMPACTION

- **Persistence** (`ctx.sessionPersistence`): per-session serialized chains, write-behind 200ms batch, crash repair — **never truncates a complete interrupted turn**: closes it with synthetic `turn/end {reason:{kind:'interrupted'}}`. Backends:
  - **JSONL**: `<root>/--<projectKey>--/<segment>/session.jsonl[.zstd]`. Header line, one event per line, **packed chunk rows** (~60% smaller; runs <3 never packed), checksummed concatenated zstd frames, torn-tail recovery, fsync file + parent dir.
  - **SQLite**: `sessions` + `events`, STRICT, WAL, `SCHEMA_VERSION=15` gated by `PRAGMA user_version` + `application_id 0x44534850`; refuse-not-migrate. Seek-capable suffix reads.
  - `SESSION_FORMAT_VERSION = 0`, direction-aware refusal, no compatibility promise.
- **Storage** (`ctx.storage`): hub = meeting point, no IO. Backends `json` (one whole file per unit, atomic rename) / `sqlite` (one doc per row). Domain layer: named KV units with zod schemas; writes queue on per-domain chain: backend durability → memory → `domain/changed`; rejected write leaves memory untouched.
- **Projection** (`ctx.sessionProjections`): pure fold `init + apply(state,event) + view(state)`; framework drives; whole-value events; cache write-behind + mandatory `turn/end` + disposal flush; `ver` mismatch discards rows; cold-read ladder cached → tail replay → refold → write-back.
- **session-query**: exact reads backend-independent; SQLite FTS5 full-text over semantic docs (chunks excluded); live shadows persisted via NOT EXISTS; five model-facing tools `session_search/event_search/trace/event_trace/event_read` — all workspace-authorized via caller's `header.cwd` lineage. **Not shipped by default.**
- **Telemetry**: `ledger` channel mirrors events 1:1 (dedupe `(session.id, seq)`); `ops` channel (agent-error, shutdown). Only first chunk per turn/step ships. `session-telemetry/record` waterfall = redaction point (fail-closed, exported copy only). OTEL LoggerProvider + BatchLogRecordProcessor + OTLP/HTTP. FULL/FEEDBACK_ONLY/DISABLED (default off).
- **Titles**: `session/title` log-only latest-wins. Provider seam `first-prompt`|`all-prompts`; deterministic fallback always fires; rename pins.
- **Compaction** (`ctx.compaction`): trigger at 80% of adapter context window (priced via token-meter at serial `agent/pre-step`); `CONTEXT_WINDOW_EXCEEDED` → forced balanced reduction; `/compact` manual. Transaction: `compaction/start` (lock) → summarize (**aux call = genuine prefix of last routed request → reuses provider KV cache**) → `compaction/summary` → single `user/message` with `surfaceOp: {op:'replace', start, end}` → `compaction/end` (lock released LAST — crash = orphaned lock). Retains priced tail (0.16), never splits tool-call/result pairs. `tool-result-pruner`: model-free head/middle/tail pruning.
- **Checkpoint policy**: `llm/stream` wrapper flushes before first chunk; `tools/execute` flushes recorded call before body; `agent/pre-step` flushes prior step. Model/tool never act on unflushed state.

---

## 12. PRODUCT SURFACES

**CLI** (`apps/cli`): modes = `profile` (boot), `plugin` (pnpm manage + reconcile bundles), `dump-config`/`dump-default-config`. `dsh web` = alias `--profile web`. Launcher flags first; unrecognized token = inner args (`dsh web --port 8080`). Source-launch `node --import tsx/esm` (ESM-only). SIGTERM→0, SIGINT→130, second signal force-exit. `DSH_TELEMETRY_DISABLED` (any value) patches telemetry row disabled at launcher level — config can't re-enable.

**Web app** (`apps/web` + `packages/client/*`): browser boots its own Cordis tree from `window.__DSH_BOOT__` manifest. Host: plain `node:http`, loopback-only, no TLS/auth (reachability, not authentication). Wire: **API gateway** — HTTP POST unary + 2 one-way WebSockets (`/api/events.mux`, `/api/events.host`); browser-trust fence (Host header loopback/trustedHosts, Origin check, sec-fetch-site). RPC surface: `session.* workspace.* host.* settings.* credentials.* llm.* agentPreset.* command.* skill.list session.export`. **Typert** = typed RPC generator: `@Remote` host methods → analyzer (TS program → compiler-independent model) → emitter writes descriptors + zod codecs + `typert.remote-client.d.ts` → client `ctx.remote.goals.create(...)` → `connection.rpc.call('/api', ...)`. Wire = endpoint + named `args` only. Gateway: exact-args, live resolution, 17 error codes, JSON-safety.

**SDK** (`packages/sdk`): newline-delimited JSON-RPC 2.0 over stdio. Requests `initialize` / `session/prompt` / `shutdown`; notifications `session.event` / `session.status` / `subagent.started|finished`. Server plugin wires stdin/stdout (stdout = protocol only). TS client spawns runtime subprocess, notification fan-out, close = shutdown→EOF→SIGTERM→SIGKILL.

**Python SDK** (`python/sdk`): twin of TS client. `DeepSeekHarness` (context manager, `run()` → `RunResult{final_response, finish_reason, events, notifications}`), `HarnessClient`. Synchronous only. Runtime resolution: explicit bin → bridge → bundled `deepseek_harness_runtime`. Zero-config = wrapper injects `DSH_CORDIS_CONFIG` (explicit parameter pass, never hidden fallback).

**Bundled runtime** (`python/sdk-runtime`): single-file exe via **`@yao-pkg/pkg` `--sea`** (Node SEA, fork of vercel/pkg). Closure staged from deploy manifest, symlink-free, whole-tree assets (Cordis bare imports defeat static analysis). Linux x64/arm64 + macOS x64/arm64 only. macOS carries mandatory `-spawn-helper` sidecar (node-pty). Shipped cordis.yml = jsonrpc-agent minimal variant. Wheels pin runtime-bin == SDK version; PEP 440 `0.1.0rc5`.

---

## 13. LAST GROUPS — MCP / HOST / UTIL / TEST-SUPPORT / UI

**MCP** (`packages/mcp/mcp-client`): one plugin per external MCP server. Stdio or Streamable HTTP. Discovers `tools/list`, registers on `ctx.tools` as `mcp__<serverName>__<rawName>` (pure function of pair; lossy-normalization gets deterministic 12-hex sha256 suffix). Resources/Prompts NOT bridged. Two-phase tool swap: fetch next generation, then dispose+register — failure keeps previous; conflict rolls back to ZERO tools from that server. Reconnect supervisor: maxAttempts 10, doubling 500ms→30s, budget resets after survival past maxDelay. Child env scrubbed (credential-shaped + `DSH_*` dropped). `examples/mcp-memory/*` = three default-off overlays.

**Host plane** (`packages/host`):
- `apiproxy` = `ctx.apiProxy`, shared RPC gateway. **Four-quadrant wire union** (transport-agnostic): ClientRequest / ServerResponse / ServerRequest / ClientResponse. Responses echo `rpcId`. ~55 methods. Security: every POST must be `application/json` → 415 otherwise (**cross-site simple requests can't run side-effectful methods**); settings/credentials/llm served only from allowlist (else `settings-not-exposed`); secrets never ride responses; privileged methods loopback-pinned. `session.export` = GET streaming ZIP (fflate, level 0-9 default 6).
- `webserver` = plain node:http carrier; exact → longest-prefix → fallback seat.
- `frontend-static` = SPA dist server (traversal 403, misses → index.html 200, index taps inject boot manifest).
- `directory-picker` family: seam with `native` (osascript/zenity/kdialog/koffi-COM) vs `browse` (in-app Miller columns, maxEntries 1000); `auto` resolves at boot.
- `plugin-inventory` = read-only Typert Remote of live loader entries.

**Util** (zero-dependency): `atomic-write` (wx temp + rename, never through symlink, file-lock, no fsync by design), `brand` (`Branded<B>` type-only), `home-paths` (`$DSH_HOME` → `~/.dsh`), `launch-environment` (immutable snapshot with per-layer provenance: process > project .env > user .env; omit = refusal), `native-command` (no-shell execFile), `output-retention` (`ItemRetainer`/`TextRetainer`, UTF-8-boundary-preserving cuts; `truncated` = budget omission), `timeout` (timing + classification ONLY, owns no termination).

**test-support** — keyless test architecture: `DSH_SNAPSHOT ∈ replay|record|refresh`. **The fixture IS the session JSONL**: `llm-replay` reconstructs each model call's chunk sequence by grouping `assistant/chunk` at `(turn, step)` boundaries (group must end in finish; missing = thrown-stream fingerprint → `replay.override.json`). Live sessions bind by first-call order; `assertConsumed()` turns silent underruns into diagnostics. `llm-mock-server` = scriptable fault server, ~24 behaviors FIFO (`success`, `rate_limit`, `context_overflow`, `stall`, `random`...). `acp-snapshot` = launcher + harness + normalizers + suite factory. `loader-smoke` = subprocess smoke.

**Client UI catalog (31 `ui-*` + support)**: browser half talks to host 3 ways — (1) **apiproxy RPC** (`session.* workspace.* host.* settings.*`), (2) **Typert remotes** (`ctx.remote.*`: commands, goals, messageFeedback, pluginInventory), (3) **push projections** (`session/projection` frames, higher-seq-wins: goal, plan, todos, permissions, tokenUsage, sessionStats, contextPressure, contextBreakdown). Notable: `ui-conversation` (chat/composer/approvals/context-meter), `ui-tool` (keyed atomic tool views terminal/read/diff/search/web), `ui-trajectory` (turn-aware ledger), `ui-settings-models` (provider config page: joins `llm.providers` + `settings.describe` + `credentials.describe`, writes `settings.mutate` + `credentials.set`, `llm.discoverModels`), `ui-workspace` + pickers, `ui-user-questions` (composer takeover, answers via POST `/api/respond`), `ui-cordis` (dynamic-plugin panel). Most Node halves **deliberately empty** — row exists in host Loader, real code ships via `exports["./client"]`.

---

## 14. WHY LAYER — AGENT NOTES (`.agents/notes/`)

Every non-trivial change carries an Agent Note (decision/mechanism/rationale). Read ~50. Headline: **this repo is a simplification machine.** Half the notes are class `simplification` — remove code/surface/vocabulary with no current consumer. Through-line: *one owner per fact, one route per outcome, delete anything without a consumer, session log = single durable reality.*

Load-bearing whys:
1. **Session log = the one reality** (`2026-06-11-event-sourced-sessions.md`; cascade: `remove-agent-boundary-mirror-events`, `collapse-trace-only-session-events`, `agent-loop-observable-state-machine` — exactly 4 loop extension events; `session-log-version-mechanism`; `remove-synthetic-log-only-turns` — **archived turn-enclosure invariant is dead**, don't trust it).
2. **Tool schemas assemble WITH the system prompt** (`2026-06-11-tool-schemas-in-prompt-assembly.md`): one `system-prompt/assemble` waterfall owns everything the model is told up front. Why tool catalog generator must BOOT each plugin — a tool schema is not statically knowable.
3. **Delete-without-consumer**: `remove-tui-package` (Web = shipped surface), `remove-sdk-project-toolchain`, `drop-image-content-block` → **re-added as durable attachments** (removal: no path honored it = silent loss; re-add: Web composer + pi-ai created real producer AND durable content-addressed path).
4. **Per-session agent presets** (`2026-08-03`) = newest structural pivot: host plane vs agent plane. Mount ~3ms/~600KB per session. Root-realm leaks reject.
5. `one-send-one-turn`; `bounded-fixed-rate-schedule` (no cron, no turn-storms); `conversational-schedule-delivery` (reminder = ordinary followup); `explicit-schedule-time-zone` (canonical UTC); `buffer-free-feedback-telemetry` (FEEDBACK_ONLY reads log at feedback time, zero memory otherwise); `production-dsh-excludes-product-subagent-providers` (codex/claude-code providers not in base — dormant packages still enter every install).

---

## 15. POSTMORTEMS (4) — HOW THE PROCESS LEAKED, AND THE GUARDRAILS

Postmortem = backward-looking failure record: what broke, mechanism, why every safety net missed it, guardrails added. Subtle + systemic + costly-to-rediscover.

- **0001 — ACP server crashed on connect**: stray `export default` dropped plugin `inject` (Loader interop), and a traced optional-service lookup failed across a shadow boundary. 178 green unit tests + 100% coverage missed both because every test mounted the plugin through a path that didn't exercise real loading. Zed editor hit `cannot get property "agents" without inject` instantly. Fix: keyless real-Loader coverage + package rules (no default exports; optional-service access rules).
- **0002 — FS snapshot tools permanently disabled**: `disabled: !!js ...` — Cordis evaluates JS expressions ONLY inside plugin `config`; entry metadata like `disabled` is consumed raw, so a truthy expression object kept filesystem tools disabled in every mode. Snapshot suite "passed" because it re-recorded `UNKNOWN_TOOL` failures as expected output — deterministic replay of the regression, not of behavior. Fix: explicit fs overlay + static-config and snapshot-result guards.
- **0003 — Web agent validated a replacement server**: the GUI session cwd was the workspace, not the app; model launched bare Vite (HTTP 200 with white screen — `window.__DSH_BOOT__` missing), then launched a second full server on another port and probed THAT. Never probed the real 3081. Background processes spawned via shell `&` bypassed job identity. Fix: current URL + runtime mode become model-visible + shell-queryable; reject standalone Vite; verify production refresh + dev HMR against external state.
- **0004 — Landlock partial notice misclassified child failures**: on old Landlock ABIs the launcher prints benign `landlock-run: partial enforcement` before every child. The harness matched `landlock-run: ` substring + any nonzero exit → ripgrep's exit-1-no-match surfaced as `SANDBOX_UNAVAILABLE`. Second layer: bash-backed filesystem search replaced the structured error with generic `SEARCH_FAILED`. Fix: status-gated fatal evidence after exact informational exclusions; filesystem search uses packaged ripgrep through `ctx.subprocess` (off sandboxed bash). Stderr remains in-band — tighter conjunction prevents accidental collision, doesn't authenticate the writer.

---

## 16. FULL TOOL UNIVERSE — 52 MODEL-VISIBLE TOOLS (24 packages, from docs/tool-catalog.md)

- **Filesystem**: `read`/`write`/`edit` (diff cards), `read_image` (attachments + image-capable route), `glob`/`grep` (packaged ripgrep, capped spill), `str_replace_editor` (only `minimal` preset).
- **Execution**: `bash` (one-shot) / `bash` (persistent PTY) / `pwsh`; `terminal_open/close/list/read/send/signal`; `run_code` (reserved, `mode: code`).
- **Delegation**: `subagent` (alias `subagent_fork`), `send_message`, `interrupt_agent`, `list_agents`, `report`.
- **Long-horizon**: `workflow`, `ralph`, `create_goal/get_goal/update_goal`, `schedule_create/delete/list`.
- **Knowledge**: `web_search` (mounted) / `web_fetch` (**off by default — SSRF**), `skill`, `session_search/event_search/trace/event_trace/event_read` (**not shipped by default**), `lsp`.
- **Self-modification**: `cordis_define/inspect_list/inspect_query/inspect_self/run/stop/undefine` — **never shipped**.
- **Human**: `ask_user_question`, `exit_plan_mode` (registered while inactive — catalog doesn't churn).
- Shipped reality ≠ catalog: `even-out-shipped-tool-rosters` = "twenty unconditional names" on both surfaces.

## 17. CONFIG CATALOG — NOTABLE FIELDS (from docs/config-catalog.md, 3151 lines, generator-gated)

`dsh-llm-deepseek`: apiKeyEnv, baseURL, thinking, reasoningEffort (default high), maxTokens 256k, defaultContextWindow 1M, streamIdleTimeoutMs 5min · `dsh-agent-default-model`: provider+model required · `dsh-agent-loop`: maxParallelToolCalls, agents[] · `dsh-sandbox-policy`: mode default **read-only** fail-safe · `dsh-user-approval`: policy ask|never · `dsh-permission-presets`: presets table + defaultPreset · `dsh-tools`: mode native|code|both, maxParallelSubCalls 10 · `dsh-tool-subagent`: provider, toolName, backgroundMode, maxDepth default 3 · `dsh-tool-jobs`: wait 30s/max 10min, maxConsecutiveWakes 3 · `dsh-compaction-basic`: thresholdRatio 0.8, retainRatio 0.16, compactionRetries 1 · `dsh-session-persistence-jsonl`: root REQUIRED (no default on purpose), packChunks true, compression zstd|none · `dsh-session-telemetry-otel`: mode FULL|FEEDBACK_ONLY|DISABLED (default off) · `dsh-web`: searchProvider/fetchProvider pinning · `dsh-mcp-client`: union StdioConfig|StreamableHttpConfig, reconnect 500→30s ×10 · `dsh-invariants`: enabled true, allow/blocklists · `dsh-credentials-local`: path, dshHome, watch.

---

## 18. REPO ENGINEERING — GATES / I18N / WEBSITE / CI

- **Gates**: `check:ci` matrix (static/coverage/snapshots+artifacts/compat/python/wine-windows), `hygiene` (knip + publint + constraints + licenses + invariants + NodeNext + runtime-closure), `doc-sync` (all doc gates). Coverage gate = per-file 100% on `packages/*/*/src`. `test:coverage` not `test` is the CI gate.
- **Generated catalogs**: tool-catalog, config-catalog, event-producer-consumer matrix, capability-seams graph, module-graph, persistence-catalog — generator + verify gates.
- **Vendored rescope** (`docs/rescope.md`): `cordis`→`@deepseek-ai/cordis` 4.0.0-rc.7, cosmokit, schemastery 3.18.0, loader, include, group, timer, hmr, logger-console. Why: every harness package declares the framework as peerDependency — publishing under upstream names would squat the registry.
- **i18n**: every doc = `foo.md` + `foo.zh.md` + `foo.i18n.yaml` (blob-hash consistency record, content-addressed). Structure must mirror 1:1 (headings, lists, tables, code fences byte-identical). Merge driver merges only when both sides verify — fail-closed. Website = VitePress **projection** (manifest copies canonical docs into disposable `.generated/`; website owns no prose; zh-CN root + en locales).
- **CI**: static, coverage (bubblewrap-prepared), snapshots+artifacts (Playwright Chromium), node 22.19/24/26 compat smokes, python 3.10 keyless suite, windows wine gates; opt-in E2B + pi-ai provider e2e (workflow_dispatch, spend real tokens); release pipelines for dsh npm family, vendor family, python wheels, landlock-run native packages; docs-pages deploy.

---

## 19. THE INVARIANTS HOLDING IT UP

1. **Model-visible ⟺ logged** — runtime invariant asserts it. New model-visible input → new session event.
2. **Everything is a plugin; registrations are effects** — no privileged core.
3. **Waterfall listeners MUST call `next()`** unless owning the decision; `agent/turn-stopping` = the one serial terminal checkpoint.
4. **Capability seam = SD + Providers + Consumer**, complete, never one role.
5. **Explicit > implicit at package boundaries** — resolve(request)→Spec; no hardcoded tunables.
6. **Fail closed, fail loud** — sandbox unavailable → error to model; unknown event type → refuse; bad boot → labeled diagnostic + exit 1; unsupported capability → UNSUPPORTED_CAPABILITY.
7. **Secrets**: empty = absent; owner-only files; redacted describe on every wire.
8. **Pre-release**: no migrations, refusal-over-migration; backends reject old on-disk formats.
9. **Typed everything**: branded ids, closed unions end in `assertNever`, merge-extensible maps fall through documented default.
10. **Defensive patterns** (docs/defensive-patterns.md): orthogonal outcomes; dispose reaches quiescence; contain callback exceptions; never hand untrusted output ambient env or predictable paths; unlink link-shaped paths.

---

## 20. VERIFICATION LEDGER (what I actually ran)

| Surface | Result |
|---|---|
| Clone + install (pnpm, 33s) + build (lib tsc+tsdown + web frontend) | PASS |
| `dsh --profile headless --dump-config` (composition engine) | PASS |
| Headless end-to-end + mock LLM (boot→loop→llm/stream→title→persist→exit 0) | PASS, exit 0 |
| Session `session.jsonl.zstd` transcript matches architecture docs | PASS |
| Web UI boot + `/api` gateway typed envelope (malformed request → `bad-request`) | PASS |
| Full TypeScript unit suite (`vitest run --maxWorkers=2`) | PASS — 803 files / 13,403 tests / 0 failures |
| Snapshot tier (keyless replay: headless, compaction, version guard, subagents, forks, resume) | PASS — 21/21 |
| Python SDK unit tests (keyless, fake runtimes) | PASS — 33/33 |
| Python SDK → real jsonrpc-agent runtime → mock LLM (stdio JSON-RPC full stack) | PASS — exit 0, final_response received, session persisted |
| `process-exit.spec.ts` earlier timeouts under full parallel load | classified — standalone 5/5 in 2.5s, green within 2-worker run (contention, not defect) |

Knowledge base: 12 parallel explorer reports (each with file:line cites) + direct reads of architecture/glossary/event-map/capability-graph/catalogs/postmortems/agent-notes/rescope/dev/testing/defensive docs + source-level verification of the vendored Cordis core.

Coverage: every package group — core, llm, api/typert, session/storage/compaction/query, fs/shell/terminal/subprocess/sandbox/code-runtime/e2b/lsp, web/skill/spill/todo/plan/workflow/jobs/schedule/goal/context/feedback/attachment, subagent/acp/hooks/preset/guard/interaction/credentials/settings/identity/workspace/extensions/invariants, mcp, host, util, test-support, client+31 ui-*, boot/bundle/sdk/examples, vendor, native/landlock, python sdk+runtime, apps, website, docs, CI.

---

## 21. THE 7 DEFENSIVE PATTERNS (docs/defensive-patterns.md, full)

Bug-class rules — each is a defect class that actually shipped or nearly shipped here.

1. **Report orthogonal outcomes independently.** Process can time out AND exit 0 (trapped signal). Surface each fact (`timedOut`, `signal`, `exitCode`) on its own field; never nest one flag's report inside another's branch — caller reads a cut-short run as clean success.
2. **Honor public contracts on BOTH sides.** `LlmAdapter.stream()` may throw OR emit `finish {error|aborted}`; `LlmRuntime.stream()` exposes model-request failures ONLY as terminal finish chunks (middleware/consumer defects stay thrown). Consumers never guess where an exception came from.
3. **Async state is not synchronous state.** `agent.followup()` has no per-message result; completion races turn boundaries; `reader.close()` fires for EOF and disposal. Never treat `agent/status`/`whenIdle()` as one follow-up's result — several queued follow-ups share one `running` interval. Automation caller defines its interval explicitly (durable inbox receipt → next whole-agent idle). Handle the "nothing to wait for" branch — a wait that can never occur hangs.
4. **Dispose must reach quiescence, not just request it.** Kill/abort then RETURN = orphans. Cleanup async: kill → await `done`; close listener/notification registries BEFORE killing so late completions stay silent.
5. **Contain callback exceptions in the dispatcher.** One throwing listener must not reject the promise it runs inside or starve later listeners. try/catch + log; one bad subscriber never breaks core lifecycle.
6. **Never hand untrusted output the ambient environment or predictable paths.** Spawned commands get scrubbed env (drop `*KEY*`/`*SECRET*`/`*TOKEN*`/`*PASSWORD*`); temp/spill files in private 0700 dir, random names, exclusive `'wx'` 0600 opens — predictable world-readable paths invite symlink races + disclosure.
7. **Unlink link-shaped paths.** Path that may be a symlink/junction: `lstatSync().isSymbolicLink()` then `unlinkSync` — unlink deletes only the link, refuses a real dir, never follows into target. Windows `rmSync(link)` throws `ERR_FS_EISDIR` on junction; recursive `rmSync` may descend THROUGH one into its target. Reserve recursive delete for known real dirs.

## 22. DEVELOPMENT GUIDE ESSENTIALS (docs/development.md, full)

- **Two-aggregate TS layout** — architectural fact the source tree hides: `tsconfig.host.json` + `tsconfig.client.json` are two SEPARATE ts.Programs because both sides declaration-merge the cordis `Context` interface under the same keys with different services; one program seeing both merges reports a collision. `tsconfig.base.json` = shared options + source `paths` map, deliberately WITHOUT `include`/`files` (leak into every package project + narrow the facade). New package registered in exactly ONE aggregate. `api/remotes` is the only split package (Host entry + Client entry that imports generated `/remote` declarations — Host tsdown must generate first).
- **Build order**: `tsc -b host` → `tsdown --env.DSH_BUILD_FACE host` (Typert runs here, generates Host reflection + Host-for-Client Remote projection) → `tsc -b client` → `tsdown client` → `build:web`. Both tsdown passes use the same workspace match; `DSH_BUILD_FACE` selects entries per phase.
- **Source plane vs artifact plane**: vitest resolves workspace imports via base `paths` to `src` (never through package `exports` to stale `lib/` — would load second copies of module singletons); built artifacts consumed only by explicit `lib`-mode subprocesses + built smokes.
- **Env**: `DEEPSEEK_API_KEY`, optional `DEEPSEEK_BASE_URL`, gitignored root `.env`. e2e self-skips without key.
- **Git hooks** (lefthook, worktree-local): `pre-commit` (staged pairing records vs staged blobs, staged-file oxlint + bounded fix retry, THIRD_PARTY_NOTICES regen, whitespace check, vendor manifest guard), `pre-merge-commit` (same pairing check), `pre-push` (`pnpm run typecheck` incl. generated Typert contracts). Hooks intentionally run NO tests/snapshots/docs/builds — contributors run relevant checks, CI owns exhaustive.
- **TODO markers by urgency**: `FIXME` (blocks release), `TODO` (fix soon), `XXX` (maybe someday).
- **Demos** (need key): `pnpm dsh --profile headless "task"`, `pnpm run demo:cordis` (self-modification), `pnpm run demo:acp`.

## 23. TESTING POLICY TIERS (docs/testing.md, full)

- **Unit** (`pnpm run test`): vitest, package + example specs + script specs. Every registry gets an HMR-safety test (dispose contributing fiber, assert cleanup).
- **Coverage gate** (`pnpm run test:coverage`): THE gating run — per-file 100% on `packages/*/*/src`. Uncovered line = often dead code the gate flags for deletion. Line coverage necessary, never sufficient. pwsh suite self-skips without real `pwsh`; CI ships pwsh and enforces full bar.
- **Real-API e2e** (`pnpm run test:e2e`): with-key, self-skip without key (DeepSeek + EXA/PERPLEXITY keys). "Inference is cheap here — we are DeepSeek, do not ration real-API tests." No-key test proves plumbing; only with-key proves the agent works against a real model.
- **Snapshot** (`pnpm run test:snapshot`): keyless, ACP boots real automation-server example + replays recorded session, diffs normalized JSON-RPC + re-persisted log; headless scenarios boot explicit composition through unexported JSONL test driver.
- **Web browser snapshot** (`pnpm run test:web`; required Linux PR gate): Chromium vs `apps/web/tests/snapshots/`, CI forces read-only replay.
- **Rules**: prefer real implementation over mock (mock only LLM/network/clock); **verify the world, not the self-report** (re-run command / re-read file externally; keyword probe on agent's own output lets a cheating agent pass); **test the real entry path** (built `lib/bin.js` under plain node — tsx masks settle races; plus `expect('default' in mod).toBe(false)` for Loader-export regressions); source plane only; every non-trivial model/protocol/human-visible change adds a keyless snapshot in the same PR.

## 24. REPO'S OWN AGENT SKILLS (.agents/skills/) + OTHER BITS

The repo dogfoods skills for its own agents (11): `dsh-archive-agent-notes`, `dsh-code-review`, `dsh-doc-site-sync`, `dsh-doc-standards`, `dsh-find-simplifications` (hunt non-obvious simplifications — the simplification machine made operational), `dsh-merging-stacked-prs`, `dsh-pre-push-checks`, `dsh-prose-standard`, `dsh-translate-docs`, `dsh-trim-cot-leakage` (fix prose that reads like leaked reasoning transcripts), `record-browser-gif`. `.claude/` contains `skills` (symlink-style mirror). `web/AGENTS.md` adds: **reject redirects on credential-bearing provider requests** — HTTP client must fail before following any redirect; regression coverage proves redirect target never contacted and every credentialed provider opts in. `.gitlab-ci.yml` = separate Python release pipeline (GitLab): runs only on `python-v*` tags, verifies tag == package.json version, builds + publishes wheels (GitHub Actions owns npm/native releases; GitLab owns PyPI).

## 25. KEY FILES TO READ FIRST (source map)

Warm-up: `docs/architecture.md` → `docs/cordis-primer.md` → `AGENTS.md` → `packages/README.md`.

| Topic | File |
|---|---|
| Cordis core (context/fiber/events/reflect) | `vendor/cordis/src/{context,fiber,events,reflect}.ts` |
| Loader + config + isolate | `vendor/loader/src/index.ts`, `vendor/loader/src/config/{entry,isolate}.ts` |
| Session log types + append | `packages/core/session/src/{types,index}.ts` |
| Turn/step driver | `packages/core/agent-loop/src/agent.ts`, `tool-calls.ts` |
| Agent + inbox + events | `packages/core/agent/src/{runtime-types,inbox}.ts` |
| Tool registry + pipeline | `packages/core/tools/src/index.ts` |
| Prompt assembly | `packages/core/system-prompt/src/index.ts` |
| LLM seam + adapter | `packages/llm/llm/src/index.ts`, `packages/llm/llm-deepseek/src/{adapter,translate}.ts` |
| Capability seams (SD/Provider/Consumer) | `docs/capability-seams.md`; template = `packages/shell/*` |
| fs seam + local | `packages/fs/fs/src/index.ts`, `packages/fs/fs-local/src/index.ts` |
| sandbox + landlock | `packages/sandbox/sandbox-local/src/index.ts`, `native/landlock-run/packages/entry/src/main.c` |
| subprocess process-tree | `packages/subprocess/subprocess-local/src/{spawn,process-inspector}.ts` |
| subagent seam + providers | `packages/subagent/subagent/src/index.ts`, `subagent/src/{continuation,descriptor}.ts` |
| Persistence JSONL | `packages/session/session-persistence-jsonl/src/index.ts`, `core/session/src/chunk-rows.ts` |
| Compaction | `packages/compaction/compaction-basic/src/{index,region,summarizer}.ts` |
| Typert RPC | `packages/typert/generator/src/analyzer.ts`, `packages/api/gateway/src/index.ts` |
| SDK protocol | `packages/sdk/protocol/src/types.ts`, `packages/sdk/server/src/server.ts` |
| Python SDK | `python/sdk/src/deepseek_harness/{client,api}.py` |
| Boot composition | `apps/cli/src/profile-boot.ts`, `packages/boot/app-boot/src/profile.ts`, `packages/bundle/base/cordis.patch.yml` |
| Web boot | `packages/client/web/src/boot.tsx`, `packages/client/connection/src/*` |
| Tool catalog (all 52) | `docs/tool-catalog.md` |
| Config catalog (all fields) | `docs/config-catalog.md` |
| Event matrix (producers/consumers) | `docs/event-producer-consumer.md` |
| Rationale (why layer) | `.agents/notes/implemented/` (esp. event-sourced-sessions, per-session-agent-presets, session-log-version-mechanism, remove-synthetic-log-only-turns) |
| Failure history | `docs/postmortem/0001..0004` |
| Test architecture | `packages/test-support/llm-replay/src/index.ts`, `packages/test-support/acp-snapshot/src/harness.ts` |
