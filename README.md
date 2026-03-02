# Quantum Calculator

This repository contains a quantum algorithm for modular addition and multiplication of two integers $x$ and $y$. The binary representation of $x$ is $x_{0}x_{1}...x_{d-1}$, which consists of $d$ bits. Similarly, $y$ has a binary representation consisting of $d$ bits. Therefore, $x$ is expressed in the computational basis as follows:

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

We use the least significant bit (LSB) convention for the binary representations of the integers. We assign qubit registers $X[j]$, $Y[i]$, and $P[k]$ for $x$, $y$, and the results of $x+y$ (or $x\cdot y$), respectively. In this system, $X[0]$, $Y[0]$, and $P[0]$ represent the LSBs of $x$, $y$, and $x+y$ (or $x \cdot y$, respectively). Furthermore, $x$ is expressed as follows:

$$
x = \sum_{j=0}^{d-1}x_{j}2^{j}
$$

where $x_{j} \in \{0, 1\}$. Thus, $x_{j}$ is associated with the $X[j]$ qubit, and similarly, $y_{i}$ is associated with the $Y[i]$ qubit in their respective registers. Now, let us outline the quantum addition of two integers, $x$ and $y$, modulo $2^{d}$.

### Quantum Addition:
**Step 1**: We transform $|0\rangle_{P}$ to its Fourier basis:

$$
|0\rangle_{P}\mapsto |\phi(0)\rangle_{P} = \frac{1}{2^{d/2}}\sum_{k=0}^{2^{d}-1}|k\rangle
$$



**Step 2**: We add $x$ into $P$ using controlled-phase rotations as follows:

$$
|x\rangle_{X}|\phi(0)\rangle_{P}\mapsto |x\rangle_{X}|\phi(x)\rangle_{P}
$$

This result is achieved by applying the phase gate $R_{d-(j+k)}$ on the $P[k]$ qubit, where $j=0,1,...,d-1$ and $k=0,1,...,d-1$ such that $j+k<d$, controlled by the $X[j]$ qubit. The phase gate $R_{m}$ is defined as:

$$
R_{m}=\begin{pmatrix}
1 & 0\\
0 & e^{2\pi i/2^{m}}
\end{pmatrix}
$$

**Step 3**: We then add $y$ into $P$. The result is:

$$
|y\rangle_{Y}|x\rangle_{X}|0\rangle_{P}\mapsto |y\rangle_{Y} |x\rangle_{X} |\phi(x+y\mod 2^{d})\rangle_{P}
$$

This result is obtained by applying the phase gate $R_{d-(j+k)}$ on the $P[k]$ qubit, controlled by the $Y[j]$ qubit, where $j+k < d$. As such, for $j=0,1,..,d-1$, the index $k$ will have the range, $k=0,1,..,d-j-1$.

**Step 4**: Finally, by applying the inverse Quantum Fourier Transform (IQFT) on the $P$ qubit register, we obtain:

$$
|\phi(x+y \mod 2^{d})\rangle_{P}\mapsto |x+y \mod 2^{d}\rangle
$$

### Quantum Multiplication:

**Step 1**: Here the goal is to obtain

$$
|y\rangle_{Y}|x\rangle_{X}|0\rangle_{P}\mapsto |y\rangle_{Y}|x\rangle_{X}|x.y \mod 2^{d}\rangle
$$

So we again initiate $|0\rangle_{P}$ in the Fourier basis $|0\rangle_{P}\mapsto |\phi(0)\rangle_{P}$.

**Step 2**: We apply the phase gate $R_{d-(i+j+k)}$ on the $P[k]$ qubit, controlled by $Y[i]$ and $X[j]$ qubits where $i+j+k<d$. As such, for the index of $Y[i]$ qubit register, $i=0,1,..,d-1$, the indices of $X[j]$ qubit and $P[k]$ qubit registers will have the range, $j=0,1,..,d-j-1$ and $k=0,1,..,d-i-j-1$, respectively. The result is

