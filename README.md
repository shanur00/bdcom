# x86 Addressing Modes Explained Simply

This table shows different ways to **access data** in x86 assembly.  Let me explain each one with easy examples! 

---

## Overview: Where Can Data Be? 

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│   1.  IMMEDIATE  →  Value is in the instruction itself     │
│   2. REGISTER   →  Value is in a CPU register              │
│   3. MEMORY     →  Value is in RAM (need address)          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 1.  Immediate Mode

**Value is written directly in the instruction**

| Syntax | Meaning | Example |
|--------|---------|---------|
| `$Imm` | Use this exact number | `$5` means the number 5 |

```asm
movl $100, %eax      ; Put the number 100 into EAX

; $100 is IMMEDIATE - the value 100 is right there in the code!
```

```
┌─────────────────────────────────────┐
│  Instruction: movl $100, %eax       │
│                     │               │
│                     ▼               │
│              Value = 100            │
│                     │               │
│                     ▼               │
│               ┌─────────┐           │
│               │ EAX=100 │           │
│               └─────────┘           │
└─────────────────────────────────────┘
```

---

## 2. Register Mode

**Value is already in a register**

| Syntax | Meaning | Example |
|--------|---------|---------|
| `rₐ` | Use value in register rₐ | `%eax` means value in EAX |

```asm
movl %eax, %ebx      ; Copy value from EAX to EBX

; %eax is REGISTER mode - value comes from the register
```

```
┌─────────────────────────────────────┐
│                                     │
│   ┌─────────┐       ┌─────────┐     │
│   │ EAX=50  │ ───▶  │ EBX=50  │     │
│   └─────────┘       └─────────┘     │
│    (source)         (destination)   │
│                                     │
└─────────────────────────────────────┘
```

---

## 3. Memory Modes (This is where it gets interesting!)

### 3a. Absolute (Direct) — `M[Imm]`

**Go directly to a fixed memory address**

| Syntax | Formula | Example |
|--------|---------|---------|
| `Imm` | `M[Imm]` | Address is the number itself |

```asm
movl 0x1000, %eax    ; Load value from memory address 0x1000

; 0x1000 is a fixed address - always goes to same place
```

```
      Memory
    ┌─────────┐
    │         │ 0x0FFC
    ├─────────┤
    │   42    │ 0x1000  ◄── Go directly here!
    ├─────────┤
    │         │ 0x1004
    └─────────┘
         │
         ▼
    ┌─────────┐
    │ EAX=42  │
    └─────────┘
```

---

### 3b.  Indirect — `M[R[rₐ]]`

**Register holds the address, go to that address**

| Syntax | Formula | Example |
|--------|---------|---------|
| `(rₐ)` | `M[R[rₐ]]` | Address is inside the register |

```asm
movl (%eax), %ebx    ; EAX contains an address, load value from that address

; If EAX = 0x2000, this loads value from memory address 0x2000
```

```
    ┌─────────────┐
    │ EAX = 0x2000│ ──┐    (EAX holds the ADDRESS)
    └─────────────┘   │
                      │
                      ▼
      Memory         
    ┌─────────┐      
    │   77    │ 0x2000 ◄── Go to address stored in EAX
    └─────────┘      
         │
         ▼
    ┌─────────┐
    │ EBX=77  │
    └─────────┘
```

---

### 3c. Base + Displacement — `M[Imm + R[rᵦ]]`

**Start from register address, add a fixed offset**

| Syntax | Formula | Example |
|--------|---------|---------|
| `Imm(rᵦ)` | `M[Imm + R[rᵦ]]` | Address = Offset + Base register |

```asm
movl 8(%ebp), %eax   ; Address = EBP + 8

; If EBP = 0x1000, this accesses address 0x1008
```

```
    ┌─────────────┐
    │ EBP = 0x1000│    (Base)
    └─────────────┘
           +
    ┌─────────────┐
    │ Offset = 8  │    (Displacement)
    └─────────────┘
           =
    ┌─────────────┐
    │ Addr= 0x1008│
    └─────────────┘
           │
           ▼
      Memory
    ┌─────────┐
    │   99    │ 0x1008 ◄── EBP + 8
    └─────────┘
```

**Common use:** Accessing local variables and function parameters! 

