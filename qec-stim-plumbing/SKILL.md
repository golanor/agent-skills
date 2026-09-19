---
name: qec-stim-plumbing
description: Build stabilizer-code simulations as plumbing on request — Stim circuits, detector error models, PyMatching/BP-OSD decoders, Sinter threshold sweeps, logical-error-rate plots, and formal circuit certification (flow-based gadget proofs, rowspace reconstruction, p=0 determinism, hash-pinned certificates) — from the user's stated spec, inside notebook-pair's `# @agent:` marker protocol. Use when the user asks for a stim circuit, a memory experiment, a DEM, a decoder hookup, a threshold or pseudo-threshold sweep, a code-capacity or circuit-level noise model, a gadget or circuit certificate, or a stabilizer/logical-operator table for a CSS, GB, BB, APM, or surface code.
version: 1.1.0
tags: [skill, qec, stim, stabilizer-codes, simulation, certification, plumbing, physics]
---

# QEC Stim Plumbing

Everything in this skill is **plumbing** in the notebook-pair sense: the outcome is known once the user states the spec, so the agent builds it with zero ceremony. The **discovery** — choosing the code, the noise model, the sweep ranges, reading the threshold plot, deciding what the decoder's failures mean — is the user's and is never pre-empted. Act only on a `# @agent:` marker (or an explicit request outside a notebook). Protocol and role split live in `notebook-pair`; this skill supplies the domain toolkit.

## Environment (first marker in any new workspace)

Check before importing; install only what the spec needs, via the project's `uv`:

| Package | Role |
|---|---|
| `stim` | Stabilizer circuit simulator, DEM extraction, tableau ops |
| `pymatching` | MWPM decoder (surface/repetition-class codes) |
| `sinter` | Parallel Monte-Carlo sampling + threshold-plot helpers |
| `ldpc` | BP-OSD / BP-LSD decoders (qLDPC: GB, BB, HGP, lifted-product) |
| `numpy`, `galois` or plain `F₂` linear algebra | parity-check construction, rank, logical operators |

Report what was installed; state versions once.

## Spec the user supplies, in a sentence or a table

A buildable spec names: **code** (family + parameters, or explicit `H_X`, `H_Z`), **experiment** (memory-X/Z, stability, a logical gate), **rounds** `d` or `n_rounds`, **noise model** (code-capacity `p`, phenomenological `p_data, p_meas`, or circuit-level `p` with the standard gate set: depolarize after 1q/2q gates, flip before measurement, flip after reset), **decoder** (MWPM / BP-OSD with `osd_order`, `max_iter`), and **sweep** (`p` list, shots or `max_errors`). A marker missing any of these gets one `[needs]` question naming the missing field — never a guessed default silently applied.

## Build recipes

**From a parity-check pair.** Given `H_X`, `H_Z` (dense `np.uint8` or sparse): assert `H_X @ H_Z.T % 2 == 0` (CSS condition — report the violating row pair if it fails); compute `k = n − rank(H_X) − rank(H_Z)`; find logical operators as `ker(H_Z) \ rowspace(H_X)` (and the dual), and check by GF(2) linear algebra that `L_X`, `L_Z` pair up (`L_X L_Zᵀ = I`) and are independent of the stabilizers; return `n, k` and, on request, distance by brute force for small codes or via a probabilistic lower bound otherwise — labeled as such. Hash what was built: SHA-256 of each matrix *and* of the RREF of its rowspace, so later artifacts pin to exact matrices rather than to a filename.

**Group-algebra / bicycle codes.** For GB, BB, HGP, and APM (affine-permutation, Hagiwara–Imai-style `J×L` arrays of permutation blocks) families, build the circulant / permutation / Kronecker blocks from the user's polynomials or group elements exactly as stated (`A = x^a + y^b + …`, or the affine maps `j ↦ αj+β`), form `H_X = [A | B]`, `H_Z = [Bᵀ | Aᵀ]` (or the array layout the user names), and hand back the matrices *before* any experiment so the user can check them against their own hand computation.

**Stim circuit.** For surface / repetition codes use `stim.Circuit.generated(...)` with the user's noise parameters. For an arbitrary CSS code write the syndrome-extraction circuit explicitly: ancilla per check (or the user's pair-measurement / streamed-chain gadget), CNOT schedule per the user's ordering (ask if unspecified — the schedule is a physics choice, hook-fault-tolerance depends on it), `MR` on ancillas, `DETECTOR` per check comparing consecutive rounds, `OBSERVABLE_INCLUDE` for each logical. Annotate coordinates so `stim` diagrams render. Emit the circuit **noise-free first** and certify it (below) before any noise is attached.

**Noise model as an explicit object.** Keep noise in one named class (`CircuitNoise`-style) with one constructor per model the user names, so the DEM's provenance is a single symbol: e.g. `uniform(p)` = `DEPOLARIZE2` after each `CX`, flip before measure, flip after reset, one idle `DEPOLARIZE1` per data qubit per round; a pair-measurement model = one combined channel per `MZZ`/`MXX` written out as its `CORRELATED_ERROR` mechanisms plus prep/measure flips on the free single-qubit ops plus the same once-per-round idle. Apply noise by transforming the certified noise-free circuit, never by hand-editing it. Anything an "every-gate-p" shorthand hides (which ops idle, whether reset flips count) is stated once in the noise class's docstring.

**Detector error model and decoder.** `dem = circuit.detector_error_model(decompose_errors=True)`; MWPM via `pymatching.Matching.from_detector_error_model(dem)`; BP-OSD via `ldpc` on the **non-decomposed** DEM's check matrix with priors from the DEM probabilities. For non-matchable codes `decompose_errors=True` will fail — report it and switch to BP-OSD without being asked to explain why.

