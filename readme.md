# Qiskit Fall Fest 2026: Geometry-Aware Quantum Cloud Challenge
**Author:** Abhimanyu Bishnoi  
**Environment:** Google Colab  

## Overview
This project establishes a verified Bell pair between two fixed port qubits across three processor models: a 2016-inspired 5-qubit graph, a 156-qubit Contemporary IBM-style backend, and a 35-qubit Hyperbolic-inspired lattice[cite: 1, 2]. The primary objective is to measure ideal and noisy baselines, then implement a justified error-detection strategy to evaluate whether the protection benefits outweigh the extra routing costs[cite: 1, 4]. The entire workflow, from circuit compilation to noisy simulation benchmarking, was executed in Google Colab.

## Methodology & Optimization
I implemented an **Explorer-level ZZ Parity Check**[cite: 4]. Because the target $\vert{}\Phi^+\rangle$ state inherently has a ZZ parity of +1, an extra ancilla qubit is used to measure the ZZ observable during the circuit[cite: 4]. If the check yields a -1 (indicating an error), the shot is discarded via post-selection[cite: 4]. 

During development, the transpiler encountered memory crashes on the 156-qubit contemporary processor due to suboptimal ancilla placement. To resolve this, I utilized `rustworkx` to generate a shortest-path distance matrix for the hardware's coupling graph. By dynamically assigning the ancilla to the physical qubit with the absolute minimum combined distance to both ports, I successfully optimized the routing pathways and stabilized the compilation.

## 9-Case Comparison Results

| Model | Case | Yield (%) | Fidelity | Verified | Max 2-qubit gates | Max explicit SWAPs |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **2016** | Ideal | 100.0 | 1.0000 | True | 7 | 0 |
| **2016** | Noisy | 100.0 | 0.8469 | True | 7 | 0 |
| **2016** | Protected | 90.5 | 0.8295 | True | 12 | 0 |
| **Contemporary** | Ideal | 100.0 | 1.0000 | True | 7 | 0 |
| **Contemporary** | Noisy | 100.0 | 0.9490 | True | 7 | 0 |
| **Contemporary** | Protected | 96.6 | 0.9570 | True | 12 | 0 |
| **Hyperbolic** | Ideal | 100.0 | 1.0000 | True | 7 | 0 |
| **Hyperbolic** | Noisy | 100.0 | 0.9572 | True | 7 | 0 |
| **Hyperbolic** | Protected | 95.8 | 0.9650 | True | 12 | 0 |

*(Data derived from 10,000 shots per setting[cite: 8, 9])*

## Interpretation of Results

**1. SWAP Decomposition & Routing Overhead**
Explicit SWAP operations are listed as 0, but the routing overhead is highly visible in the two-qubit gate counts. The logical Bell circuit requires exactly 1 CNOT[cite: 2]. Because the designated ports are separated by a distance of 3[cite: 8, 9], the transpiler inserted 2 SWAPs to connect the states. Since SWAP is not a native basis gate for these models, Qiskit decomposed each SWAP into 3 CNOTs[cite: 1]. This results in 7 total two-qubit gates for the baselines (1 logical + 6 routing). Adding the parity check ancilla increased the physical footprint to 12 two-qubit gates[cite: 9].

**2. Protection Overhead vs. Benefit**
* **2016 Model:** The protection strategy failed. Fidelity dropped from 0.8469 to 0.8295[cite: 9]. Because the historical architecture relies on a high-error synthetic noise model[cite: 1], the sheer volume of extra routing operations introduced more errors than the parity check was capable of detecting.
* **Contemporary & Hyperbolic Models:** The protection strategy succeeded. The base hardware noise is low enough that the circuits survived the increased depth of 25[cite: 1, 9]. Discarding roughly 4% of the total shots was a highly effective trade-off for the improved conditional fidelity[cite: 9].

**3. Architectural Advantage**
The Hyperbolic-inspired model achieved the highest overall fidelity in both the noisy baseline (0.9572) and the protected case (0.9650)[cite: 9]. Its coupling graph, derived from a finite hyperbolic lattice, provides alternative connection patterns that offer inherently more efficient routing cycle pathways compared to standard grids[cite: 1].

## Future System Recommendation
Future cloud quantum compilers should dynamically toggle error-detection checks based on the hardware's specific geometry and real-time noise profile. 

Applying error protection blindly actively degrades performance on older or highly constrained architectures. Qiskit transpilers must be equipped to calculate a "cost-benefit threshold"—only inserting syndrome or parity checks if the physical qubits' error rates are low enough to mathematically justify the required SWAP routing overhead[cite: 4, 5].
