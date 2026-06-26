# Documentation, file structure, and location

Authoritative sources: <https://leanprover-community.github.io/contribute/doc.html> and the
"Header and imports" / "Module docstrings" sections of the style guide.

## File header (current module-system form)

```lean
/-
Copyright (c) 2026 Jane Doe. All rights reserved.
Released under Apache 2.0 license as described in the file LICENSE.
Authors: Jane Doe, John Roe
-/
module

public import Mathlib.Algebra.Group.Defs
public import Mathlib.Data.Nat.Basic

import Mathlib.Tactic.Common

/-!
# Foos and bars

In this file we introduce `foo` and `bar` …
-/
```

- `Authors:` even for one author; **no trailing period**; comma-separated (no "and").
- The **copyright year is the year of contribution** for a new file. When **moving** existing code,
  keep the *original* year and authors (trace via `git log`/blame); don't claim sole authorship of
  relocated code — append yourself at most. *(#40658)*
- `module` on its own line, blank line, grouped `public import`s, blank line, grouped plain `import`s;
  keep each block **alphabetical**.
- The module docstring is a `/-! … -/` block with a `#`-title **and a summary** (not just a title),
  then optional `## Main definitions`, `## Main statements`, `## Notation` (mandatory if you add
  notation), `## Implementation notes`, `## References`, `## Tags`. *(#40248, #39891)*

## Per-declaration docstrings

- Every `def` and **major theorem** needs a `/-- … -/` docstring (`docBlame`/`docBlameThm` linters).
- **Describe the mathematical purpose, not the implementation** ("the `p`-th root map `k → …`", not
  "the equivalence with underlying map `id`"). A complete sentence ends in a period; **continuation
  lines are not indented**. Use backticks for Lean names, `$…$`/`$$…$$` for LaTeX, `<…>` for URLs.
  *(#40187, #39740, #40714)*
- **The docstring must match the statement** — variables, which hypothesis is on which object, and
  any edge-case condition (reference the actual hypothesis, not an informal paraphrase). Update it
  when you generalize. *(#41008, #40080, #40903)*
- Use precise terminology and correct grammar/Unicode (`étale`, `an` before a vowel, right plural;
  fully-qualified names so they link). Cross-reference related declarations and mention searchable
  keywords. Document a design decision (e.g. a `to_additive` orientation) or a diamond. *(#40942, #39884)*
- Warn the reader when a declaration is auxiliary (name it `…aux`/`private`, say so).

## Citing the literature

Add a BibTeX entry to `docs/references.bib` and cite it (`[key]` or `[text][key]`). A literature link
is a sanity check that the material is wanted; `@[stacks …]`/`@[kerodon …]` tag verbatim results.

## Location, imports, and splitting

- Put each declaration in the file it **belongs** to (use `#find_home`); put **general lemmas in
  general files**, not in a domain-specific module (a PR doing the latter may be closed). *(#39912)*
- **Minimize imports, especially `public` ones.** Start from `#min_imports`, import the minimal file
  (`Order.Defs.LinearOrder`, not `Order.Lattice`), and make an import private when downstream files
  don't need it publicly. Importing `Analysis` into an `Algebra` file is a red flag. *(#39543, #40248)*
- **Split a file** when it exceeds ~1000 lines, mixes loosely related topics, or would otherwise drag
  in new imports; a new file to avoid bloating a long one is legitimate. *(#40892)*
- A fully-**deprecated** module file keeps its original header and is marked so the import ("shake")
  linter leaves it alone (`-- shake: keep-all`). *(#40869)*
- Use `/-! … -/` for section headers (they appear in docs), `/- … -/` for technical/TODO comments,
  `--` for short inline comments, and `###` for titles inside sectioning comments.
