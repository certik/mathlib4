# PR process: scope, AI policy, conventions, and lifecycle

Authoritative sources: `contribute/index.html`, `how-to-contribute.html`, `commit.html`.

## Is it in scope?

Mathlib is **not** "all of mathematics." Ask: is the material typically taught/studied in a math
department, and does it fall within a [maintainer's interests](https://github.com/leanprover-community/mathlib4#maintainers)
(they maintain it forever)? If not, consider a **standalone repo that depends on Mathlib**. As of
mid-2026 Mathlib has 2600+ open PRs, so fit and small size matter; if unsure, ask in the
[`#mathlib4` Zulip](https://leanprover.zulipchat.com/#narrow/channel/287929-mathlib4/) first.

Style-only PRs are welcome **only** when they fix a documented style-guide violation; don't reformat
files you don't own. **Split unrelated changes** (e.g. a generalization tangential to the main goal)
into separate PRs — it makes review tractable.

## Use of AI (enforced)

- **Disclose AI use in the PR description**: which tool, and how. Add the `LLM-generated` label when a
  substantial amount of code is AI-generated.
- **Do not** write GitHub/Zulip comments with an LLM — use your own words.
- You must **understand and justify every line and design decision without an AI.**
- Low-quality LLM PRs are *summarily closed* and repeat offenders can be banned: in recent samples
  several PRs were closed for AI-policy reasons. *(#40802, #40800, #40799, #40709, #40567)*

## Search-first tooling

- `exact?` / `apply?` / `rw?` in a scratch file with `import Mathlib` to find an existing closer.
- `#find_home myDecl` for the right file. [loogle](https://loogle.lean-lang.org) and the
  [mathlib4 docs](https://leanprover-community.github.io/mathlib4_docs/) for discovery by shape/name.
- `#min_imports` to trim imports before opening the PR.

## Git workflow

Work on a branch in **your own fork**; open the PR against `master` of
`leanprover-community/mathlib4` (not your fork's `master`). If you add files, run `lake exe mk_all`.
Let CI build it. (Note: this user prefers to push branches and trigger CI themselves — don't push on
their behalf.) For a **rename**, do the pure move (or keep content changes under ~50%) in its own
commit so git tracks history; refactor in a separate commit. *(#40868)*

## Title and description conventions

Format: `<type>(<optional-scope>): <subject>`

- `<type>` ∈ `feat`, `fix`, `doc`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`.
- `<scope>` = module/dir, **omitting the `Mathlib/` prefix** (e.g. `Data/Nat/Basic`). Optional.
- `<subject>`: imperative present tense ("add", not "added"); **no leading capital**; **no trailing period**.
- `<body>`: imperative present tense; **motivation and contrast with previous behavior**.
- Questions/discussion you don't want in git history go **below a `---` line**.
- Moving/deleting declarations? List them before the `---` (`Moves:` / `Deletions:`).
- Dependencies: `- [ ] depends on: #1234`.

## Lifecycle and labels

- A green PR is reviewed within days–weeks (smaller = faster). Reviewers add `awaiting-author`.
- **Address each comment, then "Resolve conversation."** Fix via a new commit; when done, **remove
  `awaiting-author`** to re-enter the queue.
- A satisfied reviewer adds `maintainer-merge`; a maintainer runs `bors merge`/`bors r+`. A
  `delegated` PR means *you* run `bors merge` after final tweaks.
- Self-serve labels (comment the word): `WIP`, `RFC`, `awaiting-author`, `awaiting-zulip`,
  `awaiting-CI`, `easy`, `help-wanted`, `LLM-generated`, topic `t-*`.
- **`easy`** is for trivial PRs only — *not* easy if the diff > 25 lines, it changes existing
  statements/defs, adds files, or adds non-analogous `simp` lemmas/instances.
- Pushing a commit takes a PR off the merge queue; only do so if it would otherwise fail.

## How Mathlib merges (useful when researching PRs)

Mathlib merges with **Bors**, which *closes* the PR (prefixing the title `[Merged by Bors] -`)
**without** setting GitHub's `merged` flag. So `gh pr list --state merged` returns almost nothing; to
find real merged PRs, list **closed** PRs based on `master` and look for the `[Merged by Bors] -` prefix.

## Performance

- Use `Type*`, never `Type _` (extra unification work). Avoid import creep.
- Run `!bench` (comment on the PR) when you add/remove `simp` lemmas, instances, imports, classes, or
  defs, or do a nontrivial `refactor`; explain any significant regression.
