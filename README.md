```markdown
# 🖥️ Minimal x86 Bootloader

A simple project experimenting with writing a bootloader in **x86 Assembly**, building a bootable floppy disk image, and running it on QEMU.  
This is the foundation for learning **OS development** from scratch.

---

## 🚀 Setup

```bash
brew install nasm qemu make
```

---

## 🔄 Boot Flow

```
Power On → POST (Power-On Self Test) → BIOS → Bootloader → Operating System
```

---

## 📂 Build & Run

### Build
```bash
make
```

### Run with QEMU
```bash
qemu-system-i386 -fda build/main_floppy.img -drive format=raw
```

---

## 📚 Learning Notes

<details>
<summary>🔎 How the BIOS finds an OS</summary>

- **Legacy booting**
  - BIOS loads the first sector of each bootable device into memory (at location `0x7C00`).
  - BIOS checks for `0xAA55` signature.
  - If found, it starts executing code.
  - BIOS loads the OS from the first valid device (USB, SSD, etc.).
  - The precedence among devices can be modified in BIOS settings (boot priority).
</details>

<details>
<summary>📝 Symbols in NASM</summary>

- `$`: memory offset of the current line  
- `$$`: memory offset of the beginning of the section  
</details>

<details>
<summary>⚙️ Directives</summary>

Directives give hints to the assembler and are **not executed by the CPU**.  

- `ORG`: specifies where the code is expected to be loaded (e.g., `org 0x7C00`)  
- `BITS`: tells assembler to emit 16/32/64-bit code (e.g., `bits 16`)  
  - CPU starts in **Real Mode** → `bits 16` in bootloader  
  - Later, OS switches to Protected Mode (32-bit) → Long Mode (64-bit)  
- `DB`: define byte(s)  
- `DW`: define word(s) (2 bytes, little endian)  
- `TIMES`: repeat instruction or data N times  
</details>

<details>
<summary>⚡ Instructions</summary>

- `HLT`: stop CPU execution (resume on interrupt)  
- `JMP location`: unconditional jump (like C `goto`)  
- `MOV dest, src`: copy data between registers/memory/immediates  

Examples:
```nasm
mov ax, 5        ; AX = 5
mov ax, [var]    ; AX = memory[var]
mov [var], ax    ; memory[var] = AX
mov bx, ax       ; BX = AX
```

Other useful:
- `LODSB/LODSW/LODSD`: load from DS:SI into AL/AX/EAX  
- `OR dest, src`: bitwise OR, affects flags (ZF=1 if result=0)  
- `JZ location`: jump if Zero Flag is set  

**Infinite loop (boot code often stays running):**
```nasm
main:
    hlt
.halt:
    jmp .halt
```
</details>

<details>
<summary>📐 x86 Segment:Offset Addressing</summary>

- 8086 had 16-bit registers → directly addressable: 64KB  
- Actual memory: 1MB → solution: segment × 16 + offset  

Formula:
```nasm
Physical Address = Segment × 16 + Offset
```

- Segment registers: CS, DS, SS, ES  
- Offset: IP, SP, or general register  

**General Form:**
```
segment : [ base + index * scale + displacement ]
```

The segment (×16) gives the 64KB window start, and the offset gives the position within that window.
</details>
```
