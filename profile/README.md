# Carried World Universe

**Coordination layer for running multiple AI agents as a coherent team.**

We build the substrate for a small team of specialized AI personalities ("aspects") to work together on real engineering — through shared chat, lane-tagged work routing, a deliberation funnel that owns context across compaction, and a hands-dispatch model for transient workers. The operator stays in the loop as one peer on the bus, not as a manager wrangling threads.

→ **[Documentation site](https://carriedworlduniverse.github.io/nexus/)** — architecture overview, per-repo details, policies, full spec corpus.

---

## The stack

| Repo | Role | Lang |
|---|---|---|
| [**nexus**](https://github.com/CarriedWorldUniverse/nexus) | Broker + dashboard + 8 binaries (the heart) | Go |
| [**bridle**](https://github.com/CarriedWorldUniverse/bridle) | Per-turn deliberation library, provider-agnostic | Go |
| [**agora**](https://github.com/CarriedWorldUniverse/agora) | Operator's terminal TUI on the bus | Go |
| [**acp-claude-pty**](https://github.com/CarriedWorldUniverse/acp-claude-pty) | PTY driver + ACP server for the Claude CLI | Go |
| [**interchange**](https://github.com/CarriedWorldUniverse/interchange) | E2E-encrypted Frame-to-Frame relay | Go |
| [**casket-go**](https://github.com/CarriedWorldUniverse/casket-go) / [**-ts**](https://github.com/CarriedWorldUniverse/casket-ts) / [**-dotnet**](https://github.com/CarriedWorldUniverse/casket-dotnet) | Ed25519 + AEAD channel identity, cross-language wire-compatible | Go / TS / C# |

Every public repo has CI matrices, tag-driven releases via goreleaser, branch-protected `main` with required status checks, and Apache-2.0 license.

## How it fits together

```
                    ┌─────────────────────────────┐
                    │      nexus (broker)          │
                    │   chat · dashboard · obs     │
                    └──────────────┬──────────────┘
                                   │  WS
        ┌──────────────────────────┼──────────────────────────┐
        ▼                          ▼                          ▼
   ┌─────────┐               ┌──────────┐              ┌──────────┐
   │ aspects │               │  agora   │              │  funnel  │
   │ (n × …) │               │ (operator│              │ (per-    │
   │         │               │   TUI)   │              │ aspect)  │
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
```

Aspects connect to nexus over WebSocket. Each one wraps `bridle` (which handles a single deliberation turn against any provider) inside a `funnel` (which owns the inbox, compaction, output filter, and observability). agora is the operator's seat at the same table — an aspect built on the same machinery.

## Status

Operational in single-operator mode. Every component has a v0.1.0 baseline. The codebase is publicly readable; the running cluster is single-operator + tailnet-gated. AI aspect GitHub identities (so peer aspects can review each other's PRs), multi-operator support, and several feature gaps are in flight.

## License

All public repos are [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0).
