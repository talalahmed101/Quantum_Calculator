# Quantum Calculator

This repository contains a quantum algorithm for modular addition and multiplication of two integers $x$ and $y$. The binary representation of $x$ is $x_{0}x_{1}...x_{d-1}$, which consists of $d$ bits. Similarly, $ y $ has a binary representation consisting of $d$ bits. Therefore, $x$ is expressed in the computational basis as follows:

$$
|x\rangle_{d} := |x_{0}\rangle\,|x_{1}\rangle...|x_{d-1}\rangle
$$

where the subscript $d$ indicates that $x$ is represented using $d$ qubits.

We present a Qiskit function, QCalc, that takes one input, a positive integer, and outputs a quantum circuit that operates on $3d+1$ qubits and two additional ancilla qubits. This function implements the following transformation:

$$
\text{QCalc} |z\rangle_{1}|y\rangle_{d}|x\rangle_{d}|0\rangle_{d} = 
\begin{cases} 
|z\rangle_{1}|y\rangle_{d}|x\rangle_{d}|x+y \pmod{2^{d}}\rangle_{d}, & \text{if } z = 0 \\ 
|z\rangle_{1}|y\rangle_{d}|x\rangle_{d}|x \cdot y \pmod{2^{d}}\rangle_{d}, & \text{if } z = 1 
\end{cases}
$$

# Methodology

We use the least significant bit (LSB) convention for the binary representations of the integers. We assign qubit registers $X[j]$, $Y[i]$, and  $P[k]$ for $x$, $y$, and the results of $x+y$ (or $x\cdot y$), respectively. In this system, $X[0]$, $Y[0]$, and $P[0]$ represent the LSBs of $x$, \( y \), and \( x+y \) (or \( x \cdot y \), respectively). Furthermore, \( x \) is expressed as follows:

$$
x = \sum_{j=0}^{d-1}x_{j}2^{j}
$$

where \( x_{j} \in \{0, 1\} \). Thus, \( x_{j} \) is associated with the \( X[j] \) qubit, and similarly, \( y_{i} \) is associated with the \( Y[i] \) qubit in their respective registers. Now, let us outline the quantum addition of two integers, \( x \) and \( y \), modulo \( 2^{d} \).

**Step 1**: We transform \( |0\rangle_{P} \) to its Fourier basis:

$$
|0\rangle_{P}\mapsto |\phi(0)\rangle_{P} = \frac{1}{2^{d/2}}\sum_{k=0}^{2^{d}-1}|k\rangle
$$

**Step 2**: We add \( x \) into \( P \) using controlled-phase rotations as follows:

$$
|x\rangle_{X}|\phi(0)\rangle_{P}\mapsto |x\rangle_{X}|\phi(x)\rangle_{P}
$$

This result is achieved by applying the phase gate \( R_{d-(j+k)} \) on the \( P[k] \) qubit, where \( j=0,1,...,d-1 \) and \( k=0,1,...,d-1 \) such that \( j+k<d \), controlled by the \( X[j] \) qubit. The phase gate \( R_{m} \) is defined as:

$$
R_{m}=\begin{pmatrix}
1 & 0\\
0 & e^{2\pi i/2^{m}}
\end{pmatrix}
$$

**Step 3**: We then add \( y \) into \( P \). The result is:

$$
|y\rangle_{Y}|x\rangle_{X}|0\rangle_{P}\mapsto |y\rangle_{Y} |x\rangle_{X} |\phi(x+y\mod 2^{d})\rangle_{P}
$$

This result is obtained by applying the phase gate \( R_{d-(j+k)} \) on the \( P[k] \) qubit, controlled by the \( Y[j] \) qubit, where \( j+k < d \).

**Step 4**: Finally, by applying the inverse Quantum Fourier Transform (IQFT) on the \( P \) qubit register, we obtain:

$$
|\phi(x+y \mod 2^{d})\rangle_{P}\mapsto |x+y \mod 2^{d}\rangle
$$
