# Magic State Cultivation with NVIDIA CUDA-Q

![](./banner.png)

A simplified educational implementation inspired by Google's Quantum AI paper:

> **Magic State Cultivation on a Superconducting Quantum Processor**  
> Google Quantum AI (Willow Processor)

This project demonstrates the core ideas behind **magic state cultivation**, a recently proposed approach for producing high-fidelity magic states required for fault-tolerant quantum computing.

The implementation uses **NVIDIA CUDA-Q** and provides a compact simulation of:

- Raw magic-state injection
- Syndrome-based error detection
- Heralded filtering (post-selection)
- Magic-state consumption via gate teleportation
- Basic physical noise modeling

Although simplified, the example captures the key principles behind the cultivation protocol and illustrates how logical error rates can be reduced through selective acceptance of high-quality quantum states.

---

# Why Magic States Matter

Most fault-tolerant quantum error correction schemes can efficiently implement **Clifford gates**, such as:

- Hadamard (H)
- Phase (S)
- CNOT

However, Clifford gates alone are not sufficient for universal quantum computation.

To achieve universality, at least one **non-Clifford gate** is required, typically the **T gate**:

\[
T =
\begin{bmatrix}
1 & 0 \\
0 & e^{i\pi/4}
\end{bmatrix}
\]

Directly implementing fault-tolerant T gates is difficult and expensive.

Instead, modern quantum architectures prepare special resource states called **magic states**, which are later consumed through teleportation-based circuits to realize logical T operations.

---

# The Traditional Approach: Magic State Distillation

For decades, the standard solution has been **Magic State Distillation**.

The idea is straightforward:

```text
Many noisy magic states
          ↓
Distillation protocol
          ↓
One higher-quality magic state
```

While effective, distillation requires:

- Large numbers of physical qubits
- Multiple rounds of error correction
- Significant computational overhead

For large-scale quantum computers, magic-state production is expected to consume a substantial fraction of available hardware resources.

---

# Magic State Cultivation

Google Quantum AI introduced an alternative strategy known as:

## Magic State Cultivation

Instead of repeatedly distilling large batches of states, the processor attempts to **increase the quality of a single injected state** through error detection and selective survival.

Conceptually:

```text
Raw injected state
         ↓
Error-detection measurements
         ↓
Heralded filtering
         ↓
Accepted high-fidelity state
```

Rather than correcting every possible error, the system simply discards states that show evidence of faults.

This creates a powerful filtering mechanism:

```text
Bad states  → Rejected
Good states → Accepted
```

The result is a lower logical error rate at the expense of reduced yield.

Only a fraction of attempts survive, but their quality is significantly higher.

---

# Theoretical Background

## T-Magic State

A T-type magic state can be written as:

\[
|T\rangle =
\frac{|0\rangle + e^{i\pi/4}|1\rangle}{\sqrt{2}}
\]

or equivalently:

\[
|T\rangle =
\cos(\pi/8)|0\rangle +
\sin(\pi/8)|1\rangle
\]

These states lie outside the stabilizer-state manifold and therefore provide the non-Clifford resource necessary for universal quantum computation.

---

## Syndrome Measurements

A key idea from quantum error correction is measuring **stabilizers**.

Instead of measuring the quantum information directly, auxiliary observables are measured whose outcomes reveal whether an error has likely occurred.

For example, the parity operator:

\[
Z_1 Z_2
\]

checks whether two qubits have matching computational-basis parity.

Possible outcomes:

```text
+1 → No detected error
−1 → Detected error
```

The encoded quantum information itself remains protected.

---

## Heralded Filtering

The cultivation protocol relies heavily on **heralded success**.

An ancilla qubit performs stabilizer measurements:

```text
Ancilla = 0 → Accept
Ancilla = 1 → Reject
```

This process does not actively correct errors.

Instead, it identifies the experimental runs that are most likely error-free.

The surviving states consequently exhibit significantly improved fidelity.

---

## Gate Teleportation

Once a high-quality magic state is available, it can be consumed to implement a logical T gate.

A simplified teleportation circuit is:

```text
|ψ⟩ ─────●───────────────
          │
|T⟩ ─────X── Mz ──► S† correction
```

