# Certification mechanics

On-demand reference for the ladder in `SKILL.md`. Everything here is GF(2) linear algebra over vectors stim hands you; nothing is sampled until the last section. Patterns distilled from a working APM/qLDPC certification pipeline (code library → gadget verifier → whole-circuit certifier → sharded benchmark); the names below are the roles, not required identifiers.

## GF(2) toolkit (write once, reuse everywhere)

- `rref(M)` — row-reduced echelon form over F₂, returning the basis rows. Rank = number of nonzero rows.
- `in_span(basis, v)` — reduce `v` against the RREF basis; in span iff the residual is 0.
- `rowspace_hash(M)` — SHA-256 of the *RREF* bytes. Two matrices with the same rowspace hash the same; this is what pins a code, not the raw matrix hash (which pins a presentation).
- Pauli ↔ bits: a Pauli on `nq` qubits is `[x | z]` of length `2·nq`. A flow row is `[in_x | in_z | out_x | out_z (| recs)]`, length `4·nq (+ num_measurements)`.

## Rung 1 — code facts

Input: `H_X`, `H_Z` as `np.uint8`.

```
assert not ((H_X @ H_Z.T) % 2).any()          # CSS; on failure report the (row, row) pair
rX, rZ = rank(H_X), rank(H_Z)
k = n - rX - rZ
L_X = basis of ker(H_Z) modulo rowspace(H_X)   # k rows
L_Z = basis of ker(H_X) modulo rowspace(H_Z)   # k rows, chosen so L_X @ L_Z.T % 2 == I_k
```

Checks: `L_X @ H_Z.T % 2 == 0`, `L_Z @ H_X.T % 2 == 0`, `L_X @ L_Z.T % 2 == I_k`, and `rank([H_X; L_X]) == rX + k` (independence). Record `sha256(H_X)`, `sha256(H_Z)`, `rowspace_hash(H_X)`, `rowspace_hash(H_Z)`, `n, k, rX, rZ`.

## Rung 2 — gadget as a stabilizer channel

A gadget is a stim fragment on `nq` qubits: data qubits `0..w−1` carrying the check `P_S = P⊗w` (`P ∈ {X, Z}`), ancillas `w..nq−1`. Build `rows = flow_rows(circ)` from `circ.flow_generators()`: for each flow `f`, concatenate `pauli_bits(f.input_copy())`, `pauli_bits(f.output_copy())`, and (when needed) a one-hot over `f.measurements_copy()`. `basis = rref(rows)`.

Four span-membership tests, in this order (each returns a one-line reason on failure):

