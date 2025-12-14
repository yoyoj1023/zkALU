# zkALU - Zero-Knowledge Arithmetic Logic Unit

A universal Arithmetic Logic Unit (ALU) zero-knowledge proof system implementation based on the Plonky3 framework.

## 📋 Project Overview

zkALU is a virtual processor implemented using the STARK proof system, supporting basic arithmetic operations (addition and subtraction). The system includes 4 registers and can execute a series of instructions while generating zero-knowledge proofs that verify the correctness of instruction execution without revealing intermediate computation processes.

### Core Features

- **4 General-Purpose Registers**: r0, r1, r2, r3
- **Supported Instructions**:
  - `ADD`: Addition operation (dest = src1 + src2)
  - `SUB`: Subtraction operation (dest = src1 - src2)
- **Zero-Knowledge Proofs**: Uses STARK proof system to verify computation correctness
- **Finite Field Arithmetic**: Based on BabyBear finite field (p = 2³¹ - 2²⁷ + 1)

## 🛠️ Environment Requirements

### Prerequisites

- **Rust**: Version 1.70 or higher (nightly recommended for optimal performance)
- **Cargo**: Rust's package manager
- **Operating System**: Windows, Linux, or macOS

### Dependencies

All dependencies are automatically downloaded during compilation, including:

- `p3-air`: AIR (Algebraic Intermediate Representation) constraint system
- `p3-baby-bear`: BabyBear finite field implementation
- `p3-uni-stark`: STARK proof system
- `p3-merkle-tree`: Merkle tree commitment scheme
- Other cryptographic and mathematical libraries

Packages are downloaded to: `C:\Users\<username>\.cargo\registry`

## 🚀 Quick Start

### 1. Compile and Run the Example

From the project root directory:

```powershell
# In the Plonky3 directory
cd C:\VScode\zkALU\Plonky3
cargo run --release -p p3-examples --example my_alu
```

Or from the examples directory:

```powershell
# In the Plonky3/examples directory
cd C:\VScode\zkALU\Plonky3\examples
cargo run --release --example my_alu
```

### 2. Output Explanation

The program will display:

1. **Program Definition**: Shows the ALU instruction sequence
2. **Initial State**: Initial values of the 4 registers
3. **Execution Process**: Step-by-step instruction execution and register changes
4. **Final State**: Register values after execution completes
5. **Proof Generation**: STARK zero-knowledge proof generation process
6. **Proof Verification**: Validation of the proof's validity

### 3. Compiled Executable Location

After compilation, the executable is located at:

```
C:\VScode\zkALU\Plonky3\target\release\examples\my_alu.exe
```

Run directly:

```powershell
.\target\release\examples\my_alu.exe
```

## 📚 System Architecture

### Instruction Format

Each instruction contains 4 fields:

```rust
Instruction {
    op: Opcode,      // Operation type (ADD or SUB)
    dest: usize,     // Destination register (0-3)
    src1: usize,     // Source register 1 (0-3)
    src2: usize,     // Source register 2 (0-3)
}
```

### Execution Trace Structure (18 Columns)

Each row contains:

1. **Register Values** (4 columns): Current values of r0, r1, r2, r3
2. **Destination Selectors** (4 columns): dest_0, dest_1, dest_2, dest_3 (one-hot encoding)
3. **Source 1 Selectors** (4 columns): src1_0, src1_1, src1_2, src1_3 (one-hot encoding)
4. **Source 2 Selectors** (4 columns): src2_0, src2_1, src2_2, src2_3 (one-hot encoding)
5. **Operation Selectors** (2 columns): op_add, op_sub (one-hot encoding)

### AIR Constraint System

**Constraint 1: Selector Validity**
- All selectors must be boolean values (0 or 1)
- Each selector group must be one-hot encoded (exactly one equals 1)

**Constraint 2: State Transition Correctness**
- Calculate source values: `src1_val = Σ(ri × src1_i)`, `src2_val = Σ(ri × src2_i)`
- Calculate result: `result = (src1_val + src2_val) × op_add + (src1_val - src2_val) × op_sub`
- Update registers: `next_ri = current_ri × (1 - dest_i) + result × dest_i`

## 🔧 Custom Examples

### Modify Program Instructions

Edit the `main` function in `my_alu.rs`:

```rust
let mut program = vec![
    Instruction { op: Opcode::ADD, dest: 0, src1: 0, src2: 1 }, // r0 = r0 + r1
    Instruction { op: Opcode::SUB, dest: 1, src1: 2, src2: 0 }, // r1 = r2 - r0
    // Add more instructions...
];
```

### Modify Initial Register Values

```rust
let initial_regs = [
    Val::from_u64(1), // r0 = 1
    Val::from_u64(2), // r1 = 2  
    Val::from_u64(5), // r2 = 5
    Val::from_u64(0)  // r3 = 0
];
```

## 🎓 Key Learning Points

### 1. One-Hot Encoding

One-hot encoding is used for selectors to ensure only one option is selected at a time:
- `[1,0,0,0]` selects the 0th option
- `[0,1,0,0]` selects the 1st option

### 2. Conditional Constraints

Selectors implement conditional logic without branching in zero-knowledge proofs:

```rust
// result = add_result * op_add + sub_result * op_sub
// When op_add=1: result = add_result
// When op_sub=1: result = sub_result
```

### 3. State Transitions

Selectors control register updates:

```rust
// next_ri = current_ri * (1 - dest_i) + result * dest_i
// When dest_i=0: remains unchanged
// When dest_i=1: updates to result
```

## 💭 Reflection Questions

1. **Power of Conditional Constraints**: How do operation selectors implement if/else logic without branches?
2. **Optimization Approach**: If using a single `is_sub` field instead of `op_add/op_sub`, how should constraints be modified?
3. **Extensibility**: How to add a MUL (multiplication) instruction? Which parts need modification?
4. **Performance Considerations**: Why does the execution trace length need to be a power of 2?

## 📖 Advanced Topics

### CPU Feature Optimization

For optimal performance, enable all instruction sets supported by your target CPU:

```powershell
$env:RUSTFLAGS="-Ctarget-cpu=native"
cargo run --release --example my_alu
```

Supported instruction sets include: AVX2, AVX-512, BMI1/2 (x86), NEON (ARM)

### Nightly Version Optimization

Using Rust nightly enables additional optimization features:

```powershell
cargo +nightly run --release --features nightly-features --example my_alu
```

## 📄 License

This project is dual-licensed under:
- MIT License ([LICENSE-MIT](LICENSE-MIT))
- Apache 2.0 License ([LICENSE-APACHE](LICENSE-APACHE))

You may choose either license to use this project.
