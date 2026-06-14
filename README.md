# Hybrid Quantum–Classical Cyber-Defense Optimization

**Optimal Control in Cyber Defense: LQR-to-QUBO Formulation and IBM Quantum Validation**

A hybrid quantum–classical framework that formulates cyber defense as a finite-horizon state-space control problem, transforms the Linear Quadratic Regulator (LQR) optimal control objective exactly into a Quadratic Unconstrained Binary Optimization (QUBO) problem, and solves it using the Quantum Approximate Optimization Algorithm (QAOA) on both simulated and real IBM quantum hardware.

## Overview

Modern cyber defense is not a one-shot classification task — a network changes continuously as adversaries gather information, exploit vulnerable assets, and move across connected systems. This project models cyber defense as a control problem:

- **State variables** per node: compromise level, attacker knowledge, vulnerability exposure, and asset criticality
- **Control actions**: binary defense decisions (e.g., patch or do not patch)
- **Attack disturbance**: adversary pressure modeled as an external signal

A finite-horizon LQR-style objective is reduced **exactly** to a QUBO, and solved by multiple optimization methods including QAOA on real IBM quantum backends.

## Repository Structure

```
.
├── .gitignore                  # Git ignore rules
├── .env                        # Environment variables (IBM token, etc.)
├── README.md                   # This file
│
├── 📓 Jupyter Notebooks (Core Pipeline)
│   ├── Unified_Control_QUBO_VQAOA_IBM_corrected.ipynb
│   │   # Main pipeline: LQR→QUBO + CP-SAT + SA + QAOA + IBM Quantum
│   ├── 8_nodes_QUBO.ipynb
│   │   # Full local QAOA simulation on 8-node networks
│   └── LQR_QUBO_QAOA_IBM_panels.ipynb
│       # Two-panel experimental design (validation + scalability)
│
├── 📄 Paper (Springer LNCS)
│   ├── springer_cyber_qaoa_updated.tex  # LaTeX manuscript source
│   ├── springer_cyber_qaoa_updated.pdf  # Compiled manuscript
│   └── Optimal Control in Cyber Defense.pdf  # Additional reference
│
├── 📊 Experimental Results
│   ├── unified_control_qubo_qaoa_results.csv  # Aggregated all methods/sizes
│   ├── panel1_validation.csv                  # Panel 1 raw results (16 qubits)
│   ├── panel2_scalability.csv                 # Panel 2 raw results (64–152 qubits)
│   │
│   ├── fig_closed_loop_cost.png               # Closed-loop cost bar chart
│   ├── fig_qubo_energy.png                    # QUBO energy bar chart
│   ├── fig_runtime.png                        # Runtime bar chart
│   ├── panel1_validation.png                  # Panel 1 figure
│   └── panel2_scalability.png                 # Panel 2 figure
│
└── 🐍 Python Environment
    └── .venv/                                 # Virtual environment
```

## Methodology

### State-Space Cyber-Defense Model

Each of the $n$ network nodes is characterized by a 4-component state:

- **$s_i$**: Compromise level of node $i$
- **$\theta_i$**: Attacker knowledge about node $i$
- **$v_i$**: Vulnerability exposure of node $i$
- **$c_i$**: Asset criticality of node $i$

The global state vector $x_k \in \mathbb{R}^{4n}$ evolves as:

$$x_{k+1} = A x_k + B u_k + F w_k$$

where $u_k \in \{0,1\}^n$ is the binary defense action and $w_k$ is the attack disturbance. The network topology uses a Barabási–Albert scale-free graph to introduce heterogeneous connectivity.

### LQR → QUBO Transformation

For a horizon $H$, the defender optimizes binary actions $U \in \{0,1\}^{nH}$ to minimize:

$$J(U) = \sum_{k=0}^{H-1} \left(x_k^\top Q x_k + u_k^\top R u_k\right) + x_H^\top Q x_H$$

This is transformed **exactly** (verified numerically) into a QUBO:

$$\min_{U \in \{0,1\}^q} U^\top H_q U + g_q^\top U$$

The QUBO is then mapped to an Ising Hamiltonian for QAOA execution.

### Optimization Methods