| # | Property | Test |
|---|---|---|
| a | `P_S` is measured | `(P_S → I, some record set)` ∈ span: `in_span(basis, [P_S | 0 | 0])` |
| c | **no gauge leak** | `leak_space` = rows with `out = I` restricted to data-only `in`; require `dim ≤ 1` and, if 1, equal to `P_S`. A `dim ≥ 2` result means the fragment also measures an operator the code did not ask for — it is a *different* channel. |
| b | commutant preserved | for each generator `Q` of the commutant of `P_S` on the data qubits (single-qubit `P`'s and pairwise products of the conjugate Pauli), `in_span(basis, [Q | Q])` |
| d | ancillas closed out | for each ancilla `a`: `(I → Z_a)` or `(I → X_a)` ∈ span — the ancilla ends in a definite eigenstate, no entanglement left with data |

`verify_gadget(circ, w, nq, check_pauli)` → `(ok, reason)`. Certify with two engines — stim's `flow_generators()` and an independent bit-mask flow tracker (`FlowState`: propagate `[x|z]` masks through each gate by lookup, absorb measurements into the record set) — and require identical `ok` and identical `rowspace_hash(rows)`. Record `flow_sha256 = rowspace_hash(rows)` per certified gadget.

Why this is a proof and not a test: the flow generators span the whole stabilizer-channel relation; span membership over that basis decides every Pauli input at once. There is no shot count.

## Rung 3 — whole-circuit rowspace reconstruction

Input: one emitted extraction cycle `C_cycle` (noise-free) on `n` data + `n_anc` ancilla qubits, plus the target `H` (`H_X` for X-checks, `H_Z` for Z-checks).

1. `rows = flow_rows(C_cycle, with_recs=True)`.
2. Eliminate to the **data-only measured space** `M`: rows whose `out` is identity on data qubits and whose `in` is supported on data only, projected to the `in` data bits of the check type.
3. Require `rank(M) == rank(H)` and `rowspace_hash(M) == rowspace_hash(H)`. Equality of rowspaces, not containment: containment in one direction is "measures only stabilizers", the other is "measures all of them"; both are needed.
4. Round repetition: emit two cycles, split the flow rows by round, require identical RREF per round (up to the record offset).

Failure report names which direction broke: `missing stabilizers (rank M < rank H)` or `extra measured operator (rank M > rank H)` or `same rank, different space`.

## Rung 4 — ideal determinism

```
dem0 = circuit_at_p0.detector_error_model()
assert dem0.num_errors == 0                          # zero mechanisms
det, obs = circuit_at_p0.compile_detector_sampler().sample(shots, separate_observables=True)
assert not det.any() and not obs.any()
```

Anything nonzero is a bookkeeping error in the *circuit* (frame sign, detector pairing across rounds, observable definition), not a physics result. Report the first offending detector index and round.

## Rung 5 — the certificate

One JSON per benchmarked circuit. Minimum fields:

```
code:      {n, k, rank_X, rank_Z, H_X_sha256, H_Z_sha256, rowspace_X_sha256, rowspace_Z_sha256, source_file}
gadgets:   [{name, w, nq, check_pauli, flow_sha256, engines_agree: true}]
circuit:   {circuit_sha256, rounds, num_qubits, num_measurements, num_detectors, num_observables, ops_per_round}
rowspace:  {X: {rank, sha256, equals_target: true}, Z: {...}, cycle_repeats: true}
ideal:     {dem_num_errors: 0, shots, all_zero: true}
noise:     {model_name, parameters, dem_sha256}
generated: ISO timestamp, tool versions (stim, ldpc, sinter)
```

The benchmark's `json_metadata` carries the certificate path and `circuit_sha256`, so every plotted point can be traced back to the byte-identified circuit.

## Rung 6 — noisy estimate

- **Shard seeds**: `seed = int(sha256(json.dumps(config, sort_keys=True) + f":{shard}")[:16], 16)` where `config` includes the certificate hashes, `p`, decoder settings. Same config ⇒ same shards ⇒ resumable; any change ⇒ new seeds.
- **Aggregation**: sum `shots` and `failures` across shards per `p`.
- **Wilson interval** (`z = 1.96`): center `(f + z²/2)/(N + z²)`, half-width `z·sqrt(f(N−f)/N + z²/4)/(N + z²)`. Use it for every plotted bar.
- **Pseudo-threshold**: find the first adjacent pair `(p_i, p_{i+1})` with `p_L(p_i) ≤ p_i` and `p_L(p_{i+1}) > p_{i+1}`; interpolate `f = log(p_L/p)` linearly in `log p` to `f = 0` (geometric midpoint if `p_L(p_i) = 0`). Interval: bootstrap by resampling shard failure counts (binomial with each shard's observed rate), recompute the crossing per replicate, report the 2.5/97.5 percentiles. Refuse (with `[needs]`) if no sign change exists in the sweep range.

## Rejected practices (each paired with the rule it violates)

- Simulating a gadget on random inputs to "check" it — rung 2 decides every input at once; sampling is weaker and slower.
- Trusting `decompose_errors=True` on a qLDPC DEM — it either fails or silently drops hyperedges; BP-OSD on the raw DEM is the rule.
- Normal-approximation error bars at `< 10` failures — Wilson.
- Hashing the matrix file name or path — hash the matrices and their rowspaces.
- Applying noise by editing the circuit text — transform the certified noise-free circuit through the noise class so the DEM's provenance is one symbol.
