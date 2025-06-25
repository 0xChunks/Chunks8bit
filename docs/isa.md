# Chunks-8 Instruction Set Architecture (ISA)

Chunks-8 is a fixed-instruction-width 8-bit CPU. All instructions are **exactly 2 bytes (16 bits)** and are aligned on even addresses.

The architecture supports register–register operations, immediate values, memory access, and branching within a consistent, decode-friendly binary format.

---

## 🧠 Instruction Format Overview

Each instruction is 16 bits wide and consists of:


### Bit Fields

| Bits      | Name     | Description                                       |
|-----------|----------|---------------------------------------------------|
| `[15:12]` | OPCODE   | 4-bit primary opcode       |
| `[10:8]`  | REG A    | Register operand (R0–R7)                          |
| `[7:0]`   | Payload  | Operand B, immediate, offset, or extended bits    |

> The meaning of REG A and the 8-bit payload depends on the instruction type.

---

## 🔢 Instruction Type Summary

| Type | Name                 | Purpose                          | Format Encoding                         |
|------|----------------------|----------------------------------|-----------------------------------------|
| A    | ALU Register–Register| Binary operations                | REG A = dest, payload = source register |
| B    | ALU Immediate        | Register + 8-bit immediate       | REG A = dest, payload = immediate       |
| C    | Memory Access        | LOAD / STORE with 8-bit offset   | REG A = base register                   |
| D    | Control Flow         | Branches and jumps (PC-relative) | payload = signed offset                 |

---

*All instructions are self-aligned and fully decodable from the first byte.*

## 🔧 ALU Instructions: Register–Register

These instructions perform binary operations on two registers and store the result in the destination register. They use **Type A encoding** and do not modify memory.

---

### 🧩 Encoding Format (Type A)

| Bits      | Name                   | Description                                       |
|-----------|------------------------|---------------------------------------------------|
| `[15:12]` | OPCODE                 | 4-bit primary opcode                              |
| `[10:8]`  | Destination Register   | Register operand (R0–R7)                          |
| `[6:4]`   | Source Register        | Register operand (R0–R7)                          |
| `[3:0]`   | Reserved (Set to 0)    | Operand B, immediate, offset, or extended bits    |

- Always 2 bytes (16 bits)
- Reserved bits ensure consistent decoding and allow future extensions

---

### 📘 ALU Instruction Table

| Mnemonic | OPCODE (bin) | Description                       |
|----------|--------------|-----------------------------------|
| `ADD`    | `0000`       | `Rdst = Rdst + Rsrc`              |
| `SUB`    | `0001`       | `Rdst = Rdst - Rsrc`              |
| `AND`    | `0010`       | `Rdst = Rdst & Rsrc`              |
| `OR`     | `0011`       | `Rdst = Rdst | Rsrc`              |
| `CMP`    | `0100`       | `Compare Rdst - Rsrc` (set flags) |

---

### 🧪 Example

```asm
; ADD R3, R1
OPCODE:   0000
Reserved: 0
Rdst:     R3 (011)
Reserved: 0
Rsrc:     R1 (001)
Reserved: 0000

Binary:   0000 0011 0001 0000
Hex:      0x0310
```
## 🧮 ALU Instructions: Register–Immediate

These instructions perform an ALU operation between a register and an 8-bit immediate value. The result is stored in the destination register. They use a fixed 16-bit instruction format.

---

### 🧩 Encoding Format

| Bits      | Name                   | Description                                       |
|-----------|------------------------|---------------------------------------------------|
| `[15:12]` | OPCODE                 | 4-bit primary opcode                              |
| `[10:8]`  | Destination Register   | Register operand (R0–R7)                          |
| `[7:0]`   | Immediate Value        | 8-bit immediate value (0–255)                     |

> Unlike R–R instructions, all 8 low bits are used for the immediate value.

---

### 📘 Immediate Instruction Table

| Mnemonic | OPCODE (bin) | OPCODE (hex) | Description               |
|----------|--------------|--------------|---------------------------|
| `MOVI`   | `0101`       | `0x5`        | `Rdst = IMM`              |
| `ADDI`   | `0110`       | `0x6`        | `Rdst = Rdst + IMM`       |
| `SUBI`   | `0111`       | `0x7`        | `Rdst = Rdst - IMM`       |
| `ANDI`   | `1000`       | `0x8`        | `Rdst = Rdst & IMM`       |
| `ORI`    | `1001`       | `0x9`        | `Rdst = Rdst | IMM`       |

---

### 🧪 Example

```asm
; MOVI R4, #0x3C
OPCODE:   1000
Rdst:     R4 (100)
IMM:      0011 1100

Full Binary: 1000 0100 0011 1100
Hex:         0x843C
```

## 🗃️ Memory Access Instructions: LOAD & STORE

Chunks-8 uses memory-mapped I/O and register-indirect addressing with 8-bit offsets. These instructions allow reading and writing memory relative to a base register (e.g., `R1`).