The magic state is destroyed during the process.

After conditional correction, the effect is equivalent to applying:

\[
T|\psi\rangle
\]

to the data qubit.

This technique is used in essentially all major fault-tolerant quantum computing architectures.

---

# Project Structure

## 1. Raw T-State Injection

The first kernel prepares a physical magic state:

```python
h(q[0])
t(q[0])
```

The Hadamard gate creates:

\[
|+\rangle
\]

and the T gate introduces the required phase.

---

## 2. Noise Injection

A simple physical noise source is simulated using:

```python
rz(noise_perturbation, t_qubit)
```

This represents:

- Phase miscalibration
- Decoherence effects
- Control imperfections
- Frequency drift

---

## 3. Mini Error-Correcting Patch

The state is encoded into a small entangled structure:

```python
cx(t_qubit, code_qubit)
```

which provides redundancy that can be checked through parity measurements.

---

## 4. Syndrome Extraction

An ancilla qubit gathers parity information:

```python
cx(t_qubit, ancilla)
cx(code_qubit, ancilla)
```

effectively measuring:

\[
Z_1 Z_2
\]

without destroying the encoded state.

---

## 5. Heralded Acceptance Filter

The ancilla is measured:

```python
mz(ancilla)
```

Decision rule:

```text
0 → State accepted
1 → State rejected
```

This emulates the acceptance/rejection logic used in magic-state cultivation.

---

## 6. Magic Gate Teleportation

The cultivated magic state is consumed to apply a logical T operation:

```python
cx(data, magic_t)

meas_magic = mz(magic_t)

if meas_magic:
    rz(-np.pi / 2.0, data)
```

The measurement outcome determines whether a corrective phase operation must be applied.

---

# Relationship to the Willow Experiment

This code is **not a reproduction** of Google's full experiment.

The actual Willow implementation involved:

- Superconducting qubits
- Logical qubits protected by quantum error correction
- Small logical color-code patches
- Multiple rounds of stabilizer measurements
- State transfer into larger fault-tolerant codes

This CUDA-Q example focuses on the conceptual workflow:

✅ Magic-state injection

✅ Physical noise

✅ Syndrome extraction

✅ Heralded filtering

✅ State acceptance/rejection

✅ Gate teleportation

✅ Non-Clifford resource consumption

while remaining simple enough to run entirely on a classical simulator.

---

# Noise Model

The current implementation uses a simple phase-noise approximation:

```python
rz(noise_perturbation)
```

Possible future extensions include:

- Depolarizing Noise
- Amplitude Damping
- Phase Damping
- CUDA-Q `NoiseModel`
- Surface-code logical noise simulations

---

# Installation

```bash
pip install cuda-quantum
```

---

# Running the Example

```bash
python magic_state_cultivation.py
```

---

# Example Output

```text
======================================================================
CUDA-Q: MAGIC STATE CULTIVATION SIMULATION
======================================================================

[1] Raw T-State Injection:
    0: 85.4%
    1: 14.6%

[2] Heralded Cultivation:
    Accepted states: 412
    Rejected states: 4588
    Yield: 8.24%

[3] Gate Teleportation:
    {'0': 503, '1': 497}
```

---

# Educational Goals

This example is intended to help users understand:

- Why magic states are required for universal quantum computing
- Why non-Clifford gates are expensive
- How syndrome measurements detect errors
- What heralded filtering means
- How gate teleportation consumes magic states
- Why large-scale fault-tolerant quantum computing depends heavily on efficient magic-state production

In short, this project provides a compact introduction to one of the most important bottlenecks, and recent breakthroughs, in scalable quantum computing.

---

# References

1. Rosenfeld et al., *Magic State Cultivation on a Superconducting Quantum Processor*.

2. Google Quantum AI, *Bringing the Magic: Breaking a Bottleneck in Quantum Computing*.

3. Gidney, Shutty, Jones, *Magic State Cultivation: Growing T States as Cheap as CNOT Gates*.

4. Nielsen & Chuang, *Quantum Computation and Quantum Information*.

5. Campbell, Terhal & Vuillot, *Roads Towards Fault-Tolerant Universal Quantum Computation*.