# Lean 4 proofs: unit squares in an equilateral triangle, n = 2, 3, 4

Kernel-checked proofs of

```lean
theorem minSideTri_two   : minSideTri 2 = 2 + 2 / Real.sqrt 3
theorem minSideTri_three : minSideTri 3 = 3 / 2 + Real.sqrt 3
theorem minSideTri_four  : minSideTri 4 = 3 + 2 / Real.sqrt 3
```

`minSideTri n` is the infimum of the sides `s` for which `n` closed unit squares, at any positions
and angles, fit in the equilateral triangle `tri s` (vertices `(0,0)`, `(s,0)`, `(s/2, s√3/2)`)
with pairwise disjoint interiors.

`#print axioms` shows only `propext`, `Classical.choice` and `Quot.sound` (see `Axioms.lean`).
There is no `sorry`, no `native_decide` and no new `axiom`.

## Trusted base

- The Lean kernel, Lean core and Mathlib.
- The definitions in the statements: `sq` and `sqInt` (closed and open square; copied from
  evand's Lean, see below), `tri`, `PacksTri`, `minSideTri`.

Everything else is proved. In particular the Python generator in `gen/` and both Python
checkers of the paper (`checker/`, `checker2/`) are not trusted. The generator only proposes
data, and every piece of data is re-checked by the kernel (`decide +kernel`).

## How the proof is organised

| file | content |
|---|---|
| `Sqtri/Evand.lean` | Copied from evand/square-packing (commit `6e1223c`, `s12/lean/Sqpack`, MIT, see `UPSTREAM-LICENSE.txt`). Includes `sq`, `sqInt`, `packing_le_weight`, the `u = tan(θ/2)` form of "p is in the square" (`mem_sq_iff_gval`), `sq_add_pi_div_two` and `coverA`/`coverW`. |
| `Sqtri/Poly.lean` | Exact arithmetic in `ℚ(√3)` (pairs of rationals) with an exact sign test (`nonnegB_sound`, `nonnegB_complete`); polynomials in `u` and forms affine in the centre. |
| `Sqtri/Check.lean` | The certificate checker `check` and its soundness `check_sound`. |
| `Sqtri/Geom.lean` | From the checker's forms to geometry. Admissibility gives nonnegative forms (`adm_nonneg`). Nonnegative capture forms put the point in the square (`mem_sq_of_captured`). Angles reduce to `u ∈ [0,1]`. `unavoidable_of_checks` puts these together. |
| `Sqtri/MinSide.lean` | `PacksTri`, `minSideTri`, and the lower-bound argument `le_of_unavoidable`. |
| `Sqtri/Upper.lean` | Exact tests for "a square with centre and (cos, sin) in `ℚ(√3)` lies in `tri L`" and "two such squares have disjoint interiors" (a separating edge normal). Both come with soundness proofs. |
| `Sqtri/Data/N{2,3,4}/` | The generated certificates: one kernel-checked theorem per leaf (`Leaves*.lean`), glued along the tree by `check_sx`/`check_sy`/`check_su`/`check_lin` (`All.lean`). |
| `Sqtri/Main.lean` | The three theorems. |

### Lower bound

1. **Scaling.** Suppose `n` unit squares with disjoint interiors fit in `tri s` with `s < v`.
   Scale the whole picture by `v/s > 1`. This gives `n` squares of side `v/s` in `tri v`, still
   with disjoint interiors. The concentric unit squares are then pairwise disjoint closed squares
   in `tri v` (`packing_le_weight`).
2. **Unavoidable set.** The certificate's weighted points meet every closed unit square in
   `tri v` with total weight ≥ 1, and the total weight is < n. So `n ≤ W < n`, a contradiction.
3. **Checking the unavoidable set.** A pose is a centre `(x, y)` and `u = tan(θ/2) ∈ [0, 1]`.
   The square has period π/2, so this covers every angle. The range `[0,1]` is cut at
   `u = 2 − √3` and `√3/3` (θ = π/6, π/3), because margin-zero contacts occur exactly there.
   On each of the three bins a tree splits pose space:
   - at values of `x`, `y` or `u`, or
   - by the sign of an affine form.

   Every leaf carries a **Farkas–Bernstein certificate**: a polynomial identity
   `Σ c_e · (u − U0)^i (U1 − u)^(d−i) · h_e = target`. Here
   - every `c_e ≥ 0` lies in `ℚ(√3)`;
   - each `h_e ≥ 0` is a hypothesis form: one of the 12 admissibility forms (each square vertex
     lies in the triangle), a side of the centre box, or a split form (or `h_e = 1`);
   - `target` is one of the four side conditions of "point p is in the square", times `1 + u²`.

   Every term is `≥ 0` on the leaf's region, so the target is `≥ 0` there. For an empty leaf
   the identity is `Σ … + 1 = 0`, which is impossible. The kernel checks each identity
   coefficient by coefficient over `ℚ`.
   - No symmetry reduction and no chord-pair lemma are used.
   - The n = 4 family in which the captured point switches is handled by splitting on the sign
     of a capture form.

### Upper bound

Explicit packings with centres, cosines and sines in `ℚ(√3)`:

- n = 2: two squares side by side on the base.
- n = 3: Friedman's pinwheel, at angles 0, π/6, π/3 (equivalently 0, 2π/3, 4π/3).
- n = 4: three squares on the base and one on top.

Containment is checked at the four vertices. A linear function is `≥ 0` on a square once it is
`≥ 0` at its vertices. Disjointness is checked with an edge normal of one of the two squares as
a separating direction; the squares may touch.

## Build

Lean 4.33.1 and Mathlib v4.33.1 (`lean-toolchain`, `lake-manifest.json`).

```sh
lake exe cache get      # Mathlib oleans
lake build              # checks everything, including the kernel certificates
lake env lean Axioms.lean
```

## Regenerating the certificates

```sh
cd gen
uv venv .venv && uv pip install --python .venv/bin/python gmpy2 numpy scipy
.venv/bin/python tree.py ../../certificates/n4/n4_cert.json --max-depth 26 --out out/n4.json
.venv/bin/python emit2.py out/n4.json ../../certificates/n4/n4_cert.json N4 ../Sqtri/Data/N4
```

A clean build of this project (Mathlib from the cache) took 9 minutes of wall time on a
16-vCPU Linux machine. The kernel certificates account for most of it: about 12 CPU-minutes for
n = 3 and 26 CPU-minutes for n = 4. The largest single process used 8.8 GB
(`logs/build_clean_tyo4.log`, `logs/axioms.txt`).

The generator searches for multipliers with a floating-point LP (HiGHS) and then solves exactly
over `ℚ(√3)` (a small exact simplex). Only the exact result is written.

| n | points (weight) | leaves | of which empty | tree generation |
|---|---|---|---|---|
| 2 | 1 (1) | 3 | 0 | < 1 s |
| 3 | 7 (1/3 each) | 86 | 42 | ≈ 1 min |
| 4 | 3 (1 each) | 240 | 36 | ≈ 2.5 min |