```asm
movl -4(%ebp), %eax   ; Local variable 1 (EBP - 4)
movl -8(%ebp), %eax   ; Local variable 2 (EBP - 8)
movl  8(%ebp), %eax   ; Function parameter 1 (EBP + 8)
```

---

### 3d. Indexed — `M[R[rᵦ] + R[rᵢ]]`

**Add two registers to get address**

| Syntax | Formula | Example |
|--------|---------|---------|
| `(rᵦ, rᵢ)` | `M[R[rᵦ] + R[rᵢ]]` | Address = Base + Index |

```asm
movl (%eax, %ebx), %ecx   ; Address = EAX + EBX

; If EAX = 0x1000 and EBX = 0x20, accesses address 0x1020
```

```
    ┌─────────────┐
    │ EAX = 0x1000│    (Base)
    └─────────────┘
           +
    ┌─────────────┐
    │ EBX = 0x20  │    (Index)
    └─────────────┘
           =
    ┌─────────────┐
    │ Addr= 0x1020│
    └─────────────┘
```

---

### 3e.  Indexed with Displacement — `M[Imm + R[rᵦ] + R[rᵢ]]`

**Offset + Base register + Index register**

| Syntax | Formula | Example |
|--------|---------|---------|
| `Imm(rᵦ, rᵢ)` | `M[Imm + R[rᵦ] + R[rᵢ]]` | Address = Offset + Base + Index |

```asm
movl 100(%eax, %ebx), %ecx   ; Address = 100 + EAX + EBX

; If EAX = 0x1000, EBX = 0x20 → Address = 100 + 0x1000 + 0x20 = 0x1084
```

---

### 3f.  Scaled Indexed — `M[R[rᵢ] · s]`

**Multiply index by a scale factor (1, 2, 4, or 8)**

| Syntax | Formula | Example |
|--------|---------|---------|
| `(, rᵢ, s)` | `M[R[rᵢ] · s]` | Address = Index × Scale |

```asm
movl (,%eax,4), %ebx   ; Address = EAX × 4

; If EAX = 5 → Address = 5 × 4 = 20
```

**Why scale? ** For arrays! 

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   Array of integers (each 4 bytes):                         │
│                                                             │
│   Index:    0       1       2       3       4               │
│           ┌─────┬─────┬─────┬─────┬─────┐                   │
│   Memory: │ 10  │ 20  │ 30  │ 40  │ 50  │                   │
│           └─────┴─────┴─────┴─────┴─────┘                   │
│   Address: 0     4     8     12    16                       │
│                                                             │
│   To access arr[3]:                                         │
│   Index = 3, Scale = 4 (size of int)                        │
│   Address = 3 × 4 = 12 ✓                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### 3g.  Scaled Indexed + Displacement — `M[Imm + R[rᵢ] · s]`

**Offset + (Index × Scale)**

| Syntax | Formula | Example |
|--------|---------|---------|
| `Imm(, rᵢ, s)` | `M[Imm + R[rᵢ] · s]` | Address = Offset + (Index × Scale) |

```asm
movl 0x1000(,%eax,4), %ebx   ; Address = 0x1000 + (EAX × 4)

; If EAX = 3 → Address = 0x1000 + 12 = 0x100C
```

**Use case:** Array with known base address

```
   Array starts at 0x1000
   
   arr[0] at 0x1000 + (0 × 4) = 0x1000
   arr[1] at 0x1000 + (1 × 4) = 0x1004
   arr[2] at 0x1000 + (2 × 4) = 0x1008
   arr[3] at 0x1000 + (3 × 4) = 0x100C  ◄── EAX = 3
```

---

### 3h.  Scaled Indexed + Base — `M[R[rᵦ] + R[rᵢ] · s]`

**Base register + (Index × Scale)**

| Syntax | Formula | Example |
|--------|---------|---------|
| `(rᵦ, rᵢ, s)` | `M[R[rᵦ] + R[rᵢ] · s]` | Address = Base + (Index × Scale) |

```asm
movl (%eax,%ebx,4), %ecx   ; Address = EAX + (EBX × 4)

; If EAX = 0x2000, EBX = 5 → Address = 0x2000 + 20 = 0x2014
```

**Use case:** Array where base address is in a register

