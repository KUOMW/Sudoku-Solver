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

## 🚀 Setup & Execution

### Prerequisites
Make sure you have Python installed alongside the following quantum computing packages:
```bash
pip install qiskit matplotlib numpy
