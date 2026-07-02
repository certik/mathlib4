---
name: mathlib-contribution
description: >
  Guide for writing Mathlib4 (Lean 4) contributions that pass upstream review on the
  first pass. Use this whenever the user is adding or editing Lean code in a Mathlib
  repository — new theorems, lemmas, definitions, instances, structures, or files — or
  preparing/structuring a Mathlib pull request, naming a lemma, writing docstrings,
  choosing attributes (@[simp], @[ext], @[to_additive], @[simps]), generalizing
  hypotheses, or asking how to make Lean code conform to Mathlib conventions and avoid
  reviewer nitpicks. Trigger it even when the user does not say "Mathlib" explicitly but
  is clearly working in a Lean mathematics library (files under `Mathlib/`, a mathlib4
  clone) or mentions Lean theorem/proof style, naming conventions, golfing, or getting a
  Lean PR review-ready. Prefer this over generic Lean help for anything destined for
  Mathlib.
---

# Writing Mathlib contributions that pass review

Help the user produce Lean 4 code that a Mathlib reviewer can approve on the first pass.

Mathlib has unusually high and unusually *specific* standards: generality, naming, formatting,
documentation, and API design are all scrutinized. New contributors lose most of their time to
**avoidable** reviewer nitpicks and to re-proving things that already exist. The point of this
skill is to bake those standards in *while writing*, so the diff the reviewer sees is already
idiomatic and the round-trips are short.

This skill carries two things: the distilled rules (here and in `references/`), and a catalog of
real *wrong → right* corrections mined from recent merged PRs (`references/review-catalog.md`).
Apply the rules as you write; consult the catalog and references for depth and justification.

> Apply this skill's conventions to the Lean code you write, but do **not** silently restructure a
> user's unrelated existing code. Suggest improvements; let the user decide.

## Workflow

Move through four phases. They are not rigid gates, but **Phase 1 is the single highest-leverage
step** — skipping it is the most common cause of wasted work and closed PRs.

### Phase 1 — Before writing any code

The most common reviewer response to a newcomer is *"this already exists as `X`"* or *"this should
be more general."* Pre-empt both:

