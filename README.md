# Unit squares in an equilateral triangle: optimal packings for n = 2, 3, 4

Let s△(n) be the side of the smallest equilateral triangle that contains n non-overlapping unit
squares (any positions, any angles).

| n | s△(n) | ≈ | optimal packing (upper bound) | lower-bound certificate |
|---|---|---|---|---|
| 2 | 2 + 2/√3 | 3.1547 | two squares side by side on the base | the centroid, weight 1 (total 1) |
| 3 | 3/2 + √3 | 3.2321 | "pinwheel": one square on each side | 7 points of weight 1/3 (total 7/3) |
| 4 | 3 + 2/√3 | 4.1547 | three squares on the base, one on top | 3 points of weight 1 (total 3) and Lemma P |

The packings are the best known ones in Erich Friedman's Packing Center, "Squares in Triangles"
(https://erich-friedman.github.io/packing/squintri/). The certificates show that they are optimal.
The results are computer-assisted and have not been peer reviewed.

## Novelty

- Friedman's table lists n = 1 and n = 2 as "Trivial". The proof for n = 2 is included because it
  is the simplest instance of the method.
- For n = 3 and n = 4 we are not aware of a previous proof of optimality. We checked:
  - Friedman's page (as of 2026-09-30), which gives the packings but no proofs or lower bounds;
  - an arXiv search of abstracts containing "equilateral triangle", "squares" and "packing";
  - general web searches.

  We did not find a lower bound for this container. If you know of one, please open an issue.

## The method

Each lower bound is a finite weighted point set in the triangle T_v of side v = the claimed
value. The checkers prove two things:

1. **Capture.** Every closed unit square Q ⊆ T_v, at any position and any angle, contains points
   of total weight ≥ 1. A point on the boundary of Q counts. This is checked at the container
   itself, with margin zero.
2. **Budget.** The total weight is < n.

Then n squares do not fit in a triangle of side L < v.

- Suppose they did, with disjoint interiors. Translate each square by (v/L − 1)(c_i − G), where
  c_i is its centre and G the common centroid.
- The translated squares are pairwise disjoint closed unit squares in T_v.
- Each of them captures weight ≥ 1, so the total would be ≥ n. This contradicts the budget.

With the packing of side v, this gives s△(n) = v. Details are in `certificates/n*/PROOF.md`.

**Lemma P (used for n = 4).** For two points p, q at distance exactly 1, a square whose chord
along the line pq crosses two opposite edges and meets the segment [p, q] contains p or q. This
handles the middle square of the base row, which holds both lower points on its two edges.

## Two independent checkers

- **`checker/`** subdivides pose space (centre box × angle bin, with u = tan(θ/2)).
  - For fixed u, the admissible centres in a box form a convex polygon whose vertices are
    rational functions of u.
  - Capture is checked at the vertices, with exact Bernstein bounds over Q(√3).
  - The soundness argument is in `checker/README.md`; the specification is in `checker/SPEC.md`.
  - Standard library only.
- **`checker2/`** is a second checker written from the specification, the proof notes and the
  certificate format, with a different method. Its README describes the method and what its
  author read (Independence section). It verifies the same three certificates and rejects the
  negative controls.

## Reproducing

```bash
# checker 1 (Python 3.10+, standard library)
python checker/check.py certificates/n2/n2_cert.json --n 2
python checker/check.py certificates/n3/n3_cert.json --n 3 --max-depth 44 --jobs 8
python checker/check.py certificates/n4/n4_cert.json --n 4 --max-depth 32 --jobs 8
cd checker && python -m pytest -q tests          # needs pytest; about 1.5 minutes

# stand-alone check of the n = 2 lemma (needs sympy and mpmath)
python certificates/n2/check_n2.py
```

Each run prints a JSON summary with `"status": "verified"`. The summary also contains the
certificate's sha256, the exact total weight and whether it is < n. The recorded runs are
`certificates/n*/check_record.json`. Checker 2 needs sympy and gmpy2; see `checker2/README.md`.

`tools/` has the float and exact random probes used as sanity checks (not part of the proofs).
It also has the script that checks the n = 3 pinwheel packing.

## Not done yet

- A Lean formalisation of the reductions and of the checkers.
- n = 5 and n = 6. Their best known side is the same value, 2 + 4/√3, and they are work in progress.
  s△(5) = 2 + 4/√3 would also give s△(6).

## Licence

MIT (see `LICENSE`). The packings shown in the table are from Erich Friedman's Packing Center.