$$
|y\rangle_{Y}|x\rangle_{X}|0\rangle_{P}\mapsto |y\rangle_{Y}|x\rangle_{X}|\phi(x.y \mod 2^{d})\rangle
$$

**Step 3**: Finally by applying IQFT on $P[k]$ qubits, we get the desired result,

$$
|y\rangle_{Y}|x\rangle_{X}|0\rangle_{P}\mapsto |y\rangle_{Y}|x\rangle_{X}|x.y \mod 2^{d}\rangle
$$

# Quantum Circuit Construction and Qiskit Implementation

The qiskit function **QCalc**, that takes one input, a positive integer, and outputs a quantum circuit that operates on $3d+1$ qubits and two additional ancilla qubits. This function implements the following transformation:

$$
\text{QCalc} |z\rangle_{1}|y\rangle_{d}|x\rangle_{d}|0\rangle_{d} = 
\begin{cases} 
|z\rangle_{1}|y\rangle_{d}|x\rangle_{d}|x+y \pmod{2^{d}}\rangle_{d}, & \text{if } z = 0 \\ 
|z\rangle_{1}|y\rangle_{d}|x\rangle_{d}|x \cdot y \pmod{2^{d}}\rangle_{d}, & \text{if } z = 1 
\end{cases}
$$

Therefore the output, whether it should be quantum addition or multiplication, is controlled by $z$ qubit. We integrated this control qubit by introducing two ancilla qubits, $a_{1}$ and $a_{2}$ in the circuit construction. In the case of quantum addition, the phase gate $R_{d-(j+k)}$ on the $P[k]$ qubit, controlled by either $X[j]$ or $Y[j]$ qubit is modified to include a Toffoli gate with control qubits $z$ and $X[j]$ and target qubit $a_{2}$ ancilla, denoting as $\text{Toffoli}(z, X[j], a_{2})$ Then the $a_{2}$ ancilla acts as the control qubit for the controlled-phase gate $R_{d-(j+k)}$ acting on the $P[k]$ qubit. Similar construction also holds for $Y[j]$ qubits. Besides, after applying the controlled-phase gates, we uncompute the ancilla $a_{2}$ by applying the $\text{Toffoli}(z, X[j], a_{2})$ or $\text{Toffoli}(z, Y[j], a_{2})$ again.

On the other hand, for quantum multiplication, the multi-controlled phase gate acting on $P[k]$, controlled by $Y[i]$ and $X[j]$ qubits is modified into first applying $\text{Toffoli}(z, Y[i], a_{1})$ and then applying $\text{Toffoli}(a_{1}, X[j], a_{2})$, and finally controlled-phase gates $R_{d-(i+j+k)}$ acting on $P[k]$ qubits controlled with $a_{2}$ ancilla qubit. Therefore, in our circuit construction, we only require two additional ancilla qubits to implement Toffoli gates.

# Circuit Complexity and Resource Estimation

The quantum circuit associated with the **QCalc** involves multiple Toffoli gates and controlled phase gates along with NOT gate, Hadarmard gate and SWAP gates coming from Inverse QFT. With respect to the basis gate set (CX, Toffoli, RZ, RX, X), the circuit depth scales with respect to $d$ as,

$$
\text{circuit depth}\sim O\left(d^{2.12}\right)
$$

On the other hand, the total gate count of the circuit scales as

$$
\text{Total gate count}\sim O\left(d^{2.1}\right)
$$

The Toffoli count scales as $\text{Toffoli gate count}\sim O\left(d^{1.47}\right)$, and the CX count scales as $\text{Total CX count}\sim O\left(d^{2.4}\right)$. Moreover, in the case of single qubit gates, the RZ gate count scales as $\text{RZ gate count}\sim O\left(d^{2.18}\right)$ and the RX gate count scales as $\text{RX gate count}\sim O\left(d^{1.54}\right)$. In addition, in our circuit construction, the number of ancilla qubits remains 2 irrespective of $d$.