- **Search for prior art first.** Put the target statement in a scratch file with `import Mathlib`
  and try `exact?` / `apply?` / `rw?`; search [loogle](https://loogle.lean-lang.org) and the docs.
  Mathlib aspires to generality and avoids duplication, so the thing you want often already exists
  under a non-obvious name (a vector space is `Module`, a group hom is `MonoidHom`).
- **Aim for the weakest hypotheses** that still make the statement true (most general typeclasses,
  fewest assumptions). This is the number-one design request in review.
- **Find the right home.** Use `#find_home` to locate the correct file; put declarations where
  they belong, not where it is convenient. Watch for import creep (don't pull `Analysis` into an
  `Algebra` file).
- **Prove the right primitive.** Prefer the general equality/characterization over a derived
  special-case inequality — the weaker results then follow trivially.
- **Confirm fit and disclose AI.** Mathlib is *not* "all of mathematics"; if scope is unclear, ask
  on Zulip first. If AI was used, the PR description must say which tool and how, and add the
  `LLM-generated` label for substantial AI code. See `references/pr-process.md` — this is enforced.
- **Keep it small.** Many small, self-contained PRs beat one large PR.

### Phase 2 — Write it idiomatically

Apply the rules below inline as you write. The full rules and rationale live in `references/`.
The condensed, highest-frequency rules are in [The rules that matter most](#the-rules-that-matter-most).

### Phase 3 — Self-review against the catalog

Before calling it done, read `references/review-catalog.md` and check your diff against it — these
are the exact things reviewers ask people to change. Grep your diff for the cheap mechanical
violations: `erw`, `λ`, `$`, `Type _`, empty lines inside proofs, `:= by` not ending the statement
line, theorem names in camelCase, trailing periods in the PR subject.

### Phase 4 — Package the pull request

- Title: `type(scope): subject` (imperative, lowercase start, no trailing period; scope omits the
  `Mathlib/` prefix). Description gives motivation; questions/notes go below a `---` line.
- Ensure CI is green (build + linters). Run `lake exe mk_all` if you added files.
- Run `!bench` (comment on the PR) if you added/changed `simp` lemmas, instances, imports, or defs.
- Add `@[deprecated (since := "YYYY-MM-DD")] alias` for any renamed/removed *existing* public name.

Details: `references/pr-process.md`.

## The rules that matter most

These are the conventions most frequently flagged in real reviews. Each is one line with the
*why*, because understanding the reason lets you apply it correctly in new situations.

**Naming** (full rules + symbol dictionary in `references/naming.md`)
- Theorem/proof names are `snake_case`; types/structures/classes are `UpperCamelCase`; other terms
  are `lowerCamelCase`. A function is named like its return value. (Wrong: `hasFiniteProductsOfX`.)
- Translate symbols with the standard dictionary (`+`→`add`, `*`→`mul`, `⁻¹`→`inv`, `∣`→`dvd`,
  `∘`→`comp`, `≤`→`le`, …); use `one`/`mul` not `1`/`times`. Hypotheses after `of`, in order.
- **The name must describe the actual statement.** Mismatches (`pullback` vs `pushout`, `hom` vs
  `inv`, `mono` vs `epi`) and typos get flagged every time.
- Injectivity is `f_injective` (word at the end) plus an iff-form `f_inj`; extensionality is `.ext`
  with `@[ext]`. Don't put `nonempty` in a name when `[Nonempty _]` is already a typeclass argument.
- Names use American spelling (`factorization`, not `factorisation`).
- Coercion lemmas (for `⇑foo = …`) are named `coe_foo`; a property-of-an-object lemma reads
  `property_object` (`isInvertedBy_isomorphisms`); surface a disambiguating hypothesis with `_of_…`.
- A predicate is a **suffix** (`principal_surjective`); place a lemma in its **subject's namespace**
  (`_root_.Ns.foo` if defined elsewhere); `protected` short/common names.

**Statements & API** (`references/api-design.md`)
- Give an **explicit type for every argument and the return type**, even when Lean could infer it —
  it makes the statement readable on GitHub/docs.
- Put hypotheses **left of the colon** when the proof starts by introducing them. Make the bound
  variables of an **`iff` lemma implicit** so `.mp`/`.mpr` work directly.
- Hoist assumptions shared by several declarations into a `variable` block; delete typeclass
  assumptions and variables that aren't actually used.
- Reuse existing API instead of rebuilding it; add `@[simp]`/rewrite lemmas so callers never need
  to unfold your definition; use `@[simps]`/`@[simps!]` instead of hand-writing projection lemmas.
- Add attributes where they belong (`@[simp]`, `@[ext]`, `@[gcongr]`, `@[fun_prop]`, `@[to_additive]`,
  `@[to_dual]`) and **not** speculatively ("first, do no harm"). Use `@[to_additive]`/`@[to_dual]`
  to generate the additive/dual statement rather than writing it twice.
- Keep definitions `semireducible` (the `def` default); seal an API with a `structure` wrapper, not
  `irreducible`. Give `abbrev`s an explicit type.
- Put canonically-inferred **constraint** hypotheses in instance-implicit `[ ]` (`[Nonempty n]`, not
  `(_ : Nonempty n)`); reserve `( )`/`{ }` for data. Use `def`, not `abbrev`, for API maps.
- Add `@[simp]` to `*_iff` characterizations and basic `apply`/coercion lemmas; **don't** add a lemma
  that `simp` or `Iff.rfl` already proves.
- Generalize the **definition**, not just the statement: most general type, weakest structure, most
  general file (e.g. define on `PiLp` over `EuclideanSpace`; prove `IsEmbedding` first, then subtype).
- Generalize concrete homs to `FunLike`/`*HomClass`; use junk values to drop side conditions; don't
  bundle results with `∧`. `fast_instance% FunLike.…` for derived algebraic instances; a canonical
  object is a `def` (not an existence theorem); `class` only for typeclasses, else `structure`.

**Proof & formatting** (`references/style.md`)
- Lines ≤ 100 chars. Declarations are flush-left; `namespace`/`section` do not indent contents.
- `by` ends the preceding line (`:= by`), never sits on its own line. Proof body indents 2 spaces;
  a multi-line *statement* indents continuation lines 4 spaces.
- **No empty lines inside a declaration** (linter-enforced) — add a one-line comment instead.
- Use `fun x ↦ …` (not `λ`); use `<|`/`|>` (not `$`). No space after unary minus (`-a`).
- **No `erw`, no stray `rfl` after `simp`/`rw`** — that signals missing API; add the lemma instead.
- Don't squeeze a *terminal* `simp` (don't replace it with `simp?` output) — it buries the key
  lemmas and breaks on renames.
- Prefer the tight idiom: `rwa` over `rw …; exact`; `haveI` for instances; `rfl`/`inferInstance`/
  `Iso.refl _` for trivial goals; `_` for unused pattern variables; `ext` (not `ext : 1`).
- A **non-terminal `simp`** (followed by `exact`/`infer_instance`/`rw`) trips the `flexible` linter —
  use `simpa using …` or an explicit `rw`. Use `exact` (not `refine`) when there are no `?_`; `Iff.rfl`
  for a definitional iff; `simp_rw` to rewrite under binders.
- **Reach for automation** (`grind`, `simp`, `gcongr`, `positivity`, `fun_prop`, `omega`) with explicit
  lemma lists over manual ladders; `by classical` instead of a `[DecidableEq]` argument; `by_contra!`
  over `by_contra; push_neg`. Don't reformat code you aren't changing.
- **Keep proofs shallow**: a big `have` scaffold closed by a one-line `exact` is a machine-written
  tell — *fold* short `have`s inline, and *invert* a "big `have` + `exact`" into "short fact +
  `convert` + former `have` body as the main goal", so the proof reads top-down with minimal nesting.

**Documentation** (`references/documentation.md`)
- Current file header is the module form: copyright (current year), then `module`, then grouped
  `public import`s (alphabetical), then plain `import`s, then a `/-! … -/` module docstring.
- Every `def` and major theorem needs a docstring, and **the docstring must match the statement**
  (variable names, which hypothesis is on which object). Update docs/comments when you generalize.
- Use precise terminology and correct grammar/Unicode (`étale`, `an` before a vowel, right plural);
  cross-reference related declarations; cite the literature in `docs/references.bib`.
- Docstring continuation lines are **not** indented. When *moving* code, keep the original copyright
  year and authors (don't claim sole authorship of relocated code).
- Docstrings describe the mathematical **purpose**, not the implementation; a module docstring needs a
  summary, not just a title. Minimize **public** imports (`#min_imports`); keep general lemmas in general files.

**Performance**
- Use `Type*`, never `Type _` (the latter creates extra unification work). Avoid import creep.

## Pre-submission checklist

- [ ] Searched for existing/more-general results; statement uses the weakest hypotheses.
- [ ] In the right file (`#find_home`); no surprising new imports; PR is small and self-contained.
- [ ] Names follow the conventions and describe the statement; American spelling.
- [ ] Every argument and the return type has an explicit type; iff-lemma vars implicit.
- [ ] Attributes added where appropriate, not speculatively; `@[to_additive]`/`@[to_dual]` used.
- [ ] Every def/major theorem has an accurate docstring; module docstring + header present.
- [ ] Style: ≤100 cols, `:= by` placement, no `erw`, no `λ`/`$`/`Type _`, no empty proof lines.
- [ ] Renamed/removed public names carry dated `@[deprecated]` aliases.
- [ ] `lake exe mk_all` run if files added; CI green; `!bench` run if simp/instances/imports/defs.
- [ ] Constraint hypotheses in `[ ]`; `@[simp]` on iff/apply lemmas; nothing `simp`/`Iff.rfl` already proves.
- [ ] No non-terminal `simp` (use `simpa`/explicit); docstring continuation lines unindented.
- [ ] Reached for automation (`grind`/`simp`/`fun_prop`/`positivity`) over manual proofs where possible.
- [ ] Proofs read top-down with minimal nesting: short `have`s folded in; no big `have` scaffold + one-line `exact`.
- [ ] PR title is `type(scope): subject`; description has motivation; AI use disclosed.

## References

Read the relevant file when you need depth or the user pushes back on a convention:

- `references/naming.md` — capitalization rules, the symbol dictionary, structural-lemma naming
  (`.ext`, `_injective`/`_inj`, `ge`/`gt`), coercions, with real PR examples.
- `references/style.md` — layout, `calc`, focusing dots, the full *tactic idiom* substitution table
  (e.g. `rw …; exact` → `rwa`), `erw`/transparency, simp-squeezing, proof shape (fold/invert `have`s).
- `references/api-design.md` — explicit types, generality, `variable` blocks, instances, attributes,
  `@[simps]`, transparency, and the deprecation recipe.
- `references/documentation.md` — file header / module system, module docstrings, doc requirements,
  file location & splitting, citations.
- `references/pr-process.md` — scope, the AI-disclosure policy, search-first tooling, commit/PR
  title & description conventions, labels, and the Bors merge lifecycle.
- `references/review-catalog.md` — the consolidated *wrong → right* catalog, each correction cited
  to its real PR (#39365–#41076) plus the canonical examples from Mathlib's own
  PR-review guide. Read this during Phase 3 self-review, or whenever you're unsure why a pattern is
  discouraged.

When in doubt, **match the surrounding code** and ask on the [`#mathlib4` Zulip](https://leanprover.zulipchat.com/#narrow/channel/287929-mathlib4/).