---

### 🧩 Encoding Format

| Bits      | Name                   | Description                                       |
|-----------|------------------------|---------------------------------------------------|
| `[15:12]` | OPCODE                 | 4-bit primary opcode                              |
| `[10:8]`  | Destination Register   | Register operand (R0–R7)                          |
| `[6:4]`   | Base Register          |                                                   |
| `[3:0]`   | Offset                 | 8-bit signed offset (PC-relative)                 |


- The **effective address** is calculated as: `EA = REG + OFFSET`
- Supports access to RAM, ROM, or memory-mapped I/O

---

### 📘 Memory Instruction Table

| Mnemonic | OPCODE (bin) | OPCODE (hex) | Description                  |
|----------|--------------|--------------|------------------------------|
| `LOAD`   | `1010`       | `0xA`        | `Rdst = M[REG + OFFSET]`     |
| `STORE`  | `1011`       | `0xB`        | `M[REG + OFFSET] = Rsrc`     |

> The same field `REG` is used as the base for both instructions. In `LOAD`, it’s the destination register; in `STORE`, it’s the source.

---

### 🧪 Examples

```asm
; LOAD R2, [R5 + 10]
OPCODE:   1110
Rdst:     R2 (010)
Base:     R5 (101)
Offset:   1010

Binary:   1110 0010 0101 1010
Hex:      0xE25A
```
## 🔁 Control Flow Instructions: Jumps, Calls, Returns

These instructions alter the program counter (PC), allowing for conditional branching, loops, and subroutine calls. All use a 2-byte format with signed 8-bit **PC-relative offsets**.

---

### 🧩 Encoding Format

| Bits      | Name                   | Description                                       |
|-----------|------------------------|---------------------------------------------------|
| `[15:12]` | OPCODE                 | 4-bit primary opcode                              |
| `[11:8]`  | RESERVED               | Must be 0000                                      |
| `[7:0]`   | OFFSET                 | signed PC-relative jump offset                    |


- **Target address** is computed as: `PC = PC + sign_extend(OFFSET)`
- Offset range: `-128 to +127` bytes from the next instruction
- All jumps are relative, simplifying linking and bootloading

---

### 📘 Control Instruction Table

| Mnemonic | OPCODE (bin) | OPCODE (hex) | Description                     |
|----------|--------------|--------------|---------------------------------|
| `JMP`    | `1100`       | `0xC`        | Unconditional jump              |
| `JZ`     | `1101`       | `0xD`        | Jump if zero flag set           |
| `JNZ`    | `1110`       | `0xE`        | Jump if zero flag clear         |
| `RET`    | `1111`       | `0xF`        | Pop PC from stack               |

---

### 🧪 Example

```asm
; JZ +0x10 (jump forward 16 bytes if zero flag is set)
OPCODE:   1001
Offset:   0001 0000

Binary:   1101 0000 0001 0000
Hex:      0xD010

; JMP -0x08 (jump 8 bytes backward unconditionally)
OPCODE:   1100
Offset:   0b1111_1000  (2's complement -8)

Binary:   1100 0000 1111 1000
Hex:      0xC0F8
```
## 🗂️ Full Opcode Table

| OPCODE | Type   | Mnemonic | Description                        |
|--------|--------|----------|------------------------------------|
| `0000` | ALU    | `ADD`    | `Rdst = Rdst + Rsrc`               |
| `0001` | ALU    | `SUB`    | `Rdst = Rdst - Rsrc`               |
| `0010` | ALU    | `AND`    | `Rdst = Rdst & Rsrc`               |
| `0011` | ALU    | `OR`     | `Rdst = Rdst | Rsrc`               |
| `0100` | ALU    | `CMP`    | Compare `Rdst - Rsrc`, set flags   |
| `0101` | IMM    | `MOVI`   | `Rdst = Imm8`                      |
| `0110` | IMM    | `ADDI`   | `Rdst = Rdst + Imm8`               |
| `0111` | IMM    | `ANDI`   | `Rdst = Rdst & Imm8`               |
| `1000` | IMM    | `SUBI`   | `Rdst = Rdst - Imm8`               |
| `1001` | IMM    | `ORI`    | `Rdst = Rdst | Imm8`               |
| `1010` | MEM    | `LOAD`   | `Rdst = M[Base + Offset8]`         |
| `1011` | MEM    | `STORE`  | `M[Base + Offset8] = Rsrc`         |
| `1100` | CTRL   | `JMP`    | `PC += Offset8` (signed)           |
| `1101` | CTRL   | `JZ`     | If `Z==0`, jump                    |
| `1110` | CTRL   | `JNZ`    | If `Z!=0`, jump                    |
| `1111` | CTRL   | `RET`    | Pop PC                             |

> Note: IMM and CTRL instructions share some opcode numbers but are decoded based on reserved bits.

---