**Sweep.** `sinter.collect(tasks=[sinter.Task(circuit=..., decoder=..., json_metadata={...})], max_shots=..., max_errors=..., num_workers=...)`, or a sharded runner when the decoder is not a sinter plug-in: each shard's seed is SHA-derived from the *full* config (code hashes, noise model, `p`, decoder settings, shard index) so runs are reproducible and resumable, and shards aggregate by summing counts. Return logical error rate **per round** (`1 − (1 − p_L)^{1/rounds}`) alongside per-shot, with **Wilson** intervals (not normal-approximation bars — they misbehave at the low failure counts that matter); cache stats to disk keyed by the config hash so re-runs are free.

**Pseudo-threshold.** When asked: the `p_L = p` crossing by log-log interpolation between the two bracketing points, with an interval from bootstrap resampling of the per-shard failure counts. Report crossing + interval + which two points bracket it; the decision that a crossing is meaningful (enough points, monotone, no decoder saturation) is the user's.

**Plot scaffold.** `sinter.plot_error_rate(...)` or a matplotlib scaffold: log-log, one curve per `d`, physical `p` on x, logical per-round on y, error bars, the user's `p` range — no annotations, no fitted threshold line unless the user asks for the fit.

## Certification ladder: proved → checked → estimated

Every result carries the rung it sits on, and the noisy LER is the **only** rung that is sampled. Mechanics and exact GF(2) formulations live in [`reference/certification.md`](reference/certification.md); this is what gets built and in which order.

1. **Proved — code facts.** CSS orthogonality, ranks, `k`, logical basis with commutation and independence, all by GF(2) linear algebra; matrix and rowspace-RREF hashes recorded.
2. **Proved — gadget flows.** Each per-check gadget (a pair-measurement fragment, a streamed chain, the plain-CNOT reference) is verified as a **stabilizer channel**, not simulated: take `circuit.flow_generators()` — the complete generating set of relations *input Pauli → output Pauli ⊕ measurement-record set* — and decide four properties as GF(2) span membership: the check `P_S` is measured; **no gauge leak** (the data-only operators the fragment measures span exactly `⟨P_S⟩` — the failure mode of every "simplified" gadget); the commutant of `P_S` passes through unchanged; every ancilla is disentangled at the end. Exhaustive over the stabilizer group via its generators, zero sampling. Run it on two independent flow engines (stim's, and a bit-mask `FlowState` of your own) and diff — disagreements have historically been bugs in the fast path. Hash each certified gadget's flow set.
3. **Proved — whole-circuit rowspace.** Gadget correctness does not imply the assembled circuit is right (detector bookkeeping, frame propagation, round stitching). From one emitted extraction cycle, eliminate the flows down to the data-only measured space and require it to **equal** `rowspace(H_X)` / `rowspace(H_Z)` exactly — rank and RREF hash. That is the statement "this circuit measures precisely this code's stabilizers, all of them and nothing else." Also require the cycle to repeat identically across rounds.
4. **Checked — ideal determinism.** At `p = 0` the DEM has **zero** error mechanisms, and every sampled detector and every logical observable is 0 over all shots. Any frame or detector error fails this instantly (it has caught a mis-signed `m₁⊕m₃` frame and a double-counted ideal-frame correction).
5. **Pinned.** One JSON certificate per benchmarked circuit: code hashes, gadget-flow hashes, circuit SHA-256, DEM SHA-256, resource counts (qubits, ops per round, measurements), noise-model name and parameters. The LER plot cites the certificate; a benchmark without one is an anecdote.
6. **Estimated — noisy LER.** Only now: sweep, Wilson intervals, pseudo-threshold.

Rung 1 always runs. Rungs 2–5 run whenever the circuit is hand-written (not `stim.Circuit.generated`) or the user asks for a certificate; a hand-written circuit that skips them is labeled `uncertified` in its return value. The user reads the certificate; the agent never interprets a passing certificate as "the code is good" — it only says the circuit is the code.

## Structural sanity checks (always run, silently unless one fails)

- CSS commutation `H_X H_Zᵀ = 0`; `n, k` match the stated code (ladder rung 1).
- The generated circuit has `num_detectors > 0` and `num_observables == k`.
- Zero-noise determinism: at `p = 0` the DEM is empty and all detectors and observables are 0 over ≥1000 shots (rung 4).
- Detector-graph degree: an MWPM decoder on a DEM with hyperedges is a spec error, not a runtime detail — report it.
- Two engines agree: wherever a fast path exists (own flow engine, own decoder wrapper, cached matrices), diff it against the reference path on the same input before trusting it.

A failing check is reported as a one-line fact (`[failed] H_X H_Z^T ≠ 0 at rows (3,7)`, `[failed] gauge leak: measured data space dim 2`), never as a fix — the user decides.

## Silence rules (from notebook-pair)

Output that surprises you is the user's discovery — say nothing. A bug that would silently corrupt a result (wrong observable, shots too few for the error bar) is announced as *existing*, not located. The only permitted pre-emption is cost: a sweep that will take more than a few minutes gets one line with the estimate before it runs.

## Done test

The marker is `[done]` when: the requested artifact exists in the cell or library module, every sanity check above passed (or the failure was reported), every number returned is labeled with its ladder rung (proved / checked / estimated) and a hand-written circuit is either certified or marked `uncertified`, the returned object is the one the spec named (matrices, circuit, DEM, certificate, stats, or plot — not a bundle of all of them), and no cell, interpretation, or next step was added that the user did not ask for. The verifier that closes the loop is the certificate for rungs 1–5 and the user for rung 6.
