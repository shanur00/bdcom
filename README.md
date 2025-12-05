# x86 Addressing Modes Explained Simply

This table shows different ways to **access data** in x86 assembly.  Let me explain each one with easy examples! 

---

## Overview: Where Can Data Be? 

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   1.  IMMEDIATE  →  Value is in the instruction itself      │
│   2. REGISTER   →  Value is in a CPU register              │
│   3. MEMORY     →  Value is in RAM (need address)          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
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