| Method | Description |
|--------|-------------|
| **CP-SAT** | Strong classical reference (OR-Tools); solves linearized QUBO with auxiliary variables |
| **Simulated Annealing** | Dimod SA sampler baseline |
| **Greedy** | Simple diagonal-only baseline |
| **Brute Force** | Exact optimum for small instances (≤22 variables) |
| **Local QAOA** | Variational QAOA with COBYLA optimization on AerSimulator; angles are optimized, not hard-coded |
| **IBM QAOA** | Transferred optimized angles executed on real IBM quantum hardware (ibm_marrakesh) |

### Key Experimental Design

- **Panel 1 (Validation)**: 16-qubit instance ($n=4, H=4$). All methods solve the same full QUBO with no coupling pruning.
- **Panel 2 (Scalability)**: 64, 128, and 152 qubit instances ($n=8,16,19; H=8$). IBM QAOA uses coupling pruning for hardware feasibility; bitstrings are scored on the full QUBO.

## Results

### Validation Panel (16 qubits)

| Method | QUBO Energy | Gap to CP-SAT (%) |
|--------|------------|------------------|
| Greedy | 152.804 | 210.676 |
| CP-SAT | –138.064 | 0.000 (reference) |
| Local QAOA (p=2) | –136.749 | 0.952 |
| IBM QAOA (p=2) | –136.780 | 0.930 |

### Scalability Panel

| Qubits | Method | QUBO Energy | Gap (%) | Retained Couplings |
|--------|--------|------------|---------|-------------------|
| 64 | CP-SAT | –3602.902 | 0.000 | 100% |
| 64 | IBM QAOA | –3514.255 | 2.460 | 60.09% |
| 128 | CP-SAT | –6998.862 | 0.000 | 100% |
| 128 | IBM QAOA | –6560.138 | 6.269 | 43.45% |
| 152 | CP-SAT | –8183.629 | 0.000 | 100% |
| 152 | IBM QAOA | –7641.636 | 6.623 | 46.43% |

**Key observation**: IBM QAOA remains within ~1% of CP-SAT on the validation instance and within ~7% on larger hardware-scale instances up to 152 qubits. This demonstrates that a control-derived cyber-defense QUBO can be executed on real IBM quantum hardware while preserving solution quality close to a strong classical benchmark.

## Scope & Limitations

This work **does not claim quantum advantage**. The supported claims are:

1. An LQR-derived cyber-defense QUBO can be formulated and executed on current IBM quantum hardware.
2. QAOA solutions are comparable to a strong classical reference (CP-SAT) on validation instances.
3. Solution quality remains within a moderate gap on larger hardware-scale instances.

Key limitations:
- QAOA restricted to $p = 1$–$2$ for hardware feasibility
- Large-scale IBM runs use coupling truncation
- Finite time limit for CP-SAT on large instances
- Binary defense actions only (no continuous intensities)

## Dependencies

- Python 3.x
- NumPy, SciPy, Pandas, Matplotlib
- NetworkX
- Qiskit, Qiskit-Aer, Qiskit-IBM-Runtime
- D-Wave dimod (Simulated Annealing)
- Google OR-Tools (CP-SAT)
- IBM Quantum account (for hardware execution)

## Usage

The main pipeline is in `Unified_Control_QUBO_VQAOA_IBM_corrected.ipynb`. To run:

1. Install dependencies: `pip install numpy pandas matplotlib scipy networkx dimod ortools qiskit qiskit-aer qiskit-ibm-runtime`
2. Set environment variables (for IBM hardware):
   - `IBM_QUANTUM_TOKEN` — your IBM Quantum API token
   - `IBM_QUANTUM_INSTANCE` — (optional) IBM Cloud instance
3. Set `USE_IBM = True` in the configuration cell to run on hardware
4. Execute cells sequentially

## Citation

If you use this work, please cite the associated paper:

```
@inproceedings{hybrid_quantum_cyber_defense,
  title={A Hybrid Quantum--Classical State-Space Framework for Cyber-Defense Optimization: 
         LQR-to-QUBO Formulation and IBM Quantum Validation},
  author={Author(s)},
  booktitle={Springer Lecture Notes in Computer Science},
  year={2026}
}
```

## License

See `LICENSE` file (if applicable). Academic use is encouraged; please cite accordingly.