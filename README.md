## 4. Accelerator Architecture

### 4.1 Top-Level Modules

The NTT-based polynomial multiplication accelerator is composed of the following key hardware modules:

1. **Configurable Processing Element (CPE)**  
   - Core computation engine of the accelerator  
   - Executes NTT, INTT, and point-wise multiplication  
   - Contains multiple parallel butterfly units  

2. **Butterfly Units (BU)**  
   - Fundamental building block of NTT computation  
   - Performs modular addition, subtraction, and multiplication  
   - Organized in a radix-2 structure for efficient decomposition  

3. **Modular Arithmetic Unit**  
   - Handles modular multiplication and reduction  
   - Implements Barrett reduction for division-free modular arithmetic  
   - Optimized using shift-and-add operations for ASIC efficiency  

4. **Reordering Unit (CRU - Configurable Reordering Unit)**  
   - Reorders coefficients between NTT stages  
   - Eliminates need for bit-reversal memory operations  
   - Uses shift registers and feedback paths for low-latency operation  

5. **Memory Subsystem**  
   - Stores polynomial coefficients and intermediate results  
   - Organized into multiple SRAM banks (e.g., even/odd split)  
   - Supports parallel and conflict-free memory access  

6. **Control Unit**  
   - Generates control signals for all modules  
   - Manages sequencing of NTT stages  
   - Handles address generation for memory and twiddle factors  
   - Interfaces with CPU via memory-mapped registers  

---

### 4.2 NTT Computation Flow

Polynomial multiplication is performed using the following sequence:

1. Compute NTT of polynomial A  
2. Compute NTT of polynomial B  
3. Perform point-wise multiplication in NTT domain  
4. Compute inverse NTT (INTT) of the result  

This transforms polynomial multiplication from:
- **O(n²)** → **O(n log n)**

---

### 4.3 Butterfly Unit Design

Each butterfly unit computes:

u = a + b·w mod q
v = a - b·w mod q


#### Components:
- Modular multiplier (for twiddle multiplication)
- Modular adder
- Modular subtractor

#### Design Choice:
- Cooley-Tukey (CT) for NTT  
- Gentleman-Sande (GS) for INTT  

This avoids explicit bit-reversal stages and simplifies control logic.

---

### 4.4 Parallel Architecture

- The accelerator uses **4 parallel radix-2 butterfly units**
- Enables processing of multiple coefficient pairs per cycle  
- Provides a balance between:
  - Throughput  
  - Area  

---

### 4.5 Modular Multiplication

#### Technique: Barrett Reduction

- Avoids division operations  
- Uses precomputed constants  
- Implemented using:
  - Bit shifts  
  - Additions and subtractions  

#### Advantages:
- ASIC-friendly  
- Low latency  
- Deterministic execution  

---

## 5. Memory Architecture

### 5.1 Coefficient Storage

- Two SRAM banks:
  - **RAM0** → even-index coefficients  
  - **RAM1** → odd-index coefficients  

#### Benefits:
- Enables parallel access  
- Avoids read-after-write conflicts  

---

### 5.2 Data Packing

- Multiple coefficients packed into a single memory word  
- Supports SIMD-style parallel processing  

---

### 5.3 Memory Access Pattern

- Sequential read and write access  
- Same address used across memory banks  
- Simplified address generation  

---

## 6. Twiddle Factor Storage

### Design Strategy:

- Twiddle factors stored in ROM  
- Divided into:
  - **Shared ROM** → used in early stages  
  - **Private ROM** → used in final stages  

#### Advantages:
- Reduces memory usage  
- Avoids redundant storage 

---

## 7. Reordering Unit

### Problem:
NTT requires permutation of coefficients between stages

### Solution:
- Shift-register-based reordering unit  
- Feedback paths enable stage transitions  

#### Benefits:
- Eliminates bit-reversal memory  
- Simplifies memory access  
- Maintains pipeline efficiency  

---

## 8. Pipeline Design

### Pipeline Stages:
Load → Compute → Reorder → Store


### Features:
- Fully pipelined execution  
- No stalls between stages  
- Overlapping operations across stages  

---

### Throughput Optimization

- Streaming architecture  
- Continuous data flow  
- Multi-polynomial processing capability  
