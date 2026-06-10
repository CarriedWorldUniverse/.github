# Carried World Universe

**Coordination layer for running multiple AI agents as a coherent team.**

We build the substrate for a small team of specialized AI personalities ("aspects") to work together on real engineering — through shared chat, work routing, a deliberation funnel that owns context across compaction, identity rooted in CWB/Herald, and a dispatch fabric that runs agents as on-demand cloud pods. The operator stays in the loop as one peer on the bus, not as a manager wrangling threads.

→ **[Documentation site](https://carriedworlduniverse.github.io/nexus/)** — architecture overview, per-repo details, policies, full spec corpus.

---

## The stack

| Repo | Role | Lang |
|---|---|---|
| [**nexus**](https://github.com/CarriedWorldUniverse/nexus) | Broker + dashboard + aspect runtime + dispatch fabric (the heart) | Go |
| [**bridle**](https://github.com/CarriedWorldUniverse/bridle) | Per-turn deliberation library; direct-API and headless-CLI providers (Claude/OpenAI/Bedrock/Gemini APIs; Claude Code, Codex, Gemini, Antigravity, local Ollama) | Go |
| [**agora**](https://github.com/CarriedWorldUniverse/agora) | Operator's terminal TUI — a live 1:1 conversation with one agent | Go |
| [**acp-claude-pty**](https://github.com/CarriedWorldUniverse/acp-claude-pty) | PTY driver + ACP server for the Claude CLI | Go |
| [**interchange**](https://github.com/CarriedWorldUniverse/interchange) | CWB boundary gateway — public edge for the gRPC pillars + E2E relay | Go |
| [**herald**](https://github.com/CarriedWorldUniverse/herald) | CWB identity service for humans and agents; OIDC + casket-rooted assertions | Go |
| [**ledger**](https://github.com/CarriedWorldUniverse/ledger) | Aspect-first issue tracker and authority/audit pillar (standalone gRPC service) | Go |
| [**commonplace**](https://github.com/CarriedWorldUniverse/commonplace) | Knowledge pillar — agent-accessible semantic store (standalone gRPC service) | Go |
| [**cairn**](https://github.com/CarriedWorldUniverse/cairn) | Agent-native git platform — native go-git core (Forgejo lineage preserved on archived branches) | Go |
| [**custodian**](https://github.com/CarriedWorldUniverse/custodian) | External-credential vault — herald-keyed, brokered use, per-org crypto isolation | Go |
| [**cw**](https://github.com/CarriedWorldUniverse/cw) / [**cwb-client**](https://github.com/CarriedWorldUniverse/cwb-client) / [**cwb-proto**](https://github.com/CarriedWorldUniverse/cwb-proto) | CWB CLI, reusable client, and protocol definitions | Go / proto |
| [**cwb-conformance**](https://github.com/CarriedWorldUniverse/cwb-conformance) | End-to-end conformance suite exercising the pillars through their real public boundary | Go |
| [**nexus-platform**](https://github.com/CarriedWorldUniverse/nexus-platform) | Umbrella distribution repo for the deployable bundle | Go |
| [**porter**](https://github.com/CarriedWorldUniverse/porter) | casket-encrypted, searchable cloud-storage-as-a-filesystem (low-IO / offsite) | — |
| [**lynxai**](https://github.com/CarriedWorldUniverse/lynxai) | Self-hostable AI-native headless browser | Go |
| [**vessel**](https://github.com/CarriedWorldUniverse/vessel) | Avatar + voice shell for LLM backends | — |
| [**casket-go**](https://github.com/CarriedWorldUniverse/casket-go) / [**-ts**](https://github.com/CarriedWorldUniverse/casket-ts) / [**-dotnet**](https://github.com/CarriedWorldUniverse/casket-dotnet) | Ed25519 + AEAD channel identity, cross-language wire-compatible | Go / TS / C# |

Most public repos have CI matrices, tag-driven releases, branch-protected trunk with required status checks, and Apache-2.0 license. `cairn` and `lynxai` are AGPL-3.0.

## How it fits together

```
                    ┌─────────────────────────────┐
                    │      nexus (broker)          │
                    │  chat · dashboard · obs ·    │
                    │       dispatch fabric        │
                    └──────────────┬──────────────┘
                                   │  WS
        ┌──────────────────────────┼──────────────────────────┐
        ▼                          ▼                          ▼
   ┌─────────┐               ┌──────────┐              ┌──────────┐
   │ aspects │               │  agora   │              │  funnel  │
   │ (cloud  │               │ (operator│              │ (per-    │
   │  pods)  │               │   TUI)   │              │ aspect)  │
   └─────────┘               └──────────┘              └──────────┘
        │                          │                          │
        └──────────────────────────┴──────────────────────────┘
                                   │
                                   ▼
                            ┌──────────────┐
                            │   bridle     │
                            │ (1 turn, N   │
                            │ providers)   │
                            └──────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              ▼                    ▼                    ▼
       direct model APIs      headless CLIs       local + tools
       Claude/OpenAI/...      Claude/Codex/       Ollama/browser/
                              Gemini/Antigravity  shell/MCP
```

Aspects connect to nexus over WebSocket. Each one wraps `bridle` (which handles a single deliberation turn against any provider) inside a `funnel` (which owns the inbox, compaction, output filter, and observability). agora is the operator's seat at the same table — a thin client built on the same machinery.

**Dispatch fabric.** Aspects run as on-demand cloud pods on a single-node k3s cluster, not host processes. An aspect can be *addressable but napping* — its identity and inbox persist; a mention wakes its pod in seconds and it naps again when quiet. Aspects spawn their own fresh-context worker "hands" (carrying the parent's persona, under a derived identity, fully audit-threaded) and report results back into chat without blocking the conversation.

**CWB — the identity and authority plane.** Herald attests humans and agents; the pillars (Ledger for work/audit, Commonplace for knowledge, Cairn for git) run as standalone gRPC services over mTLS behind interchange as the public boundary gateway. A credential custodian brokers external secrets so agents act without raw credentials in model context. Casket provides the cross-language crypto root.

## Current design focus

- **Dispatch-native agents** — aspects as cloud pods: napping/wake-on-mention presence, aspect-owned worker fan-out, herald-rooted identities replacing host-spawned processes.
- **Multi-agent deliberation** — convening several named aspects into a thread to reach consensus, with the operator mediated (digests and batched decision-points) rather than firehosed.
- **AI-usable credentials** — a herald-keyed custodian for password, 2FA, OAuth, session, and browser-login acts without placing raw secrets in model context.
- **The CWB pillars** — Herald/Ledger/Commonplace/Cairn as gRPC-meshed, mTLS-secured services behind a public boundary gateway; conformance-tested end to end.
- **Provider breadth and control** — bridle as a complete harness over direct model APIs and self-executing CLI streams, with per-turn timing instrumentation and a local-model lane.

## Status

Honest single-operator R&D — operational, moving fast, publicly readable, not a product. The running cluster is single-operator and tailnet-gated; the codebase is open. The agent runtime (broker, funnel, dispatch) and the CWB pillars (herald, ledger, commonplace, cairn) are deployed and meshed; credential custody, multi-agent deliberation, multi-operator support, and offsite-storage are in flight. Individual repos vary from tagged-and-stable to fast-moving — check each repo's own README for where it sits.

## License

Most public repos are [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0). Check each repository's `LICENSE`; `cairn` and `lynxai` are AGPL-3.0.
