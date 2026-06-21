# Doc Review — headroom (docs) — 2026-06-21

> Review artifact / draft — NOT a KB entry. Recommendations only; nothing has been merged or deleted.

Scope: `/Users/wax/coding/headroom/docs` (markdown only, recursive). This repo has no
`knowledge-base/` tree; this is a lighter pass focused on duplication, drift, and
promotion-worthy reusable doctrine. 28 files read.

## Inventory

| Path | Purpose | "Layer" |
|------|---------|---------|
| `docs/README.md` | Generic Fumadocs/Next.js scaffold readme (boilerplate, no project content) | scaffold/boilerplate |
| `docs/auth-modes.md` | Auth-mode classifier (Payg/OAuth/Subscription) contract, decision order, extension guide | reference |
| `docs/bedrock.md` | AWS Bedrock operator guide — deploy, creds/IAM, region flags, compression policy, metrics, rollback | reference/operations |
| `docs/observability.md` | Rust-proxy Prometheus metric catalogue (Bedrock Phase D + proxy-wide Phase G) | reference/operations |
| `docs/rtk-architecture.md` | ADR-style "why RTK is wrap-CLI only, not proxy-side" decision record | decision (ADR) |
| `docs/spec/SPEC.md` | Living-spec constitution + section index (lists 001–021) | spec index/governance |
| `docs/spec/001-vision.md` | What Headroom is / is not, value prop | spec |
| `docs/spec/002-architecture.md` | Component diagram + descriptions, CCR architecture | spec |
| `docs/spec/003-adrs.md` | Architecture Decision Records (proxy vs SDK, etc.) | spec (ADR) |
| `docs/spec/004-domain-model.md` | Core entities (Session, etc.) | spec |
| `docs/spec/005-integrations.md` | Agent plugin contracts | spec |
| `docs/spec/006-actors.md` | Actor types + interactions | spec |
| `docs/spec/007-behavior.md` | Proxy modes, mode-by-mode behavior | spec |
| `docs/spec/008-capabilities.md` | Feature matrix across surfaces | spec |
| `docs/spec/009-compliance.md` | Data-handling guarantees, privacy | spec |
| `docs/spec/010-data.md` | Storage, retention, env vars | spec |
| `docs/spec/011-deployment.md` | Deployment profiles/presets/runtimes | spec |
| `docs/spec/012-diagrams.md` | Component/sequence/data-flow diagrams | spec |
| `docs/spec/013-disaster-recovery.md` | Failure modes + recovery | spec |
| `docs/spec/014-governance.md` | Decision-making, releases, repo structure | spec/governance |
| `docs/spec/015-interfaces.md` | CLI, HTTP, env var, plugin ABI surfaces | spec/reference |
| `docs/spec/016-observability.md` | Telemetry/metrics/logs (Python-era `headroom_*` metrics) | spec |
| `docs/spec/017-operations.md` | Health endpoints, logs, upgrades | spec |
| `docs/spec/018-policies.md` | Default behaviors + overrides | spec |
| `docs/spec/019-quality.md` | Test pyramid coverage | spec |
| `docs/spec/020-security.md` | Threat model, supply-chain | spec |
| `docs/spec/021-testing.md` | Test strategy per surface | spec |
| `docs/spec/022-rust-migration.md` | Rust migration motivation + phased plan (Stage 0 complete) | spec |

## De-dupe clusters

### Cluster 1 — Observability / metrics catalogue (3 files, two namespaces)
Files: `docs/observability.md`, `docs/spec/016-observability.md`, and the metrics
section embedded in `docs/bedrock.md`.

- `docs/observability.md` documents the **Rust proxy** metric surface: `bedrock_*`
  and `proxy_*` names (Phase D / Phase G), constants in
  `crates/headroom-proxy/src/observability/metric_names.rs`.
- `docs/spec/016-observability.md` documents an **older Python-era** surface:
  `headroom_requests_total`, `headroom_tokens_original`, `headroom_savings_percent`,
  etc. These metric names do not appear in `observability.md` — i.e., the two docs
  describe different (and likely divergent/stale) metric namespaces for the same
  endpoint (`/metrics` on `:8787`).
