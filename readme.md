# Qiskit Fall Fest 2026 - Basic Track
**Author**: Abhimanyu Bishnoi
**Environment**: Google Colab

## overview
this project builds a verified bell pair between two fixed ports across three processor models (2016, contemporary, hyperbolic)[cite: 1, 2]. the setup establishes ideal and noisy baselines, then adds a zz parity check to see if error detection is worth the routing cost[cite: 4]. the entire workflow and benchmarking was run in google colab.

## methodology & optimization
used the explorer-level zz parity check[cite: 4]. since the ideal $|\Phi^+\rangle$ state has a zz parity of +1, an ancilla qubit measures zz[cite: 4]. if it returns -1, the shot is dropped (post-selection)[cite: 4].

to avoid memory crashes on the 156-qubit contemporary processor during transpilation, i optimized the ancilla placement. using rustworkx, i pulled the distance matrix and explicitly assigned the ancilla to the physical qubit with the shortest total distance to both ports to minimize swap routing.

## 9-case comparison

| Model | Case | Yield (%) | Fidelity | Verified | Max 2-qubit gates | Max explicit SWAPs |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 2016 | Ideal | 100.0 | 1.0000 | True | 7 | 0 |
| 2016 | Noisy | 100.0 | 0.8469 | True | 7 | 0 |
| 2016 | Protected | 90.5 | 0.8295 | True | 12 | 0 |
| contemporary | Ideal | 100.0 | 1.0000 | True | 7 | 0 |
| contemporary | Noisy | 100.0 | 0.9490 | True | 7 | 0 |
| contemporary | Protected | 96.6 | 0.9570 | True | 12 | 0 |
| hyperbolic | Ideal | 100.0 | 1.0000 | True | 7 | 0 |
| hyperbolic | Noisy | 100.0 | 0.9572 | True | 7 | 0 |
| hyperbolic | Protected | 95.8 | 0.9650 | True | 12 | 0 |

## interpreting results

**1. swap decomposition & routing**
explicit swaps show as 0, but the routing cost is heavily reflected in the 2-qubit gates[cite: 4]. the logical circuit only needs 1 cnot[cite: 2]. since the ports are distance 3 apart, the transpiler adds 2 swaps to move the states[cite: 1, 2]. swaps aren't native, so qiskit decomposes each into 3 cnots (1 logical + 6 routing = 7 total 2-qubit gates)[cite: 1]. routing the parity check ancilla bumps this to 12.

**2. protection overhead vs benefit**
*   **2016 model:** protection failed. fidelity dropped from 0.8469 to 0.8295. the older hardware has high noise, so adding routing gates caused more errors than the parity check could catch[cite: 1, 4].
*   **contemporary & hyperbolic:** protection worked. fidelity improved because the base noise is low enough to handle the deeper circuit. discarding ~4% of shots was a good trade-off for higher fidelity[cite: 2, 4].

**3. architectural advantage**
the hyperbolic model got the best fidelity in both noisy (0.9572) and protected (0.9650) cases. the lattice geometry gives better cycle pathways for routing compared to standard grids[cite: 1].

## future system recommendation
future cloud quantum compilers need to dynamically toggle error-detection checks based on the specific hardware topology and current noise profile. blindly applying protection hurts performance on older or constrained architectures. transpilers should calculate a cost-benefit threshold and only insert ancilla checks if the physical qubits can survive the required swap routing[cite: 4, 5].
