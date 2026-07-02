# Proof & formatting style

Authoritative source: <https://leanprover-community.github.io/contribute/style.html>.
Most of these are enforced by CI linters in `Mathlib/Tactic/Linter/` (`Style`, `Whitespace`,
`EmptyLine`, `CommandStart`, `Header`, `DocString`, `DocPrime`, `OldObtain`, `Multigoal`, …), so a
violation usually shows up as a red CI check.

## Reach for automation first

The single most common proof-review request is "let a tactic do this." Prefer **`grind`**, `simp` /
`simp_rw`, `gcongr`, `positivity`, `fun_prop`, `omega`, `aesop` — usually with an explicit lemma list
(`grind [foo, bar]`, `simp [foo]`) so the proof still reads as "trivial by these facts" — over manual
case analysis, `by_cases` ladders, and long `rw` chains. Use the tactic at full strength: write
`gcongr` rather than `apply add_le_add; gcongr`, and `by fun_prop`/`by positivity` for continuity,
measurability, and positivity side goals. Don't thread `[DecidableEq _]` through an API just to make a
proof go — open with `by classical` instead.

## Layout

- **Lines ≤ 100 characters.**
- Declarations and commands are **flush-left**. Opening a `namespace`/`section` does **not** indent
  its contents.
- Spaces around `:`, `:=`, and infix operators. Put an operator/`:`/`:=` **before** a line break,
  not at the start of the next line.
- Proof body indents **2 spaces**. If the *statement* spans multiple lines, continuation lines
  indent **4 spaces** (the proof is still only 2, not 6).
- `by` goes at the **end of the preceding line** (`:= by`), never on its own line.
- **No empty lines inside a declaration** (linter-enforced). Add a short comment instead.
- A single blank line between declarations; keep the blank line between any two whose statements span
  multiple lines, and omit it only to group a run of *one-line* declarations.
- Prefer a single-line `variable` with `Type*` (`{R R' : Type*}`) over per-variable `Type u`/`Type v`.
- Structure/class fields indent 2 spaces and each gets its own docstring.
- **Don't reformat or restyle code you aren't otherwise changing** — it churns `git blame` for no
  benefit. Match the surrounding file's local style.

Canonical "do NOT" from the guide, then the fix:
```lean
-- bad
theorem mul_assoc_assoc {α:Type*}[Semigroup α](a b c d:α)
    :a*b*c*d=a*(b*(c*d))
    :=by rw[mul_assoc,
    mul_assoc]
-- good
theorem mul_assoc_assoc {α : Type*} [Semigroup α] (a b c d : α) :
    a * b * c * d = a * (b * (c * d)) := by
  rw [mul_assoc, mul_assoc]
```

## Notation choices

- `fun x ↦ e` — the `↦` arrow (`\mapsto`). **`λ` is disallowed**; `=>` is allowed but `↦` is
  preferred. Use the named-binder form `map_add' x y := …` over `map_add' := fun x y => …`.
- Use `<|` and `|>` to drop parentheses; **`$` is disallowed**. (But don't use `<|` for a single `rw`
  argument: `rw [foo h]`, not `rw [foo <| h]`.)
- No space after unary minus: `-a.left`. A space after `←` in `rw`/`simp`: `rw [← add_comm a b]`.
- Prefer the anonymous constructor `⟨…⟩` over a `where` block for a simple term; use `where`
  (no braces) for instances/structures.

## Tactic idioms reviewers ask for (wrong → right)