- `docs/bedrock.md` reproduces the Bedrock metric table (`bedrock_invoke_count_total`,
  `bedrock_invoke_latency_seconds`, `bedrock_eventstream_message_count_total`) that
  `observability.md` also carries verbatim.

Merge recommendation: Make `docs/observability.md` the **canonical** runtime metric
catalogue. (a) Reconcile `016-observability.md` against it — either update 016's
`headroom_*` table to the current Rust names or explicitly mark the Python-era names
as legacy/migrated, cross-linking `docs/observability.md`. (b) In `bedrock.md`, replace
the inline Bedrock metric table with a one-line pointer to `docs/observability.md`
(§ Bedrock route) to keep one source of truth and avoid table drift. Keep the
Bedrock-specific PromQL examples in `bedrock.md` (operational, not a catalogue dup).

### Cluster 2 — ADR content split across two homes
Files: `docs/rtk-architecture.md` (standalone ADR) and `docs/spec/003-adrs.md`
(the spec's ADR register).

Not a content duplication, but a structural inconsistency: `rtk-architecture.md` is a
full ADR ("decided, locked at Phase G PR-G3") living outside the spec ADR register.
Recommendation: add an ADR stub/cross-link in `003-adrs.md` pointing to
`docs/rtk-architecture.md` so the decision is discoverable from the spec, OR relocate it
under `spec/` as a numbered ADR. Low urgency — flag, don't force.

## Refactor recommendations

- **`docs/spec/SPEC.md`** — Section index lists 001–021 only; `022-rust-migration.md`
  exists on disk but is **not referenced** in the index table or change log. Add row
  022 (Status: in-progress) and bump the change-log. Without this the spec index is
  silently incomplete. Why: index/file drift hides an in-progress, high-importance doc.
- **`docs/spec/016-observability.md`** — Metric names (`headroom_*`) appear stale vs the
  live Rust proxy surface in `docs/observability.md` (`proxy_*`, `bedrock_*`). Either
  refresh or annotate as legacy with a forward pointer. Why: a contributor using the
  spec as "canonical source of truth" (SPEC.md's stated governance) would wire the wrong
  metric names.
- **`docs/spec/003-adrs.md` — ADR-006 contradicts the migration.** ADR-006 ("Why
  Python for Core Implementation") is still marked authoritative with no superseded
  note, while `022-rust-migration.md` (Stage 0 complete) is actively replacing the
  Python core with Rust. A contributor reading the ADR register would conclude Python
  is the deliberate core-language choice. Add a superseded/amended marker to ADR-006
  pointing to 022, or add an ADR-009 ("Why Rust core") to the register. Why: the ADR
  register is the spec's "why we chose X" surface; a live contradiction there is more
  misleading than a stale metric table.
- **Frontmatter consistency** — None of these files use YAML frontmatter; spec files use
  an inline `**Status:** done` convention, top-level docs use ad-hoc headers
  (`rtk-architecture.md` has `**Status:** decided` / `**Owner:**`; `auth-modes.md`,
  `bedrock.md`, `observability.md` have none). Not a KB tree, so frontmatter is optional —
  but if any KB-style indexing is wanted later, standardize on the spec's `**Status:**`
  line at minimum for the three top-level reference docs. Low priority.
- **Cross-link hygiene** — `auth-modes.md` and `bedrock.md` reference `REALIGNMENT/...`
  paths (e.g., `REALIGNMENT/08-phase-F-auth-mode.md`, `REALIGNMENT/04-phase-B-live-zone.md`)
  and a `project_compression_realignment_2026_05` memory note. These targets are outside
  `docs/` (not in scope here) — verify they still exist; phase-plan references are a
  common stale-link source once phases land. Flag for a link-check, not fixed here.
- **`docs/README.md`** — Pure generated Fumadocs scaffold (Next.js boilerplate, no
  Headroom content). Either replace with a real `docs/` index (point at SPEC.md +
  the three reference docs) or leave as the docs-site app readme. Currently it adds no
  doc-navigation value for a reader landing in `docs/`.

## Archive/stale candidates

- **`docs/spec/016-observability.md` (partial)** — The `headroom_*` Python-era metric
  table is likely superseded by the Rust `proxy_*`/`bedrock_*` surface. Not a
  whole-file archive; the section needs reconciliation (see refactor note). Treat the
  stale table as the archive candidate, not the file.
- **`docs/README.md`** — Boilerplate-stale (scaffold default text). Candidate for
  replacement rather than archive.
- No other clearly-obsolete whole files. The `spec/` set is marked `done` and reads as
  current intended-behavior; `022-rust-migration.md` is explicitly in-progress. Note the
  spec is dated 2026-04-16 and describes Headroom as a "Python package" first, while
  `022` + `observability.md` + `bedrock.md` + `rtk-architecture.md` describe a Rust-core
  reality — the spec as a whole is drifting behind the Rust migration but is not yet
  stale enough to archive; it should be version-bumped as the migration lands.

## Promotion candidates

These are **flagged only** — not to be merged in this pass. All are tightly coupled to
Headroom internals; generality is mostly low-to-medium.

- `docs/rtk-architecture.md` → **ai-coding-agents** — generality **medium**. The
  *decision* (RTK is a wrap-CLI command-rewrite hook, not a proxy-side output compressor;
  command-rewrite vs output-rewrite value props) is reusable RTK doctrine that the global
  repo already discusses (RTK redirect/pipeline bypass, RTK-vs-display-data hazards in
  CLAUDE.md). A distilled "RTK: command-rewrite vs output-rewrite, and why not in a hot
  proxy path" note could live in ai-coding-agents' RTK reference. The Headroom-specific
  cache-hot-zone / `log_compressor.rs` rationale stays here. Promote a *distillation*,
  not the file.
- `docs/auth-modes.md` → **ai-coding-agents** — generality **low/medium**. The mapping of
  provider auth shapes (`sk-ant-api*`, `sk-ant-oat-*`, 3-segment JWT, `AWS4-HMAC-SHA256`,
  subscription UA prefixes like `claude-code/`, `codex-cli/`) to PAYG vs OAuth vs
  subscription, and the "subscription-UA wins over token shape" rule, is reusable
  knowledge for any tool that classifies LLM client auth. But it is interwoven with
  Headroom compression-policy decisions. Promote only a small reference table
  ("LLM provider auth-header shapes and what they imply"), not the doc.
- `docs/bedrock.md` → **ai-coding-agents** — generality **low**. The AWS Bedrock
  credential-chain + IAM-permission + SigV4 + `AWS_ENDPOINT_URL_BEDROCK_RUNTIME` testing
  recipe is generically useful for anyone proxying Anthropic-on-Bedrock, but it is
  Headroom-operator framed. Flag the Bedrock testing/IAM snippet as a possible global
  tooling-reference seed; keep the file project-local.
- `docs/spec/*` → **none**. The living-spec sections are Headroom-product-specific
  (vision, domain model, capabilities, deployment). Not promotion candidates.

## Notes

- **Cross-repo overlap suspicion (RTK):** `docs/rtk-architecture.md` and `auth-modes.md`
  describe the same RTK + token-compression ecosystem that `ai-coding-agents/CLAUDE.md`
  governs at the agent-harness level. Headroom is the *proxy/engine* side; ai-coding-agents
  is the *agent-config* side. They are complementary, not duplicative, but a reader could
  reasonably expect a cross-link between "RTK in the agent harness" (ai-coding-agents) and
  "RTK in the Headroom wrap-CLI" (here). Worth a synthesis-step note.
- **Headroom itself is a Headroom MCP tool** in the ai-coding-agents harness
  (`headroom_compress` / `headroom_retrieve`). Any global "Headroom tooling reference" in
  ai-coding-agents should link back to this repo's `SPEC.md` as upstream truth rather than
  restating it — avoid creating a second drifting description of Headroom in the global repo.
- **Two metric namespaces is the single most actionable finding** — `headroom_*` (spec
  016, Python era) vs `proxy_*`/`bedrock_*` (observability.md, Rust era). This is real
  drift inside one repo, independent of any cross-repo concern, and should be reconciled
  before either doc is treated as canonical.
- No broken intra-`docs/` relative links observed in the files read; the only outbound
  references go to `REALIGNMENT/` and `crates/` paths outside scope (not verified here).
