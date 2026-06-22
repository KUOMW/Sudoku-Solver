# 4x4 Quantum Sudoku Solver using Grover's Algorithm

This repository contains a quantum computing-based solution to solve a $4 \times 4$ Sudoku puzzle using **Grover's Search Algorithm** implemented via **Qiskit**. 

Given a partially filled $4 \times 4$ Sudoku grid, the quantum circuit explores the search space of unknown cells and amplifies the probability amplitude of the unique valid configuration.

---

## 📌 Project Overview

Traditional Sudoku puzzles require filling an $n^2 \times n^2$ grid such that every row, column, and subgrid block contains all numbers from $1$ to $n^2$ without repetition. 
This project focuses on a $4 \times 4$ grid ($n=2$), representing cell values using binary qubits.

### Puzzle Definition
- **Known Values:** Values ranging from $1$ to $4$ mapped across specific grid coordinates.
- **Unknown Cells:** Represented as quantum states that are solved using phase estimation and amplitude amplification (Grover's Diffusion).
- **Qubit Encoding:** Each missing cell value ($1$ to $4$) is encoded using 2 qubits:
  - `00` $\rightarrow 4$
  - `01` $\rightarrow 3$
  - `10` $\rightarrow 2$
  - `11` $\rightarrow 1$

---

## ⚙️ Circuit Architecture

The overall system architecture consists of three core components:

1. **Marker Oracle ($U$):** - Uses multi-controlled X ($MCX$) gates and custom value-matching constraints to check whether a given configuration satisfies the row, column, and $2 \times 2$ block constraints.
   - Flips the phase of the correct state.

2. **Diffuser Circuit ($D$):**
   - Applies the inversion-about-the-mean operation to amplify the probability amplitude of the valid solution state while diminishing incorrect states.

3. **Grover Iterations ($K$):**
   - Calculates the optimal number of iterations $K$ analytically using:
     $$\theta = 2 \arcsin\left(\frac{1}{\sqrt{N}}\right)$$
     $$K = \left\lfloor \frac{\pi}{2\theta} - \frac{1}{2} \right\rceil$$

---

### Prerequisites
Make sure you have Python installed alongside the following quantum computing packages:
```bash
pip install qiskit matplotlib numpy
```
--------------------------------

## 🧠 Core Methodology & Explanation

The algorithm solves the puzzle by manipulating quantum state amplitudes in a Hilbert space of dimension $N = 2^{N_{qubit} + N_{check}}$. It consists of three structural phases: **Initialization**, **The Oracle (Marker Circuit)**, and **The Diffuser**.

### 1. State Initialization
The circuit begins by preparing a uniform superposition of all possible assignments for the unknown cells. Applying a Hadamard gate ($H$) to all $N_{qubit}$ cell qubits creates an equal probability distribution across all possible states:

$$\lvert \psi_0 \rangle = H^{\otimes N_{qubit}} \lvert 0 \rangle^{\otimes N_{qubit}} = \frac{1}{\sqrt{N}} \sum_{x=0}^{N-1} \lvert x \rangle$$

### 2. The Oracle ($U_f$): Constraint Marking & Uncomputation
The Oracle behaves as a phase indicator. For a given state $\lvert x \rangle$, if the assignment satisfies all Sudoku conditions (row, column, and block uniqueness), the oracle flips its phase:

$$U_f \lvert x \rangle = (-1)^{f(x)} \lvert x \rangle \quad \text{where} \quad f(x) = 
\begin{cases} 
    1 & \text{if } x \text{ is the valid solution} \\
    0 & \text{otherwise} 
\end{cases}$$

#### Behind the Code: How `match_gate` and `complement_4_gate` Work
To achieve this without huge, unmanageable multi-controlled gates, the implementation breaks constraints down locally:

* **Local State Matching (`complement_4_gate`):**
  Sudoku constraints dictate what numbers must fill the remaining blanks. The `complement_4_gate(a)` determines what bit flips ($X$ gates) are required to change a specific target number $a$ into the binary state $\lvert 11 \rangle$.  
  e.g.: a = 1 (binary:01), then need to add "10" to make it to be "11"
* **Permutation Checking (`match_gate`):**
  The oracle loops through all valid permutations of the missing numbers. For each permutation, it temporarily transforms the qubits using `complement_4_gate`. If the current quantum state matches that specific valid permutation, the target qubits all become $\lvert 11...1 \rangle$.
* **Ancilla Multi-Controlled X ($MCX$):**
  When a row, column, or block constraint is perfectly satisfied, an auxiliary ancilla qubit is flipped to $\lvert 1 \rangle$. Once all individual ancillas are tripped, a final phase-kickback is triggered via an $MCX$ gate wrapped in Hadamard gates on the primary flag qubit:
  $$\lvert - \rangle = \frac{\lvert 0 \rangle - \lvert 1 \rangle}{\sqrt{2}}$$
* **Uncomputation (Crucial Step):**
  To prevent the ancilla qubits from remaining entangled with our cell registers (which would cause decoherence and ruin the interference pattern), the script runs the exact inverse of the matching gates (`.inverse()`). This safely resets all helper ancillas back to $\lvert 0 \rangle$, leaving only the modified phase on the cell qubits.

### 3. The Diffuser ($D$): Inversion About the Mean
The diffuser applies a reflection operator around the uniform superposition state $\lvert \psi_0 \rangle$. Geometrically, this amplifies states with a negative phase (the solution marked by the oracle) and attenuates states with a positive phase (the incorrect paths):

$$D = 2\lvert \psi_0 \rangle \langle \psi_0 \vert - I = H^{\otimes N_{qubit}} (2\lvert 0 \rangle \langle 0 \vert - I) H^{\otimes N_{qubit}}$$

In circuit architecture, this is achieved by sandwiching a multi-controlled $Z$ operation (or an $MCX$ with $H$ gates on the target qubit) between layers of $X$ and $H$ gates across the entire cell register.

### 4. Calculating the Optimal Iterations ($K$)
Because Grover's algorithm rotates the state vector in a 2D subspace toward the target solution by an angle of $\theta$ per iteration, over-rotating will decrease the success probability. The geometric step size is defined by:

$$\theta = 2 \cdot \arcsin\left(\frac{1}{\sqrt{N}}\right)$$

The exact number of times the Grover loop ($U_f$ followed by $D$) must be repeated to maximize the probability amplitude of the correct state is:

$$K = \text{round}\left(\frac{\pi}{2\theta} - \frac{1}{2}\right)$$

For $4$ unknown cells, we have $N_{qubit} = 8$, meaning $N = 2^8 = 256$ possible computational states.

$$\theta = 2 \cdot \arcsin\left(\frac{1}{\sqrt{256}}\right) = 2 \cdot \arcsin(0.0625) \approx 0.1251 \text{ rad}$$
$$K = \text{round}\left(\frac{\pi}{2(0.1251)} - 0.5\right) = \text{round}(12.55 - 0.5) = 12 \text{ iterations}$$

Executing the loop exactly $K = 12$ times yields the final 99.99% conversion accuracy captured during the statevector simulation.

---

## 💻 Code Structure

The program script contains the following execution modules:

* **Grid Parser:** Evaluates the input string into a 2D NumPy array and identifies zero-indices.
* **Constraint Mapper:** Computes existing row, column, and block values to dynamically structure the matching subsets.
* **Gate Synthesis:** Generates logical sub-circuits (`match_gate`, `complement_4_gate`) to handle binary validation.
* **Grover Assembly:** Links the state initialization, Oracle ($U$), and Diffuser ($D$) into a unified Qiskit `QuantumCircuit`.
* **Statevector Simulation & Post-Processing:** Evaluates the quantum state using `Statevector`, pulls the `argmax` state, reverses the qubit string to match standard binary ordering, and prints the solution.
