# Carried World — repository conventions

## Branches
- **`main` is the default branch** for every repo. (No per-repo special defaults — cairn was migrated `cairn`→`main` 2026-06-06; the Forgejo lineage lives on a `forgejo` branch.)
- Work happens on short-lived branches off `main`:
  - `feature/<ticket>` — human-authored work (e.g. `feature/NEX-123-slug`).
  - `builder/<ticket>` — dispatched cloud-builder work.
  - `fix/`, `chore/`, `docs/` prefixes as appropriate.
- Merge via **squash**, then delete the branch.

## CI + branch protection
- Every code repo carries a `ci` workflow — `build + test + vet` on ubuntu/macos/windows (Go 1.26 for Go repos) — and a **protected `main`**: required CI checks, strict (branch up-to-date before merge), linear history, enforced for admins.
- Do not push to `main` directly. Open a PR; let CI gate it.

## Cross-repo changes
- A change that needs a dependency updated (a new proto RPC, a shared-lib API) lands in the **upstream repo first** (tagged/bumped), then the consumer bumps to that version.
- **Never vendor a dependency + `replace` it** in the consumer to sneak in an unreleased change — that forks the source of truth. (Lesson from herald NEX-475.)
- A builder's git/gh creds are **org-wide, not repo-locked**, so this whole sequence can be **one dispatch**: clone the upstream repo, land + push the change, then clone the consumer and bump it — in the right order. The brief's `repo` field is just the primary / idempotency key; tell the builder to clone whatever else it needs. (Caveat: the fabric's per-ticket branch + PR-verification bookkeeping is single-repo-shaped today, so multi-repo *completion detection* is still rough — a refinement to make cross-repo dispatch first-class, not a reason to avoid it.)
