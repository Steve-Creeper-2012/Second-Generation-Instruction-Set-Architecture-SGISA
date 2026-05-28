# NIS_ISA_CPU 指令集编码表 v0.0.1

> **版本**: v0.0.1 
> **Base_Architecture版本**: v0.0.4
> **架构**: 32-bit 定长 RISC, 64-bit 寄存器  
> **编码约束**: bit[31] = 0（保留位），bit[30:0] 自由编码  
> **寄存器**: D0-D31（64-bit 通用寄存器），D0 硬连线为 0

---

## 编码总览

```
bit 31    30 29 28    27 26 25 24    23    22 21 20 19 18    17 16 15 14 13    12 11 10 9 8    7 6 5 4 3 2 1 0
   0    |  Class(3)  |  Opcode(4)   | I/R |     rd(5)        |     rs1(5)       |    rs2(5)      |      Ext(8)
```

| 字段 | 位宽 | 说明 |
|------|------|------|
| R | 1 | bit[31] = 0，保留位 |
| Class | 3 | 指令大类 |
| Opcode | 4 | 子操作码 |
| I/R | 1 | 0=R-type, 1=I-type |
| rd | 5 | 目标寄存器 (D0-D31) |
| rs1 | 5 | 源寄存器1 |
| rs2 | 5 | 源寄存器2 |
| Ext | 8 | 扩展/保留 |

### 立即数型 (I-type) 变体

```
bit 31    30 29 28    27 26 25 24    23    22 21 20 19 18    17 16 15 14 13    12 11 10 9 8 7 6 5 4 3 2 1 0
   0    |  Class(3)  |  Opcode(4)   |  1  |     rd(5)        |     rs1(5)       |           imm13(13)
```

### Load/Store 专用格式

```
bit 31    30 29 28    27 26    25 24 23    22 21 20 19 18    17 16 15 14 13    12 11 10 9 8 7 6 5 4 3 2 1 0
   0    |  0 1 1    | Type(2) | Width(3) |    rd/rs2(5)     |     rs1(5)       |           offset13(13)
```

### Branch/Jump 变体格式

- **Format F** (J/JL/CBZ/CBNZ): `Class(3)=100 | Opcode(4) | F=0/1 | rd/rs1(5) | imm18(18)`
- **Format E** (条件分支): `Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13)`
- **Format G** (JALR/RET): `Class(3)=100 | Opcode(4) | F=1 | rd(5) | rs1(5) | imm13(13)`

### Atomic 格式

```
bit 31    30 29 28    27 26 25 24    23    22 21 20 19 18    17 16 15 14 13    12 11 10 9 8    7 6 5 4 3 2 1 0
   0    |  1 1 0    | Opcode(4)  | S(1) |     rd(5)        |     rs1(5)       |    rs2(5)      |   Model(8)
```

---

## 指令大类映射

| Class | 名称 | 指令数 | 说明 |
|-------|------|--------|------|
| 000 | Integer Arithmetic | 32 | ADD/SUB/AND/OR/XOR/ADC/SBC/MAX/MIN/ADDI/ANDI/ORI 等 |
| 001 | Shift & Bit Ops | 21 | LSL/LSR/ASR/ROR/REV/CLZ/CTZ/POPCNT/BFX/BFI/SEX/ZEX |
| 010 | Multiply/Divide | 16 | MUL/MULH/DIV/REM/MAC/MULW 等 |
| 011 | Load/Store | 24 | LD/LDU/ST/STU (B/BS/H/HS/W/WS/D) + LDPC |
| 100 | Branch/Jump | 16 | J/JL/JALR/RET/BEQ/BNE/BLT/BGE/CBZ/CBNZ |
| 101 | Compare & Set | 27 | CMP/CMPU/TST/TEQ/SLT/SLTU/SGT/SGE/SLE/SNE + 立即数变体 |
| 110 | Atomic & Fence | 16 | LDAR/STLR/CAS/SWAP/ADD.A/LR/SC/FENCE 等 |
| 111 | **Reserved** | — | 为未来扩展保留 (向量/浮点/系统/自定义) |

---

## 完整指令表


### 000 — Integer Arithmetic

