# Sudoku_Solver_with_Grover_Algorithm
Solving $n^2 \times n^2$ Sudoku with Grover's algorithm

Sudoku is a puzzle that involves logic and combinatorial number placement, organized on grids of size $n^2 \times n^2$ composed of $n \times n$ blocks. Sudoku aims to fill this grid with the numbers from 1 to $n^2$, ensuring that each column, each row, and each of the $n^2$ subgrids (often referred to as "boxes") contains every digit exactly once. In the classic version, the task is to complete a 9 × 9 grid so that each column, each row, and the nine $3\times 3$ boxes contain all the digits from 1 to 9 without repetition. When provided with a partially filled grid, a well-formed Sudoku puzzle has a single solution, which we aim to find.

The overall challenge of solving Sudoku puzzles organized on $n^2 \times n^2$ grids with $n \times n$ blocks is NP-complete. Although various algorithms for solving Sudoku, like backtracking and dancing links, can efficiently resolve most $9\times 9$ puzzles, the complexity tends to escalate with increasing values of $n$, which results in practical limitations concerning the properties of Sudokus that can be generated, analyzed, and solved as $n$ grows larger. For the classic $9\times 9$ Sudoku, the total number of solution grids is a staggering 6,670,903,752,021,072,936,960, or roughly $6.67 \times 10^{21}$. However, when accounting for symmetrical variations such as rotation, reflection, permutation, and relabelling, the number of fundamentally distinct solutions decreases significantly to 5,472,730,538.

Here, we use Grover’s algorithm to find out the solution of the given partially-filled Sudoku grid. For the given Sudoku grid, we identify all empty cells and all row, column and box constraints containing either single or more empty cells.The focus is to make sure all constraints are satisfied. We use quantum circuits for bit-masking method for checking if a constraint is satisfied or not. In classical case, the bit-masking first create the bitstring of $n^2$ places, ‘0000…0’ and for a row, column or a box, add ‘1’ to already present integer on that row/column/box in the respective position of the above bitstring. The constraint is satisfied only if the bitstring consists of all ‘1’. The quantum circuit construction will be part of the Oracle construction.

In this framework, each empty cell is represented by $d = \lceil \log_2(n^2) \rceil$ qubits. Therefore, with $K$ empty cells, we have a total of $Kd$ qubits tied to these empty cells. When implementing the Bit-Masking method for an $n^2 \times n^2$ Sudoku solver, the primary objective is to ensure that every constraint, whether a row, column, or box, contains each number exactly once.

For any constraint consisting of $m = n^2$ cells, let us denote the value of cell $i$ as $v_i$. We establish a Bucket Register $|B\rangle$ composed of $m$ qubits, with each qubit representing a potential Sudoku value in the range $\{1, 2, ..., m\}$. The process unfolds in three main steps:

1. Marking: For each cell $i$ in the constraint, we perform a transformation to flip the $k$-th bucket qubit when  $v_i = k$.
   
2. Parity: If all values within the constraint are unique, then each bucket qubit will be flipped exactly once, leading to the state $|11\dots1\rangle$.

3. Collision: If there is any repetition of values, the bucket qubit corresponding to the repeated value will be flipped an even number of times, reverting it to $|0\rangle$.

For every cell, represented using $d = \lceil \log_2(n^2) \rceil$ qubits, a quantum decoder will be employed to mark the relevant bucket qubit for the value $v_i = k$ through a series of multi-controlled $X$ gates (MCX), with the controls dependent on the binary representation of the Sudoku values $k$. After this setup, we carry out the following steps:

-Compute: Utilize decoders for all cells within the constraint to fill the buckets accordingly.
  
- Mark: Apply an $n^2$-controlled Toffoli gate on the bucket register to toggle a constraint flag qubit $|c_j\rangle$.

- Uncompute: Implement the inverse of the decoders to restore the bucket register to its initial state $|0\rangle$.

