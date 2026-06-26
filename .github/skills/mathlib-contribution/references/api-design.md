# Statements & API design

The hardest part of review to satisfy is design: generality, the right interface, and how a
declaration interacts with automation. Aim to make the *statement* obviously useful and the API
complete.

## Explicit types

Give the type of **every** argument and the **return type**, even when Lean could infer them — the
statement has to be readable on GitHub and in the docs.
```lean
def GoodStatement (n : ℕ) : Prop := ∃ k : ℕ, n + k = 3   -- not: def Bad (n) := ∃ k, n + k = 3
```

## Argument placement

- If the proof starts by introducing variables, put them **left of the colon** rather than behind
  `∀`/`→`. (Pattern-matching definitions keep the binder on the right.)
- For an **`iff` lemma**, make the quantified variables **implicit** so `.mp`/`.mpr` are usable. *(#41076)*
- Put canonically-inferred **constraint** hypotheses in instance-implicit `[ ]` (`[Nonempty n]`,
  `[Fintype n]`), not as explicit/anonymous `(_ : Nonempty n)`; reserve `( )`/`{ }` for data. *(#40699)*
- Make an argument explicit when it appears in the conclusion and can't be inferred; make it implicit
  when another argument determines it. Order hypotheses structural-first, "automatic" (`g _ 0 = 1`)
  last; for a dot-notation lemma, the subject + its property come first. *(#39623, #39422, #40219)*

## Generality

- Use the **weakest hypotheses / most general typeclasses** that make the statement true (`Mul` over
  `Monoid`, `Ring` over `Ring`+`Algebra ℤ`, `IsReduced` over `NoZeroDivisors`, partial measurability
  over full). Generalize on request, and proactively. *(#40248, #39873, #40836, #39812)*
- **Generalize concrete morphisms to `FunLike`/`*HomClass`**: take `[FunLike F R S] [RingHomClass F R S]
  (f : F)` rather than `(f : R →+* S)`. *(#40449)*
- **Generalize the definition, not only the statement:** define on the general type, with the weakest
  structure, in the most general file — e.g. on `PiLp` rather than `EuclideanSpace`; prove the
  `IsEmbedding` version first and derive the subtype case. *(#40634, #40799)*
- **Use junk values to drop side conditions** where the conclusion still holds (`∀ x` rather than
  `∀ x ≥ 1`). *(#40569)*
- Hoist shared assumptions into a `variable` block; **delete unused** assumptions and variables.
  *(#40976, #40944, #40928)*
- Use the most specific base needed in an instance head, and prove the form that keeps instance search
  cheap (`T3Space` directly). Don't require an instance you can derive locally with a `have`. *(#40931, #40637)*

## A complete, idiomatic API

- Supply the full set of standard operations and their companion/dual lemmas: add `Sub` alongside
  `Add`/`Neg`; the `LE` form (and `LT → LE`) for an `LT` result; `comp_left` with `comp_right`. Don't
  bundle several results with `∧` — state them as **separate theorems**. *(#40890, #40802, #40569)*
- **`@[simps]` / `@[simps!]`** auto-generate projection lemmas — use them over hand-written ones (and
  on structure operations: `@[simps, refl]`, `@[simps, symm]`). Build on an existing definition with
  inheritance (`def foo where __ := bar`) so `@[simps!]` reuses it. *(#40890, #39757, #40455)*
- **`fast_instance% FunLike.…`** derives an algebraic instance (`Semiring`, `AddCommMonoid`, …) for a
  bundled-morphism type — use it instead of proving each field by hand. *(#40515, #39637)*
- Add `@[simp]`/rewrite lemmas so downstream proofs never unfold your definition. *(#40991, #41063)*
- **Reuse existing API** instead of reconstructing it (`rTensor`, `IsLocalHomeomorphOn`,
  `Algebra.adjoin_le`, `rangeRestrict`, `toZMod`, an `_iff` via `rw`); check for a defeq type that
  already has the instance (`⨁ i, M i` is `Π₀ i, M i`). *(#41034, #41033, #40844, #39764)*
- **Don't add a lemma `simp`/`Iff.rfl`/`grind` already proves**, a definitional-equation lemma, or a
  duplicate of existing API; don't delete a *useful* characterization lemma. Inline a one-line proof
  rather than naming it. *(#40889, #40647, #40456, #39589, #41067)*
- Make a canonical object a **`def`** (with explicit parameters) rather than an existence theorem —
  `Classical.choice`-free constructions deserve real API. Use `class` only for typeclasses; otherwise
  `structure`. *(#40567, #39810)*

## Attributes

Tag where appropriate — and only where appropriate ("first, do no harm"):
`@[simp]`, `@[ext]` (numeric priority `@[ext 1100]`, not `@[ext high]`), `@[gcongr]`, `@[fun_prop]`,
`@[to_additive]`, `@[to_dual]`, `@[simps]`, and literature tags `@[stacks …]`/`@[kerodon …]`. In
particular add `@[simp]` to `*_iff` characterizations and basic `apply`/coercion lemmas (reviewers ask
across *all* analogous lemmas at once), and `@[fun_prop]` to continuity/differentiability/measurability
lemmas so `fun_prop` can use them. Don't add `@[simp]`/`@[simps]` speculatively, and don't let a
generated `simp` lemma replace a better hand-written one. *(#40715, #40739, #40451, #40080, #40931)*
Use `@[to_additive]`/`@[to_dual]` to generate the additive/dual statement rather than writing it twice.

## Transparency / definitional design

- `def` is `semireducible` by default; `abbrev` is `reducible`. Use `def` (not `abbrev`) for API maps
  that should not auto-unfold; give an `abbrev` an explicit type. Seal an API with a `structure`
  wrapper, not `irreducible`. *(#40666, #40923)*
- Don't add an instance/field whose body is just `inferInstance`. For a type with several natural
  instances (e.g. `Matrix` with different norms), **scope** the instance rather than making it global,
  and document any diamond in detail. *(#41000, #40272, #39531)*
- Avoid dependent types where a non-dependent encoding works; bundle new morphisms with `FunLike` and
  new subobjects with `SetLike`.

## Deprecation (renaming/removing public declarations)

Keep the old name as a deprecated `alias` (or deprecate with a message), **with the merge date**:
```lean
@[deprecated (since := "2026-06-25")] alias old_name := new_name
```
For `@[to_additive]` pairs deprecate both names. Deprecations may be deleted after 6 months;
brand-new declarations need none. Renaming a *type* uses `@[deprecated New (since := "…")] abbrev Old := New`.
*(#41033, #39707)*
