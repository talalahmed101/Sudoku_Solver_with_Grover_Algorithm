# Sudoku_Solver_with_Grover_Algorithm
Solving $n^2 \times n^2$ Sudoku with Grover's algorithm

Sudoku is a puzzle that involves logic and combinatorial number placement, organized on grids of size $n^2 \times n^2$ composed of $n \times n$ blocks. Sudoku aims to fill this grid with the numbers from 1 to $n^2$, ensuring that each column, each row, and each of the $n^2$ subgrids (often referred to as "boxes") contains every digit exactly once. In the classic version, the task is to complete a 9 × 9 grid so that each column, each row, and the nine $3\times 3$ boxes contain all the digits from 1 to 9 without repetition. When provided with a partially filled grid, a well-formed Sudoku puzzle has a single solution, which we aim to find.

The overall challenge of solving Sudoku puzzles organized on $n^2 \times n^2$ grids with $n \times n$ blocks is NP-complete. Although various algorithms for solving Sudoku, like backtracking and dancing links, can efficiently resolve most $9\times 9$ puzzles, the complexity tends to escalate with increasing values of $n$, which results in practical limitations concerning the properties of Sudokus that can be generated, analyzed, and solved as $n$ grows larger. For the classic $9\times 9$ Sudoku, the total number of solution grids is a staggering 6,670,903,752,021,072,936,960, or roughly $6.67 \times 10^{21}$ [1]. However, when accounting for symmetrical variations such as rotation, reflection, permutation, and relabelling, the number of fundamentally distinct solutions decreases significantly to 5,472,730,538 [1].

Here, we use Grover’s algorithm to find out the solution of the given partially-filled Sudoku grid. For the given Sudoku grid, we identify all empty cells and all row, column and box constraints containing either single or more empty cells.The focus is to make sure all constraints are satisfied. We use quantum circuits for bit-masking method for checking if a constraint is satisfied or not. In classical case, the bit-masking first create the bitstring of $n^2$ places, ‘0000…0’ and for a row, column or a box, add ‘1’ to already present integer on that row/column/box in the respective position of the above bitstring. The constraint is satisfied only if the bitstring consists of all ‘1’. The quantum circuit construction will be part of the Oracle construction.

In this framework, each empty cell is represented by $d = \lceil \log_2(n^2) \rceil$ qubits. Therefore, with $K$ empty cells, we have a total of $Kd$ qubits tied to these empty cells. When implementing the Bit-Masking method for an $n^2 \times n^2$ Sudoku solver, the primary objective is to ensure that every constraint, whether a row, column, or box, contains each number exactly once.

For any constraint consisting of $m = n^2$ cells, let us denote the value of cell $i$ as $v_i$. We establish a Bucket Register $|B\rangle$ composed of $m$ qubits, with each qubit representing a potential Sudoku value in the range $\{1, 2, ..., m\}$. The process unfolds in three main steps:

1. Marking: For each cell $i$ in the constraint, we perform a transformation to flip the $k$-th bucket qubit when  $v_i = k$.
   
2. Parity: If all values within the constraint are unique, then each bucket qubit will be flipped exactly once, leading to the state $|11\dots1\rangle$.

3. Collision: If there is any repetition of values, the bucket qubit corresponding to the repeated value will be flipped an even number of times, reverting it to $|0\rangle$.

For every cell, represented using $d = \lceil \log_2(n^2) \rceil$ qubits, a quantum decoder will be employed to mark the relevant bucket qubit for the value $v_i = k$ through a series of multi-controlled $X$ gates (MCX), with the controls dependent on the binary representation of the Sudoku values $k$. After this setup, we carry out the following steps:

- Compute: Utilize decoders for all cells within the constraint to fill the buckets accordingly.
  
- Mark: Apply an $n^2$-controlled Toffoli gate on the bucket register to toggle a constraint flag qubit $|c_j\rangle$.

- Uncompute: Implement the inverse of the decoders to restore the bucket register to its initial state $|0\rangle$.

In the following we give detailed description on the solver. The algorithm works in the following stages:
1. Classical preprocessing:
  - Convert Sudoku grid to a dictionary representation.
  - Identify empty cells.
  - Generate constraint dictionaries for rows, columns, and boxes.
2. Quantum register construction:
  - Allocate registers for variable cells.
  - Allocate registers for symbol buckets and constraint flags.
  - Build mappings from Sudoku cells to qubit blocks.
3. Oracle construction:
 - Encode Sudoku constraints into a quantum oracle.
 - Use bucket registers to detect digit presence.
 - Set constraint flags for satisfied constraints.
 - Combine constraint flags into a global solution flag.
4. Grover search:
 - Initialize superposition over all candidate assignments.
 - Apply Grover iterations using the Sudoku oracle.

