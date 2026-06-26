# Naming conventions

Authoritative source: <https://leanprover-community.github.io/contribute/naming.html>.
Names are how users find lemmas by guessing or tab-completion, so consistency is load-bearing.

## Capitalization scheme

| Kind of thing | Case | Example |
|---|---|---|
| Proofs / theorem names (terms of `Prop`) | `snake_case` | `mul_comm`, `lt_of_le_of_ne` |
| `Prop`s, `Type`s, structures, classes, inductives | `UpperCamelCase` | `Module`, `IsUnit`, `OneHom` |
| Functions / defs / other terms | `lowerCamelCase` | `padicValRat`, `objPreimage` |

Rules:
- A **function is named like its return value** (a `A → B → C` is named as a term of `C`).
- An `UpperCamelCase` name used *inside* a `snake_case` name becomes `lowerCamelCase`
  (`Nat.cast` → `…_natCast`; `IsUnit` hypothesis → `isUnit` in a lemma name; namespace `IsUnit.`).
- Acronyms are group-cased: `LE`, `RCLike`, `iSup`.
- **American spelling in names**: `factorization`, `Localization`, `FiberBundle` (not `…isation`,
  `Fibre…`). (Docstrings may use any English spelling; names may not.)

## Symbol → word dictionary

`+`→`add`, `-`→`neg`(unary)/`sub`(binary), `*`→`mul`, `/`→`div`, `⁻¹`→`inv`, `^`→`pow`,
`•`→`smul`, `∣`→`dvd`, `∑`→`sum`, `∏`→`prod`, `0`→`zero`, `1`→`one`.
`∘`→`comp`, `=`→`eq`(often omitted), `≠`→`ne`, `→`→`of`/`imp`, `↔`→`iff`, `¬`→`not`,
`∀`→`all`/`forall`, `∃`→`exists`. (`ball`/`bex` are deprecated in Mathlib.)
`∈`→`mem`, `∉`→`notMem`, `∪`→`union`, `∩`→`inter`, `⋃`→`iUnion`/`biUnion`, `⋂`→`iInter`,
`\`→`sdiff`, `ᶜ`→`compl`, `{x | p x}`→`setOf`, `{x}`→`singleton`.
`<`→`lt`/`gt`, `≤`→`le`/`ge`, `⊔`→`sup`, `⊓`→`inf`, `⨆`→`iSup`, `⨅`→`iInf`, `⊥`→`bot`, `⊤`→`top`.

## Descriptive naming

- The name describes the **conclusion**; if a prefix conveys the meaning, shorten it (`neg_neg`).
- **Subject/operation first, property as a suffix, conditions last after `of`.** So
  `coeff_C_of_ne_zero` (not `coeff_ne_zero_C`), `principal_surjective` (not `surjective_principal`),
  `liftBaseChange_range_le` (not `range_liftBaseChange_map_le`), and `f_injective`/`tfae_…`. The
  `top_` prefix marks lemmas about the `⊤` element; `_self` marks a self-application.
- A property-of-an-object lemma puts the **object last**: `isInvertedBy_isomorphisms`.
- Conditions implied by a typeclass argument are omitted (`[Nonempty α]` ⇒ no `_of_nonempty`).
- Follow infix order in the term: `neg_mul_neg` for `-a * -b`.
- Prefer `pos`/`neg`/`nonneg`/`nonpos` over `zero_lt`/`lt_zero`/`zero_le`/`le_zero`.
- `left`/`right` distinguishes variants and refers to the argument that "changes".
- Avoid a trailing `'` unless there is a real naming conflict — prefer a descriptive suffix
  (`support_add_eq_union`, not `support_add_eq'`).
- Decide naming **before** merge: don't ship a `-- XXX: which name?` comment or keep both an old and
  a new name "to be safe".

### `le`/`lt` vs `ge`/`gt`
Mathlib states things with `≤`/`<`, not `≥`/`>`. Use `le`/`lt` for the first occurrence; use
`ge`/`gt` only when (1) arguments appear in swapped order, (2) to match another relation's argument
order, (3) to describe the swapped relation, or (4) the second argument is "more variable".

## Namespace placement

- Put a lemma in the **namespace of its main subject** so dot notation works: a lemma about
  `F.obj`/`IsStationary`/`Bijective` goes in `F`/`IsStationary.`/`Bijective.`. When the declaration
  must live in another file, use the `_root_.` prefix to land it there (`theorem _root_.AlgHom.range_prodMap …`).
- Lemmas about a *carrier set* (e.g. `(unitary R : Set R)`) live **outside** the object's namespace.
- `protected` a declaration whose short name is common (`center_prod`, `IsStationary.univ`,
  `Functor.IsIso`) so opening the namespace doesn't shadow it.

## Structural-lemma patterns

- **Extensionality** `(∀ x, f x = g x) → f = g`: name `.ext`, tag `@[ext]`. The `↔` form is `.ext_iff`.
- **Injectivity**: prefer a `Function.Injective f` conclusion named `f_injective`; also provide the
  bidirectional `f x = f y ↔ x = y` named `f_inj` (often a good `@[simp]`).
- **Coercions** are named after the underlying function: a lemma `⇑foo = …` is `coe_foo`; for a
  *bundled morphism* the projection lemma is `toLinearMap_foo` (and often `@[norm_cast]`, not `@[simp]`).

## Real PR examples (wrong → right)

- camelCase → snake_case theorem name: `hasFiniteProductsOfAdditiveEssSurj` →
  `hasFiniteProducts_of_additive_of_essSurj` *(#41067)*.
- predicate as a suffix: `surjective_principal_of_finite` → `principal_surjective` *(#39848)*;
  `injective_toBoundedContinuousFunctionCLM` → `toBoundedContinuousFunctionCLM_injective` *(#40931)*.
- condition after `of`: `coeff_ne_zero_C` → `coeff_C_of_ne_zero` *(#39623)*.
- `tfae_` prefix for "the following are equivalent": `universallyInjective_tfae` → `tfae_universallyInjective` *(#39910)*.
- name must match the statement: `pullback.desc'` → `pushout.desc'` *(#41024)*; `rank_modTorsion` →
  `finrank_modTorsion` *(#40239)*; fix typos (`ntRootsFinset` → `nthRootsFinset`) *(#40906)*.
- place in the subject's namespace: `of_not_isCofinal_compl` → `IsStationary.of_not_isCofinal_compl` *(#39727)*.
- canonical guide example: `inv_is_unit_times_self_eq_1` → `IsUnit.inv_val_mul` — `mul` not `times`,
  drop the redundant `1`, `IsUnit.` namespace + `isUnit`, and name the coercion (`val`).