| 指令 | 类型 | Opcode | 编码格式 | RTL | 语义 | 专利风险 |
|------|------|--------|----------|-----|------|----------|
| **ADD** | R | 0000 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] + D[rs2]` | 64-bit signed/unsigned addition, overflow ignored | Public domain. Basic arithmetic operation, no known patent encumbrance. |
| **SUB** | R | 0001 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] - D[rs2]` | 64-bit subtraction | Public domain. |
| **ADC** | R | 0010 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] + D[rs2] + C` | Addition with carry-in from flag register | Public domain. Ancient operation. |
| **SBC** | R | 0011 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] - D[rs2] - C` | Subtraction with borrow | Public domain. |
| **AND** | R | 0100 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] & D[rs2]` | Bitwise AND | Public domain. |
| **OR** | R | 0101 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] | D[rs2]` | Bitwise OR | Public domain. |
| **XOR** | R | 0110 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] ^ D[rs2]` | Bitwise XOR | Public domain. |
| **NOR** | R | 0111 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← ~(D[rs1] | D[rs2])` | Bitwise NOR | Public domain. Universal gate, no patent. |
| **NAND** | R | 1000 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← ~(D[rs1] & D[rs2])` | Bitwise NAND | Public domain. |
| **XNOR** | R | 1001 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← ~(D[rs1] ^ D[rs2])` | Bitwise XNOR | Public domain. |
| **MAX** | R | 1010 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] >s D[rs2]) ? D[rs1] : D[rs2]` | Signed maximum | Generic DSP operation. No ISA-level patent risk. |
| **MIN** | R | 1011 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] <s D[rs2]) ? D[rs1] : D[rs2]` | Signed minimum | Generic DSP operation. |
| **MAXU** | R | 1100 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] >u D[rs2]) ? D[rs1] : D[rs2]` | Unsigned maximum | Generic. |
| **MINU** | R | 1101 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] <u D[rs2]) ? D[rs1] : D[rs2]` | Unsigned minimum | Generic. |
| **ADD.SAT** | R | 1110 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← sat_s(D[rs1] + D[rs2])` | Signed saturating addition | Saturating arithmetic is generic DSP concept. No known ISA patent. |
| **SUB.SAT** | R | 1111 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← sat_s(D[rs1] - D[rs2])` | Signed saturating subtraction | Generic. |
| **ADDI** | I | 0000 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← D[rs1] + sext(imm13)` | Add 13-bit signed immediate | Public domain. Immediate arithmetic generic. |
| **SUBI** | I | 0001 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← D[rs1] - sext(imm13)` | Subtract immediate | Public domain. |
| **ANDI** | I | 0010 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← D[rs1] & sext(imm13)` | AND immediate | Public domain. |
| **ORI** | I | 0011 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← D[rs1] | sext(imm13)` | OR immediate | Public domain. |
| **XORI** | I | 0100 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← D[rs1] ^ sext(imm13)` | XOR immediate | Public domain. |
| **NORI** | I | 0101 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← ~(D[rs1] | sext(imm13))` | NOR immediate | Public domain. |
| **NANDI** | I | 0110 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← ~(D[rs1] & sext(imm13))` | NAND immediate | Public domain. |
| **XNORI** | I | 0111 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← ~(D[rs1] ^ sext(imm13))` | XNOR immediate | Public domain. |
| **LEAHI** | I | 1000 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← D[rs1] + (sext(imm13) << 12)` | Load effective address, high 13-bit shifted by 12 | Address generation is generic. ARM ADRP uses different encoding (pg_hi21). No direct patent conflict on concept. |
| **LEALO** | I | 1001 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← D[rs1] + sext(imm13)` | Load effective address, low 13-bit | Generic. |
| **SLTI** | I | 1010 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← (D[rs1] <s sext(imm13)) ? 1 : 0` | Set if less than immediate (signed) | Comparison and set is generic. RISC-V SLTI uses different encoding. |
| **SLTIU** | I | 1011 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← (D[rs1] <u uext(imm13)) ? 1 : 0` | Set if less than immediate (unsigned) | Generic. |
| **ADDI.SAT** | I | 1100 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← sat_s(D[rs1] + sext(imm13))` | Saturating add immediate | Generic. |
| **SUBI.SAT** | I | 1101 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← sat_s(D[rs1] - sext(imm13))` | Saturating subtract immediate | Generic. |
| **NOP** | I | 1110 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `—` | No operation (D0=D0+0) | Public domain. |
| **UNIMP** | I | 1111 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `—` | Unimplemented instruction trap | Public domain. |

