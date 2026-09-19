---
name: qec-stim-plumbing
description: Build stabilizer-code simulations as plumbing on request — Stim circuits, detector error models, PyMatching/BP-OSD decoders, Sinter threshold sweeps, logical-error-rate plots — from the user's stated spec, inside notebook-pair's `# @agent:` marker protocol. Use when the user asks for a stim circuit, a memory experiment, a DEM, a decoder hookup, a threshold sweep, a code-capacity or circuit-level noise model, or a stabilizer/logical-operator table for a CSS, GB, BB, or surface code.
version: 1.0.0
tags: [skill, qec, stim, stabilizer-codes, simulation, plumbing, physics]
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

**From a parity-check pair.** Given `H_X`, `H_Z` (dense `np.uint8` or sparse): assert `H_X @ H_Z.T % 2 == 0` (CSS condition — report the violating row pair if it fails); compute `k = n − rank(H_X) − rank(H_Z)`; find logical operators as `ker(H_Z) \ rowspace(H_X)` (and the dual); return `n, k` and, on request, distance by brute force for small codes or via a probabilistic lower bound otherwise — labeled as such.

**Group-algebra / bicycle codes.** For GB, BB, and HGP families, build the circulant / Kronecker blocks from the user's polynomials or group elements exactly as stated (`A = x^a + y^b + …`), form `H_X = [A | B]`, `H_Z = [Bᵀ | Aᵀ]`, and hand back the matrices *before* any experiment so the user can check them against their own hand computation.

**Stim circuit.** For surface / repetition codes use `stim.Circuit.generated(...)` with the user's noise parameters. For an arbitrary CSS code write the syndrome-extraction circuit explicitly: ancilla per check, CNOT schedule per the user's ordering (ask if unspecified — the schedule is a physics choice, hook-fault-tolerance depends on it), `MR` on ancillas, `DETECTOR` per check comparing consecutive rounds, `OBSERVABLE_INCLUDE` for each logical. Annotate coordinates so `stim` diagrams render.

**Detector error model and decoder.** `dem = circuit.detector_error_model(decompose_errors=True)`; MWPM via `pymatching.Matching.from_detector_error_model(dem)`; BP-OSD via `ldpc` on the DEM's check matrix with priors from the DEM probabilities. For non-matchable codes `decompose_errors=True` will fail — report it and switch to BP-OSD without being asked to explain why.

**Sweep.** `sinter.collect(tasks=[sinter.Task(circuit=..., decoder=..., json_metadata={...})], max_shots=..., max_errors=..., num_workers=...)`. Return logical error rate **per round** (`1 − (1 − p_L)^{1/rounds}`) alongside per-shot, with binomial error bars; cache the `sinter.TaskStats` to disk keyed by the spec hash so re-runs are free.

**Plot scaffold.** `sinter.plot_error_rate(...)` or a matplotlib scaffold: log-log, one curve per `d`, physical `p` on x, logical per-round on y, error bars, the user's `p` range — no annotations, no fitted threshold line unless the user asks for the fit.

## Structural sanity checks (always run, silently unless one fails)

- CSS commutation `H_X H_Zᵀ = 0`; `n, k` match the stated code.
- The generated circuit has `num_detectors > 0` and `num_observables == k`.
- Zero-noise sanity: with `p = 0` the logical error rate is exactly 0 over ≥1000 shots.
- Detector-graph degree: an MWPM decoder on a DEM with hyperedges is a spec error, not a runtime detail — report it.

A failing check is reported as a one-line fact (`[failed] H_X H_Z^T ≠ 0 at rows (3,7)`), never as a fix — the user decides.

## Silence rules (from notebook-pair)

Output that surprises you is the user's discovery — say nothing. A bug that would silently corrupt a result (wrong observable, shots too few for the error bar) is announced as *existing*, not located. The only permitted pre-emption is cost: a sweep that will take more than a few minutes gets one line with the estimate before it runs.

## Done test

The marker is `[done]` when: the requested artifact exists in the cell or library module, every sanity check above passed (or the failure was reported), the returned object is the one the spec named (matrices, circuit, DEM, stats, or plot — not a bundle of all of them), and no cell, interpretation, or next step was added that the user did not ask for.
