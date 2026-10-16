# A Lean proof of optimal square packing for 19 unit squares

Nineteen unit squares can be packed in a square of side `L` exactly when
`L ≥ 3 + 4√2/3 ≈ 4.885618`. Arbitrary rotations are allowed, and touching
boundaries are allowed.

The Lean theorem is:

```lean
Sq19.square19_optimality (L : ℝ) :
  Feasible19 L ↔ 3 + 4 * Real.sqrt 2 / 3 ≤ L
```

`Feasible19 L` means that 19 independently translated and rotated closed unit
squares, with pairwise disjoint interiors, fit inside `[0,L]²`. The statement
has no remaining certificate-acceptance or case-exclusion premise.

The underlying definitions are [unit squares](Sq19Lean/SquareBase.lean#L20),
[their coordinate parametrization](Sq19Lean/SquareBase.lean#L16),
[the packing predicate](Sq19Lean/Assembly/PlainPacking.lean#L11) and
[the endpoint](Sq19Lean/Basic.lean#L7). The
[explicit packing](Sq19Lean/EndpointPacking.lean) supplies existence at the endpoint.
The theorem is [here](Sq19Lean/Assembly/Optimality.lean#L9).

The theorem compiled in the research build with Lean 4.34.1 and the pinned
Mathlib dependencies. Concrete certificate checks use `native_decide`, so the Lean
compiler and native evaluator are part of the trust boundary, alongside
`propext`, `Classical.choice` and `Quot.sound`. The theorem's axiom report
contains 4,320 Lean-generated native-check facts and no `sorryAx`; see the
[full axiom report](verification/Square19Optimality.log). The geometric soundness
arguments are checked by Lean's kernel; the large finite certificate checks run
as compiled native code. This follows the trust model of the
[s(17) formalization](https://github.com/ofitzharding/sq17-lean). It is not a
kernel-only computation claim.

**Kernel replay passed.** Every module in the theorem's import closure (13,143
modules, Mathlib included) was replayed through Lean's kernel with the
toolchain's `LeanChecker`: 0 failures ([summary](verification/kernel-replay.json)).

## Method

The proof follows the s(17) method. The possible square centres are covered by 25
cells, and every packing below the endpoint induces an occupancy pattern. 269
representative patterns are excluded by certified half-plane propagation, and four
more by certified branch trees. The three remaining families are forced by
certified replays into a near-optimal configuration, and a certified hull argument
shows that such a configuration needs side at least `3 + 4√2/3`.

## Verify and build

About 2,100 generated certificate modules (Lean source, not precompiled objects)
are release assets of [v1.0.0](https://github.com/ofitzharding/sq19/releases/tag/v1.0.0);
[`expected/generated-sources.sha256`](expected/generated-sources.sha256) lists
every file's SHA-256.

Linux x86-64 with [elan](https://github.com/leanprover/elan), Python 3.10 or
newer, `curl`, `xz` and `libgmp`. From this directory:

```sh
bash scripts/fetch.sh            # download, check and unpack the certificate sources
python3 scripts/verify.py        # build from source; check statement and axioms
```

`verify.py` builds the hand-written modules with Lake. It compiles the checker
modules to a native library and replays every certificate module with `lean`. It
prints `VERIFIED` only if the theorem has exactly the statement in
[`expected/statement.txt`](expected/statement.txt) and the axioms in
[`expected/axioms.txt`](expected/axioms.txt). A full replay takes several hours on
a 64-core, 512 GB machine, and longer on smaller ones; it resumes if interrupted.
`python3 scripts/verify.py --kernel-replay` also repeats the kernel replay above.

## Credits

This proof extends the method and checker code of the
[s(17) formalization](https://github.com/ofitzharding/sq17-lean), which it uses
as a library at a fixed version. Thanks to everyone involved with the
[Squares Project](https://github.com/jlevy/squares) for their assistance,
especially Manasseh Ahmed
([@Queuingtheorydotcom](https://github.com/Queuingtheorydotcom)).

This project was developed with substantial assistance from Claude and Codex.
The published theorem, source files and explicit trust boundary are the basis
for assessing the result.

Project sources are licensed under the [MIT License](LICENSE). Dependencies,
including Mathlib, retain their own licenses.