```asm
; C code: int arr[10]; value = arr[i];
; EAX = address of arr
; EBX = index i

movl (%eax,%ebx,4), %ecx   ; ECX = arr[i]
```

---

### 3i. Full Form — `M[Imm + R[rᵦ] + R[rᵢ] · s]`

**Most complete: Offset + Base + (Index × Scale)**

| Syntax | Formula | Example |
|--------|---------|---------|
| `Imm(rᵦ, rᵢ, s)` | `M[Imm + R[rᵦ] + R[rᵢ] · s]` | Everything combined!  |

```asm
movl 8(%eax,%ebx,4), %ecx   ; Address = 8 + EAX + (EBX × 4)

; If EAX = 0x1000, EBX = 2 → Address = 8 + 0x1000 + 8 = 0x1010
```

# Understanding MOV Instructions — Line by Line

Let me break down each sentence in simple terms!  

---

## Line 1: Source Operand

> *"The source operand designates a value that is immediate, stored in a register, or stored in memory."*

### Meaning: **Where can data COME FROM?**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   SOURCE (where data comes from) can be:                    │
│                                                             │
│   1.   IMMEDIATE  →  A constant value like $100              │
│   2.  REGISTER   →  A register like %eax                    │
│   3. MEMORY     →  A memory address like (%ebx)            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

```asm
movl $100, %eax       ; Source = Immediate ($100)
movl %ebx, %eax       ; Source = Register (%ebx)
movl (%ecx), %eax     ; Source = Memory (address in %ecx)
```

---

## Line 2: Destination Operand

> *"The destination operand designates a location that is either a register or a memory address."*