### 001 — Shift & Bit Operations

| 指令 | 类型 | Opcode | 编码格式 | RTL | 语义 | 专利风险 |
|------|------|--------|----------|-----|------|----------|
| **LSL** | R | 0000 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] << (D[rs2] & 0x3F)` | Logical shift left by register | Public domain. |
| **LSR** | R | 0001 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] >> (D[rs2] & 0x3F)` | Logical shift right by register | Public domain. |
| **ASR** | R | 0010 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] >>> (D[rs2] & 0x3F)` | Arithmetic shift right by register | Public domain. |
| **ROR** | R | 0011 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← ror(D[rs1], D[rs2] & 0x3F)` | Rotate right by register | Public domain. |
| **ROL** | R | 0100 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← rol(D[rs1], D[rs2] & 0x3F)` | Rotate left by register | Public domain. |
| **REV** | R | 0101 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← bswap(D[rs1])` | Byte reverse (big-endian swap) | Generic. ARM REV, x86 BSWAP exist but are differently encoded. Concept not patentable. |
| **REV16** | R | 0110 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← hswap(D[rs1])` | Halfword reverse | Generic. |
| **CLZ** | R | 0111 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← count_leading_zeros(D[rs1])` | Count leading zeros | Generic bit operation. ARM CLZ, x86 LZCNT. |
| **CTZ** | R | 1000 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← count_trailing_zeros(D[rs1])` | Count trailing zeros | Generic. x86 TZCNT. |
| **POPCNT** | R | 1001 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← popcount(D[rs1])` | Population count (1-bits) | Generic. x86 POPCNT. |
| **BFX** | R | 1010 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← extract(D[rs1], D[rs2][5:0], D[rs2][11:6])` | Bit field extract (lsb, width in rs2) | Bit field operations are generic. ARM UBFX/IBFX use different encoding. No ISA-level patent. |
| **BFI** | R | 1011 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← insert(D[rs1], D[rs2][5:0], D[rs2][11:6])` | Bit field insert | Generic. |
| **SEXB** | R | 1100 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← sext(D[rs1][7:0])` | Sign-extend byte to 64b | Generic. |
| **SEXH** | R | 1101 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← sext(D[rs1][15:0])` | Sign-extend halfword to 64b | Generic. |
| **ZEXB** | R | 1110 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← zext(D[rs1][7:0])` | Zero-extend byte to 64b | Generic. |
| **ZEXH** | R | 1111 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← zext(D[rs1][15:0])` | Zero-extend halfword to 64b | Generic. |
| **LSLI** | I | 0000 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← D[rs1] << (imm13 & 0x3F)` | Logical shift left immediate | Public domain. |
| **LSRI** | I | 0001 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← D[rs1] >> (imm13 & 0x3F)` | Logical shift right immediate | Public domain. |
| **ASRI** | I | 0010 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← D[rs1] >>> (imm13 & 0x3F)` | Arithmetic shift right immediate | Public domain. |
| **RORI** | I | 0011 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← ror(D[rs1], imm13 & 0x3F)` | Rotate right immediate | Public domain. |
| **ROLI** | I | 0100 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← rol(D[rs1], imm13 & 0x3F)` | Rotate left immediate | Public domain. |

### 010 — Multiply & Divide

| 指令 | 类型 | Opcode | 编码格式 | RTL | 语义 | 专利风险 |
|------|------|--------|----------|-----|------|----------|
| **MUL** | R | 0000 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] * D[rs2])[63:0]` | 64×64→64 low multiply | Public domain. |
| **MULH** | R | 0001 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] * D[rs2])[127:64] (signed)` | High 64b of signed multiply | Public domain. |
| **MULHU** | R | 0010 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] * D[rs2])[127:64] (unsigned)` | High 64b of unsigned multiply | Public domain. |
| **MULHSU** | R | 0011 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (sext(D[rs1]) * zext(D[rs2]))[127:64]` | High 64b signed×unsigned | Public domain. |
| **DIV** | R | 0100 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] /s D[rs2]` | Signed division | Public domain. |
| **DIVU** | R | 0101 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] /u D[rs2]` | Unsigned division | Public domain. |
| **REM** | R | 0110 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] %s D[rs2]` | Signed remainder | Public domain. |
| **REMU** | R | 0111 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rs1] %u D[rs2]` | Unsigned remainder | Public domain. |
| **MULW** | R | 1000 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← sext((D[rs1] * D[rs2])[31:0])` | 32-bit multiply, sign-extend result | Public domain. |
| **DIVW** | R | 1001 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← sext((D[rs1] /s D[rs2])[31:0])` | 32-bit signed divide | Public domain. |
| **REMW** | R | 1010 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← sext((D[rs1] %s D[rs2])[31:0])` | 32-bit signed remainder | Public domain. |
| **MULU** | R | 1011 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] * D[rs2])[63:0] (unsigned low)` | Unsigned low multiply (same as MUL) | Public domain. |
| **SMUL** | R | 1100 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← sat_s(D[rs1] * D[rs2])` | Saturating multiply | Generic DSP. |
| **SDIV** | R | 1101 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← sat_s(D[rs1] /s D[rs2])` | Saturating divide | Generic. |
| **MULR** | R | 1110 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← round(D[rs1] * D[rs2] / 2^64)` | Rounded high multiply (DSP) | Generic. |
| **MAC** | R | 1111 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← D[rd] + (D[rs1] * D[rs2])[63:0]` | Multiply-accumulate low | MAC is generic DSP operation. No ISA-level patent. |

### 011 — Load & Store

| 指令 | 类型 | Opcode | 编码格式 | RTL | 语义 | 专利风险 |
|------|------|--------|----------|-----|------|----------|
| **LDB** | LD-B | 00-000 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd] ← zext(M[D[rs1] + sext(offset13)][byte])` | Load byte from memory. Offset is 13-bit signed. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDBS** | LD-BS | 00-001 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd] ← sext(M[D[rs1] + sext(offset13)][signed byte])` | Load signed byte from memory. Offset is 13-bit signed. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDH** | LD-H | 00-010 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd] ← zext(M[D[rs1] + sext(offset13)][halfword])` | Load halfword from memory. Offset is 13-bit signed. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDHS** | LD-HS | 00-011 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd] ← sext(M[D[rs1] + sext(offset13)][signed halfword])` | Load signed halfword from memory. Offset is 13-bit signed. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDW** | LD-W | 00-100 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd] ← zext(M[D[rs1] + sext(offset13)][word])` | Load word from memory. Offset is 13-bit signed. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDWS** | LD-WS | 00-101 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd] ← sext(M[D[rs1] + sext(offset13)][signed word])` | Load signed word from memory. Offset is 13-bit signed. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDD** | LD-D | 00-110 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd] ← zext(M[D[rs1] + sext(offset13)][doubleword])` | Load doubleword from memory. Offset is 13-bit signed. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDPC** | LD-PC | 00-111 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd] ← M[PC + 4 + (sext(offset13) << 3)]` | Load 64-bit constant from PC-relative literal pool. Offset scaled by 8. | PC-relative addressing is generic. ARM LDR (literal) uses different encoding. No patent conflict. |
| **STB** | ST-B | 01-000 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `M[D[rs1] + sext(offset13)][7:0] ← D[rs2][7:0]` | Store byte to memory. Offset is 13-bit signed. | Generic. |
| **STH** | ST-H | 01-010 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `M[D[rs1] + sext(offset13)][15:0] ← D[rs2][15:0]` | Store halfword to memory. Offset is 13-bit signed. | Generic. |
| **STW** | ST-W | 01-100 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `M[D[rs1] + sext(offset13)][31:0] ← D[rs2][31:0]` | Store word to memory. Offset is 13-bit signed. | Generic. |
| **STD** | ST-D | 01-110 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `M[D[rs1] + sext(offset13)][63:0] ← D[rs2][63:0]` | Store doubleword to memory. Offset is 13-bit signed. | Generic. |
| **LDUB** | LDU-B | 10-000 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd], D[rs1] ← D[rs1] + sext(offset13) ← zext(M[D[rs1], D[rs1] ← D[rs1] + sext(offset13) + sext(offset13)], D[rs1] ← D[rs1] + sext(offset13)[byte], D[rs1] ← D[rs1] + sext(offset13))` | Load byte from memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after load. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDUBS** | LDU-BS | 10-001 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd], D[rs1] ← D[rs1] + sext(offset13) ← sext(M[D[rs1], D[rs1] ← D[rs1] + sext(offset13) + sext(offset13)], D[rs1] ← D[rs1] + sext(offset13)[signed byte], D[rs1] ← D[rs1] + sext(offset13))` | Load signed byte from memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after load. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDUH** | LDU-H | 10-010 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd], D[rs1] ← D[rs1] + sext(offset13) ← zext(M[D[rs1], D[rs1] ← D[rs1] + sext(offset13) + sext(offset13)], D[rs1] ← D[rs1] + sext(offset13)[halfword], D[rs1] ← D[rs1] + sext(offset13))` | Load halfword from memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after load. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDUHS** | LDU-HS | 10-011 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd], D[rs1] ← D[rs1] + sext(offset13) ← sext(M[D[rs1], D[rs1] ← D[rs1] + sext(offset13) + sext(offset13)], D[rs1] ← D[rs1] + sext(offset13)[signed halfword], D[rs1] ← D[rs1] + sext(offset13))` | Load signed halfword from memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after load. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDUW** | LDU-W | 10-100 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd], D[rs1] ← D[rs1] + sext(offset13) ← zext(M[D[rs1], D[rs1] ← D[rs1] + sext(offset13) + sext(offset13)], D[rs1] ← D[rs1] + sext(offset13)[word], D[rs1] ← D[rs1] + sext(offset13))` | Load word from memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after load. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDUWS** | LDU-WS | 10-101 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd], D[rs1] ← D[rs1] + sext(offset13) ← sext(M[D[rs1], D[rs1] ← D[rs1] + sext(offset13) + sext(offset13)], D[rs1] ← D[rs1] + sext(offset13)[signed word], D[rs1] ← D[rs1] + sext(offset13))` | Load signed word from memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after load. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **LDUD** | LDU-D | 10-110 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `D[rd], D[rs1] ← D[rs1] + sext(offset13) ← zext(M[D[rs1], D[rs1] ← D[rs1] + sext(offset13) + sext(offset13)], D[rs1] ← D[rs1] + sext(offset13)[doubleword], D[rs1] ← D[rs1] + sext(offset13))` | Load doubleword from memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after load. | Load/store architecture is generic. ARM64 and RISC-V use different encodings. No ISA-level patent risk. |
| **STUB** | STU-B | 11-000 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `M[D[rs1] + sext(offset13)][7:0] ← D[rs2][7:0]; D[rs1] ← D[rs1] + sext(offset13)` | Store byte to memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after store. | Generic. |
| **STUH** | STU-H | 11-010 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `M[D[rs1] + sext(offset13)][15:0] ← D[rs2][15:0]; D[rs1] ← D[rs1] + sext(offset13)` | Store halfword to memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after store. | Generic. |
| **STUW** | STU-W | 11-100 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `M[D[rs1] + sext(offset13)][31:0] ← D[rs2][31:0]; D[rs1] ← D[rs1] + sext(offset13)` | Store word to memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after store. | Generic. |
| **STUD** | STU-D | 11-110 | Class(3)=011 | Type(2) | Width(3) | rd/rs2(5) | rs1(5) | offset13(13) | `M[D[rs1] + sext(offset13)][63:0] ← D[rs2][63:0]; D[rs1] ← D[rs1] + sext(offset13)` | Store doubleword to memory. Offset is 13-bit signed. Post-increment: D[rs1] ← D[rs1] + sext(offset13) after store. | Generic. |

### 100 — Branch & Jump

| 指令 | 类型 | Opcode | 编码格式 | RTL | 语义 | 专利风险 |
|------|------|--------|----------|-----|------|----------|
| **J** | F | 0000 | Class(3)=100 | Opcode(4) | F=0 | rd(5) | imm18(18) | `PC ← PC + sext(imm18)<<2` | Unconditional jump, ±512KB range | Public domain. |
| **JL** | F | 0001 | Class(3)=100 | Opcode(4) | F=0 | rd(5) | imm18(18) | `D[rd] ← PC + 4; PC ← PC + sext(imm18)<<2` | Jump and link (call) | Public domain. |
| **JALR** | G | 0010 | Class(3)=100 | Opcode(4) | F=1 | rd(5) | rs1(5) | imm13(13) | `D[rd] ← PC + 4; PC ← D[rs1] + sext(imm13)` | Jump and link register | Public domain. |
| **RET** | G | 0011 | Class(3)=100 | Opcode(4) | F=1 | rd(5) | rs1(5) | imm13(13) | `PC ← D[rs1] + sext(imm13)` | Return (JALR with rd=0) | Public domain. |
| **BEQ** | E | 0100 | Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13) | `if D[rs1]==D[rs2] PC ← PC + sext(imm13)<<2` | Branch if equal, ±16KB | Public domain. |
| **BNE** | E | 0101 | Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13) | `if D[rs1]!=D[rs2] PC ← PC + sext(imm13)<<2` | Branch if not equal | Public domain. |
| **BLT** | E | 0110 | Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13) | `if D[rs1]<s D[rs2] PC ← PC + sext(imm13)<<2` | Branch if less than (signed) | Public domain. |
| **BGE** | E | 0111 | Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13) | `if D[rs1]>=s D[rs2] PC ← PC + sext(imm13)<<2` | Branch if greater/equal (signed) | Public domain. |
| **BLTU** | E | 1000 | Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13) | `if D[rs1]<u D[rs2] PC ← PC + sext(imm13)<<2` | Branch if less than (unsigned) | Public domain. |
| **BGEU** | E | 1001 | Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13) | `if D[rs1]>=u D[rs2] PC ← PC + sext(imm13)<<2` | Branch if greater/equal (unsigned) | Public domain. |
| **CBZ** | H | 1010 | Class(3)=100 | Opcode(4) | F=1 | rs1(5) | imm18(18) | `if D[rs1]==0 PC ← PC + sext(imm18)<<2` | Compare and branch if zero, ±512KB | CBZ is used in ARM64 but concept of branching on zero is generic. Different encoding avoids conflict. |
| **CBNZ** | H | 1011 | Class(3)=100 | Opcode(4) | F=1 | rs1(5) | imm18(18) | `if D[rs1]!=0 PC ← PC + sext(imm18)<<2` | Compare and branch if not zero | Generic. |
| **BGT** | E | 1100 | Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13) | `if D[rs1]>s D[rs2] PC ← PC + sext(imm13)<<2` | Branch if greater than (signed) | Public domain. |
| **BLE** | E | 1101 | Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13) | `if D[rs1]<=s D[rs2] PC ← PC + sext(imm13)<<2` | Branch if less/equal (signed) | Public domain. |
| **BGTU** | E | 1110 | Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13) | `if D[rs1]>u D[rs2] PC ← PC + sext(imm13)<<2` | Branch if greater than (unsigned) | Public domain. |
| **BLEU** | E | 1111 | Class(3)=100 | Opcode(4) | F=0 | rs1(5) | rs2(5) | imm13(13) | `if D[rs1]<=u D[rs2] PC ← PC + sext(imm13)<<2` | Branch if less/equal (unsigned) | Public domain. |

### 101 — Compare & Set

| 指令 | 类型 | Opcode | 编码格式 | RTL | 语义 | 专利风险 |
|------|------|--------|----------|-----|------|----------|
| **CMP** | R | 0000 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← compare_s(D[rs1], D[rs2])` | Signed comparison: returns -1 if <, 0 if =, 1 if > | Comparison is generic. |
| **CMPU** | R | 0001 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← compare_u(D[rs1], D[rs2])` | Unsigned comparison | Generic. |
| **TST** | R | 0010 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← ((D[rs1] & D[rs2]) != 0) ? 1 : 0` | Test bits (AND and check non-zero) | Generic. |
| **TEQ** | R | 0011 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] == D[rs2]) ? 1 : 0` | Test equal | Generic. |
| **SLT** | R | 0100 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] <s D[rs2]) ? 1 : 0` | Set if less than (signed) | Generic. RISC-V uses different encoding. |
| **SLTU** | R | 0101 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] <u D[rs2]) ? 1 : 0` | Set if less than (unsigned) | Generic. |
| **SGT** | R | 0110 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] >s D[rs2]) ? 1 : 0` | Set if greater than (signed) | Generic. |
| **SGE** | R | 0111 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] >=s D[rs2]) ? 1 : 0` | Set if greater/equal (signed) | Generic. |
| **SLE** | R | 1000 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] <=s D[rs2]) ? 1 : 0` | Set if less/equal (signed) | Generic. |
| **SNE** | R | 1001 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] != D[rs2]) ? 1 : 0` | Set if not equal | Generic. |
| **SGTU** | R | 1010 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] >u D[rs2]) ? 1 : 0` | Set if greater than (unsigned) | Generic. |
| **SGEU** | R | 1011 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] >=u D[rs2]) ? 1 : 0` | Set if greater/equal (unsigned) | Generic. |
| **SLEU** | R | 1100 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] <=u D[rs2]) ? 1 : 0` | Set if less/equal (unsigned) | Generic. |
| **SEQ** | R | 1101 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] == D[rs2]) ? 1 : 0` | Set if equal | Generic. |
| **BTST** | R | 1110 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] >> (D[rs2] & 0x3F)) & 1` | Bit test (extract single bit) | Generic. x86 BT, ARM TST. |
| **BEXT** | R | 1111 | Class(3) | I/R(1)=0 | Opcode(4) | rd(5) | rs1(5) | rs2(5) | Ext(8) | `D[rd] ← (D[rs1] >> (D[rs2] & 0x3F)) & 1` | Bit extract (alias for BTST) | Generic. |
| **CMPI** | I | 0000 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← compare_s(D[rs1], sext(imm13))` | Signed compare with immediate | Generic. |
| **CMPUI** | I | 0001 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← compare_u(D[rs1], uext(imm13))` | Unsigned compare with immediate | Generic. |
| **TSTI** | I | 0010 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← ((D[rs1] & sext(imm13)) != 0) ? 1 : 0` | Test bits with immediate | Generic. |
| **SLTI** | I | 0011 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← (D[rs1] <s sext(imm13)) ? 1 : 0` | Set if less than immediate (signed) | Generic. |
| **SLTIU** | I | 0100 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← (D[rs1] <u uext(imm13)) ? 1 : 0` | Set if less than immediate (unsigned) | Generic. |
| **SGTI** | I | 0101 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← (D[rs1] >s sext(imm13)) ? 1 : 0` | Set if greater than immediate (signed) | Generic. |
| **SGEI** | I | 0110 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← (D[rs1] >=s sext(imm13)) ? 1 : 0` | Set if greater/equal immediate (signed) | Generic. |
| **SLEI** | I | 0111 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← (D[rs1] <=s sext(imm13)) ? 1 : 0` | Set if less/equal immediate (signed) | Generic. |
| **SNEI** | I | 1000 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← (D[rs1] != sext(imm13)) ? 1 : 0` | Set if not equal immediate | Generic. |
| **SEQI** | I | 1001 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← (D[rs1] == sext(imm13)) ? 1 : 0` | Set if equal immediate | Generic. |
| **BTSTI** | I | 1010 | Class(3) | I/R(1)=1 | Opcode(4) | rd(5) | rs1(5) | imm13(13) | `D[rd] ← (D[rs1] >> (imm13 & 0x3F)) & 1` | Bit test immediate | Generic. |