These functions convert the Sudoku puzzle into data structures suitable for the quantum oracle.
<pre>
grid_to_cellvalue_dict(grid, n)
</pre>
converts the Sudoku grid into a dictionary representation.

Input:
 - grid : Sudoku grid
 - n : box dimension
   
Output:
<code>{cell_index : value}</code>

Empty cells are stored as 0. Example:
<code>{0:5, 1:3, 2:0, 3:6, ...}</code>

<pre>
empty_index_to_position_map(cellvalue_dict)
</pre>
maps each empty Sudoku cell to a position in the quantum register.

Input:
- <code>cellvalue_dict</code>

Output:
<code>{empty_cell_index : position}</code>

Example:
<code>{19:0, 40:1, 61:2}</code>. Each position corresponds to a block of d qubits.

Constraint Generation:

Three types of constraints are generated:
- Row constraints
- Column constraints
- Box constraints
  
Each constraint is represented by a dictionary
<code>{cell_index : value}</code>

<code>row_constraints_dicts(grid)</code> returns a list of dictionaries representing row constraints. Rows with no empty cells are skipped.

<code>col_constraints_dicts(grid)</code> returns a list of dictionaries representing column constraints. Columns with no empty cells are skipped.

<code>box_constraints_dicts(grid)</code> returns a list of dictionaries representing sub-box constraints. Boxes with no empty cells are skipped.

Quantum Circuit Construction:
<code>sudoku_quantum_register_circuit(...)</code> constructs the full quantum circuit and registers.
Input: <code>(n, empty_index_to_position_map, total_constraints)</code>

Output: <code>(qc, empty_cell_to_qubits, x, b, c, anc, tot_qubits)</code> which provides the quantum circuit, a mapping
empty_cell_index → qubits in x register, qubit register associated with empty cells <code>x</code>, qubit register associated with buckets <code>b</code>, qubit register associated with constraints <code>c</code>, the global ancilla qubit register <code>anc</code>, and total number of qubits <code>tot_qubits</code>.
Example of <code>empty_cell_to_qubits</code> for $n = 2$ is <code>{2: [<Qubit register=(4, "x"), index=0>, <Qubit register=(4, "x"), index=1>],  5: [<Qubit register=(4, "x"), index=2>, <Qubit register=(4, "x"), index=3>]}</code>

<code>sudoku_quantum_register_circuit_oracle(...)</code> creates a circuit specifically for the oracle construction.

Binary Encoding Utilities:

<code>bits_lsb_first(a, d)</code> returns the binary representation of integer a in LSB-first order.

Example: <code>bits_lsb_first(6,4)</code> returns <code>[0,1,1,0]</code>

<code>d_qubits(n)</code> computes number of qubits needed to encode one Sudoku symbol: <code>d = ceil(log2(n**2))</code>

Besides, <code>empty_keys_from_constraint(constraint_dict)</code> returns sorted list of empty cell indices from constraint dictionary. The function <code>nonzero_values_from_constraint(constraint_dict)</code> returns (keys, values, dict)
returns only the filled values.

These functions implement the Sudoku oracle logic.

<code>bitmask_check_for_single_constraint(...)</code> constructs logic to detect which digits appear in a constraint. For each possible digit, compare binary encoding of cell values. Flip a bucket qubit if the digit appears. The bucket register therefore represents symbol presence.

<code>bitmask_check_for_single_constraint_extended(...)</code> extends the previous function by: Checking if all digits appear, Setting a constraint flag qubit <code>qc.mcx(b_reg, c_reg[i])</code>, and finally uncomputes by applying the inverse circuit.

<code>bitmask_check_for_multi_constraints(...)</code> applies the single-constraint logic to all constraints. Each constraint gets its own flag qubit.

<code>bitmask_check_for_multi_constraints_reversed(...)</code> builds the inverse circuit to uncompute temporary workspace.

<code>bitmask_check_for_all_constraints_oracle(...)</code> constructs the complete Sudoku oracle. It evaluates all constraints, combines constraint flags, and flip global ancilla if all constraints satisfied. Afterwards, it uncomputes intermediate ancilla register associated with constraints. The oracle therefore marks valid Sudoku assignments.

<code>Grover_Diffuser_circuit(x_reg)</code> constructs the Grover Diffuser circuit for given qubit register associated with empty cells <code>x</code>.

<code>Grover_Full_Diffusion(qc, x, anc, depth, oracle_gate, Diffuse_gate)</code> constructs the Grover Diffusion for given quantum circuit, qubit register associated with empty cells <code>x</code>, and qubit register associated with global ancilla <code>anc</code>. Here depth associated with the minimum number of Grover iteration associated with the dimension of the search space. 

## References
[1] http://www.afjarvis.org.uk/sudoku/




