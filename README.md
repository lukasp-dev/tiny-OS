# 🖥️ Minimal x86 Bootloader (OS Learning Repository)

This repository documents my journey of **learning OS development from scratch**.  
The first step is building a **bootloader** in x86 assembly, packaging it into a floppy disk image, and running it on QEMU.

---

## 🚀 Setup

Install required tools (macOS example):

```bash
brew install nasm qemu make
```

---

## 🔄 Boot Flow

```
Power On → POST (Power-On Self Test) → BIOS → Bootloader → Operating System
```

- **BIOS legacy booting**
  - BIOS loads the first sector (512 bytes) of the boot device into memory at `0x7C00`.
  - If the sector ends with the signature `0xAA55`, BIOS considers it bootable.
  - Execution starts at `0x7C00`.
  - Boot device order (floppy, HDD, USB, etc.) can be changed in BIOS settings.

---

## 📂 Build & Run

### Makefile

The build process is already automated with a `Makefile`:

```make
ASM=nasm

SRC_DIR=src
BUILD_DIR=build

$(BUILD_DIR)/main_floppy.img: $(BUILD_DIR)/main.bin
        cp $(BUILD_DIR)/main.bin $(BUILD_DIR)/main_floppy.img
        truncate -s 1440k $(BUILD_DIR)/main_floppy.img

$(BUILD_DIR)/main.bin: $(SRC_DIR)/main.asm
        $(ASM) $(SRC_DIR)/main.asm -f bin -o $(BUILD_DIR)/main.bin
```

### Build
```bash
make
```

### Run
```bash
qemu-system-i386 -fda build/main_floppy.img -drive format=raw
```

---

## 📚 Learning Notes

<details>
<summary>📝 NASM Basics</summary>

- `$`: offset of the current line  
- `$$`: offset of the beginning of the section  

**Directives**
- `ORG`: origin, expected load address (`org 0x7C00`)  
- `BITS`: emit 16/32/64-bit code (bootloader starts in 16-bit real mode)  
- `DB`: define byte  
- `DW`: define word (2 bytes, little endian)  
- `TIMES`: repeat instruction/data  

**Instructions**
- `HLT`: halt CPU until next interrupt  
- `JMP`: unconditional jump  
- `MOV`: move data between registers/memory  

Example infinite loop:
```nasm
main:
    hlt
.halt:
    jmp .halt
```
</details>

<details>
<summary>📐 x86 Segmented Addressing</summary>

- 8086 had 16-bit registers → could only directly address 64KB.  
- But actual memory was 1MB → solution: **segment × 16 + offset**.

Formula:
```nasm
Physical Address = Segment × 16 + Offset
```

- Segment registers: CS, DS, SS, ES  
- Offset: IP, SP, or general-purpose register  

Example: `CS=0x07C0, IP=0x0000` → Physical address `0x7C00`
</details>

<details>
<summary>💾 Floppy Disk & .img</summary>

- A floppy disk is a legacy portable storage medium (1.44MB capacity for 3.5").  
- In OSDev, we use `.img` files to emulate an entire floppy disk.  
- `truncate -s 1440k main_floppy.img` creates an empty 1.44MB floppy image.  
- BIOS loads only the **first 512 bytes (boot sector)** into RAM at `0x7C00`.  
- QEMU uses `.img` as if it were a real disk.
</details>

<details>
<summary>🖥️ QEMU</summary>

- QEMU = Quick Emulator, a program that simulates an entire PC.  
- With `-fda main_floppy.img`, it treats the `.img` file as a floppy disk in drive A:.  
- BIOS inside QEMU:
  - Reads first 512 bytes from `.img`
  - Copies them into `0x7C00`
  - Starts executing your assembly code there
</details>
