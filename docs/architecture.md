# Chunks-8 Architecture Overview

**Chunks-8** is a custom 8-bit computer designed from scratch for learning, simulation, and eventual FPGA implementation. It supports variable-length instructions, a custom ISA, and memory-mapped I/O to enable simple games and systems programming in a compact, understandable platform.

---

## 🧠 Core Specs

| Feature             | Description                          |
|---------------------|--------------------------------------|
| **Data width**       | 8 bits (registers, ALU, memory)     |
| **Address bus**      | 16 bits (64KB addressable memory)   |
| **Instruction size** | Variable (2–3 bytes, aligned)       |
| **Registers**        | 8 general-purpose registers (R0–R7) |
| **Instruction set**  | Custom-designed for Chunks-8        |
| **Memory map**       | RAM, ROM, and I/O-mapped peripherals|

---

## 📦 Memory Layout

| Address Range     | Size     | Purpose                    |
|-------------------|----------|----------------------------|
| `0x0000–0x1FFF`   | 8 KB     | General-purpose RAM        |
| `0x2000–0x7FFF`   | 24 KB    | Graphics/video RAM         |
| `0x8000–0xFFEF`   | 32 KB    | Program ROM                |
| `0xFFF0–0xFFFF`   | 16 bytes | Memory-mapped I/O          |

### Memory-Mapped I/O (Tentative)

| Address   | Function           |
|-----------|--------------------|
| `0xFFF0`  | Output (UART TX)   |
| `0xFFF1`  | Input (UART RX)    |
| `0xFFF2`  | Button/Input Flags |
| `0xFFF3`  | Display Control    |

---

## 📚 Register File

- 8 general-purpose 8-bit registers:
  - `R0` to `R7`
- No dedicated accumulator or index registers (yet)
- Potential special use conventions:
  - `R6`: stack pointer (software managed)
  - `R7`: reserved for system call, flags, or interrupt return

---

## 🔀 Instruction Categories

| Type     | Description              | Format     | Size |
|----------|--------------------------|------------|------|
| ALU      | R–R or R–IMM operations  | Type A/B   | 2 B  |
| Memory   | LOAD, STORE              | Type C     | 2 B  |
| Control  | JUMP, CALL, RET          | Type D     | 2 B  |
| Special  | NOP, HLT, SYS            | Type E     | 2 B  |

### Sample Opcodes

| Mnemonic | Type   | Description             |
|----------|--------|-------------------------|
| `ADD`    | ALU    | `Rdst = Rdst + Rsrc`    |
| `MOVI`   | ALU    | `Rdst = Imm`            |
| `LOAD`   | Memory | `Rdst = M[addr]`        |
| `STORE`  | Memory | `M[addr] = Rsrc`        |
| `JMP`    | Ctrl   | `PC = addr`             |
| `HLT`    | Special| Halt the CPU            |

---

## 🧠 CPU Cycle Summary

- **Fetch**: read instruction bytes from program memory
- **Decode**: determine instruction type, operands
- **Execute**: perform ALU op
- **Memory**: If data memory needs to be accessed, it is done in this stage.
- **WriteBack**: write results into the register file

---

## 🛠 Planned Subsystems

| Subsystem      | Description                            |
|----------------|----------------------------------------|
| ALU            | Performs ADD, SUB, AND, OR, NOT, etc.  |
| Register File  | 8x 8-bit registers                     |
| Control Unit   | Decodes opcodes and signals execution  |
| Memory Access  | Address decoding, data bus interface   |
| Program Counter| Holds current instruction address      |
| I/O Bus        | Interface to UART, screen, etc.        |

---

## 🧩 Notes

- Instructions will be designed with clean binary encodings to make decoding straightforward in both software and Verilog.
- Variable-length instructions are aligned and self-delimiting.
- The system will support demos like Pong, Snake, or scrolling text once basic video output is implemented.

---

*Last updated: 2025-06-24*

