# Catalog of real "wrong → right" review corrections

Each entry is a change a reviewer actually requested on a recently merged PR (#39365–#41076), or a
canonical example from Mathlib's own [PR-review guide](https://leanprover-community.github.io/contribute/pr-review.html).
Use it as a lint list during self-review: scan for anything resembling a "wrong" pattern in your diff.

## Naming
- camelCase theorem name → `snake_case`. *(#41067, #41063)*
- predicate is a **suffix**: `surjective_principal` → `principal_surjective`; `injective_f` → `f_injective` (+ `f_inj`). *(#39848, #40931)*
- condition after `of`: `coeff_ne_zero_C` → `coeff_C_of_ne_zero`; drop a condition implied by a typeclass arg. *(#39623, #41006)*
- subject/operation first: `range_liftBaseChange_map_le` → `liftBaseChange_range_le`; `tfae_` prefix for TFAE; `top_` for `⊤`. *(#39958, #39910, #39844)*
- name must match the statement (`pullback`/`pushout`, `hom`/`inv`, `rank`/`finrank`); fix typos. *(#41024, #40239, #40906)*
- place in the subject's namespace (use `_root_.Ns.foo`); carrier-set lemmas stay outside it. *(#39727, #40491)*
- `coe_foo` for `⇑foo = …`; `toLinearMap_foo` (+ `@[norm_cast]`) for a bundled morphism. *(#40697, #39637)*
- avoid a meaningless trailing `'`; `protected` short/common names. *(#40210, #39891)*

## Statements / API
- iff-lemma variables implicit; constraint hypotheses in `[ ]`, not `(_ : …)`. *(#41076, #40699)*
- weakest hypotheses (`Mul` over `Monoid`; `IsReduced` over `NoZeroDivisors`; partial measurability). *(#40248, #40836, #39812)*
- generalize concrete homs to `[FunLike F R S] [RingHomClass F R S]`. *(#40449)*
- generalize the definition (`PiLp` over `EuclideanSpace`; `IsEmbedding` first); use junk values to drop side conditions. *(#40634, #40569)*
- `def` (explicit params) for a canonical object, not an existence theorem; `class` only for typeclasses else `structure`. *(#40567, #39810)*
- `@[simps]`/`@[simps!]` (and on `@[simps, refl]` etc., with `where __ := base`) over hand-written projection lemmas. *(#40890, #39757)*
- `fast_instance% FunLike.…` for derived algebraic instances. *(#40515, #39637)*
- add `@[simp]` to `*_iff`/`apply` lemmas, `@[fun_prop]` to continuity/measurability lemmas; not speculatively. *(#40715, #40080, #40931)*
- reuse API; check for a defeq type that already has the instance; don't add what `simp`/`Iff.rfl`/`grind` proves; inline one-line proofs. *(#39764, #40456, #39589)*
- add companion/dual lemmas; state results separately, not bundled with `∧`; delete unused assumptions. *(#40802, #40569, #40928)*
- scope an instance for a type with several natural ones; explain diamonds. *(#40272, #39531)*

## Proofs / formatting
- **reach for automation**: `grind`/`simp`/`gcongr`/`positivity`/`fun_prop` with explicit lemma lists, not manual ladders. *(#40293, #39616, #39744)*
- `by classical` instead of a `[DecidableEq]` API argument. *(#40210)*
- `rw …; exact` → `rwa`; non-terminal `simp; exact` → `simpa using …` (flexible linter). *(#40944, #40883)*
- `exact` not `refine` without `?_`; `by_contra!` not `by_contra; push_neg`; avoid `change` when `rw`/defeq works. *(#40714, #39912, #40239)*
- `haveI` (not `letI`) for an instance; `rintro x (rfl | rfl)`; `simp_rw` under binders; `Iff.rfl` for a defeq iff. *(#39669, #39435, #40583, #40647)*
- let `simp` intro (`map_add' x y := by simp`); `Set.mem_Icc.mpr ⟨…⟩` not bare `⟨…⟩`; drop redundant `↑`. *(#40634, #40655, #40795)*
- `_` for unused names; short namespace alias; remove dead `open`; no space after unary minus. *(#40944, #41034, #40890)*
- remove `erw`/stray `rfl` (add API); don't reformat unrelated code; golf only where it *improves* readability. *(#41014, #39531, #4702)*

## Docs / meta
- docstring matches the statement and describes **purpose** not implementation; continuation lines unindented; module docstring needs a summary. *(#41008, #39740, #40248, #40714)*
- precise terminology/grammar/Unicode; cross-references; literature citation/tags. *(#40942, #39958)*
- minimize **public** imports (`#min_imports`); general lemmas in general files; preserve copyright when moving code. *(#39543, #39912, #40658)*
- dated `@[deprecated]` alias when renaming existing decls; none for brand-new. *(#41033, #39707)*
- honest PR claims; PRs are closed for undisclosed/indefensible LLM code; questions below `---`. *(#40921, #40567)*

## Canonical examples from the PR-review guide

- **Formatting:** `theorem mul_assoc_assoc{α:Type*}…:=by rw[…]` → spaces around operators, `:` ending
  the line, `:= by` at the end, `rw` on one line.
- **Naming:** `inv_is_unit_times_self_eq_1` → `IsUnit.inv_val_mul`.
- **Location:** an instance unrelated to norms in a norm file → move via `#find_home`.
- **Import creep:** a stray `import` to reach one helper lemma → the result likely belongs elsewhere.
- **Docstring value:** a long `TFAE` theorem got a multi-paragraph docstring + a cross-reference.
- **Design:** prefer non-dependent encodings (`Mathlib.Vector` as a `List` subtype); `FunLike` for new
  morphisms, `SetLike` for new subobjects.