| Wrong | Right | PR |
|---|---|---|
| `rw […]` then `exact h` | `rwa […]` | #40944 |
| a **non-terminal** `simp` then `exact`/`infer_instance`/`rw` | `simpa using …`, or an explicit `rw` | #40883, #40889 |
| `have hI : … := …` (an instance); `letI := …` (an instance) | `haveI := …` | #40924, #39669 |
| `refine f x` with no `?_` holes | `exact f x` | #40714 |
| manual case ladder / `by_cases` + `rcases` | `grind [lemmas]` or `rcases lt_trichotomy …` | #40293, #40888 |
| `intro x hx; rcases hx with h \| h; rw [h]` | `rintro x (rfl \| rfl)` | #39435 |
| `by_contra h; push_neg at h` | `by_contra! h` | #39912 |
| `change …` to restate the goal | `rw`/`show`, or rely on defeq | #40239, #39958 |
| `ext : 1` | `ext`; `ext ⟨⟩` to split a product | #40944, #40491 |
| `simp only […] at h` (goal needs it too) | `simp only […] at h ⊢` | #40944 |
| `:= by … inferInstanceAs (…fully unfolded…)` | `:= by dsimp only [myDef]; infer_instance` | #40944 |
| trivial goal `:= by exact …` / longer | `rfl` / `inferInstance` / `Iso.refl _` / `Iff.rfl` | #41000, #40647 |
| `obtain ⟨n, hn : T n⟩ := h` (over-annotated); unused names | `obtain ⟨n, hn⟩`; `_` for unused | #40944 |
| `congr; ext` / `congr <;> ext` under a binder | `simp_rw […]` | #40583 |
| `map_add' := by intro x y; simp` | `map_add' x y := by simp` (let `simp` intro) | #40634 |
| bare `⟨…⟩` for `x ∈ Set.Icc …` (defeq abuse) | `Set.mem_Icc.mpr ⟨…⟩` | #40655 |
| explicit `@this …` in `wlog` recursion; redundant `↑` | `this …`; drop the coercion | #40976, #40795 |
| `Foo.Bar C` inside that namespace; dead/unused `open` | short alias; remove it | #40944, #41034 |

The **`flexible` linter** flags a non-terminal `simp`: `simp` can change the goal shape, so a
following brittle tactic breaks easily — close such goals with `simpa using …`.

## `calc`, focusing dots, `erw`, squeezed simps

- `calc` keyword on the line *before* the calculation; align the relation; left-justify the `_`.
- New goals use a focusing dot `·` (not indented; body indented); one tactic per line in general.
- Needing `erw`, or an extra `rfl` after `simp`/`rw`, signals **missing API** — add the lemma. *(#41014)*
- Do **not** squeeze a *terminal* `simp` (or a `simp` only followed by `ring`/`aesop`): the `simp?`
  output buries the key lemmas and breaks on renames.

## Factor long proofs

Extract reusable auxiliary lemmas and large `suffices` statements into named lemmas. Mark genuinely
file-local helpers `private` (and/or `aux` with a docstring "Not intended for use outside this file").
*(#40944, #41034, #40000.)*

## Proof shape: fold and invert `have`s

Deep indentation is a tell-tale sign of machine-written proofs: a large `have` scaffold is built up
first, then discharged by a one-line `exact` at the very end. Idiomatic Mathlib proofs instead read
top-down with as little nesting as possible. Two complementary moves (named in the review of #40973)
fix most cases:

- **`have`-folding** — inline a `have` whose proof is short. A one-tactic step (`by fun_prop`, a
  single `simp_rw`/`rw`) rarely earns a name; inlining it deletes a layer of indentation. Below, the
  nested `have hfun … ; rw [hfun, …]` folds into a single `simp_rw [hmul s, …]`.
- **`have`-inverting** — when the shape is "prove one big `have`, then a short `exact` that consumes
  it", flip it. Prove the *short* fact first (often the goal's outer shape, closed by `fun_prop` or
  another automation tactic), then `convert` (or `suffices`) down to the equality that was the
  `have`'s content, so that former `have` body becomes the **main** proof body. The
  `have … := by intro …` wrapper and the extra indentation it forced both disappear.

```lean
-- before: a big `have` scaffold discharged by a hand-built one-line `exact` (deeply nested)
have hwindow : ∀ s : ℝ, f s = (F (s + a) - F s) / F a := by
  intro s
  have h2 : … := by
    have hfun : (fun u ↦ f (s + u)) = fun u ↦ f s * f u := by funext u; rw [hmul s u]
    rw [hfun, intervalIntegral.integral_const_mul]
  have hsub : … := by …
  have hadj : … := by …
  rw [eq_div_iff ha, hsub, hadj]
exact (((hFcont.comp (continuous_id.add continuous_const)).sub hFcont).div_const (F a)).congr
  fun s ↦ (hwindow s).symm
```
```lean
-- after: prove the short fact first, `convert`, and the former `have` body is now the main goal
have hcontinuous : Continuous fun s ↦ (F (s + a) - F s) / F a := by fun_prop
convert hcontinuous with s
have h2 : … := by simp_rw [hmul s, intervalIntegral.integral_const_mul]   -- folded
have hsub : … := by …
have hadj : … := by …
rw [eq_div_iff ha, hsub, hadj]
```

Both proofs do the same mathematics, but the second dedents everything by a level, replaces the
hand-built continuity term with `fun_prop`, and reads as a sequence of steps rather than a scaffold.
*(#40973.)*
