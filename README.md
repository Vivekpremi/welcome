# NTT-Based Polynomial Multiplication Accelerator on Caravel

## 1. Overview

This project will implement a hardware accelerator for polynomial multiplication using the Number Theoretic Transform (NTT) on the Caravel SoC platform. The design will offload computationally intensive operations from the RISC-V processor, enabling efficient execution of lattice-based cryptographic algorithms such as ML-KEM (Kyber) and ML-DSA (Dilithium). The architecture is inspired from [this paper]

---

## 2. Motivation

Polynomial multiplication is a computationally intensive operation in post-quantum cryptography.

| Method | Complexity |
|-------|-----------|
| Schoolbook | O(n²)      |
| NTT-based  | O(n log n) |

This accelerator aims to accelerate polynomial multiplication using a pipelined hardware architecture, improving performance.

---

## 3. Top-Level Hardware Architecture

                  +--------------------------+
                  |     RISC-V Processor     |
                  |        (VexRiscv)        |
                  +------------+-------------+
                               |
                         Wishbone Bus
                               |
    =========================================================
    ||              NTT ACCELERATOR DATAPATH                ||
    =========================================================
                               |
        +----------------------+----------------------+
        |                                             |
+---------------+                             +---------------+
|     RAM0      |                             |     RAM1      |
| (Even Coeffs) |                             | (Odd Coeffs)  |
+-------+-------+                             +-------+-------+
        |                                             |
        +----------------------+----------------------+
                               |
                        +------+------+
                        | Address Gen |
                        |  / Control  |
                        +------+------+
                               |
                     +---------+---------+
                     |  Input Buffer /   |
                     |   Data Aligner    |
                     +---------+---------+
                               |
                +--------------+--------------+
                |   4x Butterfly Units (CPE)  |
                |-----------------------------|
                |  Twiddle Generator          |
                |  (Mod Mult Based)           |
                |-----------------------------|
                |  Modular Arithmetic         |
                |  (Mul + Add + Sub)          |
                +--------------+--------------+
                               |
                               v
                      +--------+--------+
                      | Reordering Unit |
                      |     (CRU)       |
                      +--------+--------+
                               |
        +----------------------+----------------------+
        |                                             |
+---------------+                             +---------------+
|     RAM0      |                             |     RAM1      |
| (Write Back)  |                             | (Write Back)  |
+---------------+                             +---------------+
                               |
                               v
                     Output Polynomial C(x)

---

## 4. Accelerator Microarchitecture

### 4.1 Core Modules

- Configurable Processing Element (CPE)  
- Butterfly Units (BU)  
- Modular Arithmetic Unit  
- Reordering Unit (CRU)  
- Memory Subsystem  
- Control Unit  

---

### 4.2 NTT Computation Flow

A(x), B(x)
   ↓
NTT(A), NTT(B)
   ↓
Point-wise Multiplication
   ↓
INTT(Result)
   ↓
C(x)

---

### 4.3 Butterfly Unit Architecture

#### NTT (Cooley-Tukey)

u = a + b·w mod q  
v = a - b·w mod q

#### INTT (Gentleman-Sande)

u = w · (a + b) mod q  
v = w · (a - b) mod q

---

### 4.4 Parallel Processing Architecture

- 4 parallel radix-2 butterfly units  
- Processes multiple coefficients per cycle  

---

### 4.5 Modular Arithmetic Design

#### Modular Multiplication

Technique: Barrett Reduction  

- Division-free  
- Uses precomputed constants  
- Implemented using shift-and-add operations  

---

#### Modular Addition and Subtraction

Technique: Conditional Reduction  

Addition:
- Compute s = a + b  
- If s ≥ q, subtract q  

Subtraction:
- Compute d = a - b  
- If d < 0, add q  

---

### 4.6 Memory Organization

- RAM0 → even indices  
- RAM1 → odd indices  
- Sequential access pattern for coefficients  

---

### 4.7 Twiddle Factor Generation

Twiddle factors are generated on-the-fly to reduce memory overhead and improve flexibility.

#### Generation Strategy

Twiddle factors follow a geometric progression:

w(i+1) = w(i) × w_stage mod q

Where:
- w_stage is the base twiddle for a given stage  
- w(0) = 1  

---

#### Execution Flow

For each NTT stage:

twiddle = 1  
for each butterfly:  
    use twiddle  
    twiddle = twiddle × w_stage mod q  

- Initial value is set to 1  
- Subsequent values are generated using modular multiplication  

---

#### Hardware Realization

- A modular multiplier is used for generation  
- Integrated within the butterfly datapath  
- Stage base values stored in registers (8 words total)  

---

#### Benefits

- Eliminates large twiddle ROM  
- Reduces area consumption  

---

### 4.8 Reordering Mechanism

- Shift-register-based design  
- Eliminates bit-reversal  

---

### 4.9 Pipeline Organization

Load → Compute → Reorder → Store

---

## 5. Polynomial Multiplication Dataflow

Polynomial size:
- 256 coefficients per polynomial  

Execution:
- NTT: 8 stages  
- PWM: 256 multiplications  
- INTT: same structure  

---

## 6. Output Generation

C(x) = A(x) * B(x)

- Output: 256 coefficients  

---

## 7. System Integration

- Connected via Wishbone bus  
- Controlled via memory-mapped registers  

---

## 8. Key Features

- Unified NTT/INTT architecture  
- Low area requirement  
- Reduced memory footprint  