### 110 — Atomic & Memory Barrier

| 指令 | 类型 | Opcode | 编码格式 | RTL | 语义 | 专利风险 |
|------|------|--------|----------|-----|------|----------|
| **LDAR** | A | 0000 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `D[rd] ← M[D[rs1]]; acquire` | Load-acquire (sequentially consistent load) | Acquire/release semantics are generic concepts. ARM LDAR uses different encoding. No ISA patent risk. |
| **STLR** | A | 0001 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `M[D[rs1]] ← D[rs2]; release` | Store-release (sequentially consistent store) | Generic concept. |
| **LD** | A | 0010 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `D[rd] ← M[D[rs1]]` | Plain atomic load (no ordering) | Public domain. |
| **ST** | A | 0011 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `M[D[rs1]] ← D[rs2]` | Plain atomic store (no ordering) | Public domain. |
| **CAS** | A | 0100 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `t ← M[D[rs1]]; if t == D[rs2] then M[D[rs1]] ← D[rd]; D[rd] ← t` | Compare and swap (rd=expected, rs2=desired, returns old) | CAS is generic atomic primitive. IBM, Intel, ARM all have implementations. Concept not patentable. |
| **SWAP** | A | 0101 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `t ← M[D[rs1]]; M[D[rs1]] ← D[rs2]; D[rd] ← t` | Atomic exchange | Generic. |
| **ADD.A** | A | 0110 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `M[D[rs1]] ← M[D[rs1]] + D[rs2]; D[rd] ← old` | Atomic add | Generic. |
| **AND.A** | A | 0111 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `M[D[rs1]] ← M[D[rs1]] & D[rs2]; D[rd] ← old` | Atomic AND | Generic. |
| **OR.A** | A | 1000 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `M[D[rs1]] ← M[D[rs1]] | D[rs2]; D[rd] ← old` | Atomic OR | Generic. |
| **XOR.A** | A | 1001 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `M[D[rs1]] ← M[D[rs1]] ^ D[rs2]; D[rd] ← old` | Atomic XOR | Generic. |
| **LR** | A | 1010 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `D[rd] ← M[D[rs1]]; reserve_addr ← D[rs1]` | Load-reserved (for LL/SC sequence) | LL/SC is generic (MIPS, RISC-V, ARM). Concept not patentable. |
| **SC** | A | 1011 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `if reserve_addr == D[rs1] then M[D[rs1]] ← D[rs2]; D[rd] ← 0 else D[rd] ← 1` | Store-conditional | Generic. |
| **FENCE** | A | 1100 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `Full memory barrier` | All prior memory ops visible before subsequent ops | Memory fences are generic. |
| **FENCE.R** | A | 1101 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `Read fence` | Load-load/load-store ordering barrier | Generic. |
| **FENCE.W** | A | 1110 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `Write fence` | Store-store/store-load ordering barrier | Generic. |
| **FENCE.TSO** | A | 1111 | Class(3)=110 | Opcode(4) | S(1) | rd(5) | rs1(5) | rs2(5) | Model(8) | `TSO fence` | Emits sufficient barriers to enforce TSO ordering (for x86 BT) | TSO is academic concept (SPARC). Software emulation via fences does not infringe hardware TSO patents. |