### Meaning: **Where can data GO TO?**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   DESTINATION (where data goes) can be:                     │
│                                                             │
│   1.  REGISTER  →  Like %eax                                │
│   2. MEMORY    →  Like (%ebx) or 0x1000                     │
│                                                             │
│   ✗ NOT immediate!  (Can't store into a number)             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

```asm
movl $100, %eax       ; Destination = Register ✓
movl $100, (%ebx)     ; Destination = Memory ✓
movl $100, $200       ; Destination = Immediate ✗ IMPOSSIBLE!
```

**Why?** You can't store a value INTO a constant number — that makes no sense!  

---

## Line 3: Memory-to-Memory Restriction

> *"x86-64 imposes the restriction that a move instruction cannot have both operands refer to memory locations."*

### Meaning: **Can't move directly from memory to memory! **

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   ✗ ILLEGAL:                                                │
│                                                             │
│   movl (%eax), (%ebx)    ; Memory → Memory = NOT ALLOWED!   │
│                                                             │
│         Memory              Memory                          │
│        ┌──────┐            ┌──────┐                         │
│        │  50  │  ─────✗──▶│      │                         │
│        └──────┘            └──────┘                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Line 4: Two-Step Solution

> *"Copying a value from one memory location to another requires two instructions—the first to load the source value into a register, and the second to write this register value to the destination."*

### Meaning: **Use a register as a "middleman"**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   To copy Memory → Memory, use 2 steps:                     │
│                                                             │
│   Step 1: Memory → Register                                 │
│   Step 2: Register → Memory                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

```asm
# Goal: Copy value from address in %eax to address in %ebx

movl (%eax), %ecx     ; Step 1: Load from memory into register
movl %ecx, (%ebx)     ; Step 2: Store from register into memory
```

```
    Memory A           Register           Memory B
   ┌──────┐           ┌──────┐           ┌──────┐
   │  50  │ ─Step1──▶│  50  │ ──Step2──▶│  50  │
   └──────┘           └──────┘           └──────┘
   (%eax)              %ecx               (%ebx)
```

---

## Line 5: Register Size Must Match

> *"Register operands for these instructions can be the labeled portions of any of the 16 registers, where the size of the register must match the size designated by the last character of the instruction ('b', 'w', 'l', or 'q')."*

### Meaning: **Instruction suffix must match register size**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   Suffix    Size        Register Example                    │
│   ──────    ────        ─────────────────                   │
│     b       1 byte      %al, %bl, %cl                       │
│     w       2 bytes     %ax, %bx, %cx                       │
│     l       4 bytes     %eax, %ebx, %ecx                    │
│     q       8 bytes     %rax, %rbx, %rcx                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

```asm
movb $100, %al        ; b = byte (1) → %al (1 byte) ✓
movw $100, %ax        ; w = word (2) → %ax (2 bytes) ✓
movl $100, %eax       ; l = long (4) → %eax (4 bytes) ✓
movq $100, %rax       ; q = quad (8) → %rax (8 bytes) ✓

movl $100, %al        ; l (4 bytes) with %al (1 byte) ✗ MISMATCH!
```

---

## Line 6: Partial Register Update

> *"For most cases, the mov instructions will only update the specific register bytes or memory locations indicated by the destination operand."*

### Meaning: **Only the specified bytes change, rest stays same**

```asm
# Assume %rax = 0x1234567890ABCDEF (64 bits)

movb $0xFF, %al       ; Only changes lowest 1 byte
```

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   BEFORE: %rax = 0x1234567890ABCDEF                         │
│                                                             │
│   ┌────┬────┬────┬────┬────┬────┬────┬────┐                 │
│   │ 12 │ 34 │ 56 │ 78 │ 90 │ AB │ CD │ EF │                 │
│   └────┴────┴────┴────┴────┴────┴────┴────┘                 │
│                                          ▲                  │
│                                          │                  │
│                                         %al (this byte)     │
│                                                             │
│   movb $0xFF, %al                                           │
│                                                             │
│   AFTER: %rax = 0x1234567890ABCDFF                          │
│                                                             │
│   ┌────┬────┬────┬────┬────┬────┬────┬────┐                 │
│   │ 12 │ 34 │ 56 │ 78 │ 90 │ AB │ CD │ FF │ ← Only this     │
│   └────┴────┴────┴────┴────┴────┴────┴────┘   changed!      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Line 7 & 8: The `movl` Exception! 

> *"The only exception is that when movl has a register as the destination, it will also set the high-order 4 bytes of the register to 0."*

> *"This exception arises from the convention, adopted in x86-64, that any instruction that generates a 32-bit value for a register also sets the high-order portion of the register to 0."*

### Meaning: **`movl` to register = upper 32 bits become ZERO! **

```asm
# Assume %rax = 0xFFFFFFFFFFFFFFFF (all 1s)

movl $0x12345678, %eax    ; Writes to lower 32 bits AND clears upper 32! 
```

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   BEFORE: %rax = 0xFFFFFFFFFFFFFFFF                         │
│                                                             │
│   ┌────────────────────┬────────────────────┐               │
│   │   FF FF FF FF      │   FF FF FF FF      │               │
│   │   (upper 32 bits)  │   (lower 32 bits)  │               │
│   └────────────────────┴────────────────────┘               │
│                                                             │
│   movl $0x12345678, %eax                                    │
│                                                             │
│   AFTER: %rax = 0x0000000012345678                          │
│                                                             │
│   ┌────────────────────┬────────────────────┐               │
│   │   00 00 00 00      │   12 34 56 78      │               │
│   │   (ZEROED!  )      │   (new value)      │               │
│   └────────────────────┴────────────────────┘               │
│                                                             │
│   Upper 32 bits automatically become 0!                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Comparison: `movb`, `movw` vs `movl`

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   %rax starts as: 0xFFFFFFFFFFFFFFFF                        │
│                                                             │
│   ┌────────────┬─────────────────────────────────────────┐  │
│   │ Instruction│  Result in %rax                         │  │
│   ├────────────┼─────────────────────────────────────────┤  │
│   │ movb $1,%al│  0xFFFFFFFFFFFFFF01  (only 1 byte)      │  │
│   ├────────────┼─────────────────────────────────────────┤  │
│   │ movw $1,%ax│  0xFFFFFFFFFFFF0001  (only 2 bytes)     │  │
│   ├────────────┼─────────────────────────────────────────┤  │
│   │movl $1,%eax│  0x0000000000000001  (upper CLEARED!)   │  │
│   ├────────────┼─────────────────────────────────────────┤  │
│   │movq $1,%rax│  0x0000000000000001  (full 8 bytes)     │  │
│   └────────────┴─────────────────────────────────────────┘  │
│                                                             │
│   movb, movw → Keep upper bits                              │
│   movl       → Clear upper 32 bits (EXCEPTION!)             │
│   movq       → Writes all 64 bits                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
