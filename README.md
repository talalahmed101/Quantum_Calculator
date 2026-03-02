# Quantum Calculator
This repository contains a quantum algorithm for modular addition and modular multiplication for two integers $x$ and $y$. The binary representation of $x$ is $x_{0}x_{1}...x_{d-1}$, containing $d$ bits. Similarly $y$ also has $d$ bits in its binary representation. Therefore, $x$ is represented in computational basis as

$$
|x\rangle_{d} := |x_{0}\rangle\,|x_{1}\rangle...|x_{d-1}\rangle
$$

where the subscript $d$ indicates that $x$ is expressed using $d$ qubits.

We present a qiskit function that takes one input: a positive integer and outputs a quantum circuit, QCalc, on $3d+1$ qubits and two additional ancilla qubits that inplements the following:

$$
\text{QCalc} |z\rangle_{1}|y\rangle_{d}|x\rangle_{d}|0\rangle_{d} = 
\begin{cases} 
|z\rangle_{1}|y\rangle_{d}|x\rangle_{d}|x+y \pmod{2^{d}}\rangle_{d}, & \text{if } z = 0 \\ 
|z\rangle_{1}|y\rangle_{d}|x\rangle_{d}|x \cdot y \pmod{2^{d}}\rangle_{d}, & \text{if } z = 1 
\end{cases}
$$

# Methodology

First of all, we use least significant bit (LSB) convention for the binary representations of the integers,

$$
x = \sum_{j=0}^{d-1}x_{j}2^{j}
$$

where $x_{j}\in \{0,1\}$.