---

## 寄存器约定

| 寄存器 | 名称 | 用途 |
|--------|------|------|
| D0 | ZERO | 硬连线 0，写入被忽略 |
| D1 | RA | 返回地址 (JAL/JALR 默认目标) |
| D2-D3 | A0-A1 | 函数参数 / 返回值 |
| D4-D7 | A2-A5 | 函数参数 |
| D8-D15 | T0-T7 | 临时寄存器 (调用者保存) |
| D16-D23 | S0-S7 | 保存寄存器 (被调用者保存) |
| D24 | TP | 线程指针 |
| D25 | GP | 全局指针 |
| D26 | SP | 栈指针 |
| D27 | FP | 帧指针 |
| D28 | LR | 链接寄存器 (软件约定) |
| D29 | TMP0 | 汇编器临时 |
| D30 | TMP1 | 汇编器临时 |
| D31 | TMP2 | 汇编器临时 / 标志寄存器映射 |

---

## 标志寄存器 (隐式)

通过专用寄存器 D31 的低 8 位实现（软件可读/写，硬件自动更新）：

| 位 | 名称 | 含义 |
|----|------|------|
| 0 | Z | Zero flag |
| 1 | N | Negative flag |
| 2 | C | Carry flag |
| 3 | V | Overflow flag |
| 4 | T | TSO mode (0=ARM weak, 1=x86 TSO) |
| 5 | I | Interrupt enable |
| 6 | P | Privilege mode (0=U, 1=S, 2=M) |
| 7 | - | Reserved |

---

*文档版本: NIS Basic32 v0.0.1 | 生成时间: 2026-05-27*
