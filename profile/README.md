# Carried World Universe

**Coordination layer for running multiple AI agents as a coherent team.**

We build the substrate for a small team of specialized AI personalities ("aspects") to work together on real engineering — through shared chat, lane-tagged work routing, a deliberation funnel that owns context across compaction, identity rooted in CWB/Herald, and a hands-dispatch model for transient workers. The operator stays in the loop as one peer on the bus, not as a manager wrangling threads.

→ **[Documentation site](https://carriedworlduniverse.github.io/nexus/)** — architecture overview, per-repo details, policies, full spec corpus.

---

## The stack

| Repo | Role | Lang |
|---|---|---|
| [**nexus**](https://github.com/CarriedWorldUniverse/nexus) | Broker + dashboard + aspect runtime (the heart) | Go |
| [**bridle**](https://github.com/CarriedWorldUniverse/bridle) | Per-turn deliberation library with direct-API and headless-CLI providers, including Codex CLI | Go |
| [**agora**](https://github.com/CarriedWorldUniverse/agora) | Operator's terminal TUI on the bus | Go |
| [**acp-claude-pty**](https://github.com/CarriedWorldUniverse/acp-claude-pty) | PTY driver + ACP server for the Claude CLI | Go |
| [**interchange**](https://github.com/CarriedWorldUniverse/interchange) | E2E-encrypted Frame-to-Frame relay | Go |
| [**herald**](https://github.com/CarriedWorldUniverse/herald) | CWB identity service for humans and agents; OIDC + casket-rooted assertions | Go |
| [**ledger**](https://github.com/CarriedWorldUniverse/ledger) | Aspect-first issue tracker and authority/audit surface | Go |
| [**cairn**](https://github.com/CarriedWorldUniverse/cairn) | Agent-native git platform — long-term divergent fork of Forgejo | Go |
| [**cw**](https://github.com/CarriedWorldUniverse/cw) / [**cwb-client**](https://github.com/CarriedWorldUniverse/cwb-client) / [**cwb-proto**](https://github.com/CarriedWorldUniverse/cwb-proto) | CWB CLI, reusable client, and protocol definitions | Go / proto |
| [**nexus-platform**](https://github.com/CarriedWorldUniverse/nexus-platform) | Umbrella distribution repo for the deployable bundle | Go |
| [**lynxai**](https://github.com/CarriedWorldUniverse/lynxai) | Self-hostable AI-native headless browser | Go |
| [**vessel**](https://github.com/CarriedWorldUniverse/vessel) | Desktop avatar/voice shell for LLM backends | — |
| [**casket-go**](https://github.com/CarriedWorldUniverse/casket-go) / [**-ts**](https://github.com/CarriedWorldUniverse/casket-ts) / [**-dotnet**](https://github.com/CarriedWorldUniverse/casket-dotnet) | Ed25519 + AEAD channel identity, cross-language wire-compatible | Go / TS / C# |

Most public repos have CI matrices, tag-driven releases, branch-protected trunk with required status checks, and Apache-2.0 license. Forgejo-derived or browser-runtime repos can differ: `cairn` and `lynxai` are AGPL-3.0.

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
                                   │
              ┌────────────────────┼────────────────────┐
              ▼                    ▼                    ▼
       direct model APIs      headless CLIs        local tools
       Claude/OpenAI/...      Claude/Gemini/       browser/shell/
                              Codex                MCP
```

Aspects connect to nexus over WebSocket. Each one wraps `bridle` (which handles a single deliberation turn against any provider) inside a `funnel` (which owns the inbox, compaction, output filter, and observability). agora is the operator's seat at the same table — an aspect built on the same machinery.

CWB adds the identity and authority plane: Herald attests humans and agents, Ledger records work and authorization decisions, and Nexus mediates per-aspect tokens through a custodian instead of collapsing every action into one "nexus" identity.

## Current design focus

- **Agent identity and authorization** — CWB/Herald assertions, per-aspect token custody, and authority-preserving calls into CWB pillars.
- **AI-usable credentials** — a Ledger-authorized credential custodian for password, 2FA, OAuth, session, and browser-login acts without placing raw secrets in model context.
- **Provider breadth** — bridle supports direct APIs and self-executing CLI streams, including Claude Code, Gemini CLI, and Codex CLI headless mode.
- **Native work tracking** — Ledger as the markdown-native, append-only issue and audit plane for aspect work.
- **Agent-native git** — Cairn as the long-term git/code-review host for AI and human collaboration.

## Status

Operational in single-operator mode. The core repos have tagged baselines and CI; the newer CWB repos are moving quickly. The codebase is publicly readable; the running cluster is single-operator + tailnet-gated. AI aspect GitHub identities, multi-operator support, credential custody, and the broader CWB authority plane are in flight.

## License

Most public repos are [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0). Check each repository's `LICENSE`; `cairn` and `lynxai` are AGPL-3.0.
