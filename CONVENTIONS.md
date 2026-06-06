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
- A change that needs a dependency updated (a new proto RPC, a shared-lib API) lands in the **upstream repo first** (proto/lib), which is tagged/bumped; then the consumer bumps its dependency.
- **Never vendor a dependency + `replace` it** in the consumer to sneak in an unreleased change — that forks the source of truth. (Lesson from herald NEX-475: a builder scoped to one repo vendored cwb-proto; the fix was to land the RPCs upstream first.)
- A dispatched builder is scoped to one repo, so cross-repo work must either land the dependency first or be coordinated across repos by the operator/shadow.
