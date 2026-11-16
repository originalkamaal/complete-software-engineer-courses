---
title: "Linux Kernel & OS Development from Zero to Advanced"
subtitle: "Build an operating system kernel from the first boot sector to processes, files, and multiple CPUs — then work inside Linux 6.12: modules, drivers, memory, scheduling, locking, RCU, tracing, networking, BPF, and upstream patches"
lang: en
---

# Preface

An operating system kernel is the program that every other program trusts. It decides which code runs, which memory it may touch, and what happens when something goes wrong. Learning it from documentation alone is hard, because the interesting parts happen where nobody is watching: during boot, inside an interrupt, between two CPUs.

This book teaches the kernel by building and running one. **Part I (Chapters 0–7)** builds a small 64-bit kernel from nothing: a boot sector and a UEFI application, the switch to long mode, C without a library, interrupts and the timer, physical and virtual memory with demand paging, a kernel heap, threads and preemption, user mode and system calls, a disk driver and a file system with its own `mkfs` tool, an ELF loader and a shell in user space, and finally the other CPUs, atomics, and locks. **Part II (Chapters 8–14)** moves to Linux 6.12: building and booting it under QEMU, writing modules and a character device driver, measuring processes, memory, and scheduling through `/proc`, using lockdep, KASAN, RCU, ftrace, and kprobes, following packets through the network stack, loading eBPF programs with the raw system call, and preparing an upstream-style patch series.

Three promises shape every page:

1. **Precision.** Each *Deep & Tricky* fact is labelled with its source of truth — the Intel Software Developer's Manual, the Multiboot2 or ELF specification, the C standard, a datasheet, an RFC, or a named file in the Linux source or its `Documentation/` tree.
2. **Evidence.** Every program shown with output was run, and the output is real: QEMU exit statuses, kernel panics, lockdep and KASAN reports, page-fault counts, verifier logs, packet captures. Most *Deep & Tricky* entries come with an experiment switch that builds the wrong version so you can watch it fail. Several examples were wrong in their first draft, and the book shows what happened when that taught something: a bitmap allocator that destroyed the bootloader's data, a timer that stopped because of a missing EOI, a ticket lock that collapsed under virtualisation, a NULL dereference that KASAN turned into a general protection fault.
3. **Order.** Nothing is used before it is taught; Part II refers back to the hobby kernel wherever Linux solves the same problem.

# How to use this book

## Who it is for

- **C programmers new to systems work:** read in order. Part I gives you a working mental model of the machine before Linux's scale arrives.
- **Linux users and application developers:** Chapters 0–2 can be read quickly; read Chapters 3–7 carefully — interrupts, paging, and concurrency are what Part II assumes.
- **Engineers who want to work on Linux:** you may start Part II at Chapter 8, but read every *Deep & Tricky* entry in Part I first; Chapters 9, 10, 12, and 14 are the core of everyday kernel work.

You should be comfortable with C — pointers, structs, the preprocessor, and building with `gcc` and `make` — and with the command line. The C and DSA books of this series cover what you need.

## The shape of a unit

| Part | What it gives you |
| --- | --- |
| **One-line summary** | The whole idea in one sentence. |
| **Lesson** | Complete code — assembly, C, scripts — and its real output. |
| **Deep & Tricky (DT-n)** | Precise facts that surprise experienced developers, labelled with their source of truth, usually with an experiment that shows the failure. |
| **Patterns (P-n)** | Engineering practices with their trade-offs. |
| **Mastery check** | *Predict* questions; answers at the end of each chapter. |

Each chapter ends with a **Chapter Project**: a realistic piece of work with complete code, real output, and a review checklist.

## The hobby kernel grows chapter by chapter

From Chapter 2 on, the kernel lives in a directory `os/`. Each chapter starts from the previous chapter's finished kernel, adds or replaces files from its `new/` directory, and its project adds `proj/` to make `final/`. A file listing in the text is always the complete file. The kernel runs headless in QEMU: output goes to the serial port, and the kernel ends a run by writing to QEMU's `isa-debug-exit` device, so every run ends with an exit status — 33 for success, 35 for a kernel panic, 124 when the time limit was hit. Tests are registered with `KTEST(name)` and selected with `make run CMDLINE="test=name"`; Chapter 14 runs all of them.

## Running the code

The source archive has one directory per chapter, with the files exactly as listed (each starts with a one-line `// file:` or `# file:` comment that names it). The console blocks show the exact commands, run from the chapter's directory.

**Part I** needs a Linux x86-64 machine (or VM) with `gcc`, `nasm`, `qemu-system-x86_64`, GRUB's `grub-mkrescue` with `xorriso` and the PC (`grub-pc-bin`) modules, OVMF (UEFI firmware for QEMU, Chapter 0), `clang` and `lld` (Chapter 0's UEFI program), and Python 3. On Debian or Ubuntu: `apt install build-essential nasm qemu-system-x86 grub-pc-bin grub-common xorriso mtools ovmf clang lld python3`.

**Part II** needs, in addition, the kernel build dependencies (`flex bison bc libelf-dev libssl-dev dwarves`), `busybox-static`, `cpio`, `kmod`, `curl`, `python3-ply` and `python3-git` (for `checkpatch.pl`'s SPDX check), and the Linux 6.12 source: `git clone --depth 1 --branch v6.12 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git`. Chapter 8 builds it out of tree with this book's configuration (`book_defconfig`, listed in Unit 8.2). In the code directories, `linux` and `build` are symbolic links to the source and build directories; adjust them to your paths.

Everything was run on Ubuntu 24.04 with GCC 13.3, NASM 2.16, QEMU 8.2, GRUB 2.12, and Linux 6.12, on a two-core x86-64 host without hardware virtualisation. Timings — boot times, lock throughput, the cost of `synchronize_rcu` — are from that machine under emulation and with a debug kernel; expect different numbers on yours, and the same shape.

## Reading order

| Part | Chapters | Builds on |
| --- | --- | --- |
| I — Building a kernel | 0 Boot · 1 x86-64 · 2 Freestanding C · 3 Interrupts · 4 Memory · 5 Processes · 6 Disks and files · 7 SMP and locks | C, DSA |
| II — Working in Linux | 8 Build and boot · 9 Modules · 10 Char devices · 11 Processes, memory, scheduling · 12 Locking, RCU, debugging, tracing · 13 Networking and BPF · 14 Testing and patches | Part I |

# Chapter 0 — How Computers Boot

*Goal: follow a PC from power-on to the first instruction of an operating system — firmware (legacy BIOS and UEFI), the 512-byte boot sector, multi-stage bootloaders, GRUB, and the Multiboot2 protocol that this book's hobby kernel will use — by writing and running each stage in QEMU.*

Every program in Part I of this book runs in **QEMU**, a machine emulator, with no display: the guest writes to an emulated serial port (COM1), QEMU copies that to the terminal, and the guest ends the run by writing to QEMU's `isa-debug-exit` device. A guest that writes value *v* makes QEMU exit with status `(v << 1) | 1` — so every successful run below ends with status 33 (v = 0x10). This gives us fully automated, repeatable runs of code that normally needs real hardware.

**What you will be able to do after this chapter**

- Explain what happens between power-on and the operating system: reset vector, firmware, boot device, bootloader.
- Write a legacy BIOS boot sector in 16-bit assembly that loads a second stage from disk.
- Write a UEFI application in C and boot it with OVMF firmware.
- Describe GRUB and the Multiboot2 protocol, and verify a kernel's Multiboot2 header.

## Topic 0A — Firmware

## Unit 0.1 — From power-on to firmware

**One-line summary:** When an x86 CPU is reset, it starts in 16-bit **real mode** at physical address **0xFFFFFFF0** (the reset vector), which the chipset maps to the **firmware** ROM. The firmware tests and initialises the hardware (memory controller, buses, devices), then finds a boot device and hands control to the code it loads from there. Two firmware families exist: legacy **BIOS** (1981 onwards; loads one 512-byte sector) and **UEFI** (the modern standard; loads a PE executable from a FAT partition).

```text
power on → CPU reset (real mode, CS:IP = F000:FFF0 → 0xFFFFFFF0)
        → firmware: initialise RAM, PCI, devices; build memory map and tables (ACPI, SMBIOS)
        → BIOS: read sector 0 of the boot disk to 0x7C00, check 0x55AA, jump to it
          UEFI: read EFI/BOOT/BOOTX64.EFI from the EFI system partition, run it in 64-bit mode
        → bootloader (GRUB, systemd-boot, Windows Boot Manager, …)
        → kernel
```

QEMU's default firmware is **SeaBIOS**, an open-source BIOS. It writes a log to the I/O port 0x402, which QEMU's `-debugcon` option can capture (the `-global` option places the debug console at that port). The first lines show the firmware at work before any boot code runs:

```console
$ timeout 5 qemu-system-x86_64 -display none -debugcon file:seabios.log -global isa-debugcon.iobase=0x402 -no-reboot > /dev/null 2>&1; head -6 seabios.log | cut -c1-90
SeaBIOS (version 1.16.3-debian-1.16.3-2)
BUILD: gcc: (Ubuntu 13.2.0-7ubuntu1) 13.2.0 binutils: (GNU Binutils for Ubuntu) 2.41
No Xen hypervisor found.
Running on QEMU (i440fx)
physbits: signature="AuthenticAMD", pae=yes, lm=yes, phys-bits=40, valid=yes
cpuid 0x40000000: eax 40000001, signature 'TCGTCGTCGTCG'
```

## Unit 0.2 — The legacy BIOS boot sector

**One-line summary:** A BIOS reads the first 512-byte sector of the boot disk into memory at **0x7C00** and jumps to it if the sector ends with the **signature bytes 0x55 0xAA**. The CPU is in 16-bit real mode; the BIOS passes the boot drive number in register `DL` and offers services through software interrupts (`int 0x10` video, `int 0x13` disk, `int 0x15` memory map). 510 bytes of code is too little for an OS, so the boot sector's job is to load a bigger **second stage**.

```nasm
; file: boot.asm
bits 16
org 0x7C00
start:
    jmp 0:.norm                ; normalise CS:IP to 0000:7C00 (some BIOSes use 07C0:0000)
.norm:
    xor ax, ax
    mov ds, ax
    mov es, ax
    mov ss, ax
    mov sp, 0x7C00             ; stack grows down, below the boot sector
    mov [boot_drive], dl       ; the BIOS passes the boot drive number in DL
    mov si, msg1
    call print
    ; load sector 2 to 0x7E00 with BIOS int 13h, function 02h (cylinder/head/sector addressing)
    mov ah, 0x02
    mov al, 1                  ; number of sectors
    mov ch, 0                  ; cylinder
    mov cl, 2                  ; sector (numbered from 1)
    mov dh, 0                  ; head
    mov dl, [boot_drive]
    mov bx, 0x7E00             ; ES:BX = destination
    int 0x13
    jc disk_error              ; carry flag set on failure
    jmp 0x7E00                 ; run stage 2
disk_error:
    mov si, err
    call print
    hlt

print:                         ; SI -> zero-terminated string, written to COM1 (port 0x3F8)
    lodsb
    test al, al
    jz .done
    mov dx, 0x3F8 + 5          ; line status register
.wait:
    mov ah, al
    in al, dx
    test al, 0x20              ; transmitter ready?
    mov al, ah
    jz .wait
    mov dx, 0x3F8
    out dx, al
    jmp print
.done:
    ret

boot_drive: db 0
msg1: db "Stage 1: BIOS loaded 512 bytes at 0x7C00", 13, 10, 0
err:  db "disk error", 13, 10, 0
times 510 - ($ - $$) db 0      ; pad to 510 bytes
dw 0xAA55                      ; boot signature (stored little-endian: 55 AA)

; ---- stage 2: the second sector, loaded at 0x7E00 by stage 1 ----
stage2:
    mov si, msg2
    call print
    mov al, 0x10
    out 0xF4, al               ; QEMU isa-debug-exit: end the run
    hlt
msg2: db "Stage 2: loaded from disk by stage 1 via int 13h", 13, 10, 0
times 1024 - ($ - $$) db 0
```

```console
$ nasm -f bin boot.asm -o boot.img && ls -l boot.img | awk '{print $5, $9}' && od -A d -t x1 -j 510 -N 2 boot.img | head -1
1024 boot.img
0000510 55 aa
$ timeout 30 qemu-system-x86_64 -drive format=raw,file=boot.img -serial stdio -display none -device isa-debug-exit,iobase=0xf4,iosize=0x04; echo "QEMU exit status: $?"
Stage 1: BIOS loaded 512 bytes at 0x7C00
Stage 2: loaded from disk by stage 1 via int 13h
QEMU exit status: 33
```

`nasm -f bin` produces raw machine code with no file format at all — exactly what the BIOS copies into memory. The serial port is written directly with `in`/`out` instructions; the BIOS's own video output (`int 0x10`) would go to a screen we have turned off.

### Deep & Tricky

**DT-1. The BIOS guarantees that the boot sector is loaded at physical address 0x7C00, but not which segment:offset pair points there — some BIOSes jump to `0000:7C00`, others to `07C0:0000` — so a boot sector that uses absolute addresses must first reload CS with a far jump; and the only register value it can rely on is `DL` (the boot drive).** *(Phoenix/IBM BIOS Boot Specification 1.01, §6.5.1; documented BIOS behaviour)* With `org 0x7C00`, NASM assumes offsets relative to segment 0; under `07C0:0000` every absolute address would be off by 0x7C00. Real-mode addresses are `segment × 16 + offset`, so many pairs name the same byte — which is exactly why the normalising `jmp 0:.norm` is the first instruction above. Everything else (stack, data segments, direction flag) must be set up explicitly too.

## Unit 0.3 — UEFI applications

**One-line summary:** UEFI firmware runs in **64-bit long mode** with paging already enabled, reads the **EFI System Partition** (a FAT file system), and loads a **PE/COFF executable** (`EFI/BOOT/BOOTX64.EFI` by default). The program receives an image handle and a pointer to the **system table**, whose function pointers provide console output, memory allocation, file access, and the memory map — until the OS calls `ExitBootServices` and takes over the machine.

```c
// file: efi.c
/* A minimal UEFI application, with just enough of the UEFI type definitions written out by hand. */
typedef unsigned long long UINT64;
typedef unsigned short CHAR16;
typedef UINT64 EFI_STATUS;
typedef void *EFI_HANDLE;
typedef struct { UINT64 Signature; unsigned Revision, HeaderSize, CRC32, Reserved; } EFI_TABLE_HEADER;
struct SIMPLE_TEXT_OUTPUT;
typedef EFI_STATUS (*TEXT_STRING)(struct SIMPLE_TEXT_OUTPUT *self, CHAR16 *text);
typedef struct SIMPLE_TEXT_OUTPUT { void *Reset; TEXT_STRING OutputString; } SIMPLE_TEXT_OUTPUT;
typedef struct {
    EFI_TABLE_HEADER Hdr;
    CHAR16 *FirmwareVendor;
    unsigned FirmwareRevision;
    EFI_HANDLE ConsoleInHandle;
    void *ConIn;
    EFI_HANDLE ConsoleOutHandle;
    SIMPLE_TEXT_OUTPUT *ConOut;
} EFI_SYSTEM_TABLE;

EFI_STATUS efi_main(EFI_HANDLE image, EFI_SYSTEM_TABLE *st) {
    (void)image;
    st->ConOut->OutputString(st->ConOut, (CHAR16 *)L"Hello from a UEFI application!\r\nFirmware vendor: ");
    st->ConOut->OutputString(st->ConOut, st->FirmwareVendor);
    st->ConOut->OutputString(st->ConOut, (CHAR16 *)L"\r\n");
    __asm__ volatile("outb %0, %1" : : "a"((unsigned char)0x10), "Nd"((unsigned short)0xf4));   /* exit QEMU */
    return 0;
}
```

```console
$ clang -target x86_64-unknown-windows -ffreestanding -fshort-wchar -mno-red-zone -O2 -c efi.c -o efi.o && lld-link -subsystem:efi_application -entry:efi_main -nodefaultlib -out:BOOTX64.EFI efi.o && file BOOTX64.EFI | cut -d, -f1-2
BOOTX64.EFI: PE32+ executable (EFI application) x86-64, for MS Windows
$ dd if=/dev/zero of=esp.img bs=1M count=8 status=none && mformat -i esp.img :: && mmd -i esp.img ::/EFI ::/EFI/BOOT && mcopy -i esp.img BOOTX64.EFI ::/EFI/BOOT/ && mdir -i esp.img -/ :: | grep -E "EFI|BOOTX64"
EFI          <DIR>     2026-11-15  18:20 
Directory for ::/EFI
Directory for ::/EFI/BOOT
BOOTX64  EFI      2048 2026-11-15  18:20 
$ timeout 60 qemu-system-x86_64 -bios /usr/share/qemu/OVMF.fd -drive format=raw,file=esp.img -serial stdio -display none -net none -device isa-debug-exit,iobase=0xf4,iosize=0x04 -m 256M 2>&1 | sed 's/\x1b\[[=?0-9;]*[A-Za-z]//g' | grep -E "^(Hello|Firmware|BdsDxe: starting)"; echo "QEMU exit status: ${PIPESTATUS[0]}"
BdsDxe: starting Boot0001 "UEFI QEMU HARDDISK QM00001 " from PciRoot(0x0)/Pci(0x1,0x1)/Ata(Primary,Master,0x0)
Hello from a UEFI application!
Firmware vendor: Ubuntu distribution of EDK II
QEMU exit status: 33
```

UEFI programs are Windows-format (PE) executables because the specification adopted Microsoft's calling convention and file format; `clang` targets `x86_64-unknown-windows` and `lld-link` builds the PE file, with no UEFI SDK required. `-fshort-wchar` makes `L"..."` strings UTF-16, which UEFI expects. **OVMF** is the UEFI firmware build of the EDK II project for QEMU.

## Topic 0B — Bootloaders

## Unit 0.4 — GRUB and the Multiboot2 protocol

**One-line summary:** A **bootloader** such as GRUB understands file systems and executable formats, loads the kernel (and modules such as an initial RAM disk) into memory, collects information about the machine (memory map, command line, framebuffer), and jumps to the kernel in a documented machine state. The **Multiboot2** specification defines that contract for hobby and research kernels: the kernel contains a header (magic `0xE85250D6`) in its first 32 KiB; GRUB enters it in 32-bit protected mode with `EAX = 0x36D76289` and `EBX` pointing to a tagged **boot information** structure.

```nasm
; file: mb2/header.asm
section .multiboot2
align 8
mb2_start:
    dd 0xE85250D6                                    ; magic
    dd 0                                             ; architecture 0 = i386 protected mode
    dd mb2_end - mb2_start                           ; header length
    dd -(0xE85250D6 + 0 + (mb2_end - mb2_start))     ; checksum: the four fields sum to 0 (mod 2^32)
    dw 0, 0                                          ; end tag: type 0, flags 0
    dd 8                                             ;          size 8
mb2_end:

section .text
bits 32
global _start
_start:
    cmp eax, 0x36D76289         ; did a Multiboot2 loader start us?
    jne .bad
    mov dx, 0x3F8
    mov esi, ok
.loop:
    lodsb
    test al, al
    jz .exit
    out dx, al
    jmp .loop
.bad:
.exit:
    mov al, 0x10
    out 0xF4, al
    hlt
section .rodata
ok: db "GRUB started us in 32-bit protected mode with the Multiboot2 magic in EAX", 13, 10, 0
```

```text
# file: mb2/linker.ld
ENTRY(_start)
SECTIONS {
    . = 1M;                          /* load the kernel at 1 MiB, above the legacy BIOS area */
    .boot : { KEEP(*(.multiboot2)) } /* the header must be within the first 32 KiB of the file */
    .text : { *(.text*) }
    .rodata : { *(.rodata*) }
}
```

```text
# file: mb2/grub.cfg
set timeout=0
set default=0
menuentry "hobby kernel" {
    multiboot2 /boot/kernel.elf
    boot
}
```

```console
$ cd mb2 && nasm -f elf32 header.asm -o header.o && ld -m elf_i386 -n -T linker.ld -o kernel.elf header.o && grub-file --is-x86-multiboot2 kernel.elf && echo "grub-file: valid Multiboot2 kernel"
grub-file: valid Multiboot2 kernel
$ cd mb2 && mkdir -p iso/boot/grub && cp kernel.elf iso/boot/ && cp grub.cfg iso/boot/grub/ && grub-mkrescue -o boot.iso iso > /dev/null 2>&1 && ls -l boot.iso | awk '{print $5, $9}'
5124096 boot.iso
$ cd mb2 && timeout 60 qemu-system-x86_64 -cdrom boot.iso -serial stdio -display none -no-reboot -device isa-debug-exit,iobase=0xf4,iosize=0x04; echo "QEMU exit status: $?"
GRUB started us in 32-bit protected mode with the Multiboot2 magic in EAX
QEMU exit status: 33
```

`grub-mkrescue` builds a bootable ISO image containing GRUB (for both BIOS and UEFI booting), our configuration, and the kernel. From Chapter 2 on, every kernel in this book is booted this way.

### Pattern

**P-1. Make every boot experiment automatic and observable: output on the serial port, a programmatic exit (QEMU's `isa-debug-exit`), a timeout around every run, and the exit status checked. Then a kernel test is just a command that passes or fails.** *(Practice)* Kernel bugs often hang or reboot the machine silently; `-no-reboot`, `timeout`, and serial logging turn those into visible failures. *Trade-off:* emulators are more forgiving and more deterministic than real hardware (memory starts zeroed, timing is regular) — test on real machines, or at least on several QEMU configurations, before trusting low-level code.

### Mastery check

1. **Predict.** What happens if the last two bytes of the boot sector are `AA 55` instead of `55 AA`?
2. **Predict.** In which CPU mode does a UEFI application start, and in which mode does GRUB enter a Multiboot2 kernel?

## Chapter 0 Project — A boot-chain inspector

**Goal:** a Python tool that inspects the three kinds of boot images built in this chapter and reports what the firmware or bootloader will see: for a BIOS image, the boot signature and the first instruction bytes; for a UEFI application, the PE machine type and subsystem; for a Multiboot2 kernel, the header's location, magic, and checksum — verifying each rule the firmware applies.

```python
# file: inspect_boot.py
import struct
import sys

def bios_image(path):
    data = open(path, "rb").read(512)
    sig = data[510:512]
    print(f"{path}: BIOS boot sector, signature {sig.hex(' ')} -> {'bootable' if sig == b'\x55\xaa' else 'NOT bootable'}; "
          f"first bytes {data[:5].hex(' ')} (EA = far jump)")

def uefi_app(path):
    data = open(path, "rb").read()
    pe = struct.unpack_from("<I", data, 0x3C)[0]                    # offset of the PE header
    assert data[:2] == b"MZ" and data[pe:pe + 4] == b"PE\0\0"
    machine = struct.unpack_from("<H", data, pe + 4)[0]
    subsystem = struct.unpack_from("<H", data, pe + 24 + 68)[0]    # optional header: Subsystem field
    names = {0x8664: "x86-64", 10: "EFI application"}
    print(f"{path}: PE file, machine {names.get(machine, hex(machine))}, subsystem {names.get(subsystem, subsystem)}")

def multiboot2(path):
    data = open(path, "rb").read(32768)
    for off in range(0, len(data) - 16, 8):                        # header must be 8-byte aligned in the first 32 KiB
        magic, arch, length, checksum = struct.unpack_from("<IIII", data, off)
        if magic == 0xE85250D6:
            ok = (magic + arch + length + checksum) & 0xFFFFFFFF == 0
            print(f"{path}: Multiboot2 header at file offset {off:#x}, architecture {arch}, length {length}, checksum {'valid' if ok else 'INVALID'}")
            return
    print(f"{path}: no Multiboot2 header in the first 32 KiB")

bios_image("boot.img")
uefi_app("BOOTX64.EFI")
multiboot2("mb2/kernel.elf")
```

```console
$ python3 inspect_boot.py
boot.img: BIOS boot sector, signature 55 aa -> bootable; first bytes ea 05 7c 00 00 (EA = far jump)
BOOTX64.EFI: PE file, machine x86-64, subsystem EFI application
mb2/kernel.elf: Multiboot2 header at file offset 0x60, architecture 0, length 24, checksum valid
```

The Multiboot2 header is not at file offset 0 — the ELF headers come first — which is why the specification only requires it to be within the first 32 KiB and 8-byte aligned. Each check here is one rule the real firmware or GRUB applies before it will start your code.

### Review checklist

- Why does a BIOS image need no file format, while UEFI requires PE and GRUB understands ELF?
- What information does the Multiboot2 boot-information structure give a kernel that a BIOS boot sector would have to collect itself?
- How would the boot chain change on a machine with Secure Boot enabled?

## Answers to the Chapter 0 prediction questions

**0.4 Q1.** The BIOS does not consider the disk bootable and moves on to the next boot device (or reports that no bootable device was found); the signature must be the byte sequence 55 AA at offsets 510–511.

**0.4 Q2.** UEFI applications start in 64-bit long mode with paging enabled (identity-mapped). GRUB enters a Multiboot2 kernel in 32-bit protected mode with paging disabled; switching to long mode is the kernel's job (Chapter 2).

# Chapter 1 — x86-64 for Operating-System Developers

*Goal: know the parts of the x86-64 architecture that an operating system controls directly — operating modes, registers, control registers and MSRs, privilege rings, the System V calling convention, and the instruction set as seen from NASM and from C inline assembly — practising each in ordinary Linux user programs before using it in a kernel.*

**What you will be able to do after this chapter**

- Name the x86 operating modes and the registers an OS manipulates (CR0, CR3, CR4, EFER, RFLAGS).
- Query the CPU with `cpuid` and interpret feature bits.
- Write NASM programs that call Linux system calls directly.
- Follow the System V AMD64 calling convention to mix assembly and C, and explain the red zone.
- Explain privilege rings and see a privileged instruction fault in user mode.
- Write correct GCC inline assembly with outputs, inputs, and clobbers.

## Topic 1A — The machine

## Unit 1.1 — Modes and registers

**One-line summary:** An x86-64 CPU starts in 16-bit **real mode** (1 MiB address space, no protection), can enter 32-bit **protected mode** (segments with privilege levels, optional paging), and from there 64-bit **long mode** (requires paging with PAE; segmentation mostly disabled). In long mode, programs see sixteen 64-bit general registers (`rax`…`r15`), `rip`, and `rflags`; the OS additionally controls **control registers** (`cr0` protection and paging enables, `cr2` page-fault address, `cr3` page-table root, `cr4` extensions) and **model-specific registers** (MSRs, e.g. `EFER` with the long-mode enable bit).

| Register | Key bits for an OS |
| --- | --- |
| CR0 | PE (bit 0) protected mode; WP (16) write-protect in ring 0; PG (31) paging |
| CR2 | linear address that caused the last page fault |
| CR3 | physical address of the top-level page table (PML4) |
| CR4 | PAE (5); PGE (7) global pages; OSFXSR (9) SSE; SMEP (20), SMAP (21) |
| EFER (MSR 0xC0000080) | SCE (0) `syscall`; LME (8) long-mode enable; LMA (10) long mode active; NXE (11) no-execute |
| RFLAGS | IF (9) interrupts enabled; IOPL (12–13); TF (8) single-step |

The `cpuid` instruction reports what the processor supports. This user-space program reads the vendor string and the feature bits an OS cares about:

```c
// file: cpuid.c
#include <cpuid.h>
#include <stdio.h>
#include <string.h>

static void bit(const char *name, unsigned reg, int b) { printf("  %-28s %s\n", name, (reg >> b) & 1 ? "yes" : "no"); }

int main(void) {
    unsigned a, b, c, d;
    char vendor[13] = {0};
    __cpuid(0, a, b, c, d);
    memcpy(vendor, &b, 4); memcpy(vendor + 4, &d, 4); memcpy(vendor + 8, &c, 4);
    printf("vendor: %s, highest basic leaf: %u\n", vendor, a);
    __cpuid(1, a, b, c, d);
    printf("leaf 1 (basic features):\n");
    bit("APIC on chip", d, 9);
    bit("SSE2", d, 26);
    bit("x2APIC", c, 21);
    bit("running under a hypervisor", c, 31);
    __cpuid(0x80000001, a, b, c, d);
    printf("leaf 0x80000001 (extended):\n");
    bit("long mode (64-bit)", d, 29);
    bit("NX (no-execute pages)", d, 20);
    bit("1 GiB pages", d, 26);
    bit("syscall/sysret", d, 11);
    return 0;
}
```

```console
$ gcc -O2 -Wall -o cpuid cpuid.c && ./cpuid
vendor: GenuineIntel, highest basic leaf: 36
leaf 1 (basic features):
  APIC on chip                 yes
  SSE2                         yes
  x2APIC                       yes
  running under a hypervisor   yes
leaf 0x80000001 (extended):
  long mode (64-bit)           yes
  NX (no-execute pages)        yes
  1 GiB pages                  yes
  syscall/sysret               yes
```

## Unit 1.2 — NASM and system calls

**One-line summary:** NASM uses Intel syntax (`mov dst, src`). A Linux user program can talk to the kernel without libc by placing the **system-call number** in `rax` and arguments in `rdi`, `rsi`, `rdx`, `r10`, `r8`, `r9`, then executing `syscall`; the result comes back in `rax` (negative values are `-errno`). The `syscall` instruction itself clobbers `rcx` (saved `rip`) and `r11` (saved `rflags`).

```nasm
; file: hello.asm
global _start
section .rodata
msg:    db "Hello from assembly, via the write system call", 10
len:    equ $ - msg

section .text
_start:
    mov rax, 1              ; sys_write
    mov rdi, 1              ; fd 1 = stdout
    lea rsi, [rel msg]      ; buffer (RIP-relative address)
    mov rdx, len            ; count
    syscall
    mov rdi, rax            ; exit status = number of bytes written (mod 256)
    mov rax, 60             ; sys_exit
    syscall
```

```console
$ nasm -f elf64 hello.asm -o hello.o && ld -o hello hello.o && ./hello; echo "exit status: $?"
Hello from assembly, via the write system call
exit status: 47
$ ls -l hello | awk '{print $5, "bytes"}' && objdump -d --no-show-raw-insn -M intel hello | sed -n '/<_start>:/,$p' | head -9
8904 bytes
0000000000401000 <_start>:
  401000:	mov    eax,0x1
  401005:	mov    edi,0x1
  40100a:	lea    rsi,[rip+0xfef]        # 402000 <msg>
  401011:	mov    edx,0x2f
  401016:	syscall
  401018:	mov    rdi,rax
  40101b:	mov    eax,0x3c
  401020:	syscall
```

The exit status 47 is the byte count returned by `write` (47 characters including the newline). The whole program is a few hundred bytes: no C runtime, no dynamic linker — the same freestanding situation as a kernel.

## Topic 1B — Conventions and privilege

## Unit 1.3 — The System V AMD64 calling convention

**One-line summary:** On Linux (and in our kernel), a function receives its first six integer or pointer arguments in **`rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`** and returns in **`rax`**; `rbx`, `rbp`, `r12`–`r15` are **callee-saved** (a function must restore them), all other general registers are **caller-saved**; the stack pointer must be **16-byte aligned** at each `call`; and leaf functions may use the 128 bytes below `rsp` (the **red zone**) without adjusting `rsp`.

```nasm
; file: sum.asm
; uint64_t sum_array(const uint64_t *a, uint64_t n)  -- called from C
global sum_array
section .text
sum_array:
    xor eax, eax            ; rax = 0 (writing eax zero-extends into rax)
    test rsi, rsi
    jz .done
.loop:
    add rax, [rdi]          ; rdi = a, rsi = n
    add rdi, 8
    dec rsi
    jnz .loop
.done:
    ret                     ; result in rax; no callee-saved registers touched
```

```c
// file: main_sum.c
#include <inttypes.h>
#include <stdint.h>
#include <stdio.h>

uint64_t sum_array(const uint64_t *a, uint64_t n);

int main(void) {
    uint64_t a[1000];
    uint64_t expect = 0;
    for (int i = 0; i < 1000; i++) { a[i] = (uint64_t)i * i; expect += a[i]; }
    uint64_t got = sum_array(a, 1000);
    printf("assembly sum = %" PRIu64 ", C sum = %" PRIu64 ", %s\n", got, expect, got == expect ? "equal" : "DIFFERENT");
    printf("empty array: %" PRIu64 "\n", sum_array(a, 0));
    return 0;
}
```

```console
$ nasm -f elf64 sum.asm -o sum.o && gcc -O2 -Wall main_sum.c sum.o -o sum -z noexecstack && ./sum
assembly sum = 332833500, C sum = 332833500, equal
empty array: 0
```

### Deep & Tricky

**DT-2. Kernel code must be compiled with `-mno-red-zone`: an interrupt arriving while a leaf function keeps data in the 128-byte red zone below `rsp` would push the interrupt frame onto that same memory and corrupt it — user code is safe only because the CPU switches to a different (kernel) stack on interrupts from ring 3.** *(System V AMD64 ABI, §3.2.2 "The Stack Frame"; Linux `arch/x86/Makefile` uses `-mno-red-zone`)* Kernels that handle interrupts on the current stack (as ours will in Chapter 3) would see rare, unreproducible corruption of local variables. The same reasoning is why kernels also pass `-mno-sse -mno-mmx` (or save the FPU state explicitly): the compiler would otherwise use vector registers that interrupt handlers do not preserve.

```c
// file: redzone.c
/* A leaf function: GCC keeps its locals below rsp (in the red zone) unless told not to. */
long leaf(long x) {
    volatile long a = x, b = x + 1, c = x + 2;      /* volatile: force them into memory */
    return a + b + c;
}
```

```console
$ gcc -O1 -S -o - redzone.c | grep -E "rsp" | head -4
	movq	%rdi, -24(%rsp)
	movq	%rax, -16(%rsp)
	movq	%rdi, -8(%rsp)
	movq	-24(%rsp), %rax
$ gcc -O1 -mno-red-zone -S -o - redzone.c | grep -E "rsp" | head -4
	subq	$32, %rsp
	movq	%rdi, 8(%rsp)
	movq	%rax, 16(%rsp)
	movq	%rdi, 24(%rsp)
```

With the red zone, the locals live at negative offsets from `rsp` and `rsp` never moves; with `-mno-red-zone`, the function first reserves stack space (`sub rsp`), so anything pushed by an interrupt lands below its data.

## Unit 1.4 — Privilege rings

**One-line summary:** x86 has four privilege levels; operating systems use **ring 0** (kernel) and **ring 3** (user). The current privilege level (CPL) is in the low two bits of `cs`. Privileged instructions — `cli`/`sti`, `hlt`, `lgdt`/`lidt`, moves to control registers, `in`/`out` (unless permitted by IOPL or the I/O bitmap) — raise a **general-protection fault** (#GP) in ring 3, which Linux delivers to the program as `SIGSEGV`.

```c
// file: rings.c
#include <signal.h>
#include <setjmp.h>
#include <stdint.h>
#include <stdio.h>
#include <string.h>

static sigjmp_buf env;
static void on_segv(int sig, siginfo_t *info, void *ctx) { (void)info; (void)ctx; siglongjmp(env, sig); }

static void try(const char *name, void (*f)(void)) {
    if (sigsetjmp(env, 1) == 0) {
        f();
        printf("%-34s executed\n", name);
    } else {
        printf("%-34s -> general-protection fault, delivered as SIGSEGV\n", name);
    }
}
static void do_cli(void) { __asm__ volatile("cli"); }
static void do_hlt(void) { __asm__ volatile("hlt"); }
static void do_read_cr0(void) { uint64_t v; __asm__ volatile("mov %%cr0, %0" : "=r"(v)); }
static void do_out(void) { __asm__ volatile("outb %%al, $0x80" : : "a"(0)); }
static void do_rdtsc(void) { uint32_t lo, hi; __asm__ volatile("rdtsc" : "=a"(lo), "=d"(hi)); }

int main(void) {
    struct sigaction sa;
    memset(&sa, 0, sizeof sa);
    sa.sa_sigaction = on_segv;
    sa.sa_flags = SA_SIGINFO;
    sigaction(SIGSEGV, &sa, NULL);
    uint16_t cs;
    __asm__("mov %%cs, %0" : "=r"(cs));
    printf("cs = %#x -> current privilege level %d\n", cs, cs & 3);
    try("cli (disable interrupts)", do_cli);
    try("hlt (halt the CPU)", do_hlt);
    try("mov from cr0", do_read_cr0);
    try("out to I/O port 0x80", do_out);
    try("rdtsc (allowed unless CR4.TSD=1)", do_rdtsc);
    return 0;
}
```

```console
$ gcc -O0 -Wall -o rings rings.c && ./rings
cs = 0x33 -> current privilege level 3
cli (disable interrupts)           -> general-protection fault, delivered as SIGSEGV
hlt (halt the CPU)                 -> general-protection fault, delivered as SIGSEGV
mov from cr0                       -> general-protection fault, delivered as SIGSEGV
out to I/O port 0x80               -> general-protection fault, delivered as SIGSEGV
rdtsc (allowed unless CR4.TSD=1)   executed
```

## Topic 1C — Assembly from C

## Unit 1.5 — GCC inline assembly

**One-line summary:** `__asm__ volatile("template" : outputs : inputs : clobbers)` embeds instructions in C. **Outputs** (`"=r"(x)`) and **inputs** (`"r"(y)`, `"a"(v)` for a specific register, `"Nd"(port)` for an 8-bit immediate or `dx`) tell GCC which values flow in and out; **clobbers** list registers and `"memory"` that the instructions change behind the compiler's back. `volatile` prevents the compiler from deleting or merging the statement. Kernels wrap each special instruction (`inb`, `outb`, `rdmsr`, `invlpg`, `lidt`) in a tiny inline function.

```c
// file: inline.c
#include <stdint.h>
#include <stdio.h>

static inline uint64_t rdtsc(void) {
    uint32_t lo, hi;
    __asm__ volatile("rdtsc" : "=a"(lo), "=d"(hi));             /* outputs pinned to eax and edx */
    return ((uint64_t)hi << 32) | lo;
}

static inline uint64_t add_carry(uint64_t a, uint64_t b, int *carry) {
    uint8_t c;
    __asm__("add %2, %0\n\tsetc %1" : "+r"(a), "=qm"(c) : "r"(b) : "cc");   /* "+r": read and written */
    *carry = c;
    return a;
}

static inline void cpu_relax(void) { __asm__ volatile("pause" ::: "memory"); }

int main(void) {
    uint64_t t0 = rdtsc();
    for (int i = 0; i < 1000; i++) cpu_relax();
    uint64_t t1 = rdtsc();
    int carry;
    uint64_t r = add_carry(UINT64_MAX, 2, &carry);
    printf("1000 pause instructions took %s cycles (TSC advanced)\n", t1 > t0 ? "a positive number of" : "NO");
    printf("UINT64_MAX + 2 = %llu with carry %d\n", (unsigned long long)r, carry);
    return 0;
}
```

```console
$ gcc -O2 -Wall -o inline inline.c && ./inline
1000 pause instructions took a positive number of cycles (TSC advanced)
UINT64_MAX + 2 = 1 with carry 1
```

### Deep & Tricky

**DT-3. Inline assembly that reads or writes memory the compiler cannot see must declare a `"memory"` clobber (or pass the memory as an operand); otherwise the optimiser may keep values in registers across the statement, reorder it, or delete stores it believes are dead — the classic cause of "works at -O0, broken at -O2" kernel bugs.** *(GCC manual, "Extended Asm — Clobbers and Scratch Registers": the "memory" clobber "tells the compiler that the assembly code performs memory reads or writes to items other than those listed in the input and output operands")* In the demo, a "flag" in memory is written by assembly; without the clobber, GCC uses the value it already loaded and never sees the change.

```c
// file: clobber.c
#include <stdio.h>

int flag = 0;

int without_clobber(void) {
    int before = flag;
    __asm__ volatile("movl $1, flag(%%rip)" ::);                 /* writes flag; GCC is not told */
    return before + flag;                                        /* GCC may reuse the old value */
}

int with_clobber(void) {
    int before = flag;
    __asm__ volatile("movl $1, flag(%%rip)" ::: "memory");
    return before + flag;
}

int main(void) {
    flag = 0; int a = without_clobber();
    flag = 0; int b = with_clobber();
    printf("without \"memory\" clobber: %d (stale), with clobber: %d (correct)\n", a, b);
    return 0;
}
```

```console
$ gcc -O2 -o clobber clobber.c && ./clobber
without "memory" clobber: 0 (stale), with clobber: 1 (correct)
```

### Pattern

**P-2. Confine architecture-specific instructions to small, named, inline functions (`outb`, `inb`, `read_cr3`, `wrmsr`, `cpu_relax`) with precise constraints and clobbers, and write everything else in portable C. Check generated code with `objdump -d` when it matters.** *(Practice; Linux `arch/x86/include/asm/`)* A wrong constraint is invisible in the source and fatal at run time; keeping such code in one place makes it reviewable. *Trade-off:* compiler intrinsics (`__builtin_ia32_*`, `<x86intrin.h>`) are safer than raw assembly where they exist, but kernels still need inline assembly for privileged instructions that have no intrinsics.

### Mastery check

1. **Predict.** After `mov eax, -1` in 64-bit mode, what is the value of `rax`?
2. **Predict.** Which registers may a C function called from assembly freely overwrite without saving them?

## Chapter 1 Project — `memcpy` and `strlen` in assembly, tested against C

**Goal:** implement `asm_strlen` and `asm_memcpy` in NASM following the calling convention — `strlen` with the `repne scasb` string instruction, `memcpy` with `rep movsb` (fast on modern CPUs thanks to "enhanced REP MOVSB") — then **stress-test** them against the C library on 100,000 random cases (random lengths, random alignments, overlapping-free buffers) and compare speed.

```nasm
; file: strings.asm
global asm_strlen, asm_memcpy
section .text

; size_t asm_strlen(const char *s)     rdi = s
asm_strlen:
    mov rdx, rdi            ; remember the start
    xor eax, eax            ; al = 0: the byte to search for
    mov rcx, -1             ; "infinite" count
    cld                     ; scan forwards (direction flag clear: required by the ABI anyway)
    repne scasb             ; while byte at [rdi] != al: rdi++, rcx--
    sub rdi, rdx            ; rdi now points one past the zero byte
    lea rax, [rdi - 1]
    ret

; void *asm_memcpy(void *dst, const void *src, size_t n)    rdi, rsi, rdx
asm_memcpy:
    mov rax, rdi            ; return value: dst
    mov rcx, rdx            ; count
    rep movsb               ; copy rcx bytes from [rsi] to [rdi]
    ret
```

```c
// file: test_strings.c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

size_t asm_strlen(const char *s);
void *asm_memcpy(void *dst, const void *src, size_t n);

static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec * 1e-9; }

int main(void) {
    enum { MAX = 4096 };
    static char a[MAX + 64], b[MAX + 64], c[MAX + 64];
    srand(1);
    for (int trial = 0; trial < 100000; trial++) {
        size_t off = rand() % 32, len = rand() % MAX, off2 = rand() % 32;
        for (size_t i = 0; i < len; i++) a[off + i] = (char)(1 + rand() % 255);   /* no zero bytes inside */
        a[off + len] = 0;
        if (asm_strlen(a + off) != strlen(a + off)) { printf("strlen mismatch at trial %d\n", trial); return 1; }
        memset(b, 0x55, sizeof b); memset(c, 0x55, sizeof c);
        void *r = asm_memcpy(b + off2, a + off, len);
        memcpy(c + off2, a + off, len);
        if (r != b + off2 || memcmp(b, c, sizeof b) != 0) { printf("memcpy mismatch at trial %d\n", trial); return 1; }
    }
    printf("100000 random strlen and memcpy cases agree with the C library\n");

    enum { BIG = 64 * 1024 * 1024 };
    char *src = malloc(BIG), *dst = malloc(BIG);
    memset(src, 'x', BIG - 1); src[BIG - 1] = 0;
    memset(dst, 0, BIG);                               /* touch every page first: no page faults in the timings */
    double t = now(); size_t n1 = asm_strlen(src); double t_asm = now() - t;
    t = now(); size_t n2 = strlen(src); double t_c = now() - t;
    printf("strlen of 64 MiB: repne scasb %.1f ms, glibc %.1f ms (lengths equal: %d)\n", t_asm * 1e3, t_c * 1e3, n1 == n2);
    t = now(); asm_memcpy(dst, src, BIG); double m_asm = now() - t;
    t = now(); memcpy(dst, src, BIG); double m_c = now() - t;
    printf("memcpy of 64 MiB: rep movsb %.1f ms, glibc %.1f ms\n", m_asm * 1e3, m_c * 1e3);
    return 0;
}
```

```console
$ nasm -f elf64 strings.asm -o strings.o && gcc -O2 -Wall test_strings.c strings.o -o test_strings -z noexecstack && ./test_strings
100000 random strlen and memcpy cases agree with the C library
strlen of 64 MiB: repne scasb 28.0 ms, glibc 4.0 ms (lengths equal: 1)
memcpy of 64 MiB: rep movsb 35.7 ms, glibc 10.1 ms
```

Both assembly routines are correct, and both are clearly slower than glibc on the (virtualised) machine that ran this book: glibc's `strlen` tests 16 or 32 bytes per step with vector instructions while `repne scasb` compares one byte at a time, and glibc's `memcpy` picks a strategy by size and CPU model (vector loops, or `rep movsb` where the CPU's "fast short/enhanced REP MOVSB" features make it fastest). The lesson for kernel work: a short assembly routine is easy to make *correct* and hard to make *fast* — measure before replacing compiler- or library-generated code.

### Review checklist

- Why does `asm_strlen` not need to save `rdx` or `rcx`?
- What would break if the caller left the direction flag set (`std`)?
- Why do the tests vary the starting alignment of the buffers?

## Answers to the Chapter 1 prediction questions

**1.5 Q1.** `0x00000000FFFFFFFF`: writing a 32-bit register zero-extends into the full 64-bit register (writes to 8- and 16-bit registers, by contrast, leave the upper bits unchanged).

**1.5 Q2.** The caller-saved registers: `rax`, `rcx`, `rdx`, `rsi`, `rdi`, `r8`–`r11` (and the vector registers). It must preserve `rbx`, `rbp`, `rsp`, and `r12`–`r15`.

# Chapter 2 — A Freestanding Kernel in C

*Goal: write the first version of this book's hobby kernel — booted by GRUB via Multiboot2, switched from 32-bit protected mode into 64-bit long mode by our own assembly, then running C code with no operating system and no C library underneath it: serial and VGA output, a `kprintf`, the memory map from the bootloader, and a `panic` that prints a stack trace.*

From this chapter on, the hobby kernel grows chapter by chapter in a directory `os/`. Each chapter starts from a copy of the previous chapter's kernel and shows only the files it adds or changes.

**What you will be able to do after this chapter**

- Compile and link C for a freestanding target: the right compiler flags, a linker script, and the few library functions the compiler needs.
- Switch the CPU into long mode: page tables, PAE, EFER.LME, a 64-bit GDT.
- Write to the serial port and the VGA text buffer; implement `kprintf` with `<stdarg.h>`.
- Parse the Multiboot2 boot information: command line, bootloader name, memory map.
- Build a `panic` with a frame-pointer stack trace, symbolised on the host.

## Topic 2A — The freestanding environment

## Unit 2.1 — No libc: compiler flags and the minimal runtime

**One-line summary:** A **freestanding** C program (`-ffreestanding`) may use only the headers that need no library — `<stdint.h>`, `<stddef.h>`, `<stdarg.h>`, `<stdbool.h>`, `<limits.h>` — and must provide everything else itself. Kernel code is also compiled **without** a red zone (DT-2), SSE, stack protector, and position-independent code, and linked with `-nostdlib` against a **linker script** that places it at its load address.

```text
# file: os/Makefile
CC      := gcc
CFLAGS  := -std=c11 -ffreestanding -fno-stack-protector -fno-pic -mno-red-zone -mno-sse -mno-mmx -mno-80387 \
           -mcmodel=small -fno-omit-frame-pointer -O2 -g -Wall -Wextra -Werror -Iinclude
LDFLAGS := -n -T linker.ld -nostdlib -z noexecstack -z max-page-size=0x1000 --no-warn-rwx-segments
SRCS_C  := $(wildcard kernel/*.c)
SRCS_S  := $(wildcard arch/*.asm)
OBJS    := $(SRCS_S:.asm=.o) $(SRCS_C:.c=.o)
QEMU    := qemu-system-x86_64 -serial stdio -display none -no-reboot -m 128M \
           -device isa-debug-exit,iobase=0xf4,iosize=0x04 $(QEMU_EXTRA)

kernel.elf: $(OBJS) linker.ld
	ld $(LDFLAGS) -o $@ $(OBJS)

%.o: %.asm
	nasm -f elf64 -g $< -o $@
%.o: %.c $(wildcard include/*.h)
	$(CC) $(CFLAGS) -c $< -o $@

os.iso: kernel.elf grub.cfg
	mkdir -p iso/boot/grub && cp kernel.elf iso/boot/ && cp grub.cfg iso/boot/grub/
	grub-mkrescue -o $@ iso > /dev/null 2>&1

run: os.iso
	timeout 60 $(QEMU) -cdrom os.iso; status=$$?; echo "[qemu exit status $$status]"

clean:
	rm -rf $(OBJS) kernel.elf os.iso iso
.PHONY: run clean
```

```text
# file: os/grub.cfg
set timeout=0
set default=0
menuentry "hobby os" {
    multiboot2 /boot/kernel.elf loglevel=debug answer=42
    boot
}
```

```c
// file: os/kernel/string.c
/* GCC may emit calls to these four functions even in freestanding code (DT-4), so every kernel provides them. */
#include <stddef.h>
#include <stdint.h>

void *memset(void *dst, int c, size_t n) {
    unsigned char *d = dst;
    while (n--) *d++ = (unsigned char)c;
    return dst;
}
void *memcpy(void *restrict dst, const void *restrict src, size_t n) {
    unsigned char *d = dst;
    const unsigned char *s = src;
    while (n--) *d++ = *s++;
    return dst;
}
void *memmove(void *dst, const void *src, size_t n) {
    unsigned char *d = dst;
    const unsigned char *s = src;
    if (d < s) while (n--) *d++ = *s++;
    else { d += n; s += n; while (n--) *--d = *--s; }
    return dst;
}
int memcmp(const void *a, const void *b, size_t n) {
    const unsigned char *x = a, *y = b;
    for (; n; n--, x++, y++) if (*x != *y) return *x - *y;
    return 0;
}
size_t strlen(const char *s) {
    size_t n = 0;
    while (s[n]) n++;
    return n;
}
int strncmp(const char *a, const char *b, size_t n) {
    for (; n; n--, a++, b++) {
        if (*a != *b) return (unsigned char)*a - (unsigned char)*b;
        if (!*a) return 0;
    }
    return 0;
}
```

### Deep & Tricky

**DT-4. `-ffreestanding` does not stop GCC from emitting calls to `memcpy`, `memmove`, `memset`, and `memcmp`: the GCC manual states that a freestanding environment must still provide them, and the compiler turns large struct copies and initialisations into calls — so a kernel without these functions fails to link. In hosted mode the optimiser goes further and recognises loops as `memset`/`memcpy`, which can turn the body of your own `memset` into a call to itself.** *(GCC manual, "C Language — Standards": "GCC requires the freestanding environment provide memcpy, memmove, memset and memcmp")* `-ffreestanding` implies `-fno-builtin`, which (in GCC 13) stops the loop-to-`memset` rewrite — but not the struct-copy calls, as the demo shows. Linux provides optimised versions in `arch/x86/lib/` and builds the files that implement them with flags that forbid the rewrite.

```c
// file: dt4/calls.c
struct big { char bytes[65536]; };
void copy(struct big *d, const struct big *s) { *d = *s; }          /* struct assignment */
void zero(struct big *d) { *d = (struct big){0}; }                   /* struct initialisation */
void clear(char *p, unsigned long n) { for (unsigned long i = 0; i < n; i++) p[i] = 0; }   /* a plain loop */
```

```console
$ cd dt4 && gcc -O3 -ffreestanding -mno-sse -S -o - calls.c | grep -E "call|jmp" | grep -E "mem"
	call	memcpy@PLT
	call	memset@PLT
$ cd dt4 && gcc -O3 -S -o - calls.c | grep -E "call|jmp" | grep -E "mem"
	call	memcpy@PLT
	call	memset@PLT
	jmp	memset@PLT
```

With `-ffreestanding`, the struct copy and initialisation still become `memcpy` and `memset` calls, while the loop stays a loop; in hosted mode the loop in `clear` becomes a `memset` call as well.

## Topic 2B — Into long mode

## Unit 2.2 — The boot assembly: from GRUB to 64-bit C

**One-line summary:** GRUB enters `_start` in 32-bit protected mode with paging off. To reach long mode: (1) set up a stack, (2) build page tables — here an **identity map** of the first 1 GiB with 2 MiB pages (PML4 → PDPT → PD), (3) load CR3, (4) enable PAE in CR4, (5) set EFER.LME, (6) enable paging in CR0 (this activates long mode), (7) load a GDT with a 64-bit code segment and far-jump to it. Then call `kmain(magic, info)` with the System V convention.

```nasm
; file: os/arch/boot.asm
section .multiboot2
align 8
mb2_start:
    dd 0xE85250D6, 0, mb2_end - mb2_start, -(0xE85250D6 + (mb2_end - mb2_start))
    dw 0, 0
    dd 8
mb2_end:

section .bss
align 4096
global boot_pml4
boot_pml4: resb 4096
boot_pdpt: resb 4096
boot_pd:   resb 4096
stack_bottom: resb 16384
global stack_top
stack_top:

section .rodata
align 8
gdt64:
    dq 0                                              ; 0x00: null
    dq (1<<43) | (1<<44) | (1<<47) | (1<<53)          ; 0x08: kernel code, 64-bit
    dq (1<<41) | (1<<44) | (1<<47)                    ; 0x10: kernel data
gdt64_ptr:
    dw $ - gdt64 - 1
    dq gdt64

section .text
bits 32
global _start
extern kmain
_start:
    mov esp, stack_top
    mov edi, eax                  ; Multiboot2 magic -> first argument (rdi), upper half zeroed below
    mov esi, ebx                  ; boot information pointer -> second argument (rsi)
    ; PML4[0] -> PDPT, PDPT[0] -> PD, PD[i] = i * 2 MiB (present | writable | huge)
    mov eax, boot_pdpt
    or eax, 0b11
    mov [boot_pml4], eax
    mov eax, boot_pd
    or eax, 0b11
    mov [boot_pdpt], eax
    xor ecx, ecx
.map:
    mov eax, ecx
    shl eax, 21                   ; i * 2 MiB
    or eax, 0b10000011
    mov [boot_pd + ecx * 8], eax
    inc ecx
    cmp ecx, 512
    jne .map
    mov eax, boot_pml4
    mov cr3, eax
    mov eax, cr4
    or eax, 1 << 5                ; CR4.PAE
    mov cr4, eax
    mov ecx, 0xC0000080           ; IA32_EFER
    rdmsr
    or eax, 1 << 8                ; EFER.LME
    wrmsr
    mov eax, cr0
    or eax, 1 << 31               ; CR0.PG: paging on -> long mode active (compatibility mode)
    mov cr0, eax
    lgdt [gdt64_ptr]
    jmp 0x08:long_mode            ; far jump loads CS with the 64-bit code segment

bits 64
long_mode:
    mov ax, 0x10
    mov ds, ax
    mov es, ax
    mov ss, ax
    xor ax, ax
    mov fs, ax
    mov gs, ax
    mov edi, edi                  ; zero-extend the two 32-bit arguments
    mov esi, esi
    xor ebp, ebp                  ; terminates frame-pointer stack traces
    call kmain
.hang:
    cli
    hlt
    jmp .hang
```

```text
# file: os/linker.ld
ENTRY(_start)
SECTIONS {
    . = 1M;
    _kernel_start = .;
    .boot   : { KEEP(*(.multiboot2)) }
    .text   : ALIGN(4K) { *(.text*) }
    .rodata : ALIGN(4K) { *(.rodata*) }
    .data   : ALIGN(4K) { *(.data*) }
    .bss    : ALIGN(4K) { *(COMMON) *(.bss*) }
    _kernel_end = .;
    /DISCARD/ : { *(.comment) *(.note*) *(.eh_frame*) }
}
```

## Topic 2C — Output and boot information

## Unit 2.3 — Serial port, VGA text mode, and `kprintf`

**One-line summary:** The 16550 **UART** at I/O port 0x3F8 (COM1) is programmed with a divisor (baud rate), a line format (8N1), and FIFO settings; a byte is sent when the line-status register says the transmitter is empty. The **VGA text buffer** at physical 0xB8000 holds 80 × 25 cells of (character, attribute). `kprintf` walks the format string and pulls arguments with `va_arg`, writing to both.

```c
// file: os/include/kernel.h
#pragma once
#include <stdarg.h>
#include <stdbool.h>
#include <stddef.h>
#include <stdint.h>

static inline void outb(uint16_t port, uint8_t v) { __asm__ volatile("outb %0, %1" : : "a"(v), "Nd"(port)); }
static inline uint8_t inb(uint16_t port) { uint8_t v; __asm__ volatile("inb %1, %0" : "=a"(v) : "Nd"(port)); return v; }
static inline void qemu_exit(uint8_t code) { outb(0xF4, code); }   /* QEMU exits with status (code << 1) | 1 */

void serial_init(void);
void serial_putc(char c);
void vga_clear(void);
void vga_putc(char c);
void kprintf(const char *fmt, ...) __attribute__((format(printf, 1, 2)));
int  ksnprintf(char *buf, size_t size, const char *fmt, ...) __attribute__((format(printf, 3, 4)));
int  kvsnprintf(char *buf, size_t size, const char *fmt, va_list ap);
_Noreturn void panic(const char *fmt, ...) __attribute__((format(printf, 1, 2)));
void multiboot2_parse(uint32_t magic, uintptr_t info);

void *memset(void *dst, int c, size_t n);
void *memcpy(void *restrict dst, const void *restrict src, size_t n);
void *memmove(void *dst, const void *src, size_t n);
int memcmp(const void *a, const void *b, size_t n);
size_t strlen(const char *s);
int strncmp(const char *a, const char *b, size_t n);

#define KASSERT(cond) do { if (!(cond)) panic("assertion failed: %s at %s:%d", #cond, __FILE__, __LINE__); } while (0)
extern char _kernel_start[], _kernel_end[];
```

```c
// file: os/kernel/console.c
#include <kernel.h>

#define COM1 0x3F8

void serial_init(void) {
    outb(COM1 + 1, 0x00);         /* disable UART interrupts (for now) */
    outb(COM1 + 3, 0x80);         /* DLAB = 1: next two ports are the baud divisor */
    outb(COM1 + 0, 0x01);         /* divisor 1 -> 115200 baud */
    outb(COM1 + 1, 0x00);
    outb(COM1 + 3, 0x03);         /* 8 data bits, no parity, 1 stop bit; DLAB = 0 */
    outb(COM1 + 2, 0xC7);         /* enable and clear FIFOs */
    outb(COM1 + 4, 0x0B);         /* DTR, RTS, OUT2 */
}

void serial_putc(char c) {
    if (c == '\n') serial_putc('\r');
    while (!(inb(COM1 + 5) & 0x20)) { }   /* wait for an empty transmit holding register */
    outb(COM1, (uint8_t)c);
}

static volatile uint16_t *const vga = (uint16_t *)0xB8000;
static int row, col;

void vga_clear(void) {
    for (int i = 0; i < 80 * 25; i++) vga[i] = 0x0720;   /* space, light grey on black */
    row = col = 0;
}

void vga_putc(char c) {
    if (c == '\n') { col = 0; row++; }
    else { vga[row * 80 + col] = (uint16_t)(0x0700 | (uint8_t)c); if (++col == 80) { col = 0; row++; } }
    if (row == 25) {                                      /* scroll up one line */
        memmove((void *)vga, (const void *)(vga + 80), 80 * 24 * 2);
        for (int i = 0; i < 80; i++) vga[24 * 80 + i] = 0x0720;
        row = 24;
    }
}
```

```c
// file: os/kernel/printf.c
#include <kernel.h>

struct out { char *buf; size_t size, len; };

static void put(struct out *o, char c) {
    if (o->len + 1 < o->size) o->buf[o->len] = c;
    o->len++;
}

static void put_num(struct out *o, unsigned long long v, unsigned base, bool neg, int width, char pad, bool upper) {
    char tmp[24];
    const char *digits = upper ? "0123456789ABCDEF" : "0123456789abcdef";
    int n = 0;
    do { tmp[n++] = digits[v % base]; v /= base; } while (v);
    if (neg) tmp[n++] = '-';
    if (pad == '0' && neg) { put(o, '-'); n--; width--; }      /* sign before zero padding */
    for (int i = n; i < width; i++) put(o, pad);
    while (n) put(o, tmp[--n]);
}

int kvsnprintf(char *buf, size_t size, const char *fmt, va_list ap) {
    struct out o = { buf, size, 0 };
    for (; *fmt; fmt++) {
        if (*fmt != '%') { put(&o, *fmt); continue; }
        fmt++;
        char pad = ' ';
        int width = 0, longs = 0;
        if (*fmt == '0') { pad = '0'; fmt++; }
        while (*fmt >= '0' && *fmt <= '9') width = width * 10 + (*fmt++ - '0');
        while (*fmt == 'l') { longs++; fmt++; }
        if (*fmt == 'z') { longs = 1; fmt++; }
        switch (*fmt) {
        case 'd': case 'i': {
            long long v = longs ? va_arg(ap, long long) : va_arg(ap, int);
            put_num(&o, v < 0 ? -(unsigned long long)v : (unsigned long long)v, 10, v < 0, width, pad, false);
            break;
        }
        case 'u': case 'x': case 'X': {
            unsigned long long v = longs ? va_arg(ap, unsigned long long) : va_arg(ap, unsigned);
            put_num(&o, v, *fmt == 'u' ? 10 : 16, false, width, pad, *fmt == 'X');
            break;
        }
        case 'p':
            put(&o, '0'); put(&o, 'x');
            put_num(&o, (uintptr_t)va_arg(ap, void *), 16, false, 16, '0', false);
            break;
        case 's': {
            const char *s = va_arg(ap, const char *);
            if (!s) s = "(null)";
            size_t len = strlen(s);
            for (size_t i = len; i < (size_t)width; i++) put(&o, ' ');
            while (*s) put(&o, *s++);
            break;
        }
        case 'c': put(&o, (char)va_arg(ap, int)); break;
        case '%': put(&o, '%'); break;
        default: put(&o, '%'); put(&o, *fmt); break;
        }
    }
    if (size) buf[o.len < size ? o.len : size - 1] = '\0';
    return (int)o.len;
}

int ksnprintf(char *buf, size_t size, const char *fmt, ...) {
    va_list ap; va_start(ap, fmt);
    int n = kvsnprintf(buf, size, fmt, ap);
    va_end(ap);
    return n;
}

void kprintf(const char *fmt, ...) {
    char buf[512];
    va_list ap; va_start(ap, fmt);
    kvsnprintf(buf, sizeof buf, fmt, ap);
    va_end(ap);
    for (char *p = buf; *p; p++) { serial_putc(*p); vga_putc(*p); }
}
```

## Unit 2.4 — Reading the Multiboot2 boot information

**One-line summary:** `EBX` points to the boot information: an 8-byte header (`total_size`, reserved) followed by **tags**, each with a `type` and `size`, padded to 8 bytes, ending with a type-0 tag. Type 1 is the kernel **command line**, 2 the **bootloader name**, 6 the **memory map** — entries of (base address, length, type), where type 1 means usable RAM. The memory map is how the kernel learns where it may allocate (Chapter 4).

```c
// file: os/kernel/multiboot2.c
#include <kernel.h>

struct mb2_tag { uint32_t type, size; };
struct mb2_mmap_entry { uint64_t base, length; uint32_t type, reserved; };
struct mb2_tag_mmap { uint32_t type, size, entry_size, entry_version; struct mb2_mmap_entry entries[]; };

uint64_t usable_ram_bytes;                                    /* filled in for later chapters */
struct mb2_mmap_entry usable_regions[16];
int usable_region_count;

void multiboot2_parse(uint32_t magic, uintptr_t info) {
    if (magic != 0x36D76289) panic("not started by a Multiboot2 loader (magic 0x%x)", magic);
    uint32_t total = *(uint32_t *)info;
    kprintf("boot information at %p, %u bytes\n", (void *)info, total);
    for (struct mb2_tag *t = (struct mb2_tag *)(info + 8); t->type != 0;
         t = (struct mb2_tag *)((uintptr_t)t + ((t->size + 7) & ~7u))) {
        switch (t->type) {
        case 1: kprintf("  command line: \"%s\"\n", (char *)(t + 1)); break;
        case 2: kprintf("  bootloader:   \"%s\"\n", (char *)(t + 1)); break;
        case 6: {
            struct mb2_tag_mmap *m = (struct mb2_tag_mmap *)t;
            int n = (int)((m->size - sizeof *m) / m->entry_size);
            kprintf("  memory map: %d entries\n", n);
            for (int i = 0; i < n; i++) {
                struct mb2_mmap_entry *e = (struct mb2_mmap_entry *)((uintptr_t)m->entries + (uintptr_t)i * m->entry_size);
                kprintf("    %016lx - %016lx  %8lu KiB  %s\n", e->base, e->base + e->length - 1, e->length / 1024,
                        e->type == 1 ? "usable" : e->type == 3 ? "ACPI reclaimable" : e->type == 4 ? "ACPI NVS" : "reserved");
                if (e->type == 1 && usable_region_count < 16) {
                    usable_regions[usable_region_count++] = *e;
                    usable_ram_bytes += e->length;
                }
            }
            break;
        }
        }
    }
}
```

```c
// file: os/kernel/main.c
#include <kernel.h>

void kmain(uint32_t magic, uintptr_t info) {
    serial_init();
    vga_clear();
    kprintf("hobby os: running in 64-bit long mode, kernel at %p-%p\n", (void *)_kernel_start, (void *)_kernel_end);
    multiboot2_parse(magic, info);
    extern uint64_t usable_ram_bytes;
    kprintf("usable RAM: %lu MiB\n", usable_ram_bytes / (1024 * 1024));
    char buf[64];
    int n = ksnprintf(buf, sizeof buf, "[%5d|%s|%05x|%c|%lld]", 42, "ok", 0xbeef, 'Z', -1234567890123LL);
    kprintf("ksnprintf test: %s (%d chars)\n", buf, n);
    qemu_exit(0x10);
}
```

```c
// file: os/kernel/panic.c
#include <kernel.h>

_Noreturn void panic(const char *fmt, ...) {
    char buf[256];
    va_list ap; va_start(ap, fmt);
    kvsnprintf(buf, sizeof buf, fmt, ap);
    va_end(ap);
    __asm__ volatile("cli");
    kprintf("\nKERNEL PANIC: %s\n", buf);
    qemu_exit(0x11);                                   /* exit status 35: failure */
    for (;;) __asm__ volatile("hlt");
}
```

```console
$ cd os && make -s os.iso && make -s run
hobby os: running in 64-bit long mode, kernel at 0x0000000000100000-0x000000000010a1c8
boot information at 0x000000000010f2b0, 1472 bytes
  command line: "loglevel=debug answer=42"
  bootloader:   "GRUB 2.12-1ubuntu7.3"
  memory map: 7 entries
    0000000000000000 - 000000000009fbff       639 KiB  usable
    000000000009fc00 - 000000000009ffff         1 KiB  reserved
    00000000000f0000 - 00000000000fffff        64 KiB  reserved
    0000000000100000 - 0000000007fdffff    129920 KiB  usable
    0000000007fe0000 - 0000000007ffffff       128 KiB  reserved
    00000000fffc0000 - 00000000ffffffff       256 KiB  reserved
    000000fd00000000 - 000000ffffffffff  12582912 KiB  reserved
usable RAM: 127 MiB
ksnprintf test: [   42|ok|0beef|Z|-1234567890123] (33 chars)
[qemu exit status 33]
```

The kernel occupies about 40 KiB starting at 1 MiB. QEMU's 128 MiB appear as two usable regions: the 639 KiB of conventional memory below the legacy video and BIOS area, and everything from 1 MiB up. The firmware reserves the rest. This `kprintf` has one gap that matters: it does not understand the `-` (left-justify) flag. An unknown conversion is printed literally — and from that point on every later argument is read by the wrong `%`, so the rest of the line is garbage. The project adds the flag.

### Pattern

**P-3. Keep the boot path tiny and paranoid: check the Multiboot2 magic, build the minimal page tables needed, jump to C as early as possible, and make the very first C code bring up an output channel so that everything after it can report problems.** *(Practice)* Every later failure is diagnosable only if output works; a kernel that dies before `serial_init` gives you nothing. *Trade-off:* identity-mapping the first GiB is the simplest correct choice for a teaching kernel, but real kernels map themselves into the **higher half** (Linux at `0xffffffff80000000`) so that user programs can own the low addresses.

### Mastery check

1. **Predict.** What does the CPU do if the boot code enables CR0.PG before setting CR4.PAE while EFER.LME is set?
2. **Predict.** Why is `-mno-sse` needed even though x86-64 guarantees SSE2?

## Chapter 2 Project — `panic` with a symbolised stack trace, and a finished `kprintf`

**Goal:** (1) complete `kprintf` with left-justification (`%-Ns`, `%-Nd`) and test it against the host C library's `snprintf` by compiling `printf.c` **for user space** as well — the same code in two environments; (2) make `panic` print a **stack trace** by walking saved frame pointers (`rbp` chains, available because the kernel is built with `-fno-omit-frame-pointer`), and symbolise the return addresses on the host with `addr2line`.

```c
// file: proj/kernel/printf_test.c
/* Host-side test: build kernel/printf.c into a normal program and compare with glibc's snprintf. */
#ifdef HOST_TEST
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int ksnprintf(char *buf, size_t size, const char *fmt, ...);
void serial_putc(char c) { (void)c; }                /* stubs: kprintf's output functions are kernel-only */
void vga_putc(char c) { (void)c; }

int main(void) {
    const char *fmts[] = {"[%d]", "[%5d]", "[%-5d]", "[%05d]", "[%x]", "[%08x]", "[%X]", "[%s]", "[%-8s|]", "[%8s]",
                          "[%c]", "[%u]", "[%lld]", "[%llu]", "[%%]"};
    srand(2);
    int checked = 0;
    for (int trial = 0; trial < 20000; trial++) {
        const char *f = fmts[rand() % 15];
        char a[128], b[128];
        int v = rand() - RAND_MAX / 2;
        long long big = ((long long)rand() << 32) ^ rand();
        const char *s = (const char *[]){"ok", "", "kernel", "hobby os!"}[rand() % 4];
        int na, nb;
        if (strstr(f, "s")) { na = ksnprintf(a, sizeof a, f, s); nb = snprintf(b, sizeof b, f, s); }
        else if (strstr(f, "ll")) { na = ksnprintf(a, sizeof a, f, big); nb = snprintf(b, sizeof b, f, big); }
        else if (strstr(f, "c")) { na = ksnprintf(a, sizeof a, f, 'A' + v % 26 + 26 * (v < 0)); nb = snprintf(b, sizeof b, f, 'A' + v % 26 + 26 * (v < 0)); }
        else { na = ksnprintf(a, sizeof a, f, v); nb = snprintf(b, sizeof b, f, v); }
        if (na != nb || strcmp(a, b) != 0) { printf("MISMATCH for %s: kernel '%s' (%d) vs libc '%s' (%d)\n", f, a, na, b, nb); return 1; }
        checked++;
    }
    char small[6];
    int n = ksnprintf(small, sizeof small, "%s", "truncated output");
    printf("%d random format cases match glibc; truncation: '%s' (would need %d chars)\n", checked, small, n);
    return 0;
}
#endif
```

```c
// file: proj/kernel/panic.c
#include <kernel.h>

struct frame { struct frame *next; uintptr_t ret; };

void print_backtrace(void) {
    struct frame *f;
    __asm__ volatile("mov %%rbp, %0" : "=r"(f));
    kprintf("backtrace (symbolise with: addr2line -f -e kernel.elf <addresses>):\n");
    for (int depth = 0; f && depth < 16; depth++, f = f->next) {
        if ((uintptr_t)f < (uintptr_t)_kernel_start || (uintptr_t)f >= 0x40000000) break;   /* sanity check */
        kprintf("  #%d %p\n", depth, (void *)f->ret);
    }
}

_Noreturn void panic(const char *fmt, ...) {
    char buf[256];
    va_list ap; va_start(ap, fmt);
    kvsnprintf(buf, sizeof buf, fmt, ap);
    va_end(ap);
    __asm__ volatile("cli");
    kprintf("\nKERNEL PANIC: %s\n", buf);
    print_backtrace();
    qemu_exit(0x11);
    for (;;) __asm__ volatile("hlt");
}
```

```c
// file: proj/kernel/main.c
#include <kernel.h>

__attribute__((noinline)) static int checked_divide(int a, int b) {
    KASSERT(b != 0);
    return a / b;
}

__attribute__((noinline)) static int average(const int *v, int n) {
    int sum = 0;
    for (int i = 0; i < n; i++) sum += v[i];
    return checked_divide(sum, n);
}

void kmain(uint32_t magic, uintptr_t info) {
    serial_init();
    vga_clear();
    kprintf("hobby os: running in 64-bit long mode, kernel at %p-%p\n", (void *)_kernel_start, (void *)_kernel_end);
    multiboot2_parse(magic, info);
    char buf[64];
    int n = ksnprintf(buf, sizeof buf, "[%5d|%-3s|%05x|%c|%lld]", 42, "ok", 0xbeef, 'Z', -1234567890123LL);
    kprintf("ksnprintf test: %s (%d chars)\n", buf, n);
    int values[] = {3, 4, 5};
    kprintf("average of 3 values: %d\n", average(values, 3));
    kprintf("average of 0 values: %d\n", average(values, 0));     /* triggers the assertion */
    qemu_exit(0x10);
}
```

The left-justification fix in `printf.c`: parse a `-` flag and pad on the right. The project's files live in `proj/`; the build directory `final/` is `os/` with them copied on top — and it is the starting point of Chapter 3.

```c
// file: proj/kernel/printf.c
#include <kernel.h>

struct out { char *buf; size_t size, len; };

static void put(struct out *o, char c) {
    if (o->len + 1 < o->size) o->buf[o->len] = c;
    o->len++;
}

static void put_num(struct out *o, unsigned long long v, unsigned base, bool neg, int width, char pad, bool upper, bool left) {
    char tmp[24];
    const char *digits = upper ? "0123456789ABCDEF" : "0123456789abcdef";
    int n = 0;
    do { tmp[n++] = digits[v % base]; v /= base; } while (v);
    if (neg) tmp[n++] = '-';
    int len = n;
    if (left) { while (n) put(o, tmp[--n]); for (int i = len; i < width; i++) put(o, ' '); return; }
    if (pad == '0' && neg) { put(o, '-'); n--; len--; width--; }
    for (int i = len; i < width; i++) put(o, pad);
    while (n) put(o, tmp[--n]);
}

int kvsnprintf(char *buf, size_t size, const char *fmt, va_list ap) {
    struct out o = { buf, size, 0 };
    for (; *fmt; fmt++) {
        if (*fmt != '%') { put(&o, *fmt); continue; }
        fmt++;
        char pad = ' ';
        bool left = false;
        int width = 0, longs = 0;
        for (;; fmt++) {
            if (*fmt == '-') left = true;
            else if (*fmt == '0') pad = '0';
            else break;
        }
        while (*fmt >= '0' && *fmt <= '9') width = width * 10 + (*fmt++ - '0');
        while (*fmt == 'l') { longs++; fmt++; }
        if (*fmt == 'z') { longs = 1; fmt++; }
        switch (*fmt) {
        case 'd': case 'i': {
            long long v = longs ? va_arg(ap, long long) : va_arg(ap, int);
            put_num(&o, v < 0 ? -(unsigned long long)v : (unsigned long long)v, 10, v < 0, width, pad, false, left);
            break;
        }
        case 'u': case 'x': case 'X': {
            unsigned long long v = longs ? va_arg(ap, unsigned long long) : va_arg(ap, unsigned);
            put_num(&o, v, *fmt == 'u' ? 10 : 16, false, width, pad, *fmt == 'X', left);
            break;
        }
        case 'p':
            put(&o, '0'); put(&o, 'x');
            put_num(&o, (uintptr_t)va_arg(ap, void *), 16, false, 16, '0', false, false);
            break;
        case 's': {
            const char *s = va_arg(ap, const char *);
            if (!s) s = "(null)";
            size_t len = strlen(s);
            if (!left) for (size_t i = len; i < (size_t)width; i++) put(&o, ' ');
            while (*s) put(&o, *s++);
            if (left) for (size_t i = len; i < (size_t)width; i++) put(&o, ' ');
            break;
        }
        case 'c': put(&o, (char)va_arg(ap, int)); break;
        case '%': put(&o, '%'); break;
        default: put(&o, '%'); put(&o, *fmt); break;
        }
    }
    if (size) buf[o.len < size ? o.len : size - 1] = '\0';
    return (int)o.len;
}

int ksnprintf(char *buf, size_t size, const char *fmt, ...) {
    va_list ap; va_start(ap, fmt);
    int n = kvsnprintf(buf, size, fmt, ap);
    va_end(ap);
    return n;
}

void kprintf(const char *fmt, ...) {
    char buf[512];
    va_list ap; va_start(ap, fmt);
    kvsnprintf(buf, sizeof buf, fmt, ap);
    va_end(ap);
    for (char *p = buf; *p; p++) { serial_putc(*p); vga_putc(*p); }
}
```

```console
$ rm -rf final && cp -r os final && cp -r proj/. final/ && cd final && make -s clean && echo "final = os + proj"
final = os + proj
$ cd final && gcc -O2 -DHOST_TEST -Iinclude -fno-builtin kernel/printf.c kernel/printf_test.c kernel/string.c -o printf_test && ./printf_test
20000 random format cases match glibc; truncation: 'trunc' (would need 16 chars)
$ cd final && make -s os.iso && make -s run | tee panic.log | tail -9
average of 3 values: 4

KERNEL PANIC: assertion failed: b != 0 at kernel/main.c:5
backtrace (symbolise with: addr2line -f -e kernel.elf <addresses>):
  #0 0x0000000000101627
  #1 0x0000000000101279
  #2 0x000000000010136f
  #3 0x0000000000101093
[qemu exit status 35]
$ cd final && grep -oE "#[0-9]+ 0x[0-9a-f]+" panic.log | while read -r _ a; do printf '0x%x\n' $((a - 1)); done | xargs addr2line -f -s -e kernel.elf | paste - - | awk '{printf "  %-16s %s\n", $1, $2}'
  panic            panic.c:23
  checked_divide   main.c:5
  kmain            main.c:25
  long_mode        boot.asm:81
```

The kernel's `kprintf` agrees with glibc on 20,000 random formatting cases. This is the same stress-testing idea used in the rest of this series, now applied to kernel code compiled for the host. The panic shows the failed assertion, and `addr2line` turns the raw return addresses into function names and source lines. (We subtract 1 from each address so that it points inside the `call` instruction, not at the next line.)

Look closely at the trace: `average` is **missing**. The chain reads `panic` ← `checked_divide` ← `kmain`. The reason is in the disassembly. At `-O2`, GCC compiled `return checked_divide(sum, n);` as a **tail call** — a `jmp`, not a `call`. So `average`'s stack frame was reused by `checked_divide` and no return address into `average` was ever saved. The trace shows only what is on the stack. This is why Linux, when built with frame pointers, also passes `-fno-optimize-sibling-calls`.

```console
$ cd final && objdump -d kernel.elf | grep -E "jmp .*<checked_divide>"
  10129d:	eb b1                	jmp    101250 <checked_divide>
  1012a4:	eb aa                	jmp    101250 <checked_divide>
```

### Review checklist

- Why must the frame-pointer walk stop when `rbp` leaves the kernel's address range?
- Why can the same `printf.c` be compiled for the kernel and for a host test program?
- What would break in the stack trace if the kernel were built with `-fomit-frame-pointer` (the `-O2` default)?
- Which flag would make `average` appear in the trace, and what does it cost?

## Answers to the Chapter 2 prediction questions

**2.4 Q1.** Enabling paging with EFER.LME set requires CR4.PAE; otherwise the CPU raises a general-protection fault — and since no interrupt handlers exist yet, that becomes a triple fault: QEMU resets (or exits with `-no-reboot`).

**2.4 Q2.** SSE registers are not saved by our interrupt handlers (Chapter 3) and the FPU/SSE state has not been initialised (CR4.OSFXSR is clear), so any SSE instruction the compiler emits would fault or corrupt user state; the kernel must use only general-purpose registers unless it manages the vector state explicitly.

# Chapter 3 — Interrupts and Exceptions

*Goal: teach the kernel to react to events. When the CPU hits an error (an **exception**) or a device needs attention (a **hardware interrupt**, or IRQ), the CPU stops what it is doing, saves a few registers, and jumps to a handler that the kernel registered in the **Interrupt Descriptor Table**. By the end of this chapter, the kernel handles exceptions with a register dump, runs a 100 Hz timer, reads keyboard input from the serial port, and has a small interactive shell.*

**What you will be able to do after this chapter**

- Build an IDT and assembly entry stubs that save the full CPU state into a `struct trap_frame`.
- Decode exceptions: the vector, the error code, and (for page faults) CR2.
- Program the legacy 8259 **PIC** and the 8253/8254 **PIT** timer, and acknowledge interrupts correctly.
- Move data from an interrupt handler to normal code safely, and sleep without losing a wake-up.
- Register kernel self-tests through a **linker section**, the same idea Linux uses for initcalls.

**How this chapter's code is organised.** The kernel continues from Chapter 2's `final/` directory. This chapter's new and changed files are in `new/`, and the build directory `os/` is Chapter 2's kernel with `new/` copied on top. The project's files are in `proj/`, and `final/` is `os/` plus `proj/`.

```console
$ rm -rf os && cp -r ../ch02/final os && cp -r new/. os/ && make -s -C os clean && ls os/kernel os/arch
os/arch:
boot.asm  isr.asm

os/kernel:
console.c  main.c	 panic.c  printf.c	 string.c	    test_serial.c  timer.c
idt.c	   multiboot2.c  pic.c	  printf_test.c  test_exceptions.c  test_timer.c
```

## Topic 3A — The Interrupt Descriptor Table

## Unit 3.1 — Gates, entry stubs, and the trap frame

**One-line summary:** The IDT holds 256 **gates**, one per vector. In long mode each gate is 16 bytes: the handler's 64-bit address split into three parts, the code segment selector, and a type byte (0x8E = present, ring 0, 64-bit **interrupt gate**, which also clears IF so that interrupts are off inside the handler). Vectors 0–31 are reserved for CPU exceptions. When an event arrives, the CPU pushes SS, RSP, RFLAGS, CS, RIP — and for some exceptions an error code — and jumps to the gate's address. Our assembly stubs push the remaining registers so that C code receives the whole interrupted state as one struct.

The CPU's own push looks like this on the handler's stack (higher addresses at the top):

```text
  ss       \
  rsp       |  pushed by the CPU on every interrupt or exception
  rflags    |
  cs        |
  rip      /
  error    <- pushed by the CPU for vectors 8, 10-14, 17, 21, 29, 30; our stub pushes 0 for the rest
  vector   <- pushed by our stub, so one common handler knows what happened
  rax ... r15  <- pushed by isr_common
```

```nasm
; file: new/arch/isr.asm
; Interrupt entry stubs for vectors 0-47 and the common path into C.
bits 64
section .text
extern interrupt_dispatch

%macro ISR_NOERR 1                ; the CPU pushed no error code: push a 0 so every frame has the same layout
isr%1:
    push 0
    push %1
    jmp isr_common
%endmacro
%macro ISR_ERR 1                  ; the CPU already pushed an error code
isr%1:
    push %1
    jmp isr_common
%endmacro

%assign v 0
%rep 48
  %if v == 8 || (v >= 10 && v <= 14) || v == 17 || v == 21 || v == 29 || v == 30
    ISR_ERR v
  %else
    ISR_NOERR v
  %endif
  %assign v v + 1
%endrep

isr_common:
    push rax
    push rbx
    push rcx
    push rdx
    push rsi
    push rdi
    push rbp
    push r8
    push r9
    push r10
    push r11
    push r12
    push r13
    push r14
    push r15
    cld                           ; the System V ABI requires DF = 0 on function entry
    mov rdi, rsp                  ; argument: pointer to the struct trap_frame we just built
    call interrupt_dispatch
    pop r15
    pop r14
    pop r13
    pop r12
    pop r11
    pop r10
    pop r9
    pop r8
    pop rbp
    pop rdi
    pop rsi
    pop rdx
    pop rcx
    pop rbx
    pop rax
    add rsp, 16                   ; drop vector and error code
    iretq                         ; pops rip, cs, rflags, rsp, ss: resumes the interrupted code

section .rodata
global isr_table
isr_table:
%assign v 0
%rep 48
    dq isr %+ v
  %assign v v + 1
%endrep
```

The stack stays 16-byte aligned for the `call`, as the System V ABI requires: in 64-bit mode the CPU aligns RSP to 16 before its own pushes, and 5 CPU pushes + error + vector + 15 registers = 22 × 8 = 176 bytes, a multiple of 16.

```c
// file: new/include/kernel.h
#pragma once
#include <stdarg.h>
#include <stdbool.h>
#include <stddef.h>
#include <stdint.h>

/* ---- port I/O and CPU control ---- */
static inline void outb(uint16_t port, uint8_t v) { __asm__ volatile("outb %0, %1" : : "a"(v), "Nd"(port)); }
static inline uint8_t inb(uint16_t port) { uint8_t v; __asm__ volatile("inb %1, %0" : "=a"(v) : "Nd"(port)); return v; }
static inline void io_wait(void) { outb(0x80, 0); }                 /* a write to an unused port: a short delay */
static inline void cli(void) { __asm__ volatile("cli" ::: "memory"); }
static inline void sti(void) { __asm__ volatile("sti" ::: "memory"); }
static inline void hlt(void) { __asm__ volatile("hlt" ::: "memory"); }
static inline uint64_t read_cr2(void) { uint64_t v; __asm__ volatile("mov %%cr2, %0" : "=r"(v)); return v; }
static inline void qemu_exit(uint8_t code) { outb(0xF4, code); }   /* QEMU exits with status (code << 1) | 1 */

/* ---- console ---- */
void serial_init(void);
void serial_putc(char c);
void serial_enable_rx_irq(void);
int  serial_getc(void);                     /* blocks (sleeping) until a byte arrives */
extern volatile uint64_t serial_irq_count, serial_rx_bytes;
void vga_clear(void);
void vga_putc(char c);
void kprintf(const char *fmt, ...) __attribute__((format(printf, 1, 2)));
int  ksnprintf(char *buf, size_t size, const char *fmt, ...) __attribute__((format(printf, 3, 4)));
int  kvsnprintf(char *buf, size_t size, const char *fmt, va_list ap);
_Noreturn void panic(const char *fmt, ...) __attribute__((format(printf, 1, 2)));
void print_backtrace(void);

/* ---- boot information ---- */
void multiboot2_parse(uint32_t magic, uintptr_t info);
extern const char *kernel_cmdline;
extern uint64_t usable_ram_bytes;
bool cmdline_get(const char *key, char *out, size_t n);

/* ---- interrupts ---- */
struct trap_frame {
    uint64_t r15, r14, r13, r12, r11, r10, r9, r8, rbp, rdi, rsi, rdx, rcx, rbx, rax;
    uint64_t vector, error;
    uint64_t rip, cs, rflags, rsp, ss;                               /* pushed by the CPU */
};
typedef void (*irq_handler_t)(struct trap_frame *);
void idt_init(void);
void irq_register(int irq, irq_handler_t handler);
void pic_init(void);
void pic_unmask(int irq);
void pic_eoi(int irq);
void timer_init(unsigned hz);
extern volatile uint64_t ticks;
extern unsigned timer_hz;
uint8_t rtc_read(uint8_t reg);

/* ---- kernel self-tests, registered in the .ktests linker section ---- */
struct ktest { const char *name; void (*fn)(void); } __attribute__((aligned(16)));
#define KTEST(n)                                                                              \
    static void ktest_##n(void);                                                              \
    __attribute__((used, section(".ktests"))) static const struct ktest ktest_entry_##n = {   \
        #n, ktest_##n};                                                                       \
    static void ktest_##n(void)

/* ---- library ---- */
void *memset(void *dst, int c, size_t n);
void *memcpy(void *restrict dst, const void *restrict src, size_t n);
void *memmove(void *dst, const void *src, size_t n);
int memcmp(const void *a, const void *b, size_t n);
size_t strlen(const char *s);
int strncmp(const char *a, const char *b, size_t n);

#define KASSERT(cond) do { if (!(cond)) panic("assertion failed: %s at %s:%d", #cond, __FILE__, __LINE__); } while (0)
extern char _kernel_start[], _kernel_end[];
```

```c
// file: new/kernel/idt.c
#include <kernel.h>

struct idt_gate {
    uint16_t offset_low;          /* handler address bits 0-15 */
    uint16_t selector;            /* code segment: 0x08, our 64-bit kernel code */
    uint8_t  ist;                 /* interrupt stack table index: 0 = use the current stack */
    uint8_t  type;                /* 0x8E: present, DPL 0, 64-bit interrupt gate */
    uint16_t offset_mid;          /* bits 16-31 */
    uint32_t offset_high;         /* bits 32-63 */
    uint32_t reserved;
} __attribute__((packed));
_Static_assert(sizeof(struct idt_gate) == 16, "a long-mode gate is 16 bytes");

static struct idt_gate idt[256];
extern const uint64_t isr_table[48];
static irq_handler_t irq_handlers[16];

static const char *const exception_names[32] = {
    "divide error", "debug", "NMI", "breakpoint", "overflow", "bound range", "invalid opcode", "device not available",
    "double fault", "coprocessor overrun", "invalid TSS", "segment not present", "stack fault", "general protection",
    "page fault", "reserved", "x87 FP error", "alignment check", "machine check", "SIMD FP error", "virtualization",
    "control protection", "reserved", "reserved", "reserved", "reserved", "reserved", "reserved", "hypervisor injection",
    "VMM communication", "security", "reserved"};

void idt_init(void) {
    for (int v = 0; v < 48; v++) {
        uint64_t a = isr_table[v];
        idt[v] = (struct idt_gate){ (uint16_t)a, 0x08, 0, 0x8E, (uint16_t)(a >> 16), (uint32_t)(a >> 32), 0 };
    }
    struct __attribute__((packed)) { uint16_t limit; uint64_t base; } idtr = { sizeof idt - 1, (uint64_t)idt };
    __asm__ volatile("lidt %0" : : "m"(idtr));
}

void irq_register(int irq, irq_handler_t handler) { irq_handlers[irq] = handler; }

static void dump_frame(const struct trap_frame *f) {
    kprintf("  rip=%p  cs=0x%lx  rflags=0x%lx  rsp=%p\n", (void *)f->rip, f->cs, f->rflags, (void *)f->rsp);
    kprintf("  rax=%016lx rbx=%016lx rcx=%016lx rdx=%016lx\n", f->rax, f->rbx, f->rcx, f->rdx);
    kprintf("  rsi=%016lx rdi=%016lx rbp=%016lx\n", f->rsi, f->rdi, f->rbp);
}

static void exception(struct trap_frame *f) {
    if (f->vector == 3) {                                    /* int3: report and continue after the instruction */
        kprintf("breakpoint at rip=%p\n", (void *)f->rip);
        return;
    }
    kprintf("\nEXCEPTION %lu: %s, error code 0x%lx\n", f->vector, exception_names[f->vector], f->error);
    if (f->vector == 14) {
        kprintf("  faulting address (cr2) = %p: %s, %s, %s mode\n", (void *)read_cr2(),
                f->error & 1 ? "protection violation" : "page not present",
                f->error & 2 ? "write" : "read", f->error & 4 ? "user" : "kernel");
    }
    dump_frame(f);
    panic("unhandled exception %lu at rip %p", f->vector, (void *)f->rip);
}

void interrupt_dispatch(struct trap_frame *f) {
    if (f->vector < 32) { exception(f); return; }
    int irq = (int)f->vector - 32;
    if (irq_handlers[irq]) irq_handlers[irq](f);
    pic_eoi(irq);
}
```

The self-test mechanism: each `KTEST(name) { ... }` places a `struct ktest` in a section called `.ktests`; the linker script collects them between two symbols, and `kmain` runs the test whose name is given on the kernel command line (`test=name`). Nothing needs a central list — a file that defines a test is enough.

```text
# file: new/linker.ld
ENTRY(_start)
SECTIONS {
    . = 1M;
    _kernel_start = .;
    .boot   : { KEEP(*(.multiboot2)) }
    .text   : ALIGN(4K) { *(.text*) }
    .rodata : ALIGN(4K) {
        *(.rodata*)
        . = ALIGN(16);
        __ktests_start = .;
        KEEP(*(.ktests))
        __ktests_end = .;
    }
    .data   : ALIGN(4K) { *(.data*) }
    .bss    : ALIGN(4K) { *(COMMON) *(.bss*) }
    _kernel_end = .;
    /DISCARD/ : { *(.comment) *(.note*) *(.eh_frame*) }
}
```

```c
// file: new/kernel/main.c
#include <kernel.h>

extern const struct ktest __ktests_start[], __ktests_end[];

__attribute__((weak)) void shell_run(void) { kprintf("no shell in this build\n"); }

static void run_test(const char *name) {
    for (const struct ktest *t = __ktests_start; t < __ktests_end; t++) {
        if (strncmp(t->name, name, 64) == 0) {
            kprintf("== test %s ==\n", t->name);
            t->fn();
            kprintf("== test %s passed ==\n", t->name);
            return;
        }
    }
    kprintf("unknown test '%s'; available:", name);
    for (const struct ktest *t = __ktests_start; t < __ktests_end; t++) kprintf(" %s", t->name);
    kprintf("\n");
}

void kmain(uint32_t magic, uintptr_t info) {
    serial_init();
    vga_clear();
    multiboot2_parse(magic, info);
    kprintf("hobby os: %lu MiB usable, command line \"%s\"\n", usable_ram_bytes >> 20, kernel_cmdline);
    idt_init();
    pic_init();
    timer_init(100);
    serial_enable_rx_irq();
    sti();
    char name[32];
    if (cmdline_get("test", name, sizeof name)) run_test(name);
    else shell_run();
    qemu_exit(0x10);
}
```

```c
// file: new/kernel/multiboot2.c
#include <kernel.h>

struct mb2_tag { uint32_t type, size; };
struct mb2_mmap_entry { uint64_t base, length; uint32_t type, reserved; };
struct mb2_tag_mmap { uint32_t type, size, entry_size, entry_version; struct mb2_mmap_entry entries[]; };

const char *kernel_cmdline = "";
uint64_t usable_ram_bytes;
struct mb2_mmap_entry usable_regions[16];
int usable_region_count;

void multiboot2_parse(uint32_t magic, uintptr_t info) {
    if (magic != 0x36D76289) panic("not started by a Multiboot2 loader (magic 0x%x)", magic);
    for (struct mb2_tag *t = (struct mb2_tag *)(info + 8); t->type != 0;
         t = (struct mb2_tag *)((uintptr_t)t + ((t->size + 7) & ~7u))) {
        if (t->type == 1) kernel_cmdline = (const char *)(t + 1);
        if (t->type == 6) {
            struct mb2_tag_mmap *m = (struct mb2_tag_mmap *)t;
            int n = (int)((m->size - sizeof *m) / m->entry_size);
            for (int i = 0; i < n; i++) {
                struct mb2_mmap_entry *e = (struct mb2_mmap_entry *)((uintptr_t)m->entries + (uintptr_t)i * m->entry_size);
                if (e->type == 1 && usable_region_count < 16) {
                    usable_regions[usable_region_count++] = *e;
                    usable_ram_bytes += e->length;
                }
            }
        }
    }
}

/* Find "key=value" in the command line; copy value into out. */
bool cmdline_get(const char *key, char *out, size_t n) {
    size_t klen = strlen(key);
    for (const char *p = kernel_cmdline; *p; ) {
        while (*p == ' ') p++;
        if (strncmp(p, key, klen) == 0 && p[klen] == '=') {
            p += klen + 1;
            size_t i = 0;
            while (*p && *p != ' ' && i + 1 < n) out[i++] = *p++;
            out[i] = '\0';
            return true;
        }
        while (*p && *p != ' ') p++;
    }
    return false;
}
```

The Makefile now writes `grub.cfg` on every run, so a command line can be passed as `make run CMDLINE="test=..."`, and it accepts `EXTRA_CFLAGS` for experiment builds.

```text
# file: new/Makefile
CC      := gcc
CFLAGS  := -std=c11 -ffreestanding -fno-stack-protector -fno-pic -mno-red-zone -mno-sse -mno-mmx -mno-80387 \
           -mcmodel=small -fno-omit-frame-pointer -O2 -g -Wall -Wextra -Werror -Iinclude $(EXTRA_CFLAGS)
LDFLAGS := -n -T linker.ld -nostdlib -z noexecstack -z max-page-size=0x1000 --no-warn-rwx-segments
SRCS_C  := $(wildcard kernel/*.c)
SRCS_S  := $(wildcard arch/*.asm)
OBJS    := $(SRCS_S:.asm=.o) $(SRCS_C:.c=.o)
CMDLINE ?=
TIMEOUT ?= 60
QEMU    := qemu-system-x86_64 -serial stdio -display none -no-reboot -m 128M \
           -device isa-debug-exit,iobase=0xf4,iosize=0x04 $(QEMU_EXTRA)

kernel.elf: $(OBJS) linker.ld
	ld $(LDFLAGS) -o $@ $(OBJS)

%.o: %.asm
	nasm -f elf64 -g $< -o $@
%.o: %.c $(wildcard include/*.h)
	$(CC) $(CFLAGS) -c $< -o $@

os.iso: kernel.elf
	rm -rf iso && mkdir -p iso/boot/grub && cp kernel.elf iso/boot/
	printf 'set timeout=0\nset default=0\nmenuentry "hobby os" {\n  multiboot2 /boot/kernel.elf %s\n  boot\n}\n' \
	    "$(CMDLINE)" > iso/boot/grub/grub.cfg
	grub-mkrescue -o $@ iso > /dev/null 2>&1

run: kernel.elf
	rm -f os.iso && $(MAKE) -s os.iso CMDLINE="$(CMDLINE)"
	timeout $(TIMEOUT) $(QEMU) -cdrom os.iso; status=$$?; echo "[qemu exit status $$status]"

clean:
	rm -rf $(OBJS) kernel.elf os.iso iso grub.cfg
.PHONY: run clean os.iso
```

Each exception test triggers one fault on purpose.

```c
// file: new/kernel/test_exceptions.c
#include <kernel.h>

KTEST(breakpoint) {
    __asm__ volatile("int3");                         /* vector 3: a "trap" - RIP already points past it */
    kprintf("execution continued after int3\n");
}

KTEST(pagefault) {
    volatile uint64_t *p = (volatile uint64_t *)0x80000000;   /* 2 GiB: outside our 1 GiB identity map */
    kprintf("reading %p...\n", (void *)p);
    (void)*p;
}

KTEST(divide) {
    __asm__ volatile("xor %%edx, %%edx; mov $7, %%eax; xor %%ecx, %%ecx; div %%ecx" ::: "rax", "rcx", "rdx");
}

KTEST(invalid_opcode) {
    __asm__ volatile("ud2");
}
```

The division is written in assembly on purpose: in C, dividing by zero is undefined behaviour, so the compiler is allowed to assume it never happens and may remove or reorder it.

```console
$ cd os && make -s && make -s run CMDLINE="test=breakpoint"
hobby os: 127 MiB usable, command line "test=breakpoint"
== test breakpoint ==
breakpoint at rip=0x0000000000102685
execution continued after int3
== test breakpoint passed ==
[qemu exit status 33]
$ cd os && make -s run CMDLINE="test=pagefault"
hobby os: 127 MiB usable, command line "test=pagefault"
== test pagefault ==
reading 0x0000000080000000...

EXCEPTION 14: page fault, error code 0x0
  faulting address (cr2) = 0x0000000080000000: page not present, read, kernel mode
  rip=0x00000000001026c1  cs=0x8  rflags=0x246  rsp=0x000000000010bfa0
  rax=0000000000000003 rbx=0000000080000000 rcx=000000000000000a rdx=00000000000003f8
  rsi=000000000000001d rdi=0000000000000000 rbp=000000000010bfb0

KERNEL PANIC: unhandled exception 14 at rip 0x00000000001026c1
backtrace (symbolise with: addr2line -f -e kernel.elf <addresses>):
  #0 0x0000000000101b57
  #1 0x00000000001016fb
  #2 0x000000000010121a
  #3 0x00000000001017d2
  #4 0x0000000000101093
[qemu exit status 35]
$ cd os && make -s run CMDLINE="test=nosuchtest"
hobby os: 127 MiB usable, command line "test=nosuchtest"
unknown test 'nosuchtest'; available: invalid_opcode divide pagefault breakpoint echo timer
[qemu exit status 33]
```

The breakpoint handler returns, and `iretq` resumes the test after `int3` — breakpoint is a **trap**: the saved RIP points to the next instruction. The page fault is a **fault**: the saved RIP points to the faulting instruction itself, so that a handler that fixes the problem (for example, by mapping the page — Chapter 4) can return and run it again. The error code 0 says "not present, read, kernel mode", and CR2 holds the address the program tried to touch.

```console
$ cd os && addr2line -f -s -e kernel.elf $(grep -o "rip=0x[0-9a-f]*" <(make -s run CMDLINE="test=pagefault") | head -1 | cut -d= -f2)
ktest_pagefault
test_exceptions.c:12
```

### Deep & Tricky

**DT-5. The CPU pushes an error code for only some exceptions — 8 (double fault), 10–14, 17, 21, 29, 30 — and never for hardware interrupts or `int n`. A handler that assumes the wrong layout pops one word too many or too few before `iretq`, which then loads RIP from the wrong slot.** *(Intel SDM Vol. 3A, §6.3.1 Table 6-1 "Protected-Mode Exceptions and Interrupts", "Error Code" column; §6.13 "Error Code")* That is why our stubs come in two kinds and push a dummy 0 where the CPU pushes nothing: every `struct trap_frame` then has the same shape, and `add rsp, 16` is always right. A related trap: `int 14` executed by software jumps to the page-fault gate **without** pushing an error code, so a page-fault handler that pops one would corrupt its stack. Real kernels prevent user code from doing that by giving such gates DPL 0 (a ring-3 `int 14` then raises #GP instead).

## Topic 3B — Hardware interrupts

## Unit 3.2 — The PIC and the PIT timer

**One-line summary:** On a PC, devices raise interrupt lines (IRQ 0–15) on two cascaded **8259 PICs**. At power-on the first PIC maps IRQ 0–7 to vectors 8–15 — the same vectors as CPU exceptions — so every kernel **remaps** them (here to 32–47). The **PIT** (programmable interval timer) counts down from a divisor at 1,193,182 Hz and raises IRQ 0 each time it reaches zero; divisor 11932 gives about 100 interrupts per second. After handling an IRQ the kernel must send **End Of Interrupt** (EOI) to the PIC.

```c
// file: new/kernel/pic.c
/* The legacy 8259A PIC pair: master at ports 0x20/0x21, slave at 0xA0/0xA1, slave wired to master's IRQ 2. */
#include <kernel.h>

#define PIC1_CMD 0x20
#define PIC1_DATA 0x21
#define PIC2_CMD 0xA0
#define PIC2_DATA 0xA1

void pic_init(void) {
    outb(PIC1_CMD, 0x11);  io_wait();      /* ICW1: start initialisation, ICW4 will follow */
    outb(PIC2_CMD, 0x11);  io_wait();
    outb(PIC1_DATA, 32);   io_wait();      /* ICW2: master IRQ 0-7  -> vectors 32-39 */
    outb(PIC2_DATA, 40);   io_wait();      /*       slave  IRQ 8-15 -> vectors 40-47 */
    outb(PIC1_DATA, 0x04); io_wait();      /* ICW3: master has a slave on IRQ 2 */
    outb(PIC2_DATA, 0x02); io_wait();      /*       slave's cascade identity is 2 */
    outb(PIC1_DATA, 0x01); io_wait();      /* ICW4: 8086 mode */
    outb(PIC2_DATA, 0x01); io_wait();
    outb(PIC1_DATA, 0xFF);                 /* mask everything; drivers unmask their own line */
    outb(PIC2_DATA, 0xFF);
}

void pic_unmask(int irq) {
    if (irq >= 8) {
        outb(PIC2_DATA, inb(PIC2_DATA) & ~(1 << (irq - 8)));
        irq = 2;                           /* the slave's interrupts arrive through master line 2 */
    }
    outb(PIC1_DATA, inb(PIC1_DATA) & ~(1 << irq));
}

void pic_eoi(int irq) {
#ifndef NO_EOI                             /* experiment switch for DT-6 */
    if (irq >= 8) outb(PIC2_CMD, 0x20);
    outb(PIC1_CMD, 0x20);
#else
    (void)irq;
#endif
}
```

```c
// file: new/kernel/timer.c
#include <kernel.h>

volatile uint64_t ticks;
unsigned timer_hz;

static void timer_irq(struct trap_frame *f) {
    (void)f;
    ticks++;
}

void timer_init(unsigned hz) {
    unsigned divisor = 1193182 / hz;
    timer_hz = hz;
    outb(0x43, 0x36);                      /* channel 0, low byte then high byte, mode 3 (square wave) */
    outb(0x40, (uint8_t)divisor);
    outb(0x40, (uint8_t)(divisor >> 8));
    irq_register(0, timer_irq);
    pic_unmask(0);
}

/* The CMOS real-time clock: an independent clock to check the PIT against. */
uint8_t rtc_read(uint8_t reg) {
    while (outb(0x70, 0x0A), inb(0x71) & 0x80) { }     /* wait while an update is in progress */
    outb(0x70, reg);
    return inb(0x71);
}
```

The timer test counts how many ticks arrive during one second of the **RTC** (the battery-backed real-time clock), which is read by polling and does not depend on interrupts at all.

```c
// file: new/kernel/test_timer.c
#include <kernel.h>

static void wait_for_rtc_second(void) {
    uint8_t s = rtc_read(0x00);
    while (rtc_read(0x00) == s) { }
}

KTEST(timer) {
    wait_for_rtc_second();
    uint64_t t0 = ticks;
    wait_for_rtc_second();
    uint64_t t1 = ticks;
    kprintf("timer interrupts during one RTC second: %lu (PIT programmed for %u Hz); %lu since boot\n",
            t1 - t0, timer_hz, t1);
    KASSERT(t1 - t0 >= timer_hz - 2 && t1 - t0 <= timer_hz + 2);
}
```

```console
$ cd os && make -s run CMDLINE="test=timer"
hobby os: 127 MiB usable, command line "test=timer"
== test timer ==
timer interrupts during one RTC second: 100 (PIT programmed for 100 Hz); 164 since boot
== test timer passed ==
[qemu exit status 33]
```

### Deep & Tricky

**DT-6. The 8259 does not deliver a second interrupt of the same or lower priority until it receives EOI: the in-service bit for the current IRQ stays set. Forget the EOI and the timer fires exactly once — the kernel keeps running, but time has stopped.** *(Intel 8259A datasheet, "End of Interrupt": the In-Service bit "must be reset by an EOI command"; fully nested mode blocks equal and lower priorities while it is set)* Build the same kernel with `-DNO_EOI` and run the same test: the self-check fails. Exactly one tick was counted since boot, before the measurement even started; after it, the PIC never delivers IRQ 0 again.

```console
$ rm -rf noeoi && cp -r os noeoi && cd noeoi && make -s clean && make -s EXTRA_CFLAGS=-DNO_EOI && make -s run CMDLINE="test=timer" EXTRA_CFLAGS=-DNO_EOI
hobby os: 127 MiB usable, command line "test=timer"
== test timer ==
timer interrupts during one RTC second: 0 (PIT programmed for 100 Hz); 1 since boot

KERNEL PANIC: assertion failed: t1 - t0 >= timer_hz - 2 && t1 - t0 <= timer_hz + 2 at kernel/test_timer.c:16
backtrace (symbolise with: addr2line -f -e kernel.elf <addresses>):
  #0 0x0000000000101b57
  #1 0x000000000010281b
  #2 0x00000000001017d2
  #3 0x0000000000101093
[qemu exit status 35]
```

Two more rules sit on top of DT-6. First, for an IRQ from the slave PIC (8–15) EOI must go to **both** chips. Second, the PIC can raise a **spurious** IRQ 7 (or 15) when a line drops at the wrong moment; for those the kernel must **not** send EOI (it checks the PIC's in-service register first). Our kernel never unmasks IRQ 7, so it skips that check. Linux's code for all of this is in `arch/x86/kernel/i8259.c`.

## Unit 3.3 — Serial input: from interrupt handler to normal code

**One-line summary:** When a byte arrives, the UART raises IRQ 4. The handler reads all waiting bytes from the UART into a **ring buffer** and returns. Normal code takes bytes out of the buffer. The buffer is shared between the handler and normal code, so normal code disables interrupts while it checks it. To wait for input without busy-looping, the kernel executes `hlt`, which sleeps until the next interrupt — and it must use exactly `sti; hlt` so that an interrupt cannot slip in between "the buffer is empty" and "go to sleep" (DT-7).

```c
// file: new/kernel/console.c
#include <kernel.h>

#define COM1 0x3F8

void serial_init(void) {
    outb(COM1 + 1, 0x00);         /* disable UART interrupts (for now) */
    outb(COM1 + 3, 0x80);         /* DLAB = 1: next two ports are the baud divisor */
    outb(COM1 + 0, 0x01);         /* divisor 1 -> 115200 baud */
    outb(COM1 + 1, 0x00);
    outb(COM1 + 3, 0x03);         /* 8 data bits, no parity, 1 stop bit; DLAB = 0 */
    outb(COM1 + 2, 0xC7);         /* enable and clear FIFOs */
    outb(COM1 + 4, 0x0B);         /* DTR, RTS, OUT2 (OUT2 connects the UART's interrupt line to the PIC) */
}

void serial_putc(char c) {
    if (c == '\n') serial_putc('\r');
    while (!(inb(COM1 + 5) & 0x20)) { }   /* wait for an empty transmit holding register */
    outb(COM1, (uint8_t)c);
}

/* ---- receive path: IRQ 4 fills a ring buffer, serial_getc empties it ---- */
#define RX_SIZE 256                       /* a power of two, so the indices can wrap freely */
static char rx_buf[RX_SIZE];
static volatile uint32_t rx_head, rx_tail; /* head: written only by the IRQ handler; tail: only by serial_getc */
volatile uint64_t serial_irq_count, serial_rx_bytes;

static void serial_irq(struct trap_frame *f) {
    (void)f;
    serial_irq_count++;
    while (inb(COM1 + 5) & 0x01) {        /* data ready: drain everything the FIFO holds */
        char c = (char)inb(COM1);
        serial_rx_bytes++;
        if (rx_head - rx_tail < RX_SIZE) rx_buf[rx_head++ % RX_SIZE] = c;   /* full buffer: drop the byte */
    }
}

void serial_enable_rx_irq(void) {
    irq_register(4, serial_irq);
    outb(COM1 + 1, 0x01);                 /* IER: interrupt when received data is available */
    pic_unmask(4);
}

int serial_getc(void) {
    for (;;) {
        cli();
        if (rx_head != rx_tail) {
            char c = rx_buf[rx_tail++ % RX_SIZE];
            sti();
            return (unsigned char)c;
        }
        __asm__ volatile("sti; hlt" ::: "memory");   /* DT-7: no interrupt can arrive between these two */
    }
}

static volatile uint16_t *const vga = (uint16_t *)0xB8000;
static int row, col;

void vga_clear(void) {
    for (int i = 0; i < 80 * 25; i++) vga[i] = 0x0720;   /* space, light grey on black */
    row = col = 0;
}

void vga_putc(char c) {
    if (c == '\n') { col = 0; row++; }
    else if (c == '\b') { if (col) vga[row * 80 + --col] = 0x0720; }
    else { vga[row * 80 + col] = (uint16_t)(0x0700 | (uint8_t)c); if (++col == 80) { col = 0; row++; } }
    if (row == 25) {                                      /* scroll up one line */
        memmove((void *)vga, (const void *)(vga + 80), 80 * 24 * 2);
        for (int i = 0; i < 80; i++) vga[24 * 80 + i] = 0x0720;
        row = 24;
    }
}
```

```c
// file: new/kernel/test_serial.c
#include <kernel.h>

KTEST(echo) {
    char line[80];
    size_t n = 0;
    for (;;) {
        int c = serial_getc();
        if (c == '\n' || c == '\r') break;
        if (n + 1 < sizeof line) line[n++] = (char)c;
    }
    line[n] = '\0';
    for (size_t i = 0; i < n / 2; i++) { char t = line[i]; line[i] = line[n - 1 - i]; line[n - 1 - i] = t; }
    kprintf("received %zu characters, reversed: \"%s\"\n", n, line);
    kprintf("%lu bytes arrived in %lu serial interrupts\n", serial_rx_bytes, serial_irq_count);
}
```

The input is piped into QEMU's standard input, which is connected to COM1. The `sleep` lets the kernel boot before the text is sent — just like a person who types after the prompt appears.

```console
$ cd os && (sleep 4; printf 'hello from the host, via COM1\n') | make -s run CMDLINE="test=echo"
hobby os: 127 MiB usable, command line "test=echo"
== test echo ==
received 29 characters, reversed: "1MOC aiv ,tsoh eht morf olleh"
30 bytes arrived in 2 serial interrupts
== test echo passed ==
[qemu exit status 33]
```

There are fewer interrupts than bytes: the UART has a 16-byte FIFO, and while the CPU is busy the bytes collect there; one interrupt then drains several of them. (The exact count depends on timing.) A handler must therefore always drain *all* waiting data — reading only one byte per interrupt would leave bytes stuck until the next one arrives.

### Deep & Tricky

**DT-7. `sti` enables interrupts only after the *next* instruction completes (the "interrupt shadow"), so `sti; hlt` is atomic: an interrupt that is already pending is taken after `hlt` begins and wakes it up. Written as two separate steps — check the buffer with interrupts on, then `hlt` — an IRQ that arrives between the check and `hlt` is handled, and then the CPU sleeps anyway, with data sitting in the buffer, until some unrelated interrupt happens.** *(Intel SDM Vol. 2B, STI instruction: "the processor ... does not respond to external, maskable interrupts until after the next instruction is executed"; the description gives `STI; HLT` as the example)* In our kernel the 100 Hz timer would rescue the sleeper within 10 ms, which is exactly why this bug survives testing — it appears as an occasional delay, not a hang. Linux's idle loop uses the same pairing (`native_safe_halt()` is `sti; hlt`). Chapter 5 meets the general form of this race as the **lost wake-up** problem.

### Pattern

**P-4. Keep interrupt handlers short: acknowledge the device, move the data into a buffer, update a counter, return. Do the real work in normal code that takes the data out with interrupts disabled only for the moment of the check.** *(Practice)* While a handler runs, interrupts are off (interrupt gate) and the PIC blocks its own line and all lower-priority lines until EOI, so a slow handler delays every other device and the clock. Linux formalises this split as the **top half** (the hard IRQ handler) and the **bottom half** (softirqs, tasklets, workqueues — the deferred-work mechanisms that Chapter 9's kernel threads are the simplest form of). *Trade-off:* the extra buffering adds latency and a place where data can be dropped when the buffer is full (our handler counts and drops); the alternative — processing in the handler — makes every other interrupt wait.

### Mastery check

1. **Predict.** If `isr_common` used `pop` in the same order as `push` (instead of reverse order), what would the interrupted code see after `iretq`?
2. **Predict.** In `serial_getc`, why is `rx_head` read with interrupts disabled even though only one instruction reads it?
3. **Predict.** Why would the `timer` test still pass if we forgot `pic_unmask(4)`?

## Chapter 3 Project — `kshell`: an interactive kernel shell

**Goal:** a small command shell running inside the kernel, reading lines from COM1 with basic line editing (backspace, Ctrl-U to clear the line), and offering commands to inspect the system and to trigger faults: `help`, `uptime`, `ticks`, `mem`, `echo`, `int3`, `crash`, `irqs`, `halt`. Because `kmain` calls `shell_run()` when no test is requested, adding `shell.c` replaces the weak default — no other file changes.

```c
// file: proj/kernel/shell.c
#include <kernel.h>

static size_t read_line(char *buf, size_t size) {
    size_t n = 0;
    kprintf("kshell> ");
    for (;;) {
        int c = serial_getc();
        if (c == '\r' || c == '\n') { kprintf("\n"); break; }
        if ((c == 0x7F || c == '\b') && n > 0) { n--; kprintf("\b \b"); continue; }   /* backspace */
        if (c == 0x15) { while (n) { n--; kprintf("\b \b"); } continue; }            /* Ctrl-U */
        if (c >= 0x20 && c < 0x7F && n + 1 < size) { buf[n++] = (char)c; kprintf("%c", c); }
    }
    buf[n] = '\0';
    return n;
}

static void cmd_help(const char *arg);
static void cmd_uptime(const char *arg) {
    (void)arg;
    uint64_t t = ticks;
    kprintf("up %lu.%02lu s (%lu ticks at %u Hz)\n", t / timer_hz, t % timer_hz * 100 / timer_hz, t, timer_hz);
}
static void cmd_mem(const char *arg) {
    (void)arg;
    kprintf("usable RAM %lu KiB; kernel image %p-%p (%lu KiB)\n", usable_ram_bytes >> 10, (void *)_kernel_start,
            (void *)_kernel_end, (uint64_t)(_kernel_end - _kernel_start) >> 10);
}
static void cmd_echo(const char *arg) { kprintf("%s\n", arg); }
static void cmd_int3(const char *arg) { (void)arg; __asm__ volatile("int3"); }
static void cmd_crash(const char *arg) {
    uint64_t addr = 0;
    for (const char *p = arg; *p; p++) {                         /* parse a hex address, default 0xdead0000 */
        int d = *p >= '0' && *p <= '9' ? *p - '0' : *p >= 'a' && *p <= 'f' ? *p - 'a' + 10 : -1;
        if (d >= 0) addr = addr * 16 + (uint64_t)d;
    }
    if (!addr) addr = 0xdead0000;
    kprintf("writing to 0x%lx...\n", addr);
    *(volatile uint64_t *)addr = 1;
}
static void cmd_irqs(const char *arg) {
    (void)arg;
    kprintf("timer ticks %lu, serial interrupts %lu, serial bytes %lu\n", ticks, serial_irq_count, serial_rx_bytes);
}
static void cmd_halt(const char *arg) { (void)arg; kprintf("halting\n"); qemu_exit(0x10); }

static const struct { const char *name, *help; void (*fn)(const char *); } commands[] = {
    {"help", "list commands", cmd_help},
    {"uptime", "time since the timer started", cmd_uptime},
    {"mem", "memory summary", cmd_mem},
    {"echo", "print the arguments", cmd_echo},
    {"int3", "execute a breakpoint (handled, continues)", cmd_int3},
    {"crash", "write to an unmapped address: crash [hexaddr]", cmd_crash},
    {"irqs", "interrupt counters", cmd_irqs},
    {"halt", "power off (exit QEMU)", cmd_halt},
};
#define NCOMMANDS (sizeof commands / sizeof commands[0])

static void cmd_help(const char *arg) {
    (void)arg;
    for (size_t i = 0; i < NCOMMANDS; i++) kprintf("  %-8s %s\n", commands[i].name, commands[i].help);
}

void shell_run(void) {
    kprintf("kshell ready; type 'help'\n");
    char line[128];
    for (;;) {
        if (!read_line(line, sizeof line)) continue;
        char *arg = line;
        while (*arg && *arg != ' ') arg++;
        if (*arg) *arg++ = '\0';
        size_t i;
        for (i = 0; i < NCOMMANDS; i++)
            if (strncmp(line, commands[i].name, 16) == 0) { commands[i].fn(arg); break; }
        if (i == NCOMMANDS) kprintf("unknown command '%s'\n", line);
    }
}
```

The test drives the shell like a user: each line is sent after a short pause. `\x7f` is a backspace and `\x15` is Ctrl-U, so `memx⌫` becomes `mem` and the junk before Ctrl-U disappears.

```console
$ rm -rf final && cp -r os final && cp -r proj/. final/ && make -s -C final clean && make -s -C final && echo "final = os + proj"
final = os + proj
$ cd final && (sleep 4; for l in 'help' 'memx\x7f' 'echo interrupts work' 'junk\x15uptime' 'int3' 'irqs' 'reboot' 'halt'; do printf "$l\n"; sleep 0.3; done) | make -s run
hobby os: 127 MiB usable, command line ""
kshell ready; type 'help'
kshell> help
  help     list commands
  uptime   time since the timer started
  mem      memory summary
  echo     print the arguments
  int3     execute a breakpoint (handled, continues)
  crash    write to an unmapped address: crash [hexaddr]
  irqs     interrupt counters
  halt     power off (exit QEMU)
kshell> memx 
usable RAM 130559 KiB; kernel image 0x0000000000100000-0x000000000010d358 (52 KiB)
kshell> echo interrupts work
interrupts work
kshell> junk    uptime
up 4.31 s (431 ticks at 100 Hz)
kshell> int3
breakpoint at rip=0x00000000001024e5
kshell> irqs
timer ticks 491, serial interrupts 7, serial bytes 54
kshell> reboot
unknown command 'reboot'
kshell> halt
halting
[qemu exit status 33]
$ cd final && (sleep 4; printf 'echo about to crash\ncrash 40000000\n') | make -s run
hobby os: 127 MiB usable, command line ""
kshell ready; type 'help'
kshell> echo about to crash
about to crash
kshell> crash 40000000
writing to 0x40000000...

EXCEPTION 14: page fault, error code 0x2
  faulting address (cr2) = 0x0000000040000000: page not present, write, kernel mode
  rip=0x00000000001025a9  cs=0x8  rflags=0x246  rsp=0x000000000010bef0
  rax=0000000000000006 rbx=0000000040000000 rcx=000000000000000a rdx=00000000000003f8
  rsi=0000000000000018 rdi=0000000000000000 rbp=000000000010bf00

KERNEL PANIC: unhandled exception 14 at rip 0x00000000001025a9
backtrace (symbolise with: addr2line -f -e kernel.elf <addresses>):
  #0 0x0000000000101b57
  #1 0x00000000001016fb
  #2 0x000000000010121a
  #3 0x0000000000102827
  #4 0x00000000001017ed
  #5 0x0000000000101093
[qemu exit status 35]
$ cd final && addr2line -f -s -e kernel.elf $(grep -o "rip=0x[0-9a-f]*" <((sleep 4; printf 'crash\n') | make -s run) | head -1 | cut -d= -f2)
cmd_crash
shell.c:39
```

In the captured log above, the backspace characters the kernel printed are invisible, so `memx` and `junk` still show; on a real terminal they are erased. The commands that ran prove the editing worked: `mem` and `uptime`. Note how the typed characters are echoed by the kernel itself: QEMU does not echo; the shell prints each accepted character, and prints `\b \b` (back, space, back) to erase on backspace. The crash at `0x40000000` is the first byte past our 1 GiB identity map; the write sets bit 1 of the error code, and `addr2line` points straight at the line in `cmd_crash`.

### Review checklist

- What happens if a line longer than 127 characters is typed?
- `cmd_uptime` reads `ticks` once into `t`. What could go wrong if it read `ticks` three times?
- Which command would be dangerous to offer once user programs exist, and why?

## Answers to the Chapter 3 prediction questions

**3.3 Q1.** Every register would receive another register's value — `rax` would get `r15`'s value and so on — so the interrupted code would continue with scrambled registers and almost certainly crash or compute garbage. Saving and restoring is a stack: the last register pushed must be the first popped.

**3.3 Q2.** Reading `rx_head` alone is fine, but the check and the decision that follows (take a byte, or go to sleep) must not be separated by an interrupt — see DT-7. With interrupts disabled, the state cannot change between the check and the `sti; hlt`. (On one CPU, disabling interrupts is the whole of the "lock". With several CPUs it is not enough — Chapter 7.)

**3.3 Q3.** The timer test depends only on IRQ 0. With IRQ 4 masked, serial bytes would sit in the UART and no interrupt would announce them — the `echo` test and the shell would sleep forever, but the timer would keep ticking.

# Chapter 4 — Memory Management

*Goal: give the kernel real memory management in three layers. At the bottom, a **physical frame allocator** hands out 4 KiB frames of RAM. In the middle, **paging** code maps virtual pages to those frames by editing the CPU's four-level page tables — including mapping pages lazily on a page fault. At the top, a **kernel heap** (`kmalloc`/`kfree`) carves those pages into objects of any size.*

**What you will be able to do after this chapter**

- Build a bitmap frame allocator from the Multiboot2 memory map, and know which memory must not be handed out.
- Split a 48-bit virtual address into its four table indices and an offset, and walk the tables by hand.
- Map, unmap, and translate pages; know when the TLB must be flushed.
- Handle a page fault by mapping a page and resuming the faulting instruction (demand paging).
- Write a first-fit heap allocator with splitting, coalescing, and corruption checks — and test it on the host with sanitizers.

The kernel continues from Chapter 3's `final/`. New and changed files are in `new/`; `os/` is Chapter 3's kernel plus `new/`.

```console
$ rm -rf os && cp -r ../ch03/final os && cp -r new/. os/ && make -s -C os clean && echo ready
ready
```

## Topic 4A — Physical memory

## Unit 4.1 — The frame allocator

**One-line summary:** Physical RAM is managed in 4 KiB **frames**. A **bitmap** with one bit per frame (1 = used) is the simplest allocator: 128 MiB needs 32,768 bits = 4 KiB of bitmap. Start with every bit set, clear the bits of frames inside "usable" regions of the memory map, then set them again for memory that is usable but already occupied: the first 1 MiB (BIOS data, real-mode structures), the kernel image, the bitmap itself — and the bootloader's own data (DT-8).

```c
// file: new/include/mm.h
#pragma once
#include <kernel.h>

#define PAGE_SIZE 4096UL
#define ALIGN_UP(x, a)   (((x) + (a) - 1) & ~((uint64_t)(a) - 1))
#define ALIGN_DOWN(x, a) ((x) & ~((uint64_t)(a) - 1))

/* boot information (multiboot2.c) */
struct mb2_mmap_entry { uint64_t base, length; uint32_t type, reserved; };
extern struct mb2_mmap_entry usable_regions[16];
extern int usable_region_count;
extern uintptr_t mb_info_start, mb_info_end;

/* physical frames (pmm.c) */
void     pmm_init(void);
uint64_t pmm_alloc(void);                 /* physical address of a free frame, or 0 when out of memory */
void     pmm_free(uint64_t frame);
uint64_t pmm_free_count(void);
uint64_t pmm_total_count(void);

/* page tables (vmm.c) */
#define PTE_PRESENT  (1UL << 0)
#define PTE_WRITABLE (1UL << 1)
#define PTE_USER     (1UL << 2)
#define PTE_HUGE     (1UL << 7)
#define PTE_ADDR     0x000FFFFFFFFFF000UL
#define LAZY_START   0xFFFFC00000000000UL    /* pages here are mapped on first touch */
#define LAZY_END     0xFFFFC00040000000UL
void      vmm_init(void);
bool      vmm_map(uint64_t virt, uint64_t phys, uint64_t flags);
uint64_t  vmm_unmap(uint64_t virt);           /* returns the frame that was mapped, or 0 */
bool      vmm_translate(uint64_t virt, uint64_t *phys);
uint64_t *vmm_pte(uint64_t virt, bool create);
void      vmm_walk(uint64_t virt);           /* print every level of the translation */
bool      vmm_handle_fault(struct trap_frame *f);
extern uint64_t lazy_faults;
static inline void invlpg(uint64_t virt) { __asm__ volatile("invlpg (%0)" : : "r"(virt) : "memory"); }

/* kernel heap (heap.c) */
#define HEAP_START 0xFFFF800000000000UL
void *kmalloc(size_t n);
void  kfree(void *p);
void *morecore(size_t bytes);                 /* extend the heap region by bytes; returns its old end */
struct heap_stats { size_t blocks, free_blocks, used_bytes, free_bytes, largest_free, heap_bytes; };
void  kheap_check(struct heap_stats *s);      /* validates every block; panics on corruption */
```

```c
// file: new/kernel/multiboot2.c
#include <mm.h>

struct mb2_tag { uint32_t type, size; };
struct mb2_tag_mmap { uint32_t type, size, entry_size, entry_version; struct mb2_mmap_entry entries[]; };

const char *kernel_cmdline = "";
uint64_t usable_ram_bytes;
struct mb2_mmap_entry usable_regions[16];
int usable_region_count;
uintptr_t mb_info_start, mb_info_end;         /* the bootloader's structure: must survive until we are done with it */

void multiboot2_parse(uint32_t magic, uintptr_t info) {
    if (magic != 0x36D76289) panic("not started by a Multiboot2 loader (magic 0x%x)", magic);
    mb_info_start = info;
    mb_info_end = info + *(uint32_t *)info;
    for (struct mb2_tag *t = (struct mb2_tag *)(info + 8); t->type != 0;
         t = (struct mb2_tag *)((uintptr_t)t + ((t->size + 7) & ~7u))) {
        if (t->type == 1) kernel_cmdline = (const char *)(t + 1);      /* points INTO the boot information */
        if (t->type == 6) {
            struct mb2_tag_mmap *m = (struct mb2_tag_mmap *)t;
            int n = (int)((m->size - sizeof *m) / m->entry_size);
            for (int i = 0; i < n; i++) {
                struct mb2_mmap_entry *e = (struct mb2_mmap_entry *)((uintptr_t)m->entries + (uintptr_t)i * m->entry_size);
                if (e->type == 1 && usable_region_count < 16) {
                    usable_regions[usable_region_count++] = *e;
                    usable_ram_bytes += e->length;
                }
            }
        }
    }
}

bool cmdline_get(const char *key, char *out, size_t n) {
    size_t klen = strlen(key);
    for (const char *p = kernel_cmdline; *p; ) {
        while (*p == ' ') p++;
        if (strncmp(p, key, klen) == 0 && p[klen] == '=') {
            p += klen + 1;
            size_t i = 0;
            while (*p && *p != ' ' && i + 1 < n) out[i++] = *p++;
            out[i] = '\0';
            return true;
        }
        while (*p && *p != ' ') p++;
    }
    return false;
}
```

```c
// file: new/kernel/pmm.c
/* Physical memory manager: one bit per 4 KiB frame, 1 = in use. */
#include <mm.h>

static uint8_t *bitmap;
static uint64_t nframes, nfree, next_hint;

static bool used(uint64_t f) { return bitmap[f / 8] & (1 << (f % 8)); }
static void set_used(uint64_t f) { if (!used(f)) { bitmap[f / 8] |= (uint8_t)(1 << (f % 8)); nfree--; } }
static void set_free(uint64_t f) { if (used(f)) { bitmap[f / 8] &= (uint8_t)~(1 << (f % 8)); nfree++; } }

static void reserve(uint64_t start, uint64_t end) {          /* [start, end), any alignment */
    for (uint64_t f = start / PAGE_SIZE; f < ALIGN_UP(end, PAGE_SIZE) / PAGE_SIZE && f < nframes; f++) set_used(f);
}

void pmm_init(void) {
    uint64_t top = 0;
    for (int i = 0; i < usable_region_count; i++) {
        uint64_t end = usable_regions[i].base + usable_regions[i].length;
        if (end > top) top = end;
    }
    nframes = top / PAGE_SIZE;
    uint64_t bytes = (nframes + 7) / 8;
#ifdef NAIVE_BITMAP                                           /* experiment switch for DT-8: forget the boot info */
    bitmap = (uint8_t *)ALIGN_UP((uintptr_t)_kernel_end, PAGE_SIZE);
    mb_info_start = mb_info_end = 0;
#else
    uintptr_t after = (uintptr_t)_kernel_end > mb_info_end ? (uintptr_t)_kernel_end : mb_info_end;
    bitmap = (uint8_t *)ALIGN_UP(after, PAGE_SIZE);
#endif
    memset(bitmap, 0xFF, bytes);                              /* everything "used" ... */
    nfree = 0;
    for (int i = 0; i < usable_region_count; i++) {           /* ... except whole frames inside usable regions */
        uint64_t s = ALIGN_UP(usable_regions[i].base, PAGE_SIZE) / PAGE_SIZE;
        uint64_t e = ALIGN_DOWN(usable_regions[i].base + usable_regions[i].length, PAGE_SIZE) / PAGE_SIZE;
        for (uint64_t f = s; f < e; f++) { bitmap[f / 8] &= (uint8_t)~(1 << (f % 8)); nfree++; }
    }
    reserve(0, 0x100000);                                     /* real-mode area, BIOS data, legacy video */
    reserve((uintptr_t)_kernel_start, (uintptr_t)_kernel_end);
    reserve(mb_info_start, mb_info_end);
    reserve((uintptr_t)bitmap, (uintptr_t)bitmap + bytes);
    next_hint = 0;
}

uint64_t pmm_alloc(void) {
    for (uint64_t i = 0; i < nframes; i++) {
        uint64_t f = (next_hint + i) % nframes;
        if (!used(f)) {
            set_used(f);
            next_hint = f + 1;
            return f * PAGE_SIZE;
        }
    }
    return 0;                                                 /* frame 0 is reserved, so 0 means "none" */
}

void pmm_free(uint64_t frame) {
    KASSERT(frame % PAGE_SIZE == 0 && frame / PAGE_SIZE < nframes);
    if (!used(frame / PAGE_SIZE)) panic("pmm_free: double free of frame 0x%lx", frame);
    set_free(frame / PAGE_SIZE);
}

uint64_t pmm_free_count(void) { return nfree; }
uint64_t pmm_total_count(void) { return nframes; }
```

```c
// file: new/kernel/main.c
#include <mm.h>

extern const struct ktest __ktests_start[], __ktests_end[];

__attribute__((weak)) void shell_run(void) { kprintf("no shell in this build\n"); }

static void run_test(const char *name) {
    for (const struct ktest *t = __ktests_start; t < __ktests_end; t++) {
        if (strncmp(t->name, name, 64) == 0) {
            kprintf("== test %s ==\n", t->name);
            t->fn();
            kprintf("== test %s passed ==\n", t->name);
            return;
        }
    }
    kprintf("unknown test '%s'; available:", name);
    for (const struct ktest *t = __ktests_start; t < __ktests_end; t++) kprintf(" %s", t->name);
    kprintf("\n");
}

void kmain(uint32_t magic, uintptr_t info) {
    serial_init();
    vga_clear();
    multiboot2_parse(magic, info);
    idt_init();
    pmm_init();
    vmm_init();
    kprintf("hobby os: %lu MiB usable, %lu of %lu frames free, command line \"%s\"\n", usable_ram_bytes >> 20,
            pmm_free_count(), pmm_total_count(), kernel_cmdline);
    pic_init();
    timer_init(100);
    serial_enable_rx_irq();
    sti();
    char name[32];
    if (cmdline_get("test", name, sizeof name)) run_test(name);
    else shell_run();
    qemu_exit(0x10);
}
```

```c
// file: new/kernel/test_pmm.c
#include <mm.h>

KTEST(pmm) {
    kprintf("kernel %p-%p, boot info %p-%p\n", (void *)_kernel_start, (void *)_kernel_end,
            (void *)mb_info_start, (void *)mb_info_end);
    uint64_t before = pmm_free_count();
    static uint64_t frames[1000];
    for (int i = 0; i < 1000; i++) {
        frames[i] = pmm_alloc();
        KASSERT(frames[i] != 0 && frames[i] % PAGE_SIZE == 0);
        memset((void *)frames[i], 0, PAGE_SIZE);                               /* use it, as a page table would */
    }
    kprintf("command line after zeroing 1000 allocated frames: \"%s\"\n", kernel_cmdline);
    for (int i = 0; i < 1000; i++) {
        KASSERT(frames[i] >= 0x100000);                                        /* never low memory */
        KASSERT(frames[i] >= (uint64_t)_kernel_end || frames[i] + PAGE_SIZE <= (uint64_t)_kernel_start);
        KASSERT(frames[i] >= ALIGN_UP(mb_info_end, PAGE_SIZE) || frames[i] + PAGE_SIZE <= mb_info_start);
        for (int j = i + 1; j < 1000; j++) KASSERT(frames[i] != frames[j]);    /* no frame handed out twice */
    }
    kprintf("1000 frames allocated: first 0x%lx, last 0x%lx; free %lu -> %lu\n", frames[0], frames[999], before,
            pmm_free_count());
    for (int i = 0; i < 1000; i++) pmm_free(frames[i]);
    KASSERT(pmm_free_count() == before);
    uint64_t n = 0;
    while (pmm_alloc()) n++;                                                   /* exhaust physical memory */
    kprintf("allocated every free frame: %lu frames = %lu MiB; then pmm_alloc() returns 0\n", n, n * PAGE_SIZE >> 20);
    KASSERT(n == before);
}
```

```console
$ cd os && make -s && make -s run CMDLINE="test=pmm"
hobby os: 127 MiB usable, 32457 of 32736 frames free, command line "test=pmm"
== test pmm ==
kernel 0x0000000000100000-0x0000000000114238, boot info 0x0000000000127068-0x0000000000127658
command line after zeroing 1000 allocated frames: "test=pmm"
1000 frames allocated: first 0x115000, last 0x4fe000; free 32457 -> 31457
allocated every free frame: 32457 frames = 126 MiB; then pmm_alloc() returns 0
== test pmm passed ==
[qemu exit status 33]
```

### Deep & Tricky

**DT-8. The memory map says "usable", but at boot some of that usable RAM is still in use: the bootloader places its boot-information structure (and any modules) in ordinary usable memory. The kernel must not overwrite it until it has finished reading it — which includes every pointer it kept into it, such as our `kernel_cmdline`.** *(Multiboot2 specification, §3.6 "Boot information format": the boot information "may be placed anywhere in memory by the boot loader … It is the operating system's responsibility to avoid overwriting this memory until it is done using it.")* GRUB put it in usable memory a little above our kernel image. Build with `-DNAIVE_BITMAP`, which forgets to reserve that range: the allocator then hands out its frame like any other, the test zeroes it as a page table would be zeroed, and the command line we kept a pointer to is gone.

```console
$ rm -rf naive && cp -r os naive && cd naive && make -s clean && make -s EXTRA_CFLAGS=-DNAIVE_BITMAP && make -s run CMDLINE="test=pmm" EXTRA_CFLAGS=-DNAIVE_BITMAP 2>&1 | head -6
hobby os: 127 MiB usable, 32458 of 32736 frames free, command line "test=pmm"
== test pmm ==
kernel 0x0000000000100000-0x0000000000114238, boot info 0x0000000000000000-0x0000000000000000
command line after zeroing 1000 allocated frames: ""
1000 frames allocated: first 0x116000, last 0x4fd000; free 32458 -> 31458
allocated every free frame: 32458 frames = 126 MiB; then pmm_alloc() returns 0
```

The command line is now empty, and nothing else complains: the test's check on the boot-information range cannot help, because this build has forgotten the range entirely, and the test goes on to pass. Nothing crashed at the moment of damage; the kernel would only notice later, when it read the data again — if it noticed at all. This is what makes such bugs hard to find. Linux solves the same problem with **memblock**, an early allocator that knows about the kernel image, the initrd, and firmware-reserved ranges before the real page allocator starts.

## Topic 4B — Virtual memory

## Unit 4.2 — Four-level paging: map, unmap, translate

**One-line summary:** An x86-64 virtual address uses 48 bits: four 9-bit **indices** (PML4, PDPT, PD, PT — each table has 512 eight-byte entries) and a 12-bit **offset** in the 4 KiB page. Bits 48–63 must copy bit 47 (**canonical** addresses), which splits the space into a lower half (user programs, later) and a higher half (kernel). Each entry holds a physical address plus flags: present, writable, user, and "huge" (at the PD level it maps a whole 2 MiB page — our boot identity map). To map a page, walk from CR3 and create missing tables on the way.

```text
virtual address 0xFFFF800000003123:

  bits 63-48 | 47-39 (PML4) | 38-30 (PDPT) | 29-21 (PD) | 20-12 (PT) | 11-0 (offset)
  ffff       | 256          | 0            | 0          | 3          | 0x123
```

Our kernel's tables are allocated from the frame allocator; frames below 1 GiB are identity-mapped (virtual address = physical address) by the boot code, so the kernel can read and write a table simply by using its physical address as a pointer.

```c
// file: new/kernel/vmm.c
#include <mm.h>

extern uint64_t boot_pml4[512];               /* from boot.asm; CR3 points to it */
static uint64_t *pml4 = boot_pml4;
uint64_t lazy_faults;

static unsigned idx(uint64_t virt, int level) { return (virt >> (12 + 9 * level)) & 511; }   /* level 3 = PML4 */

static uint64_t *next_table(uint64_t *table, unsigned i, bool create) {
    if (!(table[i] & PTE_PRESENT)) {
        if (!create) return NULL;
        uint64_t frame = pmm_alloc();
        if (!frame) return NULL;
        memset((void *)frame, 0, PAGE_SIZE);   /* identity mapped: physical address usable as a pointer */
        table[i] = frame | PTE_PRESENT | PTE_WRITABLE;
    }
    if (table[i] & PTE_HUGE) return NULL;      /* a 2 MiB or 1 GiB page: there is no lower table */
    return (uint64_t *)(table[i] & PTE_ADDR);
}

uint64_t *vmm_pte(uint64_t virt, bool create) {
    uint64_t *t = pml4;
    for (int level = 3; level > 0; level--)
        if (!(t = next_table(t, idx(virt, level), create))) return NULL;
    return &t[idx(virt, 0)];
}

bool vmm_map(uint64_t virt, uint64_t phys, uint64_t flags) {
    KASSERT(virt % PAGE_SIZE == 0 && phys % PAGE_SIZE == 0);
    uint64_t *pte = vmm_pte(virt, true);
    if (!pte) return false;
    if (*pte & PTE_PRESENT) panic("vmm_map: 0x%lx is already mapped", virt);
    *pte = phys | flags | PTE_PRESENT;         /* not-present -> present needs no TLB flush */
    return true;
}

uint64_t vmm_unmap(uint64_t virt) {
    uint64_t *pte = vmm_pte(virt, false);
    if (!pte || !(*pte & PTE_PRESENT)) return 0;
    uint64_t phys = *pte & PTE_ADDR;
    *pte = 0;
    invlpg(virt);                              /* DT-9: the TLB may still hold the old translation */
    return phys;
}

bool vmm_translate(uint64_t virt, uint64_t *phys) {
    uint64_t *t = pml4;
    for (int level = 3; level >= 0; level--) {
        uint64_t e = t[idx(virt, level)];
        if (!(e & PTE_PRESENT)) return false;
        if (level == 0 || (e & PTE_HUGE)) {
            uint64_t page = 1UL << (12 + 9 * level);                   /* 4 KiB, 2 MiB or 1 GiB */
            *phys = (e & PTE_ADDR & ~(page - 1)) | (virt & (page - 1));
            return true;
        }
        t = (uint64_t *)(e & PTE_ADDR);
    }
    return false;
}

void vmm_walk(uint64_t virt) {
    static const char *const names[] = {"PT", "PD", "PDPT", "PML4"};
    kprintf("walk of %p:\n", (void *)virt);
    uint64_t *t = pml4;
    for (int level = 3; level >= 0; level--) {
        unsigned i = idx(virt, level);
        uint64_t e = t[i];
        kprintf("  %-4s[%3u] = %016lx  %s%s%s%s\n", names[level], i, e, e & PTE_PRESENT ? "P" : "-",
                e & PTE_WRITABLE ? "W" : "-", e & PTE_USER ? "U" : "-", e & PTE_HUGE ? " huge" : "");
        if (!(e & PTE_PRESENT)) { kprintf("  -> not mapped\n"); return; }
        if (level == 0 || (e & PTE_HUGE)) break;
        t = (uint64_t *)(e & PTE_ADDR);
    }
    uint64_t phys;
    vmm_translate(virt, &phys);
    kprintf("  -> physical %p\n", (void *)phys);
}

/* Demand paging: a fault in the lazy zone gets a fresh zeroed frame, and the faulting instruction is retried. */
bool vmm_handle_fault(struct trap_frame *f) {
    uint64_t addr = read_cr2();
    if (addr < LAZY_START || addr >= LAZY_END || (f->error & 1)) return false;   /* error bit 0: page was present */
    uint64_t frame = pmm_alloc();
    if (!frame) return false;
    memset((void *)frame, 0, PAGE_SIZE);
    vmm_map(ALIGN_DOWN(addr, PAGE_SIZE), frame, PTE_WRITABLE);
    lazy_faults++;
    return true;
}

void vmm_init(void) {
    uint64_t cr3;
    __asm__ volatile("mov %%cr3, %0" : "=r"(cr3));
    KASSERT((cr3 & PTE_ADDR) == (uint64_t)boot_pml4);
}
```

`idt.c` gains one line: before treating a page fault as fatal, it asks `vmm_handle_fault`. If that returns true, the handler returns, `iretq` restarts the faulting instruction (it is a *fault* — RIP still points at it), and this time the access succeeds.

```c
// file: new/kernel/idt.c
#include <mm.h>

struct idt_gate {
    uint16_t offset_low;
    uint16_t selector;
    uint8_t  ist;
    uint8_t  type;
    uint16_t offset_mid;
    uint32_t offset_high;
    uint32_t reserved;
} __attribute__((packed));
_Static_assert(sizeof(struct idt_gate) == 16, "a long-mode gate is 16 bytes");

static struct idt_gate idt[256];
extern const uint64_t isr_table[48];
static irq_handler_t irq_handlers[16];

static const char *const exception_names[32] = {
    "divide error", "debug", "NMI", "breakpoint", "overflow", "bound range", "invalid opcode", "device not available",
    "double fault", "coprocessor overrun", "invalid TSS", "segment not present", "stack fault", "general protection",
    "page fault", "reserved", "x87 FP error", "alignment check", "machine check", "SIMD FP error", "virtualization",
    "control protection", "reserved", "reserved", "reserved", "reserved", "reserved", "reserved", "hypervisor injection",
    "VMM communication", "security", "reserved"};

void idt_init(void) {
    for (int v = 0; v < 48; v++) {
        uint64_t a = isr_table[v];
        idt[v] = (struct idt_gate){ (uint16_t)a, 0x08, 0, 0x8E, (uint16_t)(a >> 16), (uint32_t)(a >> 32), 0 };
    }
    struct __attribute__((packed)) { uint16_t limit; uint64_t base; } idtr = { sizeof idt - 1, (uint64_t)idt };
    __asm__ volatile("lidt %0" : : "m"(idtr));
}

void irq_register(int irq, irq_handler_t handler) { irq_handlers[irq] = handler; }

static void dump_frame(const struct trap_frame *f) {
    kprintf("  rip=%p  cs=0x%lx  rflags=0x%lx  rsp=%p\n", (void *)f->rip, f->cs, f->rflags, (void *)f->rsp);
    kprintf("  rax=%016lx rbx=%016lx rcx=%016lx rdx=%016lx\n", f->rax, f->rbx, f->rcx, f->rdx);
    kprintf("  rsi=%016lx rdi=%016lx rbp=%016lx\n", f->rsi, f->rdi, f->rbp);
}

static void exception(struct trap_frame *f) {
    if (f->vector == 3) {
        kprintf("breakpoint at rip=%p\n", (void *)f->rip);
        return;
    }
    if (f->vector == 14 && vmm_handle_fault(f)) return;          /* demand paging: retry the instruction */
    kprintf("\nEXCEPTION %lu: %s, error code 0x%lx\n", f->vector, exception_names[f->vector], f->error);
    if (f->vector == 14) {
        kprintf("  faulting address (cr2) = %p: %s, %s, %s mode\n", (void *)read_cr2(),
                f->error & 1 ? "protection violation" : "page not present",
                f->error & 2 ? "write" : "read", f->error & 4 ? "user" : "kernel");
    }
    dump_frame(f);
    panic("unhandled exception %lu at rip %p", f->vector, (void *)f->rip);
}

void interrupt_dispatch(struct trap_frame *f) {
    if (f->vector < 32) { exception(f); return; }
    int irq = (int)f->vector - 32;
    if (irq_handlers[irq]) irq_handlers[irq](f);
    pic_eoi(irq);
}
```

```c
// file: new/kernel/test_paging.c
#include <mm.h>

KTEST(paging) {
    vmm_walk((uint64_t)&pmm_alloc);                           /* kernel code: inside the 2 MiB boot pages */

    uint64_t frame = pmm_alloc();
    uint64_t a = 0xFFFF800000003000UL, b = 0xFFFFA00000000000UL;
    KASSERT(vmm_map(a, frame, PTE_WRITABLE) && vmm_map(b, frame, PTE_WRITABLE));
    vmm_walk(a);
    char *pa = (char *)a, *pb = (char *)b;
    memcpy(pa, "written through a", 18);
    kprintf("read through b: \"%s\" (two virtual pages, one frame)\n", pb);
    KASSERT(vmm_unmap(a) == frame && vmm_unmap(b) == frame);
    uint64_t phys;
    kprintf("after unmap: translate(a) %s\n", vmm_translate(a, &phys) ? "succeeds" : "fails");
    pmm_free(frame);
}

KTEST(lazy) {
    uint64_t free_before = pmm_free_count();
    volatile uint64_t *zone = (volatile uint64_t *)LAZY_START;
    kprintf("lazy zone at %p: first read returns %lu\n", (void *)zone, zone[0]);
    zone[1] = 42;                                              /* same page: no new fault */
    zone[512 * 5] = 7;                                         /* 5 pages further: one more fault */
    for (int page = 100; page < 110; page++) zone[512 * page] = (uint64_t)page;
    kprintf("page faults handled: %lu; frames used: %lu (data pages + new page tables)\n", lazy_faults,
            free_before - pmm_free_count());
    KASSERT(lazy_faults == 12 && zone[1] == 42 && zone[512 * 105] == 105);
}
```

```console
$ cd os && make -s && make -s run CMDLINE="test=paging"
hobby os: 127 MiB usable, 32457 of 32736 frames free, command line "test=paging"
== test paging ==
walk of 0x0000000000102300:
  PML4[  0] = 0000000000109023  PW-
  PDPT[  0] = 000000000010a023  PW-
  PD  [  0] = 00000000000000e3  PW- huge
  -> physical 0x0000000000102300
walk of 0xffff800000003000:
  PML4[256] = 0000000000116003  PW-
  PDPT[  0] = 0000000000117003  PW-
  PD  [  0] = 0000000000118003  PW-
  PT  [  3] = 0000000000115003  PW-
  -> physical 0x0000000000115000
read through b: "written through a" (two virtual pages, one frame)
after unmap: translate(a) fails
== test paging passed ==
[qemu exit status 33]
$ cd os && make -s run CMDLINE="test=lazy"
hobby os: 127 MiB usable, 32457 of 32736 frames free, command line "test=lazy"
== test lazy ==
lazy zone at 0xffffc00000000000: first read returns 0
page faults handled: 12; frames used: 15 (data pages + new page tables)
== test lazy passed ==
[qemu exit status 33]
```

Reading the walk of `pmm_alloc`'s address: PML4 index 0 → PDPT index 0 → a PD entry marked **huge**, so the walk stops one level early and the physical address equals the virtual one (identity map). The new mapping at `0xFFFF800000003000` needed three new tables (PDPT, PD, PT) — its PML4 entry is 256, the first index of the higher half. Twelve lazy faults: one for page 0 (the read — reading an unmapped lazy page maps a zero page), one for page 5, and ten for pages 100–109; frames used is higher than twelve because the first fault also had to create the PDPT, PD and PT for the zone.

### Deep & Tricky

**DT-9. The CPU caches translations in the TLB (translation lookaside buffer) and does not watch the page tables for changes. After changing or removing a present entry, the kernel must invalidate that translation (`invlpg`, or reload CR3 for everything non-global) — otherwise the CPU may keep using the old frame.** *(Intel SDM Vol. 3A, §4.10.4 "Invalidation of TLBs and Paging-Structure Caches": "the processor does not maintain coherence between the TLBs and the page tables… software should invalidate TLB entries after modifying a paging-structure entry")* Making a not-present entry present is the one change that needs no flush, because non-present entries are not cached. On multi-CPU systems every other CPU's TLB must be flushed too — a **TLB shootdown**, done with an inter-processor interrupt; it is one of the more expensive operations in Linux's memory management.

```c
// file: new/kernel/test_tlb.c
#include <mm.h>

KTEST(tlb) {
    uint64_t fa = pmm_alloc(), fb = pmm_alloc();
    *(volatile uint64_t *)fa = 0xAAAA;                         /* write the frames through the identity map */
    *(volatile uint64_t *)fb = 0xBBBB;
    uint64_t v = 0xFFFFB00000000000UL;
    vmm_map(v, fa, PTE_WRITABLE);
    kprintf("v -> frame A: reads 0x%lx\n", *(volatile uint64_t *)v);       /* the translation is now cached */
    uint64_t *pte = vmm_pte(v, false);
    *pte = fb | PTE_PRESENT | PTE_WRITABLE;                                /* retarget v to frame B ... */
    kprintf("PTE now says frame B, no flush: reads 0x%lx\n", *(volatile uint64_t *)v);
    invlpg(v);                                                             /* ... but only now does the CPU notice */
    kprintf("after invlpg: reads 0x%lx\n", *(volatile uint64_t *)v);
}
```

```console
$ cd os && make -s && make -s run CMDLINE="test=tlb"
hobby os: 127 MiB usable, 32457 of 32736 frames free, command line "test=tlb"
== test tlb ==
v -> frame A: reads 0xaaaa
PTE now says frame B, no flush: reads 0xaaaa
after invlpg: reads 0xbbbb
== test tlb passed ==
[qemu exit status 33]
```

On QEMU (and on real hardware) the second read still returns frame A's value: the page table says one thing, and the CPU does another. Nothing guarantees the stale entry survives — an interrupt or another access could evict it — which is exactly what makes a missing flush a bug that appears only sometimes.

## Topic 4C — The kernel heap

## Unit 4.3 — `kmalloc` and `kfree`

**One-line summary:** The heap is a virtual region at `HEAP_START` that grows by mapping fresh frames at its end (`morecore`, like `sbrk` in user space). Inside it, each allocation has a **header** (magic number, size, free flag, links to its neighbours). `kmalloc` finds the first free block that is large enough (**first fit**) and **splits** it if much is left over; `kfree` marks the block free and **coalesces** it with free neighbours so that the heap does not crumble into tiny pieces. The magic number lets `kfree` and `kheap_check` detect a corrupted header or a double free.

```c
// file: new/kernel/heap.c
/* A first-fit heap with boundary checks. Portable C: also compiled for the host test in this chapter's project. */
#include <mm.h>

#define MAGIC 0x6B6D616Cu                        /* "kmal" */
#define GROW  (64 * 1024)                        /* extend the heap at least this much at a time */

struct block {
    uint32_t magic;
    uint32_t free;
    size_t size;                                 /* payload bytes, multiple of 16 */
    struct block *prev, *next;                   /* neighbours in address order */
};
_Static_assert(sizeof(struct block) == 32, "header keeps payloads 16-byte aligned");

static struct block *head, *tail;

static void *payload(struct block *b) { return b + 1; }
static char *end_of(struct block *b) { return (char *)payload(b) + b->size; }

static void split(struct block *b, size_t n) {   /* keep n bytes in b; the rest becomes a new free block */
    if (b->size < n + sizeof(struct block) + 16) return;   /* remainder too small to be useful */
    struct block *r = (struct block *)((char *)payload(b) + n);
    *r = (struct block){MAGIC, 1, b->size - n - sizeof(struct block), b, b->next};
    if (b->next) b->next->prev = r; else tail = r;
    b->next = r;
    b->size = n;
}

static void merge_with_next(struct block *b) {    /* b and b->next are both free and adjacent */
    struct block *n = b->next;
    b->size += sizeof(struct block) + n->size;
    b->next = n->next;
    if (n->next) n->next->prev = b; else tail = b;
    n->magic = 0;                                 /* a stale pointer to n must not look valid */
}

static struct block *grow(size_t n) {
    size_t bytes = ALIGN_UP(n + sizeof(struct block), GROW);
    char *p = morecore(bytes);
    if (!p) return NULL;
    if (tail && tail->free && end_of(tail) == p) { tail->size += bytes; return tail; }
    struct block *b = (struct block *)p;
    *b = (struct block){MAGIC, 1, bytes - sizeof(struct block), tail, NULL};
    if (tail) tail->next = b; else head = b;
    tail = b;
    return b;
}

void *kmalloc(size_t n) {
    if (n == 0 || n > (1UL << 40)) return NULL;
    n = ALIGN_UP(n, 16);
    struct block *b;
    for (b = head; b; b = b->next)
        if (b->free && b->size >= n) break;
    if (!b && !(b = grow(n))) return NULL;
    split(b, n);
    b->free = 0;
    return payload(b);
}

void kfree(void *p) {
    if (!p) return;
    struct block *b = (struct block *)p - 1;
    if (b->magic != MAGIC) panic("kfree(%p): bad header magic 0x%x (corruption or not from kmalloc)", p, b->magic);
    if (b->free) panic("kfree(%p): double free", p);
    b->free = 1;
    if (b->next && b->next->free) merge_with_next(b);
    if (b->prev && b->prev->free) merge_with_next(b->prev);
}

void kheap_check(struct heap_stats *s) {
    *s = (struct heap_stats){0};
    struct block *prev = NULL;
    for (struct block *b = head; b; prev = b, b = b->next) {
        if (b->magic != MAGIC) panic("heap: block %p has bad magic 0x%x", (void *)b, b->magic);
        if (b->prev != prev) panic("heap: block %p has a broken prev link", (void *)b);
        if (b->size % 16) panic("heap: block %p has unaligned size %zu", (void *)b, b->size);
        if (prev && prev->free && b->free) panic("heap: adjacent free blocks %p and %p", (void *)prev, (void *)b);
        s->blocks++;
        s->heap_bytes += sizeof(struct block) + b->size;
        if (b->free) {
            s->free_blocks++;
            s->free_bytes += b->size;
            if (b->size > s->largest_free) s->largest_free = b->size;
        } else {
            s->used_bytes += b->size;
        }
    }
    if (prev != tail) panic("heap: tail pointer is wrong");
}
```

```c
// file: new/kernel/morecore.c
/* The kernel's heap growth: map fresh frames at the end of the heap region. */
#include <mm.h>

static uint64_t heap_end = HEAP_START;

void *morecore(size_t bytes) {
    uint64_t old = heap_end;
    for (uint64_t v = heap_end; v < heap_end + bytes; v += PAGE_SIZE) {
        uint64_t frame = pmm_alloc();
        if (!frame || !vmm_map(v, frame, PTE_WRITABLE)) return NULL;   /* (a real kernel would roll back) */
    }
    heap_end += bytes;
    return (void *)old;
}
```

```c
// file: new/kernel/test_heap.c
#include <mm.h>

static uint32_t rng = 12345;
static uint32_t rnd(void) { rng = rng * 1103515245 + 12345; return rng >> 8; }

KTEST(heap) {
    struct heap_stats s;
    char *a = kmalloc(100), *b = kmalloc(200), *c = kmalloc(300);
    kprintf("kmalloc(100)=%p kmalloc(200)=%p kmalloc(300)=%p\n", (void *)a, (void *)b, (void *)c);
    kprintf("gaps: %ld and %ld bytes (16-byte rounding + 32-byte header)\n", (long)(b - a), (long)(c - b));
    kfree(b);
    char *d = kmalloc(150);
    kprintf("after kfree(b), kmalloc(150)=%p: %s\n", (void *)d, d == b ? "reuses b's block (first fit)" : "new block");
    kfree(a); kfree(c); kfree(d);
    kheap_check(&s);
    kprintf("all freed: %zu block(s), %zu free bytes in one piece\n", s.blocks, s.largest_free);

    static char *ptr[500];
    static size_t len[500];
    uint64_t frames_before = pmm_free_count();
    for (int op = 0; op < 20000; op++) {
        int i = (int)(rnd() % 500);
        if (ptr[i]) {
            for (size_t k = 0; k < len[i]; k++) KASSERT(ptr[i][k] == (char)i);   /* contents intact? */
            kfree(ptr[i]);
            ptr[i] = NULL;
        } else {
            len[i] = 1 + rnd() % (rnd() % 8 == 0 ? 16384 : 256);
            ptr[i] = kmalloc(len[i]);
            KASSERT(ptr[i] && (uintptr_t)ptr[i] % 16 == 0);
            memset(ptr[i], i, len[i]);
        }
        if (op % 5000 == 0) kheap_check(&s);
    }
    kheap_check(&s);
    kprintf("after 20000 random operations: %zu blocks (%zu free), used %zu, free %zu, largest free %zu, heap %zu KiB\n",
            s.blocks, s.free_blocks, s.used_bytes, s.free_bytes, s.largest_free, s.heap_bytes >> 10);
    for (int i = 0; i < 500; i++) kfree(ptr[i]);
    kheap_check(&s);
    kprintf("all freed again: %zu block(s); heap took %lu frames from the frame allocator\n", s.blocks,
            frames_before - pmm_free_count());
    KASSERT(s.blocks == 1 && s.free_blocks == 1);
}
```

```console
$ cd os && make -s && make -s run CMDLINE="test=heap"
hobby os: 127 MiB usable, 32457 of 32736 frames free, command line "test=heap"
== test heap ==
kmalloc(100)=0xffff800000000020 kmalloc(200)=0xffff8000000000b0 kmalloc(300)=0xffff8000000001a0
gaps: 144 and 240 bytes (16-byte rounding + 32-byte header)
after kfree(b), kmalloc(150)=0xffff8000000000b0: reuses b's block (first fit)
all freed: 1 block(s), 65504 free bytes in one piece
after 20000 random operations: 306 blocks (60 free), used 291296, free 223200, largest free 123632, heap 512 KiB
all freed again: 1 block(s); heap took 112 frames from the frame allocator
== test heap passed ==
[qemu exit status 33]
```

The gaps show the cost of the design: a 100-byte request occupies 112 bytes plus a 32-byte header, so `b` is 144 bytes after `a`. After freeing everything, coalescing restores a single free block — the strongest check that splitting and merging agree. The heap never gives memory back to the frame allocator; Linux's slab allocators do (Chapter 11).

### Pattern

**P-5. Build each memory layer on the one below with a narrow interface — frames (`pmm_alloc`/`pmm_free`), pages (`vmm_map`/`vmm_unmap`), bytes (`kmalloc`/`kfree`) — and give each layer a self-check that can run at any time (`kheap_check`, the frame allocator's double-free panic).** *(Practice)* The narrow interface is what lets `heap.c` run unchanged on the host in the project below, where sanitizers and fast iteration are available; the self-checks turn silent corruption into an immediate panic near its cause. Linux has the same layers — memblock, the buddy allocator, `vmalloc`, and the slab allocators — and the same idea in `CONFIG_DEBUG_VM`, `CONFIG_SLUB_DEBUG`, and KASAN (Chapter 12). *Trade-off:* every check costs time on every operation; production kernels compile most of them out and turn them on only for debugging.

### Mastery check

1. **Predict.** Which four indices does the virtual address `0x00007FFFFFFFF000` (the top page of the lower half) produce?
2. **Predict.** Why does `vmm_handle_fault` refuse faults whose error code has bit 0 set?
3. **Predict.** After `kfree(b)` in the heap test, a `kmalloc(150)` got `b`'s block back. What would `kmalloc(250)` have returned?

## Chapter 4 Project — Fuzz the heap on the host, and inspect memory from the shell

**Goal:** (1) compile the unchanged `heap.c` into a **host** program with AddressSanitizer and UndefinedBehaviorSanitizer, drive it with a long random sequence checked against a simple model, and show that `kheap_check` catches a buffer overflow that corrupts a neighbouring header; (2) make shell commands self-registering (the same linker-section idea as `KTEST`) and add memory commands: `pt <addr>` walks the page tables, `frames` and `heap` print allocator statistics, and `touch <n>` uses demand paging.

```c
// file: proj/host/heap_fuzz.c
/* Host harness for kernel/heap.c: same allocator code, a fake morecore, sanitizers on. */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "mm.h"

static _Alignas(4096) char arena[64 << 20];
static size_t arena_used;

void *morecore(size_t bytes) {                     /* like the kernel's: contiguous growth, NULL when full */
    if (arena_used + bytes > sizeof arena) return NULL;
    void *p = arena + arena_used;
    arena_used += bytes;
    return p;
}

_Noreturn void panic(const char *fmt, ...) {
    va_list ap; va_start(ap, fmt);
    fputs("PANIC: ", stdout); vprintf(fmt, ap); putchar('\n');
    va_end(ap);
    exit(2);
}

int main(int argc, char **argv) {
    enum { SLOTS = 4096 };
    static char *ptr[SLOTS];
    static size_t len[SLOTS];
    struct heap_stats s;
    srand(7);
    long ops = 2000000, mallocs = 0, checks = 0;
    if (argc > 1 && strcmp(argv[1], "overflow") == 0) {
        char *a = kmalloc(40), *b = kmalloc(40);
        printf("a=%p b=%p; writing 60 bytes into a's 48-byte block...\n", (void *)a, (void *)b);
        memset(a, 'x', 60);                         /* overruns into b's header */
        kheap_check(&s);
        puts("not detected");
        return 0;
    }
    for (long op = 0; op < ops; op++) {
        int i = rand() % SLOTS;
        if (ptr[i]) {
            for (size_t k = 0; k < len[i]; k++)
                if (ptr[i][k] != (char)(i ^ len[i])) panic("slot %d corrupted at byte %zu", i, k);
            kfree(ptr[i]);
            ptr[i] = NULL;
        } else {
            int r = rand() % 100;
            len[i] = r < 70 ? 1 + rand() % 64 : r < 97 ? 1 + rand() % 2048 : 1 + rand() % 200000;
            ptr[i] = kmalloc(len[i]);
            if (!ptr[i]) panic("out of memory at op %ld", op);
            if ((uintptr_t)ptr[i] % 16) panic("misaligned pointer %p", (void *)ptr[i]);
            memset(ptr[i], i ^ (int)len[i], len[i]);
            mallocs++;
        }
        if (op % 100000 == 0) { kheap_check(&s); checks++; }
    }
    kheap_check(&s);
    printf("%ld operations (%ld kmalloc), %ld full heap checks: no corruption\n", ops, mallocs, checks + 1);
    printf("live: %zu used bytes in %zu blocks; free: %zu bytes in %zu blocks (largest %zu); arena %zu KiB\n",
           s.used_bytes, s.blocks - s.free_blocks, s.free_bytes, s.free_blocks, s.largest_free, arena_used >> 10);
    printf("utilisation: %.1f%% of the arena holds live data\n", 100.0 * s.used_bytes / arena_used);
    for (int i = 0; i < SLOTS; i++) kfree(ptr[i]);
    kheap_check(&s);
    printf("after freeing everything: %zu block(s)\n", s.blocks);
    return 0;
}
```

```c
// file: proj/include/shell.h
#pragma once
#include <kernel.h>

struct shell_cmd { const char *name, *help; void (*fn)(const char *arg); } __attribute__((aligned(32)));
#define SHELL_CMD(n, helptext)                                                                    \
    static void cmd_##n(const char *arg);                                                         \
    __attribute__((used, section(".shellcmds"))) static const struct shell_cmd shellcmd_##n = {   \
        #n, helptext, cmd_##n};                                                                   \
    static void cmd_##n(const char *arg)
uint64_t parse_hex(const char *s);
```

```c
// file: proj/kernel/shell.c
#include <shell.h>
#include <mm.h>

extern const struct shell_cmd __shellcmds_start[], __shellcmds_end[];

uint64_t parse_hex(const char *s) {
    uint64_t v = 0;
    if (s[0] == '0' && s[1] == 'x') s += 2;
    for (; *s && *s != ' '; s++) {
        int d = *s >= '0' && *s <= '9' ? *s - '0' : *s >= 'a' && *s <= 'f' ? *s - 'a' + 10 : -1;
        if (d < 0) break;
        v = v * 16 + (uint64_t)d;
    }
    return v;
}

static size_t read_line(char *buf, size_t size) {
    size_t n = 0;
    kprintf("kshell> ");
    for (;;) {
        int c = serial_getc();
        if (c == '\r' || c == '\n') { kprintf("\n"); break; }
        if ((c == 0x7F || c == '\b') && n > 0) { n--; kprintf("\b \b"); continue; }
        if (c == 0x15) { while (n) { n--; kprintf("\b \b"); } continue; }
        if (c >= 0x20 && c < 0x7F && n + 1 < size) { buf[n++] = (char)c; kprintf("%c", c); }
    }
    buf[n] = '\0';
    return n;
}

SHELL_CMD(help, "list commands") {
    (void)arg;
    for (const struct shell_cmd *c = __shellcmds_start; c < __shellcmds_end; c++) kprintf("  %-8s %s\n", c->name, c->help);
}
SHELL_CMD(uptime, "time since the timer started") {
    (void)arg;
    uint64_t t = ticks;
    kprintf("up %lu.%02lu s (%lu ticks at %u Hz)\n", t / timer_hz, t % timer_hz * 100 / timer_hz, t, timer_hz);
}
SHELL_CMD(echo, "print the arguments") { kprintf("%s\n", arg); }
SHELL_CMD(crash, "write to an address: crash [hexaddr]") {
    uint64_t addr = parse_hex(arg);
    if (!addr) addr = 0xdead0000;
    kprintf("writing to 0x%lx...\n", addr);
    *(volatile uint64_t *)addr = 1;
}
SHELL_CMD(halt, "power off (exit QEMU)") { (void)arg; kprintf("halting\n"); qemu_exit(0x10); }

void shell_run(void) {
    kprintf("kshell ready; type 'help'\n");
    char line[128];
    for (;;) {
        if (!read_line(line, sizeof line)) continue;
        char *arg = line;
        while (*arg && *arg != ' ') arg++;
        if (*arg) *arg++ = '\0';
        const struct shell_cmd *c;
        for (c = __shellcmds_start; c < __shellcmds_end; c++)
            if (strncmp(line, c->name, 16) == 0) { c->fn(arg); break; }
        if (c == __shellcmds_end) kprintf("unknown command '%s'\n", line);
    }
}
```

```c
// file: proj/kernel/cmd_mm.c
#include <shell.h>
#include <mm.h>

SHELL_CMD(pt, "walk the page tables: pt <hexaddr>") { vmm_walk(parse_hex(arg)); }

SHELL_CMD(frames, "physical frame allocator statistics") {
    (void)arg;
    uint64_t f = pmm_free_count(), t = pmm_total_count();
    kprintf("frames: %lu free of %lu (%lu MiB free)\n", f, t, f * PAGE_SIZE >> 20);
}

SHELL_CMD(heap, "kernel heap statistics: heap [n] allocates n bytes first") {
    uint64_t n = 0;
    for (const char *p = arg; *p >= '0' && *p <= '9'; p++) n = n * 10 + (uint64_t)(*p - '0');
    if (n) kprintf("kmalloc(%lu) = %p\n", n, kmalloc(n));
    struct heap_stats s;
    kheap_check(&s);
    kprintf("heap: %zu KiB mapped, %zu blocks, %zu used bytes, %zu free bytes (largest %zu)\n", s.heap_bytes >> 10,
            s.blocks, s.used_bytes, s.free_bytes, s.largest_free);
}

SHELL_CMD(touch, "touch n pages of the lazy zone: touch <n>") {
    uint64_t n = 0;
    for (const char *p = arg; *p >= '0' && *p <= '9'; p++) n = n * 10 + (uint64_t)(*p - '0');
    uint64_t before = lazy_faults;
    for (uint64_t i = 0; i < n; i++) ((volatile char *)LAZY_START)[i * PAGE_SIZE] = 1;
    kprintf("touched %lu pages: %lu new page faults\n", n, lazy_faults - before);
}
```

The linker script collects the new section the same way as `.ktests`:

```text
# file: proj/linker.ld
ENTRY(_start)
SECTIONS {
    . = 1M;
    _kernel_start = .;
    .boot   : { KEEP(*(.multiboot2)) }
    .text   : ALIGN(4K) { *(.text*) }
    .rodata : ALIGN(4K) {
        *(.rodata*)
        . = ALIGN(16);
        __ktests_start = .;
        KEEP(*(.ktests))
        __ktests_end = .;
        . = ALIGN(32);
        __shellcmds_start = .;
        KEEP(*(.shellcmds))
        __shellcmds_end = .;
    }
    .data   : ALIGN(4K) { *(.data*) }
    .bss    : ALIGN(4K) { *(COMMON) *(.bss*) }
    _kernel_end = .;
    /DISCARD/ : { *(.comment) *(.note*) *(.eh_frame*) }
}
```

```console
$ rm -rf final && cp -r os final && cp -r proj/. final/ && make -s -C final clean && make -s -C final && echo "final = os + proj"
final = os + proj
$ cd final && gcc -std=c11 -O1 -g -fsanitize=address,undefined -fno-omit-frame-pointer -Iinclude host/heap_fuzz.c kernel/heap.c kernel/string.c -o heap_fuzz 2>&1 | head; time ./heap_fuzz
2000000 operations (1001045 kmalloc), 21 full heap checks: no corruption
live: 6351792 used bytes in 2090 blocks; free: 5369168 bytes in 270 blocks (largest 3225248); arena 11520 KiB
utilisation: 53.8% of the arena holds live data
after freeing everything: 1 block(s)

real	0m16.369s
user	0m16.159s
sys	0m0.010s
$ cd final && ./heap_fuzz overflow; echo "exit status $?"
a=0x55eaa71a2020 b=0x55eaa71a2070; writing 60 bytes into a's 48-byte block...
PANIC: heap: block 0x55eaa71a2050 has bad magic 0x78787878
exit status 2
$ cd final && (sleep 4; for l in 'help' 'pt 0x1000' 'pt ffff800000000000' 'frames' 'heap 5000' 'touch 20' 'touch 20' 'frames' 'pt ffffc00000003000' 'halt'; do printf "$l\n"; sleep 0.3; done) | make -s run
hobby os: 127 MiB usable, 32456 of 32736 frames free, command line ""
kshell ready; type 'help'
kshell> help
  touch    touch n pages of the lazy zone: touch <n>
  heap     kernel heap statistics: heap [n] allocates n bytes first
  frames   physical frame allocator statistics
  pt       walk the page tables: pt <hexaddr>
  halt     power off (exit QEMU)
  crash    write to an address: crash [hexaddr]
  echo     print the arguments
  uptime   time since the timer started
  help     list commands
kshell> pt 0x1000
walk of 0x0000000000001000:
  PML4[  0] = 0000000000109023  PW-
  PDPT[  0] = 000000000010a023  PW-
  PD  [  0] = 00000000000000e3  PW- huge
  -> physical 0x0000000000001000
kshell> pt ffff800000000000
walk of 0xffff800000000000:
  PML4[256] = 0000000000000000  ---
  -> not mapped
kshell> frames
frames: 32456 free of 32736 (126 MiB free)
kshell> heap 5000
kmalloc(5000) = 0xffff800000000020
heap: 64 KiB mapped, 2 blocks, 5008 used bytes, 60464 free bytes (largest 60464)
kshell> touch 20
touched 20 pages: 20 new page faults
kshell> touch 20
touched 20 pages: 0 new page faults
kshell> frames
frames: 32414 free of 32736 (126 MiB free)
kshell> pt ffffc00000003000
walk of 0xffffc00000003000:
  PML4[384] = 000000000012c023  PW-
  PDPT[  0] = 000000000012d023  PW-
  PD  [  0] = 000000000012e023  PW-
  PT  [  3] = 0000000000131063  PW-
  -> physical 0x0000000000131000
kshell> halt
halting
[qemu exit status 33]
```

The fuzzer ran two million operations through the very code the kernel uses, with ASan and UBSan watching every access to the harness's own memory. The overflow test shows what the header magic is for: a 60-byte write into a 48-byte block destroys the next block's header, and the next `kheap_check` panics and names the damaged block. (ASan does not complain, because from its point of view the whole arena is one valid array — a heap inside a heap hides overflows from it. Linux has the same problem and solves it by teaching KASAN about the slab allocator's object boundaries.)

In the shell, `pt 0x1000` walks into the huge 2 MiB identity page; the heap address is mapped with 4 KiB pages; the second `touch 20` causes no faults because the pages are already mapped.

### Review checklist

- What happens to the heap and the frame allocator if `morecore` fails in the middle of a multi-page growth?
- `kmalloc` scans every block from the start. How long does an allocation take with 100,000 blocks in the heap, and what data structure would fix it?
- Which shell command would be unsafe to run from an interrupt handler, and why?

## Answers to the Chapter 4 prediction questions

**4.3 Q1.** `0x00007FFFFFFFF000` → PML4 index 255, PDPT index 511, PD index 511, PT index 511, offset 0. (Bit 47 is 0, so it is the last canonical page of the lower half; the next page, `0x0000800000000000`, is non-canonical.)

**4.3 Q2.** Error-code bit 0 set means the page *was* present and the access broke a permission rule (for example, a write to a read-only page). Mapping a fresh frame there would call `vmm_map` on an already-mapped address — and, worse, would silently "fix" a real protection error.

**4.3 Q3.** The freed block of `b` holds 208 bytes, too small for 256 (250 rounded up), and `a` and `c` are in use, so first fit would pass it and use memory after `c` — the free tail of the heap.

# Chapter 5 — Threads, Processes, and System Calls

*Goal: make the kernel run many things "at once". First, kernel **threads** with their own stacks and a **context switch** between them. Then **preemption**: the timer interrupt takes the CPU away from a thread that does not give it up. Then **user mode**: programs running in ring 3, each in its own address space, unable to touch the kernel or each other, asking the kernel for services through **system calls**. When a user program misbehaves, the kernel kills it and carries on.*

**What you will be able to do after this chapter**

- Write a context switch in a dozen instructions and explain what it saves and why that is enough.
- Build a round-robin scheduler with sleeping, blocking on wait queues, and joining.
- Preempt from the timer interrupt safely — and know the two traps (DT-10).
- Set up a GDT with user segments and a TSS, enter ring 3 with `iretq`, and come back on interrupts and system calls.
- Give each process its own page tables, and validate every pointer a process hands to the kernel.

The kernel continues from Chapter 4's `final/`. New and changed files are in `new/`; `os/` is Chapter 4's kernel plus `new/`.

```console
$ rm -rf os && cp -r ../ch04/final os && cp -r new/. os/ && make -s -C os clean && echo ready
ready
```

## Topic 5A — Kernel threads

## Unit 5.1 — The context switch

**One-line summary:** A thread is a stack plus a saved stack pointer. To switch from thread A to thread B, `switch_to` pushes the registers that the C calling convention says a called function must preserve (RBX, RBP, R12–R15), saves RSP into A's task structure, loads B's saved RSP, pops B's registers, and returns — into B, at the point where B itself once called `switch_to`. Everything else (the other registers, RIP, the C variables) is already on each thread's own stack. A *new* thread is given a stack that looks as if it had called `switch_to` from the start of a small **trampoline**, which calls the thread's function.

```nasm
; file: new/arch/switch.asm
bits 64
section .text
extern sched_finish_switch
extern task_exit

; void switch_to(uint64_t *save_rsp, uint64_t load_rsp)
global switch_to
switch_to:
    push rbp                      ; callee-saved registers: the caller expects them unchanged
    push rbx
    push r12
    push r13
    push r14
    push r15
    mov [rdi], rsp                ; save the old thread's stack pointer
    mov rsp, rsi                  ; ... and continue on the new thread's stack
    pop r15
    pop r14
    pop r13
    pop r12
    pop rbx
    pop rbp
    ret                           ; returns wherever the new thread called switch_to (or to the trampoline)

; A new thread's first "return" lands here, with r12 = function and r13 = argument.
global thread_trampoline
thread_trampoline:
    call sched_finish_switch
    sti                           ; we came from schedule(), which runs with interrupts off
    mov rdi, r13
    call r12
    xor edi, edi
    call task_exit                ; the function returned: the thread exits with status 0

; void enter_user(uint64_t rip, uint64_t rsp): build an interrupt frame for ring 3 and "return" to it.
global enter_user
enter_user:
    mov ax, 0x1B                  ; user data selector (0x18) with RPL 3
    mov ds, ax
    mov es, ax
    push 0x1B                     ; ss
    push rsi                      ; rsp
    push 0x202                    ; rflags: IF = 1, so the timer can preempt user code
    push 0x23                     ; cs: user code selector (0x20) with RPL 3
    push rdi                      ; rip
    xor eax, eax                  ; leak no kernel values into user registers
    xor ebx, ebx
    xor ecx, ecx
    xor edx, edx
    xor esi, esi
    xor edi, edi
    xor ebp, ebp
    xor r8d, r8d
    xor r9d, r9d
    xor r10d, r10d
    xor r11d, r11d
    xor r12d, r12d
    xor r13d, r13d
    xor r14d, r14d
    xor r15d, r15d
    iretq
```

A new thread's kernel stack, prepared by `kthread_create` (higher addresses at the top):

```text
  top of stack (16-byte aligned)
  thread_trampoline   <- "return address" for switch_to's ret
  0                   <- rbp
  0                   <- rbx
  fn                  <- r12
  arg                 <- r13
  0                   <- r14
  0                   <- r15      <- task->rsp
```

```c
// file: new/include/proc.h
#pragma once
#include <mm.h>

enum task_state { TASK_RUNNABLE, TASK_RUNNING, TASK_SLEEPING, TASK_BLOCKED, TASK_ZOMBIE };
struct task;
struct waitq { struct task *head; };

struct task {
    uint64_t rsp;                 /* saved kernel stack pointer while not running */
    uint64_t *pml4;               /* address space */
    void *kstack;                 /* KSTACK_SIZE bytes */
    int pid;
    enum task_state state;
    char name[16];
    bool user, killed;
    int exit_code;
    uint64_t wake_tick, runtime_ticks, switches;
    struct waitq exit_wait;       /* tasks waiting for this one to exit */
    struct task *next;            /* run queue or wait queue link */
    struct task *all_next;        /* list of every task */
};
#define KSTACK_SIZE 16384

static inline uint64_t irq_save(void) {
    uint64_t f;
    __asm__ volatile("pushfq; pop %0; cli" : "=r"(f) : : "memory");
    return f;
}
static inline void irq_restore(uint64_t f) { if (f & 0x200) sti(); }

extern struct task *current;
extern volatile bool need_resched;
extern bool preempt_enabled;
void sched_init(void);
struct task *kthread_create(const char *name, void (*fn)(void *), void *arg);
struct task *task_create(const char *name, void (*fn)(void *), void *arg, uint64_t *pml4, bool user);
void schedule(void);                       /* interrupts must be off */
void yield(void);
void sleep_ticks(uint64_t n);
void wait_on(struct waitq *q);             /* interrupts must be off */
void wake_all(struct waitq *q);
_Noreturn void task_exit(int code);
int  task_join(struct task *t);            /* waits for t to exit; returns its exit code */
void sched_tick(void);                     /* called by the timer interrupt */
struct task *task_first(void);
struct task *task_find(int pid);
const char *task_state_name(enum task_state s);

/* user mode */
void gdt_init(void);
void tss_set_rsp0(uint64_t rsp0);
void syscall_dispatch(struct trap_frame *f);
struct task *process_create(const char *name, const void *image, size_t size);
struct user_prog { const char *name; const char *start, *end; };
extern const struct user_prog user_progs[];
const struct user_prog *user_prog_find(const char *name);
```

```c
// file: new/kernel/sched.c
#include <proc.h>

extern void switch_to(uint64_t *save_rsp, uint64_t load_rsp);
extern char thread_trampoline[];
extern char stack_top[];

#define SLICE_TICKS 5                         /* 50 ms at 100 Hz */

struct task *current;
volatile bool need_resched;
bool preempt_enabled = true;
static struct task *rq_head, *rq_tail, *idle, *all_tasks, *to_free;
static int next_pid;
static unsigned slice;

static void rq_push(struct task *t) {
    t->next = NULL;
    if (rq_tail) rq_tail->next = t; else rq_head = t;
    rq_tail = t;
    if (current == idle) need_resched = true;   /* the idle task should give way at once */
}

static struct task *rq_pop(void) {
    struct task *t = rq_head;
    if (t && !(rq_head = t->next)) rq_tail = NULL;
    return t;
}

static struct task *task_alloc(const char *name) {
    struct task *t = kmalloc(sizeof *t);
    KASSERT(t);
    memset(t, 0, sizeof *t);
    t->pid = next_pid++;
    for (int i = 0; i < 15 && name[i]; i++) t->name[i] = name[i];
    t->pml4 = kernel_pml4;
    uint64_t f = irq_save();
    t->all_next = all_tasks;
    all_tasks = t;
    irq_restore(f);
    return t;
}

static void prepare_stack(struct task *t, void (*fn)(void *), void *arg) {
    t->kstack = kmalloc(KSTACK_SIZE);
    KASSERT(t->kstack);
    uint64_t *sp = (uint64_t *)((char *)t->kstack + KSTACK_SIZE);
    *--sp = (uint64_t)thread_trampoline;
    *--sp = 0;                 /* rbp */
    *--sp = 0;                 /* rbx */
    *--sp = (uint64_t)fn;      /* r12 */
    *--sp = (uint64_t)arg;     /* r13 */
    *--sp = 0;                 /* r14 */
    *--sp = 0;                 /* r15 */
    t->rsp = (uint64_t)sp;
}

struct task *task_create(const char *name, void (*fn)(void *), void *arg, uint64_t *pml4, bool user) {
    struct task *t = task_alloc(name);
    prepare_stack(t, fn, arg);
    t->pml4 = pml4;
    t->user = user;
    uint64_t f = irq_save();
    t->state = TASK_RUNNABLE;
    rq_push(t);
    irq_restore(f);
    return t;
}

struct task *kthread_create(const char *name, void (*fn)(void *), void *arg) {
    return task_create(name, fn, arg, kernel_pml4, false);
}

static void idle_loop(void *arg) {
    (void)arg;
    for (;;) hlt();                           /* interrupts are on: any interrupt wakes us */
}

void sched_init(void) {
    current = task_alloc("kmain");           /* the boot thread becomes task 0 */
    current->kstack = (void *)((uintptr_t)stack_top - KSTACK_SIZE);   /* boot.asm reserved 16 KiB below stack_top */
    current->state = TASK_RUNNING;
    idle = task_alloc("idle");
    prepare_stack(idle, idle_loop, NULL);
    idle->state = TASK_RUNNABLE;             /* never in the run queue: chosen only when it is empty */
}

/* Runs in the context of whichever task is resuming. Frees the task that exited before us. */
void sched_finish_switch(void) {
    struct task *z = to_free;
    if (!z || z == current) return;
    to_free = NULL;
    kfree(z->kstack);
    z->kstack = NULL;
    if (z->pml4 != kernel_pml4) { vmm_destroy_space(z->pml4); z->pml4 = NULL; }
}

void schedule(void) {
    struct task *prev = current;
    if (prev->state == TASK_RUNNING) {
        prev->state = TASK_RUNNABLE;
        if (prev != idle) rq_push(prev);     /* the idle task is never queued */
    }
    struct task *next = rq_pop();
    if (!next) next = idle;
    need_resched = false;
    slice = 0;
    if (next == prev) { prev->state = TASK_RUNNING; return; }
    if (prev->state == TASK_ZOMBIE) to_free = prev;    /* cannot free the stack we are standing on */
    next->state = TASK_RUNNING;
    next->switches++;
    current = next;
    tss_set_rsp0((uint64_t)next->kstack + KSTACK_SIZE);  /* where the CPU puts the frame on an interrupt from ring 3 */
    if (next->pml4 != prev->pml4) __asm__ volatile("mov %0, %%cr3" : : "r"(next->pml4) : "memory");
    switch_to(&prev->rsp, next->rsp);
    sched_finish_switch();                   /* we are "next" now, possibly much later */
}

void yield(void) { uint64_t f = irq_save(); schedule(); irq_restore(f); }

void sleep_ticks(uint64_t n) {
    uint64_t f = irq_save();
    current->wake_tick = ticks + n;
    current->state = TASK_SLEEPING;
    schedule();
    irq_restore(f);
}

void wait_on(struct waitq *q) {
    current->state = TASK_BLOCKED;
    current->next = q->head;
    q->head = current;
    schedule();
}

void wake_all(struct waitq *q) {
    while (q->head) {
        struct task *t = q->head;
        q->head = t->next;
        if (t->state == TASK_BLOCKED) { t->state = TASK_RUNNABLE; rq_push(t); }
    }
}

_Noreturn void task_exit(int code) {
    cli();
    current->exit_code = code;
    current->state = TASK_ZOMBIE;
    wake_all(&current->exit_wait);
    schedule();
    panic("a zombie was scheduled");
}

int task_join(struct task *t) {
    uint64_t f = irq_save();
    while (t->state != TASK_ZOMBIE) wait_on(&t->exit_wait);
    irq_restore(f);
    return t->exit_code;
}

void sched_tick(void) {                       /* interrupt context */
    for (struct task *t = all_tasks; t; t = t->all_next)
        if (t->state == TASK_SLEEPING && t->wake_tick <= ticks) { t->state = TASK_RUNNABLE; rq_push(t); }
    current->runtime_ticks++;
    if (preempt_enabled && current != idle && ++slice >= SLICE_TICKS) need_resched = true;
}

struct task *task_first(void) { return all_tasks; }
struct task *task_find(int pid) {
    for (struct task *t = all_tasks; t; t = t->all_next) if (t->pid == pid) return t;
    return NULL;
}
const char *task_state_name(enum task_state s) {
    static const char *const n[] = {"runnable", "running", "sleeping", "blocked", "zombie"};
    return n[s];
}
```

`sched_finish_switch` exists because of a chicken-and-egg problem: an exiting task cannot free its own kernel stack while it is still running on it. So it only records itself, and whichever task runs next frees it. Linux has the same step under the same name idea (`finish_task_switch`).

Everything the scheduler touches is also touched by the timer interrupt, so every function above runs with interrupts disabled while it changes the run queue — on one CPU, that is the whole of the locking. The kernel heap is shared too: one thread may be preempted in the middle of `kmalloc`, and another may then call `kfree`. So `heap.c` gains an interrupt-disabling guard around `kmalloc` and `kfree`; only the three marked lines changed.

```c
// file: new/kernel/heap.c
/* A first-fit heap with boundary checks. Interrupt-safe in the kernel; plain C on the host (HOST_TEST). */
#include <mm.h>
#ifdef HOST_TEST
#define HEAP_LOCK()   do { } while (0)
#define HEAP_UNLOCK() do { } while (0)
#else
#include <proc.h>
#define HEAP_LOCK()   uint64_t heap_flags_ = irq_save()                 /* changed: one CPU, so "no interrupts" = locked */
#define HEAP_UNLOCK() irq_restore(heap_flags_)
#endif

#define MAGIC 0x6B6D616Cu
#define GROW  (64 * 1024)

struct block {
    uint32_t magic;
    uint32_t free;
    size_t size;
    struct block *prev, *next;
};
_Static_assert(sizeof(struct block) == 32, "header keeps payloads 16-byte aligned");

static struct block *head, *tail;

static void *payload(struct block *b) { return b + 1; }
static char *end_of(struct block *b) { return (char *)payload(b) + b->size; }

static void split(struct block *b, size_t n) {
    if (b->size < n + sizeof(struct block) + 16) return;
    struct block *r = (struct block *)((char *)payload(b) + n);
    *r = (struct block){MAGIC, 1, b->size - n - sizeof(struct block), b, b->next};
    if (b->next) b->next->prev = r; else tail = r;
    b->next = r;
    b->size = n;
}

static void merge_with_next(struct block *b) {
    struct block *n = b->next;
    b->size += sizeof(struct block) + n->size;
    b->next = n->next;
    if (n->next) n->next->prev = b; else tail = b;
    n->magic = 0;
}

static struct block *grow(size_t n) {
    size_t bytes = ALIGN_UP(n + sizeof(struct block), GROW);
    char *p = morecore(bytes);
    if (!p) return NULL;
    if (tail && tail->free && end_of(tail) == p) { tail->size += bytes; return tail; }
    struct block *b = (struct block *)p;
    *b = (struct block){MAGIC, 1, bytes - sizeof(struct block), tail, NULL};
    if (tail) tail->next = b; else head = b;
    tail = b;
    return b;
}

void *kmalloc(size_t n) {
    if (n == 0 || n > (1UL << 40)) return NULL;
    n = ALIGN_UP(n, 16);
    HEAP_LOCK();                                                       /* changed */
    struct block *b;
    for (b = head; b; b = b->next)
        if (b->free && b->size >= n) break;
    if (b || (b = grow(n))) { split(b, n); b->free = 0; }
    HEAP_UNLOCK();                                                     /* changed */
    return b ? payload(b) : NULL;
}

void kfree(void *p) {
    if (!p) return;
    HEAP_LOCK();
    struct block *b = (struct block *)p - 1;
    if (b->magic != MAGIC) panic("kfree(%p): bad header magic 0x%x (corruption or not from kmalloc)", p, b->magic);
    if (b->free) panic("kfree(%p): double free", p);
    b->free = 1;
    if (b->next && b->next->free) merge_with_next(b);
    if (b->prev && b->prev->free) merge_with_next(b->prev);
    HEAP_UNLOCK();
}

void kheap_check(struct heap_stats *s) {
    HEAP_LOCK();
    *s = (struct heap_stats){0};
    struct block *prev = NULL;
    for (struct block *b = head; b; prev = b, b = b->next) {
        if (b->magic != MAGIC) panic("heap: block %p has bad magic 0x%x", (void *)b, b->magic);
        if (b->prev != prev) panic("heap: block %p has a broken prev link", (void *)b);
        if (b->size % 16) panic("heap: block %p has unaligned size %zu", (void *)b, b->size);
        if (prev && prev->free && b->free) panic("heap: adjacent free blocks %p and %p", (void *)prev, (void *)b);
        s->blocks++;
        s->heap_bytes += sizeof(struct block) + b->size;
        if (b->free) {
            s->free_blocks++;
            s->free_bytes += b->size;
            if (b->size > s->largest_free) s->largest_free = b->size;
        } else {
            s->used_bytes += b->size;
        }
    }
    if (prev != tail) panic("heap: tail pointer is wrong");
    HEAP_UNLOCK();
}
```

(The lock lines in `kfree` and `kheap_check` are new too; they are the same two lines.)

```c
// file: new/kernel/test_threads.c
#include <proc.h>

static void cooperative(void *arg) {
    for (int i = 0; i < 3; i++) {
        kprintf("%s%d ", (const char *)arg, i);
        yield();                                  /* give the CPU away voluntarily */
    }
}

KTEST(threads) {
    preempt_enabled = false;                      /* only yield() switches */
    struct task *a = kthread_create("A", cooperative, "A");
    struct task *b = kthread_create("B", cooperative, "B");
    task_join(a);
    task_join(b);
    kprintf("\nA switched in %lu times, B %lu times\n", a->switches, b->switches);
    preempt_enabled = true;
}
```

```console
$ cd os && make -s && make -s run CMDLINE="test=threads"
hobby os: 127 MiB usable, 32182 of 32736 frames free, command line "test=threads"
== test threads ==
A0 B0 A1 B1 A2 B2 
A switched in 4 times, B 4 times
== test threads passed ==
[qemu exit status 33]
```

Each `yield` hands the CPU to the next thread in the queue, so the output alternates. `kmain` itself is a task: `task_join` blocks it on A's `exit_wait` queue, and A's `task_exit` wakes it.

## Unit 5.2 — Preemption from the timer

**One-line summary:** A thread that never yields would keep the CPU forever. With **preemption**, the timer interrupt counts each thread's time slice (5 ticks here) and sets `need_resched` when it runs out; at the very end of interrupt handling — after EOI — the kernel calls `schedule()`. The interrupted thread is suspended *inside its interrupt handler*, with its whole user or kernel state saved in the trap frame on its own stack; when it is chosen again, `switch_to` returns, the handler finishes, and `iretq` resumes it exactly where it was.

```c
// file: new/kernel/timer.c
#include <proc.h>

volatile uint64_t ticks;
unsigned timer_hz;

static void timer_irq(struct trap_frame *f) {
    (void)f;
    ticks++;
    sched_tick();
#ifdef SWITCH_BEFORE_EOI                    /* experiment switch for DT-10: switch inside the handler */
    if (need_resched) schedule();
#endif
}

void timer_init(unsigned hz) {
    unsigned divisor = 1193182 / hz;
    timer_hz = hz;
    outb(0x43, 0x36);
    outb(0x40, (uint8_t)divisor);
    outb(0x40, (uint8_t)(divisor >> 8));
    irq_register(0, timer_irq);
    pic_unmask(0);
}

uint8_t rtc_read(uint8_t reg) {
    while (outb(0x70, 0x0A), inb(0x71) & 0x80) { }
    outb(0x70, reg);
    return inb(0x71);
}
```

`idt.c` changes in four ways: the end of `interrupt_dispatch` is where preemption and deferred killing happen; vector 128 is the system-call gate, and its gate has DPL 3 so that ring-3 code may use `int 0x80`; and exceptions raised in ring 3 kill the process instead of panicking the kernel.

```nasm
; file: new/arch/isr128.asm
bits 64
section .text
extern isr_common_entry
global isr128
isr128:                           ; int 0x80: the system-call vector; no error code
    push 0
    push 128
    jmp isr_common_entry
```

```nasm
; file: new/arch/isr.asm
bits 64
section .text
extern interrupt_dispatch

%macro ISR_NOERR 1
isr%1:
    push 0
    push %1
    jmp isr_common
%endmacro
%macro ISR_ERR 1
isr%1:
    push %1
    jmp isr_common
%endmacro

%assign v 0
%rep 48
  %if v == 8 || (v >= 10 && v <= 14) || v == 17 || v == 21 || v == 29 || v == 30
    ISR_ERR v
  %else
    ISR_NOERR v
  %endif
  %assign v v + 1
%endrep

global isr_common_entry
isr_common_entry:
isr_common:
    push rax
    push rbx
    push rcx
    push rdx
    push rsi
    push rdi
    push rbp
    push r8
    push r9
    push r10
    push r11
    push r12
    push r13
    push r14
    push r15
    cld
    mov rdi, rsp
    call interrupt_dispatch
    pop r15
    pop r14
    pop r13
    pop r12
    pop r11
    pop r10
    pop r9
    pop r8
    pop rbp
    pop rdi
    pop rsi
    pop rdx
    pop rcx
    pop rbx
    pop rax
    add rsp, 16
    iretq

section .rodata
global isr_table
isr_table:
%assign v 0
%rep 48
    dq isr %+ v
  %assign v v + 1
%endrep
```

(The only change to `isr.asm` is the exported label `isr_common_entry`, so that the new stub can share the common path.)

```c
// file: new/kernel/idt.c
#include <proc.h>

struct idt_gate {
    uint16_t offset_low;
    uint16_t selector;
    uint8_t  ist;
    uint8_t  type;
    uint16_t offset_mid;
    uint32_t offset_high;
    uint32_t reserved;
} __attribute__((packed));
_Static_assert(sizeof(struct idt_gate) == 16, "a long-mode gate is 16 bytes");

static struct idt_gate idt[256];
extern const uint64_t isr_table[48];
extern char isr128[];
static irq_handler_t irq_handlers[16];

static const char *const exception_names[32] = {
    "divide error", "debug", "NMI", "breakpoint", "overflow", "bound range", "invalid opcode", "device not available",
    "double fault", "coprocessor overrun", "invalid TSS", "segment not present", "stack fault", "general protection",
    "page fault", "reserved", "x87 FP error", "alignment check", "machine check", "SIMD FP error", "virtualization",
    "control protection", "reserved", "reserved", "reserved", "reserved", "reserved", "reserved", "hypervisor injection",
    "VMM communication", "security", "reserved"};

static void set_gate(int v, uint64_t a, uint8_t type) {
    idt[v] = (struct idt_gate){ (uint16_t)a, 0x08, 0, type, (uint16_t)(a >> 16), (uint32_t)(a >> 32), 0 };
}

void idt_init(void) {
    for (int v = 0; v < 48; v++) set_gate(v, isr_table[v], 0x8E);   /* ring 0 only */
    set_gate(128, (uint64_t)isr128, 0xEE);                           /* DPL 3: user code may execute int 0x80 */
    struct __attribute__((packed)) { uint16_t limit; uint64_t base; } idtr = { sizeof idt - 1, (uint64_t)idt };
    __asm__ volatile("lidt %0" : : "m"(idtr));
}

void irq_register(int irq, irq_handler_t handler) { irq_handlers[irq] = handler; }

static void dump_frame(const struct trap_frame *f) {
    kprintf("  rip=%p  cs=0x%lx  rflags=0x%lx  rsp=%p\n", (void *)f->rip, f->cs, f->rflags, (void *)f->rsp);
    kprintf("  rax=%016lx rbx=%016lx rcx=%016lx rdx=%016lx\n", f->rax, f->rbx, f->rcx, f->rdx);
    kprintf("  rsi=%016lx rdi=%016lx rbp=%016lx\n", f->rsi, f->rdi, f->rbp);
}

static void user_exception(struct trap_frame *f) {
    int sig = f->vector == 0 ? 8 : f->vector == 6 ? 4 : 11;          /* like SIGFPE, SIGILL, SIGSEGV */
    kprintf("[pid %d %s] killed: %s at rip %p", current->pid, current->name, exception_names[f->vector],
            (void *)f->rip);
    if (f->vector == 14) kprintf(", address %p (error 0x%lx)", (void *)read_cr2(), f->error);
    kprintf("\n");
    task_exit(-sig);
}

static void exception(struct trap_frame *f) {
    if (f->vector == 3) {
        kprintf("breakpoint at rip=%p\n", (void *)f->rip);
        return;
    }
    if ((f->cs & 3) == 3) { user_exception(f); return; }             /* a user bug is not a kernel bug */
    if (f->vector == 14 && vmm_handle_fault(f)) return;
    kprintf("\nEXCEPTION %lu: %s, error code 0x%lx\n", f->vector, exception_names[f->vector], f->error);
    if (f->vector == 14) {
        kprintf("  faulting address (cr2) = %p: %s, %s, %s mode\n", (void *)read_cr2(),
                f->error & 1 ? "protection violation" : "page not present",
                f->error & 2 ? "write" : "read", f->error & 4 ? "user" : "kernel");
    }
    dump_frame(f);
    panic("unhandled exception %lu at rip %p", f->vector, (void *)f->rip);
}

void interrupt_dispatch(struct trap_frame *f) {
    if (f->vector == 128) syscall_dispatch(f);
    else if (f->vector < 32) exception(f);
    else {
        int irq = (int)f->vector - 32;
        if (irq_handlers[irq]) irq_handlers[irq](f);
        pic_eoi(irq);                                                /* DT-10: acknowledge BEFORE switching */
    }
    if (current && current->killed && (f->cs & 3) == 3) task_exit(-9);   /* deferred kill, on the way to user mode */
    if (need_resched) schedule();
}
```

The serial console now blocks a waiting thread on a wait queue instead of halting the whole CPU, and the receive interrupt wakes it:

```c
// file: new/kernel/console.c
#include <proc.h>

#define COM1 0x3F8

void serial_init(void) {
    outb(COM1 + 1, 0x00);
    outb(COM1 + 3, 0x80);
    outb(COM1 + 0, 0x01);
    outb(COM1 + 1, 0x00);
    outb(COM1 + 3, 0x03);
    outb(COM1 + 2, 0xC7);
    outb(COM1 + 4, 0x0B);
}

void serial_putc(char c) {
    if (c == '\n') serial_putc('\r');
    while (!(inb(COM1 + 5) & 0x20)) { }
    outb(COM1, (uint8_t)c);
}

#define RX_SIZE 256
static char rx_buf[RX_SIZE];
static volatile uint32_t rx_head, rx_tail;
static struct waitq rx_wait;
volatile uint64_t serial_irq_count, serial_rx_bytes;

static void serial_irq(struct trap_frame *f) {
    (void)f;
    serial_irq_count++;
    while (inb(COM1 + 5) & 0x01) {
        char c = (char)inb(COM1);
        serial_rx_bytes++;
        if (rx_head - rx_tail < RX_SIZE) rx_buf[rx_head++ % RX_SIZE] = c;
    }
    wake_all(&rx_wait);                         /* changed: wake the readers */
}

void serial_enable_rx_irq(void) {
    irq_register(4, serial_irq);
    outb(COM1 + 1, 0x01);
    pic_unmask(4);
}

int serial_getc(void) {
    uint64_t f = irq_save();
    while (rx_head == rx_tail) wait_on(&rx_wait);   /* changed: sleep this thread, run others */
    char c = rx_buf[rx_tail++ % RX_SIZE];
    irq_restore(f);
    return (unsigned char)c;
}

static volatile uint16_t *const vga = (uint16_t *)0xB8000;
static int row, col;

void vga_clear(void) {
    for (int i = 0; i < 80 * 25; i++) vga[i] = 0x0720;
    row = col = 0;
}

void vga_putc(char c) {
    if (c == '\n') { col = 0; row++; }
    else if (c == '\b') { if (col) vga[row * 80 + --col] = 0x0720; }
    else { vga[row * 80 + col] = (uint16_t)(0x0700 | (uint8_t)c); if (++col == 80) { col = 0; row++; } }
    if (row == 25) {
        memmove((void *)vga, (const void *)(vga + 80), 80 * 24 * 2);
        for (int i = 0; i < 80; i++) vga[24 * 80 + i] = 0x0720;
        row = 24;
    }
}
```

The `while (empty) wait_on(...)` loop with interrupts disabled is the general form of DT-7's `sti; hlt`: the check and the decision to sleep happen with interrupts off, so the wake-up cannot be lost. `schedule()` switches away with interrupts off; another task (or the idle task) turns them on.

```c
// file: new/kernel/test_preempt.c
#include <proc.h>

static volatile uint64_t counts[2];
static volatile bool stop;

static void spinner(void *arg) {                 /* never yields */
    int i = (int)(uintptr_t)arg;
    while (!stop) counts[i]++;
}

KTEST(preempt) {
    struct task *a = kthread_create("spin0", spinner, (void *)0);
    struct task *b = kthread_create("spin1", spinner, (void *)1);
    sleep_ticks(100);                            /* one second; the spinners share the CPU meanwhile */
    stop = true;
    task_join(a);
    task_join(b);
    kprintf("spin0: %lu iterations, %lu ticks of CPU, switched in %lu times\n", counts[0], a->runtime_ticks, a->switches);
    kprintf("spin1: %lu iterations, %lu ticks of CPU, switched in %lu times\n", counts[1], b->runtime_ticks, b->switches);
    KASSERT(a->runtime_ticks >= 40 && b->runtime_ticks >= 40);
}
```

```console
$ cd os && make -s && make -s run CMDLINE="test=preempt"
hobby os: 127 MiB usable, 32182 of 32736 frames free, command line "test=preempt"
== test preempt ==
spin0: 259174463 iterations, 55 ticks of CPU, switched in 12 times
spin1: 235362396 iterations, 50 ticks of CPU, switched in 11 times
== test preempt passed ==
[qemu exit status 33]
```

Two threads that never yield each got about half of the second, in slices of 5 ticks — about 10 slices each.

### Deep & Tricky

**DT-10. When the timer handler switches to another thread, it must already have sent EOI. A thread preempted inside its timer handler is suspended until it is scheduled again — and if it still owes the PIC an EOI, the PIC blocks IRQ 0 (and all lower-priority lines) until then. The next thread then runs with no timer: if it never yields, nothing ever preempts it, and the suspended thread never runs again to send the EOI.** *(Intel 8259A datasheet, "End of Interrupt" — as in DT-6; Linux's x86 IRQ entry code acknowledges the interrupt and only reschedules on the way out, in `irqentry_exit`)* The second trap of the same kind: a *new* thread starts from `schedule()`, which runs with interrupts off — so the trampoline must execute `sti` itself, or the thread runs forever with interrupts disabled. Build with `-DSWITCH_BEFORE_EOI` and the same test hangs:

```console
$ rm -rf eoi && cp -r os eoi && cd eoi && make -s clean && make -s EXTRA_CFLAGS=-DSWITCH_BEFORE_EOI && make -s run CMDLINE="test=preempt" EXTRA_CFLAGS=-DSWITCH_BEFORE_EOI TIMEOUT=15
hobby os: 127 MiB usable, 32182 of 32736 frames free, command line "test=preempt"
== test preempt ==
qemu-system-x86_64: terminating on signal 15 from pid 2527 (timeout)
[qemu exit status 124]
```

The kernel prints the test header and then nothing until `timeout` kills QEMU after 15 seconds (exit status 124): `spin0` was switched in by the first timer interrupt with IRQ 0 still in service, and has run alone ever since.

## Topic 5B — User mode

## Unit 5.3 — Ring 3: GDT, TSS, and user pages

**One-line summary:** To run code in ring 3 the CPU needs (1) **user segments** in the GDT (code and data with DPL 3); (2) a **TSS** (task state segment) whose `rsp0` field tells the CPU which kernel stack to switch to when an interrupt arrives while user code runs — user code's own stack cannot be trusted; and (3) pages mapped with the **user** bit, at every level of the page tables. Entering ring 3 is done by building an interrupt frame by hand and executing `iretq` (`enter_user` above).

```c
// file: new/kernel/gdt.c
#include <proc.h>

struct tss {
    uint32_t reserved0;
    uint64_t rsp0, rsp1, rsp2;     /* stack for interrupts arriving from ring 3 (rsp0) */
    uint64_t reserved1;
    uint64_t ist[7];
    uint64_t reserved2;
    uint16_t reserved3, iomap_base;
} __attribute__((packed));
_Static_assert(sizeof(struct tss) == 104, "64-bit TSS");

static struct tss tss;
static uint64_t gdt[7];

void gdt_init(void) {
    gdt[0] = 0;
    gdt[1] = 0x00209A0000000000;   /* 0x08 kernel code: present, DPL 0, code, long mode (L bit 53) */
    gdt[2] = 0x0000920000000000;   /* 0x10 kernel data: present, DPL 0, writable */
    gdt[3] = 0x0000F20000000000;   /* 0x18 user data:   present, DPL 3, writable */
    gdt[4] = 0x0020FA0000000000;   /* 0x20 user code:   present, DPL 3, code, long mode */
    uint64_t base = (uint64_t)&tss, limit = sizeof tss - 1;
    gdt[5] = (limit & 0xFFFF) | ((base & 0xFFFFFF) << 16) | (0x89UL << 40)   /* 0x28: available 64-bit TSS */
           | (((limit >> 16) & 0xF) << 48) | (((base >> 24) & 0xFF) << 56);
    gdt[6] = base >> 32;           /* a system descriptor takes two slots in long mode */
    tss.iomap_base = sizeof tss;   /* no I/O permission bitmap: ring 3 may not use in/out at all */
    struct __attribute__((packed)) { uint16_t limit; uint64_t base; } gdtr = { sizeof gdt - 1, (uint64_t)gdt };
    __asm__ volatile("lgdt %0\n\t"
                     "pushq $0x08\n\t"            /* reload CS with a far return */
                     "leaq 1f(%%rip), %%rax\n\t"
                     "pushq %%rax\n\t"
                     "lretq\n"
                     "1:\n\t"
                     "mov $0x10, %%ax\n\t"
                     "mov %%ax, %%ds\n\t"
                     "mov %%ax, %%es\n\t"
                     "mov %%ax, %%ss\n\t"
                     "mov $0x28, %%ax\n\t"
                     "ltr %%ax"                    /* load the task register */
                     : : "m"(gdtr) : "rax", "memory");
}

void tss_set_rsp0(uint64_t rsp0) { tss.rsp0 = rsp0; }
```

Each process gets its own PML4. The kernel's half (entries 256–511) is shared by copying the PML4 entries — they point to the same PDPTs, which `vmm_init` now creates up front so that later kernel mappings appear in every address space automatically. The lower half is private, except for the first GiB's identity map (kernel-only, no user bit), which is shared by pointing at the same page directory.

```c
// file: new/include/mm.h
#pragma once
#include <kernel.h>

#define PAGE_SIZE 4096UL
#define ALIGN_UP(x, a)   (((x) + (a) - 1) & ~((uint64_t)(a) - 1))
#define ALIGN_DOWN(x, a) ((x) & ~((uint64_t)(a) - 1))

struct mb2_mmap_entry { uint64_t base, length; uint32_t type, reserved; };
extern struct mb2_mmap_entry usable_regions[16];
extern int usable_region_count;
extern uintptr_t mb_info_start, mb_info_end;

void     pmm_init(void);
uint64_t pmm_alloc(void);
void     pmm_free(uint64_t frame);
uint64_t pmm_free_count(void);
uint64_t pmm_total_count(void);

#define PTE_PRESENT  (1UL << 0)
#define PTE_WRITABLE (1UL << 1)
#define PTE_USER     (1UL << 2)
#define PTE_HUGE     (1UL << 7)
#define PTE_ADDR     0x000FFFFFFFFFF000UL
#define LAZY_START   0xFFFFC00000000000UL
#define LAZY_END     0xFFFFC00040000000UL
#define USER_TOP     0x0000800000000000UL    /* first non-canonical address: user space is below */
extern uint64_t *kernel_pml4;
void      vmm_init(void);
bool      vmm_map(uint64_t virt, uint64_t phys, uint64_t flags);            /* in the kernel's tables */
bool      vmm_map_in(uint64_t *pml4, uint64_t virt, uint64_t phys, uint64_t flags);
uint64_t  vmm_unmap(uint64_t virt);
bool      vmm_translate(uint64_t virt, uint64_t *phys);
uint64_t *vmm_pte(uint64_t virt, bool create);
void      vmm_walk(uint64_t virt);
bool      vmm_handle_fault(struct trap_frame *f);
uint64_t *vmm_new_space(void);
void      vmm_destroy_space(uint64_t *pml4);
bool      user_range_ok(uint64_t *pml4, uint64_t addr, uint64_t len, bool write);
extern uint64_t lazy_faults;
static inline void invlpg(uint64_t virt) { __asm__ volatile("invlpg (%0)" : : "r"(virt) : "memory"); }

#define HEAP_START 0xFFFF800000000000UL
void *kmalloc(size_t n);
void  kfree(void *p);
void *morecore(size_t bytes);
struct heap_stats { size_t blocks, free_blocks, used_bytes, free_bytes, largest_free, heap_bytes; };
void  kheap_check(struct heap_stats *s);
```

```c
// file: new/kernel/vmm.c
#include <mm.h>

extern uint64_t boot_pml4[512];
uint64_t *kernel_pml4 = boot_pml4;
uint64_t lazy_faults;

static unsigned idx(uint64_t virt, int level) { return (virt >> (12 + 9 * level)) & 511; }

static uint64_t *next_table(uint64_t *table, unsigned i, bool create, uint64_t user) {
    if (!(table[i] & PTE_PRESENT)) {
        if (!create) return NULL;
        uint64_t frame = pmm_alloc();
        if (!frame) return NULL;
        memset((void *)frame, 0, PAGE_SIZE);
        table[i] = frame | PTE_PRESENT | PTE_WRITABLE;
    }
#ifndef NO_USER_INTERMEDIATE                  /* experiment switch for DT-11 */
    table[i] |= user;                          /* a user page needs the U bit at every level */
#else
    (void)user;
#endif
    if (table[i] & PTE_HUGE) return NULL;
    return (uint64_t *)(table[i] & PTE_ADDR);
}

static uint64_t *pte_in(uint64_t *pml4, uint64_t virt, bool create, uint64_t user) {
    uint64_t *t = pml4;
    for (int level = 3; level > 0; level--)
        if (!(t = next_table(t, idx(virt, level), create, user))) return NULL;
    return &t[idx(virt, 0)];
}

uint64_t *vmm_pte(uint64_t virt, bool create) { return pte_in(kernel_pml4, virt, create, 0); }

bool vmm_map_in(uint64_t *pml4, uint64_t virt, uint64_t phys, uint64_t flags) {
    KASSERT(virt % PAGE_SIZE == 0 && phys % PAGE_SIZE == 0);
    KASSERT(!(flags & PTE_USER) || virt < USER_TOP);
    uint64_t *pte = pte_in(pml4, virt, true, flags & PTE_USER);
    if (!pte) return false;
    if (*pte & PTE_PRESENT) panic("vmm_map: 0x%lx is already mapped", virt);
    *pte = phys | flags | PTE_PRESENT;
    return true;
}

bool vmm_map(uint64_t virt, uint64_t phys, uint64_t flags) { return vmm_map_in(kernel_pml4, virt, phys, flags); }

uint64_t vmm_unmap(uint64_t virt) {
    uint64_t *pte = vmm_pte(virt, false);
    if (!pte || !(*pte & PTE_PRESENT)) return 0;
    uint64_t phys = *pte & PTE_ADDR;
    *pte = 0;
    invlpg(virt);
    return phys;
}

/* Walk; succeed only if every level is present (and, if required, user-accessible / writable). */
static bool walk(uint64_t *pml4, uint64_t virt, uint64_t need, uint64_t *phys) {
    uint64_t *t = pml4;
    for (int level = 3; level >= 0; level--) {
        uint64_t e = t[idx(virt, level)];
        if ((e & (PTE_PRESENT | need)) != (PTE_PRESENT | need)) return false;
        if (level == 0 || (e & PTE_HUGE)) {
            uint64_t page = 1UL << (12 + 9 * level);
            if (phys) *phys = (e & PTE_ADDR & ~(page - 1)) | (virt & (page - 1));
            return true;
        }
        t = (uint64_t *)(e & PTE_ADDR);
    }
    return false;
}

bool vmm_translate(uint64_t virt, uint64_t *phys) { return walk(kernel_pml4, virt, 0, phys); }

bool user_range_ok(uint64_t *pml4, uint64_t addr, uint64_t len, bool write) {
    if (len == 0) return true;
    if (addr + len < addr || addr + len > USER_TOP) return false;          /* wraps, or reaches the kernel half */
    for (uint64_t p = ALIGN_DOWN(addr, PAGE_SIZE); p < addr + len; p += PAGE_SIZE)
        if (!walk(pml4, p, PTE_USER | (write ? PTE_WRITABLE : 0), NULL)) return false;
    return true;
}

void vmm_walk(uint64_t virt) {
    static const char *const names[] = {"PT", "PD", "PDPT", "PML4"};
    kprintf("walk of %p:\n", (void *)virt);
    uint64_t *t = kernel_pml4;
    for (int level = 3; level >= 0; level--) {
        unsigned i = idx(virt, level);
        uint64_t e = t[i];
        kprintf("  %-4s[%3u] = %016lx  %s%s%s%s\n", names[level], i, e, e & PTE_PRESENT ? "P" : "-",
                e & PTE_WRITABLE ? "W" : "-", e & PTE_USER ? "U" : "-", e & PTE_HUGE ? " huge" : "");
        if (!(e & PTE_PRESENT)) { kprintf("  -> not mapped\n"); return; }
        if (level == 0 || (e & PTE_HUGE)) break;
        t = (uint64_t *)(e & PTE_ADDR);
    }
    uint64_t phys;
    vmm_translate(virt, &phys);
    kprintf("  -> physical %p\n", (void *)phys);
}

bool vmm_handle_fault(struct trap_frame *f) {
    uint64_t addr = read_cr2();
    if (addr < LAZY_START || addr >= LAZY_END || (f->error & 1)) return false;
    uint64_t frame = pmm_alloc();
    if (!frame) return false;
    memset((void *)frame, 0, PAGE_SIZE);
    vmm_map(ALIGN_DOWN(addr, PAGE_SIZE), frame, PTE_WRITABLE);
    lazy_faults++;
    return true;
}

uint64_t *vmm_new_space(void) {
    uint64_t *pml4 = (uint64_t *)pmm_alloc(), *pdpt = (uint64_t *)pmm_alloc();
    if (!pml4 || !pdpt) return NULL;
    memset(pml4, 0, PAGE_SIZE);
    memset(pdpt, 0, PAGE_SIZE);
    uint64_t *kernel_pdpt = (uint64_t *)(kernel_pml4[0] & PTE_ADDR);
    pdpt[0] = kernel_pdpt[0];                          /* share the first GiB identity map (no U bit) */
    pml4[0] = (uint64_t)pdpt | PTE_PRESENT | PTE_WRITABLE;
    for (int i = 256; i < 512; i++) pml4[i] = kernel_pml4[i];   /* share the kernel half */
    return pml4;
}

static void free_table(uint64_t *t, int level) {        /* level 2 = PDPT, 1 = PD, 0 = PT */
    for (int i = 0; i < 512; i++) {
        if (!(t[i] & PTE_PRESENT)) continue;
        if (level == 0) pmm_free(t[i] & PTE_ADDR);
        else free_table((uint64_t *)(t[i] & PTE_ADDR), level - 1);
    }
    pmm_free((uint64_t)t);
}

void vmm_destroy_space(uint64_t *pml4) {
    ((uint64_t *)(pml4[0] & PTE_ADDR))[0] = 0;          /* detach the shared identity map before freeing */
    for (int i = 0; i < 256; i++)
        if (pml4[i] & PTE_PRESENT) free_table((uint64_t *)(pml4[i] & PTE_ADDR), 2);
    pmm_free((uint64_t)pml4);
}

void vmm_init(void) {
    uint64_t cr3;
    __asm__ volatile("mov %%cr3, %0" : "=r"(cr3));
    KASSERT((cr3 & PTE_ADDR) == (uint64_t)kernel_pml4);
    for (int i = 256; i < 512; i++) {                   /* pre-create the kernel half's PDPTs (1 MiB) */
        uint64_t frame = pmm_alloc();
        memset((void *)frame, 0, PAGE_SIZE);
        kernel_pml4[i] = frame | PTE_PRESENT | PTE_WRITABLE;
    }
}
```

## Unit 5.4 — System calls and processes

**One-line summary:** A user program asks the kernel for a service by executing `int 0x80` with the call number in RAX and arguments in RDI, RSI; the result comes back in RAX. The kernel sees the call as one more interrupt: the trap frame holds the user's registers, so reading arguments and writing the result is just reading and writing `f->rdi` and `f->rax`. Every argument is untrusted: a pointer must be checked to lie in user space and be mapped with the right permissions before the kernel touches it.

```c
// file: new/kernel/syscall.c
#include <proc.h>

enum { SYS_WRITE, SYS_EXIT, SYS_GETPID, SYS_SLEEP, SYS_YIELD, SYS_TICKS };
#define EFAULT 14
#define EINVAL 22
#define ENOSYS 38

static long sys_write(uint64_t buf, uint64_t len) {
    if (len > 4096) return -EINVAL;
#ifndef TRUST_USER_POINTERS                    /* experiment switch for DT-12 */
    if (!user_range_ok(current->pml4, buf, len, false)) return -EFAULT;
#endif
    const char *p = (const char *)buf;         /* CR3 is this process's: user addresses work directly */
    for (uint64_t i = 0; i < len; i++) { serial_putc(p[i]); vga_putc(p[i]); }
    return (long)len;
}

void syscall_dispatch(struct trap_frame *f) {
    long r;
    switch (f->rax) {
    case SYS_WRITE:  r = sys_write(f->rdi, f->rsi); break;
    case SYS_EXIT:   task_exit((int)f->rdi);
    case SYS_GETPID: r = current->pid; break;
    case SYS_SLEEP:  sleep_ticks(f->rdi * timer_hz / 1000 + 1); r = 0; break;
    case SYS_YIELD:  yield(); r = 0; break;
    case SYS_TICKS:  r = (long)ticks; break;
    default:         r = -ENOSYS;
    }
    f->rax = (uint64_t)r;
}
```

```c
// file: new/kernel/process.c
#include <proc.h>

#define USER_CODE        0x40000000UL       /* 1 GiB: just above the shared identity map */
#define USER_STACK_TOP   0x80000000UL
#define USER_STACK_PAGES 4

extern void enter_user(uint64_t rip, uint64_t rsp);

static void user_start(void *arg) {
    (void)arg;
    enter_user(USER_CODE, USER_STACK_TOP);       /* never returns: from now on this task lives in ring 3 */
}

static bool map_zeroed(uint64_t *pml4, uint64_t virt, const char *src, size_t n) {
    uint64_t frame = pmm_alloc();
    if (!frame) return false;
    memset((void *)frame, 0, PAGE_SIZE);
    if (src) memcpy((void *)frame, src, n);
    return vmm_map_in(pml4, virt, frame, PTE_WRITABLE | PTE_USER);
}

struct task *process_create(const char *name, const void *image, size_t size) {
    uint64_t *pml4 = vmm_new_space();
    if (!pml4) return NULL;
    for (size_t off = 0; off < size; off += PAGE_SIZE)
        if (!map_zeroed(pml4, USER_CODE + off, (const char *)image + off, size - off < PAGE_SIZE ? size - off : PAGE_SIZE))
            return NULL;
    for (int i = 1; i <= USER_STACK_PAGES; i++)
        if (!map_zeroed(pml4, USER_STACK_TOP - (uint64_t)i * PAGE_SIZE, NULL, 0)) return NULL;
    return task_create(name, user_start, NULL, pml4, true);
}

#define PROG(n) extern const char _binary_user_##n##_bin_start[], _binary_user_##n##_bin_end[];
PROG(hello) PROG(badptr) PROG(crash) PROG(priv) PROG(counter) PROG(spin)
#define ENTRY(n) {#n, _binary_user_##n##_bin_start, _binary_user_##n##_bin_end}
const struct user_prog user_progs[] = {
    ENTRY(hello), ENTRY(badptr), ENTRY(crash), ENTRY(priv), ENTRY(counter), ENTRY(spin), {NULL, NULL, NULL}};

const struct user_prog *user_prog_find(const char *name) {
    for (const struct user_prog *p = user_progs; p->name; p++)
        if (strncmp(p->name, name, 16) == 0) return p;
    return NULL;
}
```

The user programs are small C programs with their own tiny "C library" — a header of system-call wrappers. They are linked to run at `0x40000000`, converted to flat binaries, and embedded in the kernel image as data (`ld -b binary`); Chapter 6 replaces this with a real ELF loader reading files from a disk.

```c
// file: new/user/ulib.h
#pragma once
typedef unsigned long size_t;

static inline long syscall2(long n, long a, long b) {
    long r;
    __asm__ volatile("int $0x80" : "=a"(r) : "a"(n), "D"(a), "S"(b) : "memory");
    return r;
}
static inline long u_write(const void *buf, size_t n) { return syscall2(0, (long)buf, (long)n); }
static inline _Noreturn void u_exit(int code) { syscall2(1, code, 0); for (;;) { } }
static inline long u_getpid(void) { return syscall2(2, 0, 0); }
static inline void u_sleep_ms(long ms) { syscall2(3, ms, 0); }
static inline long u_ticks(void) { return syscall2(5, 0, 0); }

static inline size_t u_strlen(const char *s) { size_t n = 0; while (s[n]) n++; return n; }
static inline void u_puts(const char *s) { u_write(s, u_strlen(s)); }
static inline void u_putnum(long v) {
    char b[24];
    int i = 23;
    int neg = v < 0;
    unsigned long u = neg ? -(unsigned long)v : (unsigned long)v;
    b[i] = 0;
    do { b[--i] = (char)('0' + u % 10); u /= 10; } while (u);
    if (neg) b[--i] = '-';
    u_puts(b + i);
}
static inline void u_puthex(unsigned long v) {
    char b[19] = "0x";
    for (int i = 0; i < 16; i++) b[2 + i] = "0123456789abcdef"[(v >> (60 - 4 * i)) & 15];
    b[18] = 0;
    u_puts(b);
}

int main(void);
__attribute__((section(".text.start"), used)) void _start(void) { u_exit(main()); }
```

```text
# file: new/user/user.ld
ENTRY(_start)
SECTIONS {
    . = 0x40000000;
    .text : { *(.text.start) *(.text*) }
    .rodata : { *(.rodata*) }
    .data : { *(.data*) *(.bss*) *(COMMON) }   /* bss inside .data: the flat binary then contains its zeros */
    /DISCARD/ : { *(.comment) *(.note*) *(.eh_frame*) }
}
```

```c
// file: new/user/hello.c
#include "ulib.h"

int main(void) {
    unsigned short cs;
    __asm__ volatile("mov %%cs, %0" : "=r"(cs));
    u_puts("hello from user space: pid ");
    u_putnum(u_getpid());
    u_puts(", cs=");
    u_puthex(cs);
    u_puts(" -> ring ");
    u_putnum(cs & 3);
    u_puts("\n");
    return 7;
}
```

```c
// file: new/user/badptr.c
#include "ulib.h"

int main(void) {
    u_puts("write(kernel image address)  = ");
    u_putnum(u_write((const char *)0x100000, 16));
    u_puts("\nwrite(kernel heap address)   = ");
    u_putnum(u_write((const char *)0xFFFF800000000000UL, 16));
    u_puts("\nwrite(0x10000000, not user)   = ");
    u_putnum(u_write((const char *)0x10000000, 16));
    u_puts("\nwrite(own string, 3 bytes)   = ");
    long r = u_write("ok ", 3);
    u_putnum(r);
    u_puts("\n");
    return 0;
}
```

```c
// file: new/user/crash.c
#include "ulib.h"

int main(void) {
    u_puts("crash: reading kernel memory at 0x100000\n");
    return *(volatile int *)0x100000;             /* identity-mapped, but without the user bit */
}
```

```c
// file: new/user/priv.c
#include "ulib.h"

int main(void) {
    u_puts("priv: executing cli in ring 3\n");
    __asm__ volatile("cli");
    return 0;
}
```

```c
// file: new/user/counter.c
#include "ulib.h"

static long counter;                              /* each process has its own copy at the same address */

int main(void) {
    long pid = u_getpid();
    for (int i = 0; i < 4; i++) {
        counter += pid;
        u_puts("  pid ");
        u_putnum(pid);
        u_puts(": &counter=");
        u_puthex((unsigned long)&counter);
        u_puts(" counter=");
        u_putnum(counter);
        u_puts("\n");
        u_sleep_ms(30);
    }
    return (int)counter;
}
```

```c
// file: new/user/spin.c
#include "ulib.h"

int main(void) {
    volatile unsigned long n = 0;
    for (;;) n++;                                 /* never yields, never makes a system call */
}
```

```text
# file: new/Makefile
CC      := gcc
CFLAGS  := -std=c11 -ffreestanding -fno-stack-protector -fno-pic -mno-red-zone -mno-sse -mno-mmx -mno-80387 \
           -mcmodel=small -fno-omit-frame-pointer -O2 -g -Wall -Wextra -Werror -Iinclude $(EXTRA_CFLAGS)
UCFLAGS := -std=c11 -ffreestanding -fno-stack-protector -fno-pic -mno-sse -mno-mmx -mno-80387 -O2 -Wall -Wextra -Werror
LDFLAGS := -n -T linker.ld -nostdlib -z noexecstack -z max-page-size=0x1000 --no-warn-rwx-segments
SRCS_C  := $(wildcard kernel/*.c)
SRCS_S  := $(wildcard arch/*.asm)
USER    := hello badptr crash priv counter spin
OBJS    := $(SRCS_S:.asm=.o) $(SRCS_C:.c=.o) $(USER:%=user/%.bin.o)
CMDLINE ?=
TIMEOUT ?= 60
QEMU    := qemu-system-x86_64 -serial stdio -display none -no-reboot -m 128M \
           -device isa-debug-exit,iobase=0xf4,iosize=0x04 $(QEMU_EXTRA)

kernel.elf: $(OBJS) linker.ld
	ld $(LDFLAGS) -o $@ $(OBJS)

%.o: %.asm
	nasm -f elf64 -g $< -o $@
%.o: %.c $(wildcard include/*.h)
	$(CC) $(CFLAGS) -c $< -o $@

user/%.elf: user/%.c user/ulib.h user/user.ld
	$(CC) $(UCFLAGS) -c $< -o user/$*.o
	ld -T user/user.ld -nostdlib -z noexecstack --no-warn-rwx-segments -o $@ user/$*.o
user/%.bin: user/%.elf
	objcopy -O binary $< $@
user/%.bin.o: user/%.bin
	ld -r -b binary -z noexecstack -o $@ $<

os.iso: kernel.elf
	rm -rf iso && mkdir -p iso/boot/grub && cp kernel.elf iso/boot/
	printf 'set timeout=0\nset default=0\nmenuentry "hobby os" {\n  multiboot2 /boot/kernel.elf %s\n  boot\n}\n' \
	    "$(CMDLINE)" > iso/boot/grub/grub.cfg
	grub-mkrescue -o $@ iso > /dev/null 2>&1

run: kernel.elf
	rm -f os.iso && $(MAKE) -s os.iso CMDLINE="$(CMDLINE)"
	timeout $(TIMEOUT) $(QEMU) -cdrom os.iso; status=$$?; echo "[qemu exit status $$status]"

clean:
	rm -rf $(OBJS) kernel.elf os.iso iso grub.cfg user/*.o user/*.elf user/*.bin
.PHONY: run clean os.iso
.PRECIOUS: user/%.elf user/%.bin
```

```c
// file: new/kernel/main.c
#include <proc.h>

extern const struct ktest __ktests_start[], __ktests_end[];

__attribute__((weak)) void shell_run(void) { kprintf("no shell in this build\n"); }

static void run_test(const char *name) {
    for (const struct ktest *t = __ktests_start; t < __ktests_end; t++) {
        if (strncmp(t->name, name, 64) == 0) {
            kprintf("== test %s ==\n", t->name);
            t->fn();
            kprintf("== test %s passed ==\n", t->name);
            return;
        }
    }
    kprintf("unknown test '%s'; available:", name);
    for (const struct ktest *t = __ktests_start; t < __ktests_end; t++) kprintf(" %s", t->name);
    kprintf("\n");
}

void kmain(uint32_t magic, uintptr_t info) {
    serial_init();
    vga_clear();
    multiboot2_parse(magic, info);
    gdt_init();
    idt_init();
    pmm_init();
    vmm_init();
    sched_init();
    kprintf("hobby os: %lu MiB usable, %lu of %lu frames free, command line \"%s\"\n", usable_ram_bytes >> 20,
            pmm_free_count(), pmm_total_count(), kernel_cmdline);
    pic_init();
    timer_init(100);
    serial_enable_rx_irq();
    sti();
    char name[32];
    if (cmdline_get("test", name, sizeof name)) run_test(name);
    else shell_run();
    qemu_exit(0x10);
}
```

```c
// file: new/kernel/test_user.c
#include <proc.h>

static int run(const char *name) {
    const struct user_prog *p = user_prog_find(name);
    KASSERT(p);
    struct task *t = process_create(p->name, p->start, (size_t)(p->end - p->start));
    KASSERT(t);
    int code = task_join(t);
    kprintf("[%s exited with status %d]\n", name, code);
    return code;
}

KTEST(user) {
    uint64_t frames = pmm_free_count();
    KASSERT(run("hello") == 7);
    KASSERT(run("badptr") == 0);
    KASSERT(run("crash") == -11);
    KASSERT(run("priv") == -11);
    yield();                                      /* let the last zombie be freed */
    kprintf("frames before %lu, after %lu: every process's memory was returned\n", frames, pmm_free_count());
    KASSERT(pmm_free_count() == frames);
}

KTEST(isolation) {
    const struct user_prog *p = user_prog_find("counter");
    struct task *a = process_create("counter", p->start, (size_t)(p->end - p->start));
    struct task *b = process_create("counter", p->start, (size_t)(p->end - p->start));
    int ca = task_join(a), cb = task_join(b);
    kprintf("pid %d exited with %d, pid %d exited with %d\n", a->pid, ca, b->pid, cb);
    KASSERT(ca == 4 * a->pid && cb == 4 * b->pid);
}
```

```console
$ cd os && make -s && ls -l user/*.bin | awk '{print $5, $9}' && make -s run CMDLINE="test=user"
599 user/badptr.bin
688 user/counter.bin
122 user/crash.bin
510 user/hello.bin
111 user/priv.bin
48 user/spin.bin
hobby os: 127 MiB usable, 32182 of 32736 frames free, command line "test=user"
== test user ==
hello from user space: pid 2, cs=0x0000000000000023 -> ring 3
[hello exited with status 7]
write(kernel image address)  = -14
write(kernel heap address)   = -14
write(0x10000000, not user)   = -14
write(own string, 3 bytes)   = ok 3
[badptr exited with status 0]
crash: reading kernel memory at 0x100000
[pid 4 crash] killed: page fault at rip 0x0000000040000046, address 0x0000000000100000 (error 0x5)
[crash exited with status -11]
priv: executing cli in ring 3
[pid 5 priv] killed: general protection at rip 0x0000000040000046
[priv exited with status -11]
frames before 32182, after 32182: every process's memory was returned
== test user passed ==
[qemu exit status 33]
$ cd os && make -s run CMDLINE="test=isolation"
hobby os: 127 MiB usable, 32182 of 32736 frames free, command line "test=isolation"
== test isolation ==
  pid 2: &counter=0x00000000400002a8 counter=2
  pid 3: &counter=0x00000000400002a8 counter=3
  pid 3: &counter=0x00000000400002a8 counter=6
  pid 2: &counter=0x00000000400002a8 counter=4
  pid 3: &counter=0x00000000400002a8 counter=9
  pid 2: &counter=0x00000000400002a8 counter=6
  pid 3: &counter=0x00000000400002a8 counter=12
  pid 2: &counter=0x00000000400002a8 counter=8
pid 2 exited with 8, pid 3 exited with 12
== test isolation passed ==
[qemu exit status 33]
```

Read the results in order. `hello` really ran in ring 3 (CS = 0x23, low two bits 3). `badptr` asked the kernel to read from three addresses that are not user memory (the third, 256 MiB, lies in the kernel-only identity map), and got `-14` (`EFAULT`) each time — the kernel checked before touching anything. `crash` read a page that exists but has no user bit: a protection-violation page fault (error 5 = present + user mode), and the kernel killed the process with "signal" 11. `priv` executed `cli`, which ring 3 may not do: general protection fault, killed. After all four exits, every frame came back. In `isolation`, both processes use `counter` at the same virtual address, but each sees only its own value: the same address, two different frames.

### Deep & Tricky

**DT-11. A page is accessible from ring 3 only if the user bit is set in the entry at *every* level of the walk — PML4, PDPT, PD and PT. The effective permission is the AND of all levels. A kernel that sets `PTE_USER` only on the last-level entry gets page faults on perfectly "mapped" user pages.** *(Intel SDM Vol. 3A, §4.6.1 "Determination of Access Rights": a linear address is a user-mode address if U/S is 1 "in every paging-structure entry controlling the translation")* The same AND rule applies to the writable bit. Build with `-DNO_USER_INTERMEDIATE` (the intermediate entries do not get the U bit) and the very first instruction of `hello` faults:

```console
$ rm -rf nouser && cp -r os nouser && cd nouser && make -s clean && make -s EXTRA_CFLAGS=-DNO_USER_INTERMEDIATE && make -s run CMDLINE="test=user" EXTRA_CFLAGS=-DNO_USER_INTERMEDIATE 2>&1 | sed -n 2,4p
== test user ==
[pid 2 hello] killed: page fault at rip 0x0000000040000000, address 0x0000000040000000 (error 0x5)
[hello exited with status -11]
```

The fault address is the entry point itself, `0x40000000`, and the error code 5 says "present, user mode": the page *is* mapped — the CPU refused because an upper level said "supervisor only".

**DT-12. A system call that uses a user-supplied pointer without checking it turns the kernel into a "confused deputy": the user cannot read kernel memory directly (DT-11 stops it), but it can ask the kernel to read it on its behalf — and the kernel, running in ring 0, is allowed to.** *(Linux `arch/x86/include/asm/uaccess.h`: `access_ok()` "Checks if a pointer to a block of memory in user space is valid"; every `copy_from_user`/`copy_to_user` performs this check — and since Linux 4.x, x86 also enables SMAP so that the hardware faults on any kernel access to user pages outside these helpers)* Build with `-DTRUST_USER_POINTERS` and run `badptr`: the kernel happily prints 16 bytes of its own image (and would fault on an unmapped address — in ring 0, which is a kernel panic triggered by any user program).

```console
$ rm -rf trust && cp -r os trust && cd trust && make -s clean && make -s EXTRA_CFLAGS=-DTRUST_USER_POINTERS && make -s run CMDLINE="test=user" EXTRA_CFLAGS=-DTRUST_USER_POINTERS 2>&1 | sed -n 4,16p | cat -v | cut -c1-110
[hello exited with status 7]^M
write(kernel image address)  = M-VPRM-h^@^@^@^@^X^@^@^@^RM-/M--^W16^M
write(kernel heap address)   = lamk^@^@^@^@p^@^@^@^@^@^@^@16^M
write(0x10000000, not user)   = ^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@16^M
write(own string, 3 bytes)   = ok 3^M
[badptr exited with status 0]^M
crash: reading kernel memory at 0x100000^M
[pid 4 crash] killed: page fault at rip 0x0000000040000046, address 0x0000000000100000 (error 0x5)^M
[crash exited with status -11]^M
priv: executing cli in ring 3^M
[pid 5 priv] killed: general protection at rip 0x0000000040000046^M
[priv exited with status -11]^M
frames before 32182, after 32182: every process's memory was returned^M
```

The first `write` dumped raw bytes of the kernel image to the console — `M-VPRM-h` is `D6 50 52 E8`, the start of our Multiboot2 header, made visible by `cat -v`. The second printed kernel heap data: `lamk` is the heap's block magic `0x6B6D616C`. The third read 16 bytes of physical memory at 256 MiB through the kernel's identity map. All three return "success" (16). Had the program passed an address the kernel has not mapped at all, the read would have faulted *in ring 0* — a kernel panic triggered by an unprivileged program.

### Pattern

**P-6. Act on a task only at safe points: anything (an interrupt, another task, the shell) may set a flag — `need_resched`, `killed` — but the flag is acted upon only at a well-defined boundary: the end of interrupt handling, just before returning to the interrupted code.** *(Practice; Linux checks `TIF_NEED_RESCHED` and `TIF_SIGPENDING` in `exit_to_user_mode_loop()`)* At that point the task holds no locks and its full state is in the trap frame, so switching away from it or killing it cannot leave a data structure half-updated. *Trade-off:* the action is delayed — until the next interrupt or system call; a task looping in user mode with interrupts on is reached within one timer tick, but a task blocked in the kernel must be woken first (the project's `kill` does that for sleeping tasks).

### Mastery check

1. **Predict.** `switch_to` saves only six registers. Why is it not a bug that RAX, RCX, RDX, RSI, RDI and R8–R11 are not saved?
2. **Predict.** What would happen on the first interrupt from ring 3 if `tss_set_rsp0` were never called?
3. **Predict.** User programs are compiled *with* the red zone (no `-mno-red-zone`). Why is that safe for them but not for the kernel?

## Chapter 5 Project — Process control from the shell

**Goal:** turn the shell into a small process manager: `run <prog>` starts a user program and waits for it, `run <prog> &` starts it in the background, `ps` lists every task with its state, CPU time and context-switch count, and `kill <pid>` stops a process. Because the shell is itself a task blocked in `serial_getc`, it stays responsive even while a background process spins in an endless loop — preemption at work.

```c
// file: proj/kernel/cmd_proc.c
#include <shell.h>
#include <proc.h>

static int parse_int(const char *s) {
    int v = 0;
    while (*s >= '0' && *s <= '9') v = v * 10 + (*s++ - '0');
    return v;
}

SHELL_CMD(run, "run a user program: run <name> [&]") {
    char name[16];
    size_t n = 0;
    while (*arg && *arg != ' ' && n + 1 < sizeof name) name[n++] = *arg++;
    name[n] = '\0';
    while (*arg == ' ') arg++;
    bool background = *arg == '&';
    const struct user_prog *p = user_prog_find(name);
    if (!p) {
        kprintf("no program '%s'; available:", name);
        for (p = user_progs; p->name; p++) kprintf(" %s", p->name);
        kprintf("\n");
        return;
    }
    struct task *t = process_create(p->name, p->start, (size_t)(p->end - p->start));
    if (!t) { kprintf("out of memory\n"); return; }
    if (background) { kprintf("[%d] %s started in the background\n", t->pid, p->name); return; }
    kprintf("[%s exited with status %d]\n", p->name, task_join(t));
}

SHELL_CMD(ps, "list tasks") {
    (void)arg;
    kprintf("  PID  STATE     KIND    CPU(ticks)  SWITCHES  NAME\n");
    for (struct task *t = task_first(); t; t = t->all_next) {
        if (t->state == TASK_ZOMBIE) kprintf("%5d  exit %-4d %-7s %10lu  %8lu  %s\n", t->pid, t->exit_code,
                                             t->user ? "user" : "kernel", t->runtime_ticks, t->switches, t->name);
        else kprintf("%5d  %-9s %-7s %10lu  %8lu  %s\n", t->pid, task_state_name(t->state),
                     t->user ? "user" : "kernel", t->runtime_ticks, t->switches, t->name);
    }
}

SHELL_CMD(kill, "kill a user process: kill <pid>") {
    int pid = parse_int(arg);
    uint64_t f = irq_save();
    struct task *t = task_find(pid);
    if (!t || !t->user || t->state == TASK_ZOMBIE) {
        irq_restore(f);
        kprintf("no live user process %d\n", pid);
        return;
    }
    t->killed = true;                                          /* P-6: acted on when it next leaves the kernel */
    if (t->state == TASK_SLEEPING) t->wake_tick = 0;           /* sleeping in a system call: wake it at the next tick */
    irq_restore(f);
    task_join(t);
    kprintf("[%d %s] killed, exit status %d\n", pid, t->name, t->exit_code);
}
```

```console
$ rm -rf final && cp -r os final && cp -r proj/. final/ && make -s -C final clean && make -s -C final && echo "final = os + proj"
final = os + proj
$ cd final && (sleep 4; for l in 'run hello' 'run spin &' 'run counter &' 'uptime' 'ps' 'run nosuch' 'kill 3' 'kill 3' 'ps' 'halt'; do printf "$l\n"; sleep 0.5; done) | make -s run
hobby os: 127 MiB usable, 32182 of 32736 frames free, command line ""
kshell ready; type 'help'
kshell> run hello
hello from user space: pid 2, cs=0x0000000000000023 -> ring 3
[hello exited with status 7]
kshell> run spin &
[3] spin started in the background
kshell> run counter &
[4] counter started in the background
kshell>   pid 4: &counter=0x00000000400002a8 counter=4
  pid 4: &counter=0x00000000400002a8 counter=8
  pid 4: &counter=0x00000000400002a8 counter=12
  pid 4: &counter=0x00000000400002a8 counter=16
uptime
up 4.98 s (498 ticks at 100 Hz)
kshell> ps
  PID  STATE     KIND    CPU(ticks)  SWITCHES  NAME
    4  exit 16   user             0         5  counter
    3  runnable  user           155         8  spin
    2  exit 7    user             0         1  hello
    1  runnable  kernel         392         2  idle
    0  running   kernel           1         6  kmain
kshell> run nosuch
no program 'nosuch'; available: hello badptr crash priv counter spin
kshell> kill 3
[3 spin] killed, exit status -9
kshell> kill 3
no live user process 3
kshell> ps
  PID  STATE     KIND    CPU(ticks)  SWITCHES  NAME
    4  exit 16   user             0         5  counter
    3  exit -9   user           256        11  spin
    2  exit 7    user             0         1  hello
    1  runnable  kernel         487         4  idle
    0  running   kernel           2        11  kmain
kshell> halt
halting
[qemu exit status 33]
```

While `spin` loops forever in ring 3, the shell still answers every line within a time slice, and `counter` makes progress — three runnable tasks sharing one CPU. `ps` shows `spin` with almost all the CPU ticks. `kill 3` sets the flag; at the next timer interrupt, on its way back to ring 3, `spin` meets the check in `interrupt_dispatch` and exits with status -9. A second `kill 3` finds a zombie and refuses.

### Review checklist

- Zombies stay in the task list forever. What should remove them, and when is it safe to free the `struct task` itself?
- `kill` of a task blocked on a wait queue does nothing until it is woken. How would you fix that without breaking the wait queue?
- What happens if the shell runs `run counter` (foreground) and the user wants to kill it?

## Answers to the Chapter 5 prediction questions

**5.4 Q1.** `switch_to` is called like an ordinary C function, and the System V calling convention says a function may destroy RAX, RCX, RDX, RSI, RDI and R8–R11 — so the caller (`schedule`) already assumes they are lost. Only the callee-saved registers must look unchanged when `switch_to` "returns". The interrupted program's registers are not in danger either: they were saved in its trap frame at interrupt entry.

**5.4 Q2.** RSP0 would be 0. On the first interrupt from ring 3, the CPU would switch to stack address 0 and push the interrupt frame below it — to addresses near `0xFFFFFFFFFFFFFFF8`, which are not mapped. That is a page fault while delivering an interrupt → double fault → (with no double-fault handler stack) triple fault: the machine resets.

**5.4 Q3.** When an interrupt arrives while user code runs, the CPU switches to the kernel stack from the TSS before pushing anything, so the 128 bytes below the user's RSP are never touched by interrupt delivery. Kernel code is interrupted *on its own stack*, so the CPU pushes the frame right where the red zone lives (DT-2).

# Chapter 6 — Disks, a File System, and Loading Programs

*Goal: stop embedding programs in the kernel image. The kernel gets a **disk driver** (ATA in PIO mode), a small on-disk **file system** of our own design ("hobbyfs") with a host tool that builds disk images, **file descriptors** and file system calls, and an **ELF loader** that starts programs stored on the disk. The project is a shell that runs in user mode and starts other programs from the disk — the shape of every Unix since 1971.*

**What you will be able to do after this chapter**

- Drive an IDE disk with port I/O: identify it, read and write sectors.
- Design an on-disk layout — superblock, bitmaps, inodes, direct and indirect blocks, directories — and write both sides of it: a `mkfs` tool on the host and a reader in the kernel.
- Give each process a file-descriptor table, and implement `open`, `read`, `readdir`, `close`, `spawn`, and `wait`.
- Load an ELF executable by its program headers — and treat every byte of it as hostile input.

The kernel continues from Chapter 5's `final/`. New and changed files are in `new/`; `os/` is Chapter 5's kernel plus `new/`.

```console
$ rm -rf os && cp -r ../ch05/final os && cp -r new/. os/ && make -s -C os clean && echo ready
ready
```

## Topic 6A — The disk

## Unit 6.1 — ATA PIO: talking to a disk through I/O ports

**One-line summary:** The classic IDE/ATA controller has a **command block** at ports 0x1F0–0x1F7 (primary channel): a data port, a sector count, the LBA (logical block address — the sector number) in three ports plus four bits in the drive register, and a command/status port. To read: wait until the drive is not **BSY**, write drive, count and LBA, send command 0x20 (READ SECTORS), then for each sector wait for **DRQ** (data request) and read 256 16-bit words from the data port. This is **PIO** (programmed I/O): the CPU copies every byte itself. Real drivers use **DMA** (the device writes memory directly) and interrupts; PIO is the simplest correct start.

```c
// file: new/kernel/ata.c
/* ATA PIO driver for the primary channel's master drive, LBA28, polling (no interrupts). */
#include <kernel.h>
#include <io16.h>

#define ATA_DATA    0x1F0
#define ATA_ERROR   0x1F1
#define ATA_COUNT   0x1F2
#define ATA_LBA0    0x1F3
#define ATA_LBA1    0x1F4
#define ATA_LBA2    0x1F5
#define ATA_DRIVE   0x1F6
#define ATA_CMD     0x1F7          /* write: command; read: status */
#define ATA_CONTROL 0x3F6          /* write: device control; read: alternate status */

#define ST_ERR 0x01
#define ST_DRQ 0x08
#define ST_DF  0x20
#define ST_BSY 0x80

uint64_t ata_sectors, ata_reads, ata_writes;
char ata_model[41];

static void delay400ns(void) { for (int i = 0; i < 4; i++) (void)inb(ATA_CONTROL); }   /* ~100 ns per read */

static int wait_ready(void) {
    for (;;) {
        uint8_t st = inb(ATA_CMD);
        if (st & ST_BSY) continue;
        if (st & (ST_ERR | ST_DF)) return -1;
        return 0;
    }
}

static int wait_drq(void) {
    for (;;) {
        uint8_t st = inb(ATA_CMD);
        if (st & ST_BSY) continue;
        if (st & (ST_ERR | ST_DF)) return -1;
        if (st & ST_DRQ) return 0;
    }
}

int ata_init(void) {
    outb(ATA_CONTROL, 0x02);                          /* nIEN: no interrupts, we poll */
    outb(ATA_DRIVE, 0xA0);                            /* select master */
    delay400ns();
    outb(ATA_COUNT, 0); outb(ATA_LBA0, 0); outb(ATA_LBA1, 0); outb(ATA_LBA2, 0);
    outb(ATA_CMD, 0xEC);                              /* IDENTIFY DEVICE */
    if (inb(ATA_CMD) == 0) return -1;                 /* status 0: no drive */
    while (inb(ATA_CMD) & ST_BSY) { }
    if (inb(ATA_LBA1) || inb(ATA_LBA2)) return -1;    /* an ATAPI (CD-ROM) or SATA signature, not ATA */
    if (wait_drq() < 0) return -1;
    uint16_t id[256];
    for (int i = 0; i < 256; i++) id[i] = inw(ATA_DATA);
    for (int i = 0; i < 20; i++) {                    /* model string: words 27-46, bytes swapped */
        ata_model[2 * i] = (char)(id[27 + i] >> 8);
        ata_model[2 * i + 1] = (char)id[27 + i];
    }
    for (int i = 39; i >= 0 && ata_model[i] == ' '; i--) ata_model[i] = '\0';
    ata_sectors = id[60] | (uint64_t)id[61] << 16;    /* number of LBA28-addressable sectors */
    return 0;
}

static void issue(uint32_t lba, uint8_t count, uint8_t cmd) {
    outb(ATA_DRIVE, (uint8_t)(0xE0 | ((lba >> 24) & 0x0F)));  /* LBA mode, master, LBA bits 24-27 */
    outb(ATA_COUNT, count);
    outb(ATA_LBA0, (uint8_t)lba);
    outb(ATA_LBA1, (uint8_t)(lba >> 8));
    outb(ATA_LBA2, (uint8_t)(lba >> 16));
    outb(ATA_CMD, cmd);
}

int ata_read(uint32_t lba, uint8_t count, void *buf) {
    if (!count || lba + count > ata_sectors) return -1;
    uint64_t f = 0;
    __asm__ volatile("pushfq; pop %0; cli" : "=r"(f) : : "memory");   /* one command at a time */
    int r = wait_ready();
    if (r == 0) {
        issue(lba, count, 0x20);                      /* READ SECTORS */
        uint16_t *p = buf;
        for (int s = 0; s < count && r == 0; s++) {
            delay400ns();                             /* status is not valid immediately after a command */
            if ((r = wait_drq()) == 0) {
                __asm__ volatile("rep insw" : "+D"(p), "+c"((uint64_t){256}) : "d"(ATA_DATA) : "memory");
            }
        }
        ata_reads += count;
    }
    if (f & 0x200) sti();
    return r;
}

int ata_write(uint32_t lba, uint8_t count, const void *buf) {
    if (!count || lba + count > ata_sectors) return -1;
    uint64_t f = 0;
    __asm__ volatile("pushfq; pop %0; cli" : "=r"(f) : : "memory");
    int r = wait_ready();
    if (r == 0) {
        issue(lba, count, 0x30);                      /* WRITE SECTORS */
        const uint16_t *p = buf;
        for (int s = 0; s < count && r == 0; s++) {
            delay400ns();
            if ((r = wait_drq()) == 0) {
                __asm__ volatile("rep outsw" : "+S"(p), "+c"((uint64_t){256}) : "d"(ATA_DATA) : "memory");
            }
        }
        if (r == 0) { outb(ATA_CMD, 0xE7); r = wait_ready(); }   /* CACHE FLUSH: make it durable */
        ata_writes += count;
    }
    if (f & 0x200) sti();
    return r;
}
```

The `rep insw` operand `"+c"((uint64_t){256})` uses a compound literal as a scratch counter: the instruction counts RCX down to zero, and the asm statement must say that RCX is modified. `inw` is a new helper for 16-bit port reads in `kernel.h`'s style:

```c
// file: new/include/io16.h
#pragma once
#include <stdint.h>
static inline uint16_t inw(uint16_t port) { uint16_t v; __asm__ volatile("inw %1, %0" : "=a"(v) : "Nd"(port)); return v; }
```

## Topic 6B — A file system

## Unit 6.2 — hobbyfs: the on-disk layout

**One-line summary:** A file system turns "sector 81,233" into "/docs/numbers.txt". Ours uses 1 KiB **blocks** (two sectors). Block 1 is the **superblock** (sizes and where everything is); block 2 is the **inode bitmap**, block 3 the **block bitmap**; then the **inode table** (64-byte inodes, 16 per block); then data. An **inode** describes one file or directory: type, size, 12 **direct** block numbers and one **indirect** block (a block full of 256 more block numbers) — so a file can grow to 12 + 256 blocks = 268 KiB. A **directory** is a file whose content is an array of 32-byte entries (inode number, name). Inode 1 is the root directory. This is the design of the original Unix file system and of ext2, in miniature.

```text
block:   0       1        2          3          4 ... 19         20 ...
       [boot] [super] [inode bmp] [block bmp] [inode table]   [data blocks ...]

inode (64 bytes): type | links | size | direct[12] | indirect | mtime

file block n  ->  n < 12 :  inode.direct[n]
                  n < 268:  ((uint32_t *)block(inode.indirect))[n - 12]
```

The layout is defined once, in a header shared by the kernel and the host tool. Every on-disk structure uses fixed-width types, and its size is checked at compile time (DT-13):

```c
// file: new/include/hobbyfs.h
#pragma once
#include <stdint.h>

#define HBFS_MAGIC       0x53464248u        /* "HBFS" as little-endian bytes */
#define HBFS_BLOCK       1024
#define HBFS_NDIRECT     12
#define HBFS_NINDIRECT   (HBFS_BLOCK / 4)
#define HBFS_MAXBLOCKS   (HBFS_NDIRECT + HBFS_NINDIRECT)
#define HBFS_ROOT        1
#define HBFS_NAME        28
enum { HBFS_FREE = 0, HBFS_FILE = 1, HBFS_DIR = 2 };

struct hbfs_super {
    uint32_t magic, block_size, total_blocks, inode_count;
    uint32_t inode_bitmap, block_bitmap, inode_table, inode_table_blocks, data_start;
    uint32_t free_blocks, free_inodes;
};
struct hbfs_inode {
    uint16_t type, links;
    uint32_t size;
    uint32_t direct[HBFS_NDIRECT];
    uint32_t indirect;
    uint32_t mtime;
};
struct hbfs_dirent {
    uint32_t inode;                          /* 0: unused slot */
    char name[HBFS_NAME];                    /* NUL-terminated unless exactly 28 bytes long */
};
#define HBFS_INODES_PER_BLOCK (HBFS_BLOCK / sizeof(struct hbfs_inode))

_Static_assert(sizeof(struct hbfs_super) == 44, "on-disk superblock layout");
_Static_assert(sizeof(struct hbfs_inode) == 64, "on-disk inode layout");
_Static_assert(sizeof(struct hbfs_dirent) == 32, "on-disk directory entry layout");
```

The host tool builds an image from a directory tree. It is ordinary C — and the only code that ever *writes* a hobbyfs; the kernel's side is read-only in this chapter.

```c
// file: new/tools/mkhobbyfs.c
/* mkhobbyfs IMAGE BLOCKS SRCDIR: build a hobbyfs image containing the tree under SRCDIR. */
#include <dirent.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include "hobbyfs.h"

static uint8_t *img;
static struct hbfs_super *sb;

static uint8_t *block(uint32_t b) { return img + (size_t)b * HBFS_BLOCK; }
static void die(const char *m, const char *x) { fprintf(stderr, "mkhobbyfs: %s %s\n", m, x ? x : ""); exit(1); }

static uint32_t alloc_bit(uint32_t bitmap_block, uint32_t limit, uint32_t *free_count) {
    uint8_t *bm = block(bitmap_block);
    for (uint32_t i = 0; i < limit; i++)
        if (!(bm[i / 8] & (1 << (i % 8)))) { bm[i / 8] |= (uint8_t)(1 << (i % 8)); (*free_count)--; return i; }
    die("out of space", NULL);
    return 0;
}
static uint32_t alloc_block(void) { return alloc_bit(sb->block_bitmap, sb->total_blocks, &sb->free_blocks); }
static uint32_t alloc_inode(void) { return alloc_bit(sb->inode_bitmap, sb->inode_count, &sb->free_inodes); }
static struct hbfs_inode *inode(uint32_t ino) {
    return (struct hbfs_inode *)block(sb->inode_table + ino / HBFS_INODES_PER_BLOCK) + ino % HBFS_INODES_PER_BLOCK;
}

static void write_data(uint32_t ino, const uint8_t *data, size_t size) {
    struct hbfs_inode *ip = inode(ino);
    size_t nblocks = (size + HBFS_BLOCK - 1) / HBFS_BLOCK;
    if (nblocks > HBFS_MAXBLOCKS) die("file too large", NULL);
    for (size_t n = 0; n < nblocks; n++) {
        uint32_t b = alloc_block();
        size_t len = size - n * HBFS_BLOCK < HBFS_BLOCK ? size - n * HBFS_BLOCK : HBFS_BLOCK;
        memcpy(block(b), data + n * HBFS_BLOCK, len);
        if (n < HBFS_NDIRECT) ip->direct[n] = b;
        else {
            if (!ip->indirect) ip->indirect = alloc_block();
            ((uint32_t *)block(ip->indirect))[n - HBFS_NDIRECT] = b;
        }
    }
    ip->size = (uint32_t)size;
}

static uint32_t add_tree(const char *path, uint32_t parent) {
    uint32_t ino = alloc_inode();
    if (parent == 0) parent = ino;                       /* the root is its own parent */
    inode(ino)->type = HBFS_DIR;
    inode(ino)->links = 2;
    struct dirent **names;
    int n = scandir(path, &names, NULL, alphasort);      /* sorted: the image is reproducible */
    if (n < 0) die("cannot read directory", path);
    struct hbfs_dirent *ents = calloc((size_t)n + 2, sizeof *ents);
    int count = 0;
    ents[count].inode = ino; strcpy(ents[count++].name, ".");
    ents[count].inode = parent; strcpy(ents[count++].name, "..");
    for (int i = 0; i < n; i++) {
        const char *name = names[i]->d_name;
        if (name[0] == '.') { free(names[i]); continue; }
        if (strlen(name) >= HBFS_NAME) die("name too long:", name);
        char child[4096];
        snprintf(child, sizeof child, "%s/%s", path, name);
        struct stat st;
        if (stat(child, &st) < 0) die("cannot stat", child);
        uint32_t cino;
        if (S_ISDIR(st.st_mode)) cino = add_tree(child, ino);
        else {
            FILE *f = fopen(child, "rb");
            if (!f) die("cannot open", child);
            uint8_t *data = malloc((size_t)st.st_size + 1);
            if (fread(data, 1, (size_t)st.st_size, f) != (size_t)st.st_size) die("short read", child);
            fclose(f);
            cino = alloc_inode();
            inode(cino)->type = HBFS_FILE;
            inode(cino)->links = 1;
            inode(cino)->mtime = (uint32_t)st.st_mtime;
            write_data(cino, data, (size_t)st.st_size);
            free(data);
        }
        ents[count].inode = cino;
        strcpy(ents[count++].name, name);
        free(names[i]);
    }
    free(names);
    write_data(ino, (const uint8_t *)ents, (size_t)count * sizeof *ents);
    free(ents);
    return ino;
}

int main(int argc, char **argv) {
    if (argc != 4) { fprintf(stderr, "usage: mkhobbyfs IMAGE BLOCKS SRCDIR\n"); return 2; }
    uint32_t total = (uint32_t)atoi(argv[2]);
    if (total < 64 || total > HBFS_BLOCK * 8) die("BLOCKS must be 64..8192", NULL);
    img = calloc(total, HBFS_BLOCK);
    sb = (struct hbfs_super *)block(1);
    *sb = (struct hbfs_super){HBFS_MAGIC, HBFS_BLOCK, total, 256, 2, 3, 4, 256 / HBFS_INODES_PER_BLOCK, 0, total, 256};
    sb->data_start = sb->inode_table + sb->inode_table_blocks;
    for (uint32_t b = 0; b < sb->data_start; b++) alloc_block();   /* metadata blocks are "used" */
    alloc_inode();                                                  /* inode 0 means "no inode" */
    if (add_tree(argv[3], 0) != HBFS_ROOT) die("root must be inode 1", NULL);
    FILE *out = fopen(argv[1], "wb");
    if (!out || fwrite(img, HBFS_BLOCK, total, out) != total || fclose(out)) die("cannot write", argv[1]);
    printf("%s: %u blocks (%u free), %u inodes (%u free)\n", argv[1], total, sb->free_blocks, sb->inode_count,
           sb->free_inodes);
    return 0;
}
```

### Deep & Tricky

**DT-13. A C struct is not a file format: the compiler may insert padding between members and at the end (only "implementation-defined" alignment rules decide how much), and the byte order of multi-byte integers is the CPU's. Data that crosses a machine boundary — a disk image built on one machine and read on another, a network packet — must use fixed-width types, an explicit byte order, and a layout whose size is verified.** *(C11 §6.7.2.1 ¶15 and ¶17: "There may be unnamed padding within a structure object, but not at its beginning … There may be unnamed padding at the end"; alignment is implementation-defined, §6.2.8)* hobbyfs is little-endian only (both writer and reader are x86), and every on-disk struct is ordered so that no padding is needed — the `_Static_assert`s prove it on every build. The demo shows how silently a field order changes a size:

```c
// file: dt13/layout.c
#include <stdint.h>
#include <stdio.h>
#include <stddef.h>

struct naive  { uint16_t type; uint32_t size; uint16_t links; };   /* same fields, careless order */
struct sorted { uint16_t type; uint16_t links; uint32_t size; };
struct packed_naive { uint16_t type; uint32_t size; uint16_t links; } __attribute__((packed));

int main(void) {
    printf("naive:  %zu bytes (size at offset %zu)\n", sizeof(struct naive), offsetof(struct naive, size));
    printf("sorted: %zu bytes (size at offset %zu)\n", sizeof(struct sorted), offsetof(struct sorted, size));
    printf("packed: %zu bytes (size at offset %zu, misaligned)\n", sizeof(struct packed_naive),
           offsetof(struct packed_naive, size));
    return 0;
}
```

```console
$ cd dt13 && gcc -O2 -Wall -o layout layout.c && ./layout && gcc -m32 -O2 -o layout32 layout.c 2>/dev/null && ./layout32 || echo "(no 32-bit toolchain installed)"
naive:  12 bytes (size at offset 4)
sorted: 8 bytes (size at offset 4)
packed: 8 bytes (size at offset 2, misaligned)
(no 32-bit toolchain installed)
```

`packed` removes the padding but makes the 4-byte field misaligned, which costs speed on x86 and can fault on other CPUs (and taking its address is a trap of its own). Ordering fields from largest to smallest alignment, or as here in pairs that fill each 4-byte slot, avoids both. Linux's on-disk headers (`include/uapi/linux/`, ext4's `struct ext4_inode`) use explicitly sized, explicitly little-endian types (`__le32`) for the same reason.

## Unit 6.3 — Reading hobbyfs in the kernel

**One-line summary:** Mounting reads and checks the superblock. Reading an inode is reading the block that contains it. Reading file bytes at offset `off` means: block index `off / 1024`, look up its disk block through the direct or indirect pointers (`bmap`), read it, copy the needed part. Looking up a path walks it component by component from the root, searching each directory's entries. Each process gets a small **file-descriptor table**: a descriptor is an index into it, and each entry remembers the inode and the current offset.

```c
// file: new/kernel/fs.c
/* hobbyfs, read-only. All functions return a negative errno on failure. */
#include <proc.h>
#include <hobbyfs.h>

static struct hbfs_super sb;
static bool mounted;
uint64_t fs_block_reads;

static int read_block(uint32_t b, void *buf) {
    if (b >= sb.total_blocks && mounted) return -EIO;
    fs_block_reads++;
    return ata_read(b * 2, 2, buf) < 0 ? -EIO : 0;
}

int fs_mount(void) {
    uint8_t buf[HBFS_BLOCK];
    mounted = false;
    if (read_block(1, buf) < 0) return -EIO;
    memcpy(&sb, buf, sizeof sb);
    if (sb.magic != HBFS_MAGIC || sb.block_size != HBFS_BLOCK) return -EINVAL;
    if ((uint64_t)sb.total_blocks * 2 > ata_sectors) return -EINVAL;     /* the disk is smaller than claimed */
    mounted = true;
    return 0;
}

int fs_stat(uint32_t ino, struct hbfs_inode *out) {
    if (ino == 0 || ino >= sb.inode_count) return -ENOENT;
    uint8_t buf[HBFS_BLOCK];
    int r = read_block(sb.inode_table + ino / HBFS_INODES_PER_BLOCK, buf);
    if (r < 0) return r;
    memcpy(out, buf + (ino % HBFS_INODES_PER_BLOCK) * sizeof *out, sizeof *out);
    return out->type == HBFS_FREE ? -ENOENT : 0;
}

static int bmap(const struct hbfs_inode *ip, uint32_t n, uint32_t *out) {   /* file block n -> disk block */
    if (n < HBFS_NDIRECT) { *out = ip->direct[n]; return 0; }
    if (n >= HBFS_MAXBLOCKS || !ip->indirect) return -EIO;
    uint32_t ind[HBFS_NINDIRECT];
    int r = read_block(ip->indirect, ind);
    *out = ind[n - HBFS_NDIRECT];
    return r;
}

long fs_read(uint32_t ino, uint64_t off, void *buf, uint64_t n) {
    struct hbfs_inode ip;
    int r = fs_stat(ino, &ip);
    if (r < 0) return r;
    if (off >= ip.size) return 0;
    if (n > ip.size - off) n = ip.size - off;
    uint8_t blk[HBFS_BLOCK];
    uint64_t done = 0;
    while (done < n) {
        uint64_t pos = off + done, in_block = pos % HBFS_BLOCK;
        uint64_t chunk = HBFS_BLOCK - in_block < n - done ? HBFS_BLOCK - in_block : n - done;
        uint32_t b;
        if ((r = bmap(&ip, (uint32_t)(pos / HBFS_BLOCK), &b)) < 0 || (r = read_block(b, blk)) < 0) return r;
        memcpy((uint8_t *)buf + done, blk + in_block, chunk);
        done += chunk;
    }
    return (long)done;
}

int fs_dirent(uint32_t dir, uint32_t index, struct hbfs_dirent *out) {   /* 1: got entry, 0: end */
    long r = fs_read(dir, (uint64_t)index * sizeof *out, out, sizeof *out);
    if (r < 0) return (int)r;
    return r == sizeof *out;
}

int fs_lookup(const char *path) {                    /* absolute path -> inode number */
    if (!mounted) return -EIO;
    if (path[0] != '/') return -ENOENT;
    uint32_t ino = HBFS_ROOT;
    while (*path) {
        while (*path == '/') path++;
        if (!*path) break;
        char name[HBFS_NAME];
        size_t len = 0;
        while (path[len] && path[len] != '/') len++;
        if (len >= HBFS_NAME) return -ENAMETOOLONG;
        memcpy(name, path, len);
        name[len] = '\0';
        path += len;
        struct hbfs_inode ip;
        int r = fs_stat(ino, &ip);
        if (r < 0) return r;
        if (ip.type != HBFS_DIR) return -ENOTDIR;
        struct hbfs_dirent d;
        uint32_t i = 0, found = 0;
        while ((r = fs_dirent(ino, i++, &d)) == 1)
            if (d.inode && strncmp(d.name, name, HBFS_NAME) == 0) { found = d.inode; break; }
        if (r < 0) return r;
        if (!found) return -ENOENT;
        ino = found;
    }
    return (int)ino;
}
```

The task structure gains a file table and an argument string; error numbers and the new interfaces join `proc.h`:

```c
// file: new/include/proc.h
#pragma once
#include <mm.h>
#include <io16.h>

#define EIO 5
#define ENOENT 2
#define ESRCH 3
#define ENOEXEC 8
#define EBADF 9
#define ENOMEM 12
#define EFAULT 14
#define ENOTDIR 20
#define EISDIR 21
#define EINVAL 22
#define EMFILE 24
#define ENAMETOOLONG 36
#define ENOSYS 38

enum task_state { TASK_RUNNABLE, TASK_RUNNING, TASK_SLEEPING, TASK_BLOCKED, TASK_ZOMBIE };
struct task;
struct waitq { struct task *head; };

#define NFILES 8
struct file { bool used, dir; uint32_t ino; uint64_t off; };

struct task {
    uint64_t rsp;
    uint64_t *pml4;
    void *kstack;
    int pid;
    enum task_state state;
    char name[16];
    bool user, killed;
    int exit_code;
    uint64_t wake_tick, runtime_ticks, switches;
    struct waitq exit_wait;
    struct task *next;
    struct task *all_next;
    struct file files[NFILES];    /* new: descriptors 3..7 (0-2 are the console) */
    uint64_t entry;               /* new: user entry point */
};
#define KSTACK_SIZE 16384

static inline uint64_t irq_save(void) {
    uint64_t f;
    __asm__ volatile("pushfq; pop %0; cli" : "=r"(f) : : "memory");
    return f;
}
static inline void irq_restore(uint64_t f) { if (f & 0x200) sti(); }

extern struct task *current;
extern volatile bool need_resched;
extern bool preempt_enabled;
void sched_init(void);
struct task *kthread_create(const char *name, void (*fn)(void *), void *arg);
struct task *task_create(const char *name, void (*fn)(void *), void *arg, uint64_t *pml4, bool user);
void schedule(void);
void yield(void);
void sleep_ticks(uint64_t n);
void wait_on(struct waitq *q);
void wake_all(struct waitq *q);
_Noreturn void task_exit(int code);
int  task_join(struct task *t);
void sched_tick(void);
struct task *task_first(void);
struct task *task_find(int pid);
const char *task_state_name(enum task_state s);

void gdt_init(void);
void tss_set_rsp0(uint64_t rsp0);
void syscall_dispatch(struct trap_frame *f);

/* disk, file system, programs */
int  ata_init(void);
int  ata_read(uint32_t lba, uint8_t count, void *buf);
int  ata_write(uint32_t lba, uint8_t count, const void *buf);
extern uint64_t ata_sectors, ata_reads, ata_writes, fs_block_reads;
extern char ata_model[41];
struct hbfs_inode;
struct hbfs_dirent;
int  fs_mount(void);
int  fs_lookup(const char *path);
int  fs_stat(uint32_t ino, struct hbfs_inode *out);
long fs_read(uint32_t ino, uint64_t off, void *buf, uint64_t n);
int  fs_dirent(uint32_t dir, uint32_t index, struct hbfs_dirent *out);
long elf_load(uint64_t *pml4, uint32_t ino, uint64_t *entry);
struct task *process_spawn(const char *path, const char *args, long *err);
long console_read_line(char *buf, uint64_t n);

#define USER_CODE_MIN    0x40000000UL       /* below: the shared, kernel-only identity map */
#define USER_STACK_TOP   0x80000000UL
#define USER_STACK_PAGES 4
#define USER_ARGS        (USER_STACK_TOP - 256)   /* the argument string lives here */
```

## Unit 6.4 — The ELF loader and file system calls

**One-line summary:** An ELF executable starts with a header (magic `\x7fELF`, class, machine, entry point, where the **program headers** are). Each program header of type `PT_LOAD` says: map `p_memsz` bytes at virtual address `p_vaddr`, fill the first `p_filesz` of them from file offset `p_offset`, and zero the rest (that is the `.bss`). The loader does exactly that, page by page, into the new process's address space — after checking every number, because the file comes from the disk and the disk is not trusted (DT-14).

```c
// file: new/kernel/elf.c
#include <proc_extra.h>

struct elf64_ehdr {
    uint8_t ident[16];
    uint16_t type, machine;
    uint32_t version;
    uint64_t entry, phoff, shoff;
    uint32_t flags;
    uint16_t ehsize, phentsize, phnum, shentsize, shnum, shstrndx;
};
struct elf64_phdr {
    uint32_t type, flags;
    uint64_t offset, vaddr, paddr, filesz, memsz, align;
};
_Static_assert(sizeof(struct elf64_ehdr) == 64 && sizeof(struct elf64_phdr) == 56, "ELF64 layouts");
#define PT_LOAD 1

static uint64_t frame_of(uint64_t *pml4, uint64_t virt) {        /* map a zeroed page if needed; return its frame */
    uint64_t phys;
    if (vmm_translate_in(pml4, virt, &phys)) return phys;
    uint64_t frame = pmm_alloc();
    if (!frame) return 0;
    memset((void *)frame, 0, PAGE_SIZE);
    if (!vmm_map_in(pml4, virt, frame, PTE_WRITABLE | PTE_USER)) { pmm_free(frame); return 0; }
    return frame;
}

long elf_load(uint64_t *pml4, uint32_t ino, uint64_t *entry) {
    struct elf64_ehdr eh;
    if (fs_read(ino, 0, &eh, sizeof eh) != sizeof eh) return -ENOEXEC;
    if (memcmp(eh.ident, "\x7f" "ELF", 4) || eh.ident[4] != 2 || eh.ident[5] != 1 ||   /* 64-bit, little-endian */
        eh.type != 2 || eh.machine != 62 || eh.phentsize != sizeof(struct elf64_phdr) || eh.phnum > 16)
        return -ENOEXEC;
    bool entry_ok = false;
    for (int i = 0; i < eh.phnum; i++) {
        struct elf64_phdr ph;
        if (fs_read(ino, eh.phoff + (uint64_t)i * sizeof ph, &ph, sizeof ph) != sizeof ph) return -ENOEXEC;
        if (ph.type != PT_LOAD) continue;
#ifndef TRUST_ELF                                      /* experiment switch for DT-14 */
        if (ph.filesz > ph.memsz || ph.vaddr < USER_CODE_MIN || ph.memsz > 64 * 1024 * 1024 ||
            ph.vaddr + ph.memsz > USER_ARGS - USER_STACK_PAGES * PAGE_SIZE || ph.vaddr + ph.memsz < ph.vaddr)
            return -ENOEXEC;
#endif
        for (uint64_t va = ALIGN_DOWN(ph.vaddr, PAGE_SIZE); va < ph.vaddr + ph.memsz; va += PAGE_SIZE) {
            uint64_t frame = frame_of(pml4, va);
            if (!frame) return -ENOMEM;
            /* the part of [vaddr, vaddr + filesz) that falls in this page comes from the file */
            uint64_t lo = va > ph.vaddr ? va : ph.vaddr;
            uint64_t hi = va + PAGE_SIZE < ph.vaddr + ph.filesz ? va + PAGE_SIZE : ph.vaddr + ph.filesz;
            if (lo < hi && fs_read(ino, ph.offset + (lo - ph.vaddr), (void *)(frame + (lo - va)), hi - lo) != (long)(hi - lo))
                return -ENOEXEC;                       /* the rest of the page stays zero: .bss */
        }
        if (eh.entry >= ph.vaddr && eh.entry < ph.vaddr + ph.memsz) entry_ok = true;
    }
#ifndef TRUST_ELF
    if (!entry_ok) return -ENOEXEC;
#else
    (void)entry_ok;
#endif
    *entry = eh.entry;
    return 0;
}
```

`vmm_translate_in` is the per-address-space version of `vmm_translate`; `mm.h` and `vmm.c` gain it (two lines each):

```c
// file: new/include/mm_extra.h
#pragma once
#include <mm.h>
bool vmm_translate_in(uint64_t *pml4, uint64_t virt, uint64_t *phys);
```

```c
// file: new/kernel/vmm_extra.c
#include <mm.h>
#include <mm_extra.h>

/* Like vmm_translate, for any address space (the walk in vmm.c is static, so repeat the four levels here). */
bool vmm_translate_in(uint64_t *pml4, uint64_t virt, uint64_t *phys) {
    uint64_t *t = pml4;
    for (int level = 3; level >= 0; level--) {
        uint64_t e = t[(virt >> (12 + 9 * level)) & 511];
        if (!(e & PTE_PRESENT)) return false;
        if (level == 0 || (e & PTE_HUGE)) {
            uint64_t page = 1UL << (12 + 9 * level);
            *phys = (e & PTE_ADDR & ~(page - 1)) | (virt & (page - 1));
            return true;
        }
        t = (uint64_t *)(e & PTE_ADDR);
    }
    return false;
}
```

(Rather than reprint `mm.h` once more, the declaration lives in a small extra header; files that need it include `proc_extra.h`, which pulls in both.)

```c
// file: new/include/proc_extra.h
#pragma once
#include <proc.h>
#include <mm_extra.h>
```

Now process creation reads the program from the disk:

```c
// file: new/kernel/process.c
#include <proc_extra.h>

extern void enter_user(uint64_t rip, uint64_t rsp);

static void user_start(void *entry) {
    enter_user((uint64_t)entry, USER_ARGS);    /* the stack starts just below the argument string */
}

struct task *process_spawn(const char *path, const char *args, long *err) {
    int ino = fs_lookup(path);
    if (ino < 0) { *err = ino; return NULL; }
    uint64_t *pml4 = vmm_new_space();
    if (!pml4) { *err = -ENOMEM; return NULL; }
    uint64_t entry;
    long r = elf_load(pml4, (uint32_t)ino, &entry);
    uint64_t top = 0;
    for (int i = 1; r == 0 && i <= USER_STACK_PAGES; i++) {
        uint64_t frame = pmm_alloc();
        if (!frame) { r = -ENOMEM; break; }
        memset((void *)frame, 0, PAGE_SIZE);
        vmm_map_in(pml4, USER_STACK_TOP - (uint64_t)i * PAGE_SIZE, frame, PTE_WRITABLE | PTE_USER);
        if (i == 1) top = frame;
    }
    if (r < 0) { vmm_destroy_space(pml4); *err = r; return NULL; }
    size_t n = 0;                                        /* copy the arguments into the top stack page */
    for (; args && args[n] && n < 255; n++) ((char *)top)[PAGE_SIZE - 256 + n] = args[n];
    const char *base = path;
    for (const char *p = path; *p; p++) if (*p == '/') base = p + 1;
    struct task *t = task_create(base, user_start, (void *)entry, pml4, true);   /* everything set before it can run */
    t->entry = entry;                                   /* for information only (ps, debugging) */
    return t;
}
```

The entry point travels to the new task as the thread function's argument, which is stored on its stack before `task_create` makes it runnable. Setting it in the task structure *after* `task_create` would be a race: with preemption, the new task could start before the assignment.

The line discipline for console input — echo, backspace, and returning a whole line at a time, like a Unix terminal in "canonical" mode:

```c
// file: new/kernel/tty.c
#include <proc.h>

long console_read_line(char *buf, uint64_t n) {
    uint64_t len = 0;
    while (len + 1 < n) {
        int c = serial_getc();
        if (c == '\r') c = '\n';
        if (c == 0x7F || c == '\b') { if (len) { len--; kprintf("\b \b"); } continue; }
        if (c == 0x04 && len == 0) return 0;              /* Ctrl-D on an empty line: end of input */
        if (c != '\n' && (c < 0x20 || c >= 0x7F)) continue;
        buf[len++] = (char)c;
        kprintf("%c", c);
        if (c == '\n') break;
    }
    return (long)len;
}
```

The new system calls. Every pointer is validated, every descriptor range-checked, and strings are copied in byte by byte with a length limit:

```c
// file: new/kernel/syscall.c
#include <proc_extra.h>
#include <hobbyfs.h>

enum { SYS_WRITE, SYS_EXIT, SYS_GETPID, SYS_SLEEP, SYS_YIELD, SYS_TICKS,
       SYS_OPEN, SYS_READ, SYS_CLOSE, SYS_READDIR, SYS_SPAWN, SYS_WAIT };

struct u_dirent { uint32_t ino, size, type; char name[HBFS_NAME]; };   /* what readdir gives user space */

static long copy_string_in(char *dst, uint64_t src, size_t max) {
    for (size_t i = 0; i < max; i++) {
        if ((i == 0 || (src + i) % PAGE_SIZE == 0) && !user_range_ok(current->pml4, src + i, 1, false)) return -EFAULT;
        if (!(dst[i] = ((const char *)src)[i])) return (long)i;
    }
    return -ENAMETOOLONG;
}

static struct file *fd_get(uint64_t fd) {
    if (fd < 3 || fd >= NFILES || !current->files[fd].used) return NULL;
    return &current->files[fd];
}

static long sys_write(uint64_t buf, uint64_t len) {
    if (len > 4096) return -EINVAL;
    if (!user_range_ok(current->pml4, buf, len, false)) return -EFAULT;
    const char *p = (const char *)buf;
    for (uint64_t i = 0; i < len; i++) { serial_putc(p[i]); vga_putc(p[i]); }
    return (long)len;
}

static long sys_open(uint64_t upath) {
    char path[128];
    long r = copy_string_in(path, upath, sizeof path);
    if (r < 0) return r;
    int ino = fs_lookup(path);
    if (ino < 0) return ino;
    struct hbfs_inode ip;
    if ((r = fs_stat((uint32_t)ino, &ip)) < 0) return r;
    for (int fd = 3; fd < NFILES; fd++) {
        if (!current->files[fd].used) {
            current->files[fd] = (struct file){true, ip.type == HBFS_DIR, (uint32_t)ino, 0};
            return fd;
        }
    }
    return -EMFILE;
}

static long sys_read(uint64_t fd, uint64_t buf, uint64_t n) {
    if (n > 65536) n = 65536;
    if (!user_range_ok(current->pml4, buf, n, true)) return -EFAULT;
    if (fd == 0) return console_read_line((char *)buf, n);
    struct file *f = fd_get(fd);
    if (!f) return -EBADF;
    if (f->dir) return -EISDIR;
    long r = fs_read(f->ino, f->off, (void *)buf, n);
    if (r > 0) f->off += (uint64_t)r;
    return r;
}

static long sys_readdir(uint64_t fd, uint64_t index, uint64_t out) {
    struct file *f = fd_get(fd);
    if (!f) return -EBADF;
    if (!f->dir) return -ENOTDIR;
    if (!user_range_ok(current->pml4, out, sizeof(struct u_dirent), true)) return -EFAULT;
    struct hbfs_dirent d;
    int r;
    do { r = fs_dirent(f->ino, (uint32_t)index++, &d); } while (r == 1 && d.inode == 0);   /* skip empty slots */
    if (r <= 0) return r;
    struct hbfs_inode ip;
    if ((r = fs_stat(d.inode, &ip)) < 0) return r;
    struct u_dirent *u = (struct u_dirent *)out;
    u->ino = d.inode;
    u->size = ip.size;
    u->type = ip.type;
    memcpy(u->name, d.name, HBFS_NAME);
    return 1;
}

static long sys_spawn(uint64_t upath, uint64_t uargs) {
    char path[128], args[128] = "";
    long r = copy_string_in(path, upath, sizeof path);
    if (r < 0) return r;
    if (uargs && (r = copy_string_in(args, uargs, sizeof args)) < 0) return r;
    struct task *t = process_spawn(path, args, &r);
    return t ? t->pid : r;
}

static long sys_wait(uint64_t pid) {
    struct task *t = task_find((int)pid);
    if (!t || !t->user || t == current) return -ESRCH;
    return task_join(t);
}

void syscall_dispatch(struct trap_frame *f) {
    long r;
    switch (f->rax) {
    case SYS_WRITE:   r = sys_write(f->rdi, f->rsi); break;
    case SYS_EXIT:    task_exit((int)f->rdi);
    case SYS_GETPID:  r = current->pid; break;
    case SYS_SLEEP:   sleep_ticks(f->rdi * timer_hz / 1000 + 1); r = 0; break;
    case SYS_YIELD:   yield(); r = 0; break;
    case SYS_TICKS:   r = (long)ticks; break;
    case SYS_OPEN:    r = sys_open(f->rdi); break;
    case SYS_READ:    sti(); r = sys_read(f->rdi, f->rsi, f->rdx); break;   /* may block for a long time */
    case SYS_CLOSE:   { struct file *fl = fd_get(f->rdi); r = fl ? (fl->used = false, 0) : -EBADF; break; }
    case SYS_READDIR: r = sys_readdir(f->rdi, f->rsi, f->rdx); break;
    case SYS_SPAWN:   sti(); r = sys_spawn(f->rdi, f->rsi); break;
    case SYS_WAIT:    r = sys_wait(f->rdi); break;
    default:          r = -ENOSYS;
    }
    f->rax = (uint64_t)r;
}
```

`read` and `spawn` turn interrupts back on (`sti`) before doing slow work: the `int 0x80` gate cleared IF on entry, and a disk read or a wait for a keypress with interrupts off would freeze the timer. (The ATA driver disables them again around each command.)

The user-side library gains the new calls; programs are now ordinary ELF files on the disk:

```c
// file: new/user/ulib.h
#pragma once
typedef unsigned long size_t;

static inline long syscall3(long n, long a, long b, long c) {
    long r;
    __asm__ volatile("int $0x80" : "=a"(r) : "a"(n), "D"(a), "S"(b), "d"(c) : "memory");
    return r;
}
#define syscall2(n, a, b) syscall3(n, a, b, 0)
static inline long u_write(const void *buf, size_t n) { return syscall2(0, (long)buf, (long)n); }
static inline _Noreturn void u_exit(int code) { syscall2(1, code, 0); for (;;) { } }
static inline long u_getpid(void) { return syscall2(2, 0, 0); }
static inline void u_sleep_ms(long ms) { syscall2(3, ms, 0); }
static inline long u_ticks(void) { return syscall2(5, 0, 0); }
static inline long u_open(const char *path) { return syscall2(6, (long)path, 0); }
static inline long u_read(long fd, void *buf, size_t n) { return syscall3(7, fd, (long)buf, (long)n); }
static inline long u_close(long fd) { return syscall2(8, fd, 0); }
struct u_dirent { unsigned int ino, size, type; char name[28]; };
static inline long u_readdir(long fd, long index, struct u_dirent *d) { return syscall3(9, fd, index, (long)d); }
static inline long u_spawn(const char *path, const char *args) { return syscall2(10, (long)path, (long)args); }
static inline long u_wait(long pid) { return syscall2(11, pid, 0); }
#define U_ARGS ((const char *)0x7FFFFF00)     /* the kernel puts the argument string here */

static inline size_t u_strlen(const char *s) { size_t n = 0; while (s[n]) n++; return n; }
static inline int u_streq(const char *a, const char *b) { while (*a && *a == *b) { a++; b++; } return *a == *b; }
static inline void u_puts(const char *s) { u_write(s, u_strlen(s)); }
static inline void u_putnum(long v) {
    char b[24];
    int i = 23;
    int neg = v < 0;
    unsigned long u = neg ? -(unsigned long)v : (unsigned long)v;
    b[i] = 0;
    do { b[--i] = (char)('0' + u % 10); u /= 10; } while (u);
    if (neg) b[--i] = '-';
    u_puts(b + i);
}
static inline void u_puthex(unsigned long v) {
    char b[19] = "0x";
    for (int i = 0; i < 16; i++) b[2 + i] = "0123456789abcdef"[(v >> (60 - 4 * i)) & 15];
    b[18] = 0;
    u_puts(b);
}

int main(void);
__attribute__((section(".text.start"), used)) void _start(void) { u_exit(main()); }
```

```c
// file: new/user/ls.c
#include "ulib.h"

int main(void) {
    const char *path = U_ARGS[0] ? U_ARGS : "/";
    long fd = u_open(path);
    if (fd < 0) { u_puts("ls: cannot open "); u_puts(path); u_puts("\n"); return 1; }
    struct u_dirent d;
    for (long i = 0; u_readdir(fd, i, &d) == 1; i++) {
        u_puts(d.type == 2 ? "d " : "- ");
        char num[12];
        int k = 11;
        num[k] = 0;
        unsigned v = d.size;
        do { num[--k] = (char)('0' + v % 10); v /= 10; } while (v);
        while (k > 4) num[--k] = ' ';
        u_puts(num + k);
        u_puts("  ");
        u_puts(d.name);
        u_puts("\n");
    }
    u_close(fd);
    return 0;
}
```

```c
// file: new/user/cat.c
#include "ulib.h"

static char buf[2048];

int main(void) {
    long fd = u_open(U_ARGS);
    if (fd < 0) { u_puts("cat: "); u_puts(U_ARGS); u_puts(": error "); u_putnum(fd); u_puts("\n"); return 1; }
    long n;
    while ((n = u_read(fd, buf, sizeof buf)) > 0) u_write(buf, (size_t)n);
    u_close(fd);
    return n < 0 ? 1 : 0;
}
```

```c
// file: new/user/wc.c
#include "ulib.h"

static char buf[1024];

int main(void) {
    long fd = u_open(U_ARGS);
    if (fd < 0) { u_puts("wc: cannot open\n"); return 1; }
    long lines = 0, words = 0, bytes = 0, n;
    int in_word = 0;
    while ((n = u_read(fd, buf, sizeof buf)) > 0) {
        for (long i = 0; i < n; i++) {
            char c = buf[i];
            bytes++;
            if (c == '\n') lines++;
            int space = c == ' ' || c == '\n' || c == '\t';
            if (!space && !in_word) words++;
            in_word = !space;
        }
    }
    u_putnum(lines); u_puts(" "); u_putnum(words); u_puts(" "); u_putnum(bytes); u_puts(" "); u_puts(U_ARGS); u_puts("\n");
    return 0;
}
```

The build now produces ELF files, a host tool, a root directory, and a disk image. The image also contains `evil`: a copy of `hello` whose first program header has been edited to load at a kernel address (DT-14).

```python
# file: new/tools/make_evil.py
"""Copy an ELF and point its first PT_LOAD at the kernel heap: a hostile executable."""
import struct, sys
data = bytearray(open(sys.argv[1], "rb").read())
phoff, = struct.unpack_from("<Q", data, 32)
for i in range(struct.unpack_from("<H", data, 56)[0]):
    off = phoff + i * 56
    if struct.unpack_from("<I", data, off)[0] == 1:          # PT_LOAD
        struct.pack_into("<Q", data, off + 16, 0xFFFF800000000000)   # p_vaddr
        break
open(sys.argv[2], "wb").write(data)
```

```text
# file: new/Makefile
CC      := gcc
CFLAGS  := -std=c11 -ffreestanding -fno-stack-protector -fno-pic -mno-red-zone -mno-sse -mno-mmx -mno-80387 \
           -mcmodel=small -fno-omit-frame-pointer -O2 -g -Wall -Wextra -Werror -Iinclude $(EXTRA_CFLAGS)
UCFLAGS := -std=c11 -ffreestanding -fno-stack-protector -fno-pic -mno-sse -mno-mmx -mno-80387 -O2 -Wall -Wextra -Werror
LDFLAGS := -n -T linker.ld -nostdlib -z noexecstack -z max-page-size=0x1000 --no-warn-rwx-segments
SRCS_C  := $(wildcard kernel/*.c)
SRCS_S  := $(wildcard arch/*.asm)
OBJS    := $(SRCS_S:.asm=.o) $(SRCS_C:.c=.o)
USER    := $(basename $(notdir $(wildcard user/*.c)))
CMDLINE ?=
TIMEOUT ?= 60
QEMU    := qemu-system-x86_64 -serial stdio -display none -no-reboot -m 128M -boot d \
           -drive file=disk.img,format=raw,index=0,media=disk \
           -device isa-debug-exit,iobase=0xf4,iosize=0x04 $(QEMU_EXTRA)

all: kernel.elf disk.img

kernel.elf: $(OBJS) linker.ld
	ld $(LDFLAGS) -o $@ $(OBJS)

%.o: %.asm
	nasm -f elf64 -g $< -o $@
%.o: %.c $(wildcard include/*.h)
	$(CC) $(CFLAGS) -c $< -o $@

user/%.elf: user/%.c user/ulib.h user/user.ld
	$(CC) $(UCFLAGS) -c $< -o user/$*.o
	ld -T user/user.ld -nostdlib -z noexecstack -z max-page-size=0x1000 --no-warn-rwx-segments -o $@ user/$*.o

tools/mkhobbyfs: tools/mkhobbyfs.c include/hobbyfs.h
	gcc -std=c11 -D_DEFAULT_SOURCE -O2 -Wall -Wextra -Werror -Iinclude -o $@ $<

disk.img: tools/mkhobbyfs $(USER:%=user/%.elf) tools/make_evil.py
	rm -rf rootfs && mkdir -p rootfs/bin rootfs/etc rootfs/docs
	for p in $(USER); do cp user/$$p.elf rootfs/bin/$$p; done
	python3 tools/make_evil.py rootfs/bin/hello rootfs/bin/evil
	printf 'Welcome to hobby os.\nPrograms live in /bin; try: ls /bin\n' > rootfs/etc/motd
	seq 1 5000 > rootfs/docs/numbers.txt
	cp include/hobbyfs.h rootfs/docs/
	tools/mkhobbyfs $@ 8192 rootfs

os.iso: kernel.elf
	rm -rf iso && mkdir -p iso/boot/grub && cp kernel.elf iso/boot/
	printf 'set timeout=0\nset default=0\nmenuentry "hobby os" {\n  multiboot2 /boot/kernel.elf %s\n  boot\n}\n' \
	    "$(CMDLINE)" > iso/boot/grub/grub.cfg
	grub-mkrescue -o $@ iso > /dev/null 2>&1

run: kernel.elf disk.img
	rm -f os.iso && $(MAKE) -s os.iso CMDLINE="$(CMDLINE)"
	timeout $(TIMEOUT) $(QEMU) -cdrom os.iso; status=$$?; echo "[qemu exit status $$status]"

clean:
	rm -rf $(OBJS) kernel.elf os.iso iso grub.cfg user/*.o user/*.elf user/*.bin tools/mkhobbyfs rootfs disk.img
.PHONY: all run clean os.iso
.PRECIOUS: user/%.elf
```

`kmain` mounts the disk and, if the command line says `init=/bin/...`, runs that program as the first process — the way Linux runs `/sbin/init`:

```c
// file: new/kernel/main.c
#include <proc.h>

extern const struct ktest __ktests_start[], __ktests_end[];

__attribute__((weak)) void shell_run(void) { kprintf("no shell in this build\n"); }

static void run_test(const char *name) {
    for (const struct ktest *t = __ktests_start; t < __ktests_end; t++) {
        if (strncmp(t->name, name, 64) == 0) {
            kprintf("== test %s ==\n", t->name);
            t->fn();
            kprintf("== test %s passed ==\n", t->name);
            return;
        }
    }
    kprintf("unknown test '%s'; available:", name);
    for (const struct ktest *t = __ktests_start; t < __ktests_end; t++) kprintf(" %s", t->name);
    kprintf("\n");
}

void kmain(uint32_t magic, uintptr_t info) {
    serial_init();
    vga_clear();
    multiboot2_parse(magic, info);
    gdt_init();
    idt_init();
    pmm_init();
    vmm_init();
    sched_init();
    kprintf("hobby os: %lu MiB usable, command line \"%s\"\n", usable_ram_bytes >> 20, kernel_cmdline);
    if (ata_init() == 0) kprintf("disk: \"%s\", %lu sectors (%lu KiB)\n", ata_model, ata_sectors, ata_sectors / 2);
    int r = fs_mount();
    kprintf("hobbyfs: %s\n", r == 0 ? "mounted" : "mount failed");
    pic_init();
    timer_init(100);
    serial_enable_rx_irq();
    sti();
    char arg[32];
    if (cmdline_get("test", arg, sizeof arg)) run_test(arg);
    else if (cmdline_get("init", arg, sizeof arg)) {
        long err;
        struct task *t = process_spawn(arg, "", &err);
        if (!t) kprintf("cannot start %s: error %ld\n", arg, err);
        else kprintf("[init %s exited with status %d]\n", arg, task_join(t));
    } else shell_run();
    qemu_exit(0x10);
}
```

The earlier tests and the kernel shell's `run` command used embedded programs; they now start programs from the disk:

```c
// file: new/kernel/test_user.c
#include <proc.h>

static int run(const char *path, const char *args) {
    long err;
    struct task *t = process_spawn(path, args, &err);
    if (!t) { kprintf("[%s: spawn failed, error %ld]\n", path, err); return (int)err; }
    int code = task_join(t);
    kprintf("[%s exited with status %d]\n", path, code);
    return code;
}

KTEST(user) {
    KASSERT(run("/bin/hello", "") == 7);
    KASSERT(run("/bin/badptr", "") == 0);
    KASSERT(run("/bin/crash", "") == -11);
    KASSERT(run("/bin/priv", "") == -11);
}

KTEST(isolation) {
    long err;
    struct task *a = process_spawn("/bin/counter", "", &err), *b = process_spawn("/bin/counter", "", &err);
    int ca = task_join(a), cb = task_join(b);
    kprintf("pid %d exited with %d, pid %d exited with %d\n", a->pid, ca, b->pid, cb);
    KASSERT(ca == 4 * a->pid && cb == 4 * b->pid);
}

KTEST(fs) {
    uint64_t reads0 = fs_block_reads;
    KASSERT(run("/bin/ls", "/") == 0);
    KASSERT(run("/bin/ls", "/docs") == 0);
    KASSERT(run("/bin/cat", "/etc/motd") == 0);
    KASSERT(run("/bin/wc", "/docs/numbers.txt") == 0);
    KASSERT(run("/bin/cat", "/no/such/file") == 1);
    KASSERT(run("/bin/nosuch", "") == -ENOENT);
    kprintf("block reads: %lu\n", fs_block_reads - reads0);
}

KTEST(elf) {
    KASSERT(run("/etc/motd", "") == -ENOEXEC);          /* not an ELF file at all */
    KASSERT(run("/bin/evil", "") == -ENOEXEC);          /* ELF, but it wants to load into the kernel */
}
```

```c
// file: new/kernel/cmd_proc.c
#include <shell.h>
#include <proc.h>

static int parse_int(const char *s) {
    int v = 0;
    while (*s >= '0' && *s <= '9') v = v * 10 + (*s++ - '0');
    return v;
}

SHELL_CMD(run, "run a program from the disk: run /bin/name [args] [&]") {
    char path[64];
    size_t n = 0;
    while (*arg && *arg != ' ' && n + 1 < sizeof path) path[n++] = *arg++;
    path[n] = '\0';
    while (*arg == ' ') arg++;
    char args[64];
    n = 0;
    while (*arg && *arg != '&' && n + 1 < sizeof args) args[n++] = *arg++;
    while (n && args[n - 1] == ' ') n--;
    args[n] = '\0';
    bool background = *arg == '&';
    long err;
    struct task *t = process_spawn(path, args, &err);
    if (!t) { kprintf("run: %s: error %ld\n", path, err); return; }
    if (background) { kprintf("[%d] %s started in the background\n", t->pid, t->name); return; }
    kprintf("[%s exited with status %d]\n", t->name, task_join(t));
}

SHELL_CMD(ps, "list tasks") {
    (void)arg;
    kprintf("  PID  STATE     KIND    CPU(ticks)  SWITCHES  NAME\n");
    for (struct task *t = task_first(); t; t = t->all_next) {
        if (t->state == TASK_ZOMBIE) kprintf("%5d  exit %-4d %-7s %10lu  %8lu  %s\n", t->pid, t->exit_code,
                                             t->user ? "user" : "kernel", t->runtime_ticks, t->switches, t->name);
        else kprintf("%5d  %-9s %-7s %10lu  %8lu  %s\n", t->pid, task_state_name(t->state),
                     t->user ? "user" : "kernel", t->runtime_ticks, t->switches, t->name);
    }
}

SHELL_CMD(kill, "kill a user process: kill <pid>") {
    int pid = parse_int(arg);
    uint64_t f = irq_save();
    struct task *t = task_find(pid);
    if (!t || !t->user || t->state == TASK_ZOMBIE) {
        irq_restore(f);
        kprintf("no live user process %d\n", pid);
        return;
    }
    t->killed = true;
    if (t->state == TASK_SLEEPING) t->wake_tick = 0;
    irq_restore(f);
    task_join(t);
    kprintf("[%d %s] killed, exit status %d\n", pid, t->name, t->exit_code);
}
```

```console
$ cd os && make -s 2>&1 | tail -3; ls -l disk.img user/hello.elf | awk '{print $5, $9}'
disk.img: 8192 blocks (8080 free), 256 inodes (238 free)
8388608 disk.img
5200 user/hello.elf
$ cd os && make -s run CMDLINE="test=fs"
hobby os: 127 MiB usable, command line "test=fs"
disk: "QEMU HARDDISK", 16384 sectors (8192 KiB)
hobbyfs: mounted
== test fs ==
d     160  .
d     160  ..
d     384  bin
d     128  docs
d      96  etc
[/bin/ls exited with status 0]
d     128  .
d     160  ..
-    1256  hobbyfs.h
-   23893  numbers.txt
[/bin/ls exited with status 0]
Welcome to hobby os.
Programs live in /bin; try: ls /bin
[/bin/cat exited with status 0]
5000 5000 23893 /docs/numbers.txt
[/bin/wc exited with status 0]
cat: /no/such/file: error -2
[/bin/cat exited with status 1]
[/bin/nosuch: spawn failed, error -2]
block reads: 348
== test fs passed ==
[qemu exit status 33]
$ cd os && make -s run CMDLINE="test=user" | tail -6
[/bin/crash exited with status -11]
priv: executing cli in ring 3
[pid 5 priv] killed: general protection at rip 0x0000000040000049
[/bin/priv exited with status -11]
== test user passed ==
[qemu exit status 33]
```

`numbers.txt` is 23,893 bytes — 24 blocks, so its last twelve blocks are reached through the indirect block; `wc` read it through 1 KiB `read` calls and got the right line count. Every block access goes to the disk: the block-read counter shows what a missing **buffer cache** costs (each `read` re-reads the inode, and blocks past the twelfth re-read the indirect block as well). Linux keeps file data in the **page cache** and metadata in the **buffer** and **dentry** caches for exactly this reason (Chapter 11).

### Deep & Tricky

**DT-14. An executable file is input like any other, and a loader that trusts its headers lets the file decide where the kernel writes. `p_vaddr`, `p_memsz`, `p_offset`, `p_filesz`, and the entry point must all be checked against the address space before anything is mapped or copied.** *(ELF-64 Object File Format / System V gABI, "Program Header": `p_filesz` "may be zero" and "must not be greater than" `p_memsz`; Linux `fs/binfmt_elf.c` rejects segments with `p_filesz > p_memsz` and addresses beyond `TASK_SIZE` before mapping)* Our `evil` program asks to be loaded at `0xFFFF800000000000` — the kernel heap, shared by every address space. The checking loader refuses with `-ENOEXEC`. Build the trusting variant and the result is worse than a refusal or a clean crash. The address `0xFFFF800000000000` is already mapped — it is the kernel heap — so the loader's "reuse the page if it is already mapped" step (`frame_of`) hands back the heap's own frame, and `fs_read` copies the program's bytes over the kernel's heap blocks. Shortly afterwards the kernel follows a heap pointer that is now program code and dies with a page fault at a nonsense address. A file on the disk has written into kernel memory.

```console
$ cd os && make -s run CMDLINE="test=elf" | tail -5
== test elf ==
[/etc/motd: spawn failed, error -8]
[/bin/evil: spawn failed, error -8]
== test elf passed ==
[qemu exit status 33]
$ rm -rf trusting && cp -r os trusting && cd trusting && make -s clean && make -s EXTRA_CFLAGS=-DTRUST_ELF && make -s run CMDLINE="test=elf" EXTRA_CFLAGS=-DTRUST_ELF 2>&1 | sed -n 5,8p
disk.img: 8192 blocks (8080 free), 256 inodes (238 free)
[/etc/motd: spawn failed, error -8]

EXCEPTION 14: page fault, error code 0x2
  faulting address (cr2) = 0x0000441f0f66fefb: page not present, write, kernel mode
```

### Pattern

**P-7. Define a persistent or shared data format once — in one header used by every program that reads or writes it — and give it a tool on the host: the image builder, a checker, a dumper. Test the reader in the kernel against images made by the host tool.** *(Practice)* `hobbyfs.h` is compiled into both `mkhobbyfs` and the kernel, so they cannot disagree about a field's offset; the host tool can be debugged with ordinary tools and sanitizers; and the image is a plain file you can inspect with `od`. Linux file systems follow the same split: `mkfs.ext4` and `e2fsck` (e2fsprogs) live outside the kernel and share the on-disk definitions. *Trade-off:* two implementations of one format must be kept in step — a format version or feature flags in the superblock (ext4's `s_feature_incompat`) are how real file systems survive that.

### Mastery check

1. **Predict.** What is the largest file hobbyfs can hold, and which part of the design sets the limit?
2. **Predict.** `fs_read` on a file of 5,000 bytes, starting at offset 1,000, for 3,000 bytes: which file blocks does it read?
3. **Predict.** Why does the ELF loader call `frame_of` (reuse a page if already mapped) instead of always mapping a new page?

## Chapter 6 Project — A user-space shell

**Goal:** move the shell out of the kernel. `/bin/sh` is an ordinary user program: it reads a line with `read(0, ...)` (the kernel does the echo and backspace), splits off the command, runs `/bin/<command>` with `spawn`, and waits for it. Built-ins: `help`, `exit`, and `motd`. Started as `init=/bin/sh`, it is the first process, and when it exits the machine halts — just as Linux panics when `init` dies.

```c
// file: proj/user/sh.c
#include "ulib.h"

static char line[128], path[64];

int main(void) {
    long fd = u_open("/etc/motd");
    if (fd >= 0) { long n; char buf[256]; while ((n = u_read(fd, buf, sizeof buf)) > 0) u_write(buf, (size_t)n); u_close(fd); }
    for (;;) {
        u_puts("$ ");
        long n = u_read(0, line, sizeof line);
        if (n <= 0) { u_puts("\n"); return 0; }          /* end of input */
        if (line[n - 1] == '\n') line[--n] = '\0';
        char *cmd = line;
        while (*cmd == ' ') cmd++;
        if (!*cmd) continue;
        char *args = cmd;
        while (*args && *args != ' ') args++;
        if (*args) *args++ = '\0';
        while (*args == ' ') args++;
        if (u_streq(cmd, "exit")) return 0;
        if (u_streq(cmd, "help")) { u_puts("built-ins: help exit; programs: ls /bin\n"); continue; }
        size_t k = 0;
        if (cmd[0] != '/') for (const char *p = "/bin/"; *p; p++) path[k++] = *p;
        for (const char *p = cmd; *p && k + 1 < sizeof path; p++) path[k++] = *p;
        path[k] = '\0';
        long t0 = u_ticks();
        long pid = u_spawn(path, args);
        if (pid < 0) {
            u_puts("sh: ");
            u_puts(cmd);
            u_puts(pid == -2 ? ": command not found\n" : pid == -8 ? ": not an executable\n" : ": cannot run\n");
            continue;
        }
        long status = u_wait(pid);
        if (status != 0) { u_puts("[exit status "); u_putnum(status); u_puts("]\n"); }
        if (u_ticks() - t0 > 100) { u_puts("[took "); u_putnum((u_ticks() - t0) * 10); u_puts(" ms]\n"); }
    }
}
```

```console
$ rm -rf final && cp -r os final && cp -r proj/. final/ && make -s -C final clean && make -s -C final 2>&1 | tail -2 && echo "final = os + proj"
disk.img: 8192 blocks (8073 free), 256 inodes (237 free)
final = os + proj
$ cd final && (sleep 4; for l in 'ls' 'ls /bin' 'cat /docs/nothing' 'wc /docs/numbers.txt' 'hello' 'crash' '/etc/motd' 'evil' 'frobnicate' 'counter' 'exit'; do printf "$l\n"; sleep 0.6; done) | make -s run CMDLINE="init=/bin/sh"
hobby os: 127 MiB usable, command line "init=/bin/sh"
disk: "QEMU HARDDISK", 16384 sectors (8192 KiB)
hobbyfs: mounted
Welcome to hobby os.
Programs live in /bin; try: ls /bin
$ ls
d     160  .
d     160  ..
d     416  bin
d     128  docs
d      96  etc
$ ls /bin
d     416  .
d     160  ..
-    5280  badptr
-    7232  cat
-    5472  counter
-    4776  crash
-    5200  evil
-    5200  hello
-    5224  ls
-    4752  priv
-    6496  sh
-    4616  spin
-    6432  wc
$ cat /docs/nothing
cat: /docs/nothing: error -2
[exit status 1]
$ wc /docs/numbers.txt
5000 5000 23893 /docs/numbers.txt
$ hello
hello from user space: pid 7, cs=0x0000000000000023 -> ring 3
[exit status 7]
$ crash
crash: reading kernel memory at 0x100000
[pid 8 crash] killed: page fault at rip 0x0000000040000049, address 0x0000000000100000 (error 0x5)
[exit status -11]
$ /etc/motd
sh: /etc/motd: not an executable
$ evil
sh: evil: not an executable
$ frobnicate
sh: frobnicate: command not found
$ counter
  pid 9: &counter=0x00000000400002d0 counter=9
  pid 9: &counter=0x00000000400002d0 counter=18
  pid 9: &counter=0x00000000400002d0 counter=27
  pid 9: &counter=0x00000000400002d0 counter=36
[exit status 36]
$ exit
[init /bin/sh exited with status 0]
[qemu exit status 33]
```

The whole session is user-mode code calling the kernel: the shell, `ls`, `cat`, `wc`, the crashing program (killed, and the shell reports its status), a data file the loader rejects, the hostile ELF, an unknown command. `exit` ends `init`, and the kernel shuts down.

### Review checklist

- `process_spawn` leaks the address space if `task_create` fails (it cannot, today — it asserts). What would a careful error path look like?
- `sys_read` re-reads the inode from disk on every call. Sketch a cache: what is its key, and what must invalidate an entry once the file system can be written?
- The shell passes arguments as one string at a fixed address. What does a real `execve` do instead, and what does the stack look like at `_start`?

## Answers to the Chapter 6 prediction questions

**6.4 Q1.** 12 direct + 256 indirect blocks of 1 KiB = 268 KiB (274,432 bytes). The limit is the single indirect block: one more level (a doubly-indirect block, as ext2 has) would add 256 × 256 blocks = 64 MiB.

**6.4 Q2.** Bytes 1,000–3,999 cover file blocks 0 (bytes 0–1023), 1, 2, and 3 (bytes 3072–4095): four blocks, all direct, the first and last only partly copied.

**6.4 Q3.** Two `PT_LOAD` segments may share a page — typically the end of the code and the start of the data, when the linker packs them tightly. Mapping a fresh page for the second segment would either hit "already mapped" or throw away what the first segment put there.

# Chapter 7 — Concurrency: Atomics, Locks, and More Than One CPU

*Goal: understand what changes when two things really happen at the same time. Until now, "concurrency" in our kernel meant interrupts and preemption on one CPU, and disabling interrupts was enough. In this chapter we boot the other CPUs of a multi-core machine, watch an innocent `counter++` lose updates, and build the tools that fix it: atomic instructions, spinlocks, interrupt-safe spinlocks, and fair ticket locks. We also meet the part of the x86 memory model that surprises everyone once.*

**What you will be able to do after this chapter**

- Explain why `volatile` does not make an operation atomic, and which instructions do.
- Write a spinlock with acquire/release ordering, and know when it must also disable interrupts.
- Find the CPUs in the ACPI tables and start them with the INIT–SIPI–SIPI sequence through a real-mode trampoline.
- Measure lock contention and fairness, and explain why a ticket lock is fair and a test-and-set lock is not.
- Recognise store→load reordering on x86 and fix it with a full fence.

The kernel continues from Chapter 6's `final/`. New and changed files are in `new/`; `os/` is Chapter 6's kernel plus `new/`.

```console
$ rm -rf os && cp -r ../ch06/final os && cp -r new/. os/ && make -s -C os clean && echo ready
ready
```

## Topic 7A — Atomic operations and memory order

## Unit 7.1 — Why `counter++` is not one step

**One-line summary:** `counter++` compiles to three steps — load, add, store. If two CPUs do them at the same time, both may load the same old value and both store old + 1: one update is lost. `volatile` only forces the compiler to perform each load and store; it does not make the three steps indivisible. An **atomic read-modify-write** instruction does: on x86, an instruction with the `lock` prefix (`lock add`, `lock xadd`, `lock cmpxchg`) or `xchg` with memory, which the CPU executes as one indivisible operation on the cache line. In C11 these are `atomic_fetch_add` and friends, or GCC's `__atomic` built-ins.

```c
// file: dt15/race.c
#include <pthread.h>
#include <stdio.h>
#include <stdatomic.h>

#define N 2000000
static volatile long plain;            /* volatile: every access really happens, but not atomically */
static atomic_long atomic;

static void *worker(void *arg) {
    (void)arg;
    for (int i = 0; i < N; i++) {
        plain++;                        /* load, add, store: three steps */
        atomic_fetch_add_explicit(&atomic, 1, memory_order_relaxed);   /* one locked instruction */
    }
    return NULL;
}

int main(void) {
    pthread_t t[2];
    for (int i = 0; i < 2; i++) pthread_create(&t[i], NULL, worker, NULL);
    for (int i = 0; i < 2; i++) pthread_join(t[i], NULL);
    printf("expected %d\nvolatile: %ld (%ld updates lost)\natomic:   %ld\n", 2 * N, plain, 2L * N - plain,
           atomic_load(&atomic));
    return 0;
}
```

```console
$ cd dt15 && gcc -O2 -pthread -o race race.c && ./race && objdump -d race | grep -A8 "<worker>:" | grep -E "add|lock"
expected 4000000
volatile: 3264559 (735441 updates lost)
atomic:   4000000
    1277:	48 83 c0 01          	add    $0x1,%rax
    1282:	f0 48 83 05 8d 2d 00 	lock addq $0x1,0x2d8d(%rip)        # 4018 <atomic>
```

The disassembly shows the difference: the volatile counter is a separate load and store around an `add`; the atomic one is a single `lock add` to memory.

### Deep & Tricky

**DT-15. In C, two threads accessing the same non-atomic object, at least one of them writing, without synchronisation, is a *data race*, and a data race is undefined behaviour — not merely "an unpredictable value". `volatile` does not change that: it constrains the compiler's treatment of accesses, not their atomicity or ordering with respect to other threads.** *(C11 §5.1.2.4 ¶25: "The execution of a program contains a data race if it contains two conflicting actions in different threads, at least one of which is not atomic, and neither happens before the other. Any such data race results in undefined behavior."; GCC manual, "Volatiles": volatile does not imply atomicity)* The lost updates above are the visible part. The invisible part is that the compiler is allowed to assume no other thread changes a non-atomic, non-volatile variable — it may keep it in a register for a whole loop. The Linux kernel predates C11 atomics and uses its own primitives (`READ_ONCE`, `WRITE_ONCE`, `atomic_t`, `smp_mb()`), documented in `Documentation/memory-barriers.txt`, but the rule is the same: shared data is touched only through those.

## Unit 7.2 — x86 reorders a store with a later load

**One-line summary:** Each x86 core has a **store buffer**: a store is written there first and reaches the cache (where other cores can see it) a little later. Meanwhile the core continues, and a later *load* from a different address may be satisfied before the earlier store is visible. So on x86 — which is otherwise strongly ordered — "store x, then load y" on one core and "store y, then load x" on another can *both* read the old values. A full fence (`mfence`, or any `lock`-prefixed instruction, or C11 `atomic_thread_fence(memory_order_seq_cst)`) drains the store buffer first. GCC compiles the C11 fence to `lock or $0, (%rsp)` — a locked no-op on the stack — because on most x86 CPUs it is cheaper than `mfence` and just as strong.

```c
// file: dt16/sb.c
/* The "store buffering" litmus test. Can r1 == 0 && r2 == 0? */
#include <pthread.h>
#include <stdio.h>
#include <stdatomic.h>
#include <string.h>

static volatile int x, y, r1, r2;
static atomic_int arrived, phase;           /* a two-thread barrier */
static int use_fence;
static long both_zero;
#define ROUNDS 200000

static void barrier(void) {
    int p = atomic_load(&phase);
    if (atomic_fetch_add(&arrived, 1) == 1) { atomic_store(&arrived, 0); atomic_fetch_add(&phase, 1); }
    else while (atomic_load(&phase) == p) { }
}

static void *thread_b(void *arg) {
    (void)arg;
    for (int i = 0; i < ROUNDS; i++) {
        barrier();                          /* round starts: x == y == 0 */
        y = 1;
        if (use_fence) atomic_thread_fence(memory_order_seq_cst);
        else __asm__ volatile("" ::: "memory");   /* stops the compiler, not the CPU */
        r2 = x;
        barrier();                          /* round ends */
        barrier();                          /* thread A has checked and reset */
    }
    return NULL;
}

int main(int argc, char **argv) {
    use_fence = argc > 1 && strcmp(argv[1], "fence") == 0;
    pthread_t b;
    pthread_create(&b, NULL, thread_b, NULL);
    for (int i = 0; i < ROUNDS; i++) {      /* the main thread is "thread A" */
        barrier();
        x = 1;
        if (use_fence) atomic_thread_fence(memory_order_seq_cst);
        else __asm__ volatile("" ::: "memory");
        r1 = y;
        barrier();
        if (r1 == 0 && r2 == 0) both_zero++;
        x = y = 0;
        barrier();
    }
    pthread_join(b, NULL);
    printf("%s: r1 == 0 && r2 == 0 in %ld of %d rounds\n", use_fence ? "with mfence" : "no fence   ", both_zero, ROUNDS);
    return 0;
}
```

```console
$ cd dt16 && gcc -O2 -pthread -o sb sb.c && ./sb && ./sb fence && objdump -d sb | grep -E "mfence|lock or" | sed "s/^ *//" | cut -c1-60
no fence   : r1 == 0 && r2 == 0 in 80321 of 200000 rounds
with mfence: r1 == 0 && r2 == 0 in 0 of 200000 rounds
119c:	f0 48 83 0c 24 00    	lock orq $0x0,(%rsp)
1394:	f0 48 83 0c 24 00    	lock orq $0x0,(%rsp)
```

### Deep & Tricky

**DT-16. x86 is not sequentially consistent: "Loads may be reordered with older stores to different locations". Two cores that each store a flag and then read the other's flag can both miss each other — which breaks every hand-written mutual-exclusion algorithm (Dekker, Peterson) that does not use a fence.** *(Intel SDM Vol. 3A, §9.2.3.4 (§8.2.3.4 in older editions) "Loads May Be Reordered with Earlier Stores to Different Locations", example 8-3)* Without a fence, the test above observes "both zero" in a large fraction of rounds on this machine; with the fence, never. (The program's label says "mfence"; the disassembly shows that GCC chose the equivalent locked instruction.) All other orderings (load→load, store→store, load→store) are preserved on x86, which is why a spinlock's *release* can be a plain store — but on ARM or POWER almost every pair can be reordered, which is why portable code states its ordering needs (`memory_order_acquire`, `release`) instead of relying on x86 behaviour.

## Topic 7B — Spinlocks

## Unit 7.3 — Spinlocks, and spinlocks that disable interrupts

**One-line summary:** A **spinlock** is a word that is 0 when free. To take it, atomically exchange 1 into it (`xchg`); if the old value was 1, someone else holds it — wait, reading (not writing) until it looks free, then try again. Taking the lock is an **acquire** operation (later accesses cannot move before it); releasing — storing 0 — is a **release** (earlier accesses cannot move after it). On one CPU there is one more danger: if code holding a lock is interrupted, and the interrupt handler wants the same lock, the handler spins forever — the holder cannot run until the handler returns. A lock that is ever taken in an interrupt handler must be taken elsewhere with interrupts disabled: `spin_lock_irqsave`.

```c
// file: new/include/smp.h
#pragma once
#include <proc.h>

#define PTE_PWT (1UL << 3)            /* page-level write-through */
#define PTE_PCD (1UL << 4)            /* page-level cache disable: for device registers */

static inline void cpu_relax(void) { __asm__ volatile("pause" ::: "memory"); }

typedef struct {
    volatile uint32_t locked;         /* 0 free, 1 held */
    volatile uint32_t owner;          /* debugging: CPU id + 1 of the holder, 0 if free */
    const char *name;
    void *taken_at;                   /* debugging: return address of the spin_lock call */
    uint64_t contended;               /* how many acquisitions had to wait */
} spinlock_t;
#define SPINLOCK_INIT(n) { 0, 0, n, NULL, 0 }

typedef struct { volatile uint32_t next, serving; } ticketlock_t;

int  cpu_id(void);                    /* LAPIC id of the running CPU (0 before the LAPIC is mapped) */
void spin_lock(spinlock_t *l);
void spin_unlock(spinlock_t *l);
uint64_t spin_lock_irqsave(spinlock_t *l);
void spin_unlock_irqrestore(spinlock_t *l, uint64_t flags);
void ticket_lock(ticketlock_t *l);
void ticket_unlock(ticketlock_t *l);

extern int ncpus;
int  smp_init(void);                  /* start all CPUs; returns how many are online */
void smp_start(void (*fn)(int ap_index));   /* run fn on every AP (indices 1..ncpus-1) */
void smp_wait(void);                  /* wait until they have all returned */
void smp_run(void (*fn)(int ap_index));     /* smp_start + smp_wait */
void smp_printf(const char *fmt, ...) __attribute__((format(printf, 1, 2)));
int  acpi_find_cpus(uint32_t *lapic_phys, uint8_t *apic_ids, int max);
```

```c
// file: new/kernel/spinlock.c
#include <smp.h>

__attribute__((noinline)) void spin_lock(spinlock_t *l) {
    uint32_t me = (uint32_t)cpu_id() + 1;
    if (l->owner == me)                                          /* spinlocks are not recursive: certain deadlock */
        panic("spinlock '%s': deadlock - already held by this CPU (taken at %p, wanted again at %p)", l->name,
              l->taken_at, __builtin_return_address(0));
    if (__atomic_exchange_n(&l->locked, 1, __ATOMIC_ACQUIRE)) {
        __atomic_fetch_add(&l->contended, 1, __ATOMIC_RELAXED);
        do {
            while (__atomic_load_n(&l->locked, __ATOMIC_RELAXED)) cpu_relax();   /* spin on a read: no cache-line ping-pong */
        } while (__atomic_exchange_n(&l->locked, 1, __ATOMIC_ACQUIRE));
    }
    l->owner = me;
    l->taken_at = __builtin_return_address(0);
}

void spin_unlock(spinlock_t *l) {
    l->owner = 0;
    __atomic_store_n(&l->locked, 0, __ATOMIC_RELEASE);          /* on x86: a plain mov */
}

uint64_t spin_lock_irqsave(spinlock_t *l) {
    uint64_t f = irq_save();                                     /* interrupts off FIRST, then the lock */
    spin_lock(l);
    return f;
}

void spin_unlock_irqrestore(spinlock_t *l, uint64_t f) {
    spin_unlock(l);
    irq_restore(f);
}

/* Ticket lock: take a number, wait until it is served. First come, first served. */
void ticket_lock(ticketlock_t *l) {
    uint32_t mine = __atomic_fetch_add(&l->next, 1, __ATOMIC_RELAXED);
    while (__atomic_load_n(&l->serving, __ATOMIC_ACQUIRE) != mine) cpu_relax();
}

void ticket_unlock(ticketlock_t *l) {
    __atomic_store_n(&l->serving, l->serving + 1, __ATOMIC_RELEASE);
}
```

The test uses the RTC's periodic interrupt (IRQ 8, about 1,024 per second) as an interrupt handler that shares a lock with a kernel thread. The thread holds the lock for a while, over and over; the handler takes it on every interrupt.

```c
// file: new/kernel/test_irqlock.c
#include <smp.h>

static spinlock_t stats_lock = SPINLOCK_INIT("stats");
static volatile uint64_t rtc_events, thread_updates;

static void rtc_irq(struct trap_frame *f) {
    (void)f;
    spin_lock(&stats_lock);                   /* interrupt context: interrupts are already off */
    rtc_events++;
    spin_unlock(&stats_lock);
    outb(0x70, 0x0C);                         /* read RTC register C: acknowledges the interrupt */
    (void)inb(0x71);
}

static void rtc_periodic(bool on) {
    outb(0x70, 0x8B);
    uint8_t b = inb(0x71);
    outb(0x70, 0x8B);
    outb(0x71, on ? (b | 0x40) : (b & ~0x40));   /* register B bit 6: periodic interrupt enable */
    outb(0x70, 0x0C);
    (void)inb(0x71);
}

KTEST(irqlock) {
    irq_register(8, rtc_irq);
    rtc_periodic(true);
    pic_unmask(8);
    for (int round = 0; round < 50; round++) {
#ifdef IRQLOCK_BUG                            /* experiment switch for DT-17 */
        spin_lock(&stats_lock);
#else
        uint64_t f = spin_lock_irqsave(&stats_lock);
#endif
        for (volatile int i = 0; i < 200000; i++) { }   /* a slow critical section */
        thread_updates++;
#ifdef IRQLOCK_BUG
        spin_unlock(&stats_lock);
#else
        spin_unlock_irqrestore(&stats_lock, f);
#endif
        sleep_ticks(1);
    }
    rtc_periodic(false);
    kprintf("50 critical sections in the thread, %lu RTC interrupts took the same lock\n", rtc_events);
    KASSERT(thread_updates == 50 && rtc_events > 0);
}
```

```console
$ cd os && make -s 2>&1 | tail -1; make -s run CMDLINE="test=irqlock" | tail -3
disk.img: 8192 blocks (8073 free), 256 inodes (237 free)
50 critical sections in the thread, 509 RTC interrupts took the same lock
== test irqlock passed ==
[qemu exit status 33]
```

### Deep & Tricky

**DT-17. On a single CPU, a plain spinlock shared with an interrupt handler is a deadlock waiting for the right timing: the thread takes the lock, the interrupt arrives, the handler spins on the lock — and the thread that would release it cannot run until the handler returns. The fix is to disable local interrupts *before* taking such a lock, everywhere outside the handler.** *(Linux `Documentation/locking/spinlocks.rst`: if a lock is used in interrupt handlers, "you need to use the irq-disabling versions of the spinlocks"; lockdep reports violations as "inconsistent {HARDIRQ-ON-W} -> {IN-HARDIRQ-W} usage")* Build with `-DIRQLOCK_BUG`: our spinlock's owner check turns what would be a silent hang into an immediate report, with both call sites.

```console
$ rm -rf irqbug && cp -r os irqbug && cd irqbug && make -s clean && make -s EXTRA_CFLAGS=-DIRQLOCK_BUG 2>&1 | tail -1; make -s run CMDLINE="test=irqlock" EXTRA_CFLAGS=-DIRQLOCK_BUG | sed -n 5,7p
disk.img: 8192 blocks (8073 free), 256 inodes (237 free)

KERNEL PANIC: spinlock 'stats': deadlock - already held by this CPU (taken at 0x00000000001060ca, wanted again at 0x0000000000106042)
backtrace (symbolise with: addr2line -f -e kernel.elf <addresses>):
$ cd irqbug && addr2line -f -s -e kernel.elf $(grep -o "0x0000000000[0-9a-f]*" <(make -s run CMDLINE="test=irqlock" EXTRA_CFLAGS=-DIRQLOCK_BUG) | head -2) | paste - -
ktest_irqlock	test_irqlock.c:35
rtc_irq	test_irqlock.c:10
```

The first address is in the test's critical section, the second in `rtc_irq` — exactly the pair lockdep would report. The order matters in `spin_lock_irqsave` too: disabling interrupts *after* taking the lock would leave a window of a few instructions with the same deadlock.

## Topic 7C — Symmetric multiprocessing

## Unit 7.4 — Waking up the other CPUs

**One-line summary:** At power-on only one CPU runs — the **bootstrap processor** (BSP). The others, **application processors** (APs), wait in a halted state. The firmware lists all CPUs in the ACPI **MADT** table, each with its **local APIC** ID. The BSP wakes an AP by sending it, through its own local APIC, an **INIT** inter-processor interrupt and then a **SIPI** (startup IPI) whose vector is a page number: the AP starts in 16-bit real mode at that page (here 0x8000). A small **trampoline** there repeats our boot sequence — protected mode, PAE, the kernel's page tables, long mode — and calls `ap_main` on a stack prepared by the BSP.

```nasm
; file: new/arch/trampoline.nasm
; AP startup code. Assembled as a flat binary, copied to physical 0x8000; SIPI vector 0x08 starts it at 0x0800:0000.
bits 16
org 0x8000
start:
    jmp short real_start
align 8
tramp_cr3:   dq 0                 ; 0x8008: filled in by the BSP
tramp_stack: dq 0                 ; 0x8010
tramp_entry: dq 0                 ; 0x8018
real_start:
    cli
    cld
    xor ax, ax
    mov ds, ax
    lgdt [gdt_ptr]
    mov eax, cr0
    or eax, 1                     ; protected mode
    mov cr0, eax
    jmp 0x08:start32

bits 32
start32:
    mov ax, 0x10
    mov ds, ax
    mov es, ax
    mov ss, ax
    mov eax, cr4
    or eax, 1 << 5                ; PAE
    mov cr4, eax
    mov eax, [tramp_cr3]          ; the kernel's PML4 (below 4 GiB)
    mov cr3, eax
    mov ecx, 0xC0000080           ; EFER.LME
    rdmsr
    or eax, 1 << 8
    wrmsr
    mov eax, cr0
    or eax, 1 << 31               ; paging on: long mode
    mov cr0, eax
    jmp 0x18:start64

bits 64
start64:
    mov rsp, [tramp_stack]
    mov rax, [tramp_entry]
    call rax                      ; ap_main: never returns

align 8
gdt:
    dq 0
    dq 0x00CF9A000000FFFF         ; 0x08: 32-bit code, flat
    dq 0x00CF92000000FFFF         ; 0x10: 32-bit data, flat
    dq 0x00209A0000000000         ; 0x18: 64-bit code
gdt_ptr:
    dw $ - gdt - 1
    dd gdt
```

```nasm
; file: new/arch/tramp_blob.asm
section .rodata
global tramp_start, tramp_end
tramp_start:
    incbin "arch/trampoline.bin"
tramp_end:
```

```c
// file: new/kernel/acpi.c
/* Find the CPUs: Multiboot2 tag 14 (ACPI 1.0 RSDP) -> RSDT -> MADT -> processor local APIC entries. */
#include <smp.h>

struct sdt_header {
    char signature[4];
    uint32_t length;
    uint8_t revision, checksum;
    char oem_id[6], oem_table_id[8];
    uint32_t oem_revision, creator_id, creator_revision;
} __attribute__((packed));
_Static_assert(sizeof(struct sdt_header) == 36, "ACPI table header");

static bool checksum_ok(const void *p, uint32_t len) {     /* every ACPI table's bytes sum to 0 mod 256 */
    uint8_t sum = 0;
    for (uint32_t i = 0; i < len; i++) sum = (uint8_t)(sum + ((const uint8_t *)p)[i]);
    return sum == 0;
}

int acpi_find_cpus(uint32_t *lapic_phys, uint8_t *apic_ids, int max) {
    const uint8_t *rsdp = NULL;
    for (uint32_t *t = (uint32_t *)(mb_info_start + 8); t[0] != 0; t = (uint32_t *)((uintptr_t)t + ((t[1] + 7) & ~7u)))
        if (t[0] == 14) rsdp = (const uint8_t *)(t + 2);       /* a copy of the RSDP, made by the bootloader */
    if (!rsdp || memcmp(rsdp, "RSD PTR ", 8) || !checksum_ok(rsdp, 20)) return -1;
    const struct sdt_header *rsdt = (const struct sdt_header *)(uintptr_t)*(const uint32_t *)(rsdp + 16);
    if (memcmp(rsdt->signature, "RSDT", 4) || !checksum_ok(rsdt, rsdt->length)) return -1;
    const uint32_t *entries = (const uint32_t *)(rsdt + 1);
    for (uint32_t i = 0; i < (rsdt->length - sizeof *rsdt) / 4; i++) {
        const struct sdt_header *h = (const struct sdt_header *)(uintptr_t)entries[i];
        if (memcmp(h->signature, "APIC", 4) || !checksum_ok(h, h->length)) continue;
        *lapic_phys = *(const uint32_t *)(h + 1);
        int n = 0;
        for (const uint8_t *p = (const uint8_t *)h + 44; p < (const uint8_t *)h + h->length && n < max; p += p[1])
            if (p[0] == 0 && (*(const uint32_t *)(p + 4) & 1)) apic_ids[n++] = p[3];   /* type 0, enabled */
        return n;
    }
    return -1;
}
```

```c
// file: new/kernel/smp.c
#include <smp.h>

#define LAPIC_VIRT 0xFFFFD00000000000UL
#define TRAMP_PHYS 0x8000UL

struct tramp_data { uint64_t cr3, stack, entry; };
struct __attribute__((packed)) desc_ptr { uint16_t limit; uint64_t base; };

extern const char tramp_start[], tramp_end[];
static volatile uint32_t *lapic;
static struct desc_ptr bsp_gdtr, bsp_idtr;
static spinlock_t print_lock = SPINLOCK_INIT("print");
int ncpus = 1;
static volatile int aps_online, next_index = 1;
static void (*volatile work)(int);
static volatile uint32_t generation, finished;

static uint32_t lapic_read(uint32_t reg) { return lapic[reg / 4]; }
static void lapic_write(uint32_t reg, uint32_t v) { lapic[reg / 4] = v; }

int cpu_id(void) { return lapic ? (int)(lapic_read(0x20) >> 24) : 0; }

static void send_ipi(uint8_t apic_id, uint32_t command) {
    lapic_write(0x310, (uint32_t)apic_id << 24);                 /* ICR high: destination */
    lapic_write(0x300, command);                                  /* ICR low: writing it sends the IPI */
    while (lapic_read(0x300) & (1 << 12)) cpu_relax();           /* delivery status: pending */
}

void smp_printf(const char *fmt, ...) {
    char buf[256];
    va_list ap; va_start(ap, fmt);
    kvsnprintf(buf, sizeof buf, fmt, ap);
    va_end(ap);
    uint64_t f = spin_lock_irqsave(&print_lock);                  /* one line at a time, whole */
    for (char *p = buf; *p; p++) serial_putc(*p);
    spin_unlock_irqrestore(&print_lock, f);
}

static _Noreturn void ap_main(void) {
    __asm__ volatile("lgdt %0\n\t"                                 /* the BSP's GDT and IDT */
                     "pushq $0x08\n\t"
                     "leaq 1f(%%rip), %%rax\n\t"
                     "pushq %%rax\n\t"
                     "lretq\n"
                     "1:\n\t"
                     "mov $0x10, %%ax\n\t"
                     "mov %%ax, %%ds\n\t"
                     "mov %%ax, %%es\n\t"
                     "mov %%ax, %%ss\n\t"
                     "lidt %1"
                     : : "m"(bsp_gdtr), "m"(bsp_idtr) : "rax", "memory");
    int index = __atomic_fetch_add(&next_index, 1, __ATOMIC_RELAXED);
    __atomic_fetch_add(&aps_online, 1, __ATOMIC_RELEASE);
    uint32_t seen = 0;
    for (;;) {                                                     /* wait for work; interrupts stay off on APs */
        while (__atomic_load_n(&generation, __ATOMIC_ACQUIRE) == seen) cpu_relax();
        seen = generation;
        work(index);
        __atomic_fetch_add(&finished, 1, __ATOMIC_RELEASE);
    }
}

static void wait_ticks(uint64_t n) { uint64_t t = ticks + n; while (ticks < t) hlt(); }

int smp_init(void) {
    static bool done;
    if (done) return ncpus;
    done = true;
    uint32_t lapic_phys;
    uint8_t ids[16];
    int n = acpi_find_cpus(&lapic_phys, ids, 16);
    if (n <= 0) { kprintf("smp: no MADT, staying on one CPU\n"); return 1; }
    vmm_map(LAPIC_VIRT, lapic_phys, PTE_WRITABLE | PTE_PCD | PTE_PWT);   /* device registers: uncached */
    lapic = (volatile uint32_t *)LAPIC_VIRT;
    lapic_write(0xF0, lapic_read(0xF0) | 0x100);                  /* spurious-vector register: APIC enabled */
    int me = cpu_id();
    kprintf("smp: MADT lists %d CPUs, local APICs at 0x%x; BSP is APIC %d\n", n, lapic_phys, me);
    __asm__ volatile("sgdt %0; sidt %1" : "=m"(bsp_gdtr), "=m"(bsp_idtr));
    memcpy((void *)TRAMP_PHYS, tramp_start, (size_t)(tramp_end - tramp_start));
    volatile struct tramp_data *td = (volatile struct tramp_data *)(TRAMP_PHYS + 8);
    td->cr3 = (uint64_t)kernel_pml4;
    td->entry = (uint64_t)ap_main;
    for (int i = 0; i < n; i++) {
        if (ids[i] == me) continue;
        int before = aps_online;
        td->stack = (uint64_t)kmalloc(KSTACK_SIZE) + KSTACK_SIZE;
        send_ipi(ids[i], 0x4500);                                  /* INIT, level assert */
        wait_ticks(2);                                             /* >= 10 ms */
        for (int attempt = 0; attempt < 2 && aps_online == before; attempt++) {
            send_ipi(ids[i], 0x4600 | (TRAMP_PHYS >> 12));         /* STARTUP, vector = page 0x08 */
            for (int w = 0; w < 50 && aps_online == before; w++) wait_ticks(1);
        }
        if (aps_online == before) kprintf("smp: APIC %d did not start\n", ids[i]);
    }
    ncpus = 1 + aps_online;
    return ncpus;
}

void smp_start(void (*fn)(int)) {
    finished = 0;
    work = fn;
    __atomic_fetch_add(&generation, 1, __ATOMIC_RELEASE);
}

void smp_wait(void) {   /* the BSP sleeps between timer ticks instead of competing with the APs for host CPU time */
    while (__atomic_load_n(&finished, __ATOMIC_ACQUIRE) < (uint32_t)(ncpus - 1)) hlt();
}
void smp_run(void (*fn)(int)) { smp_start(fn); smp_wait(); }
```

The APs run with interrupts disabled and never enter the scheduler: they are workers that `smp_run` hands a function to. Making the scheduler itself multi-processor — per-CPU run queues, per-CPU `current`, a TSS per CPU, TLB shootdowns — is a large step, and the one Linux has taken with great care; Chapter 11 measures the result — two CPUs, a load balancer, and per-CPU run queues dividing the work.

```c
// file: new/kernel/test_smp.c
#include <smp.h>

#define N 1000000
static volatile uint64_t plain;
static uint64_t atomic_counter, locked_counter;
static spinlock_t counter_lock = SPINLOCK_INIT("counter");

static void hello(int index) {
    int local;
    smp_printf("  AP %d: LAPIC id %d, stack near %p\n", index, cpu_id(), (void *)&local);
}
static void inc_plain(int index)  { (void)index; for (int i = 0; i < N; i++) plain++; }
static void inc_atomic(int index) { (void)index; for (int i = 0; i < N; i++) __atomic_fetch_add(&atomic_counter, 1, __ATOMIC_RELAXED); }
static void inc_locked(int index) {
    (void)index;
    for (int i = 0; i < N; i++) { spin_lock(&counter_lock); locked_counter++; spin_unlock(&counter_lock); }
}

static void phase(const char *name, void (*fn)(int), volatile uint64_t *counter) {
    uint64_t t0 = ticks;
    smp_run(fn);
    uint64_t expected = (uint64_t)(ncpus - 1) * N;
    kprintf("  %-9s %8lu of %8lu (%3lu%% lost) in %3lu ticks\n", name, *counter, expected,
            (expected - *counter) * 100 / expected, ticks - t0);
}

KTEST(smp) {
    int n = smp_init();
    kprintf("%d CPUs online\n", n);
    KASSERT(n > 1);
    smp_run(hello);
    kprintf("%d APs each increment a shared counter %d times:\n", n - 1, N);
    phase("volatile", inc_plain, &plain);
    phase("atomic", inc_atomic, (volatile uint64_t *)&atomic_counter);
    phase("spinlock", inc_locked, (volatile uint64_t *)&locked_counter);
    kprintf("  the spinlock was contended in %lu of %lu acquisitions\n", counter_lock.contended,
            (uint64_t)(n - 1) * N);
    KASSERT(atomic_counter == (uint64_t)(n - 1) * N && locked_counter == atomic_counter);
}
```

The Makefile builds the trampoline as a flat binary and starts QEMU with four CPUs, using multi-threaded TCG so that the virtual CPUs really run in parallel on the host:

```text
# file: new/Makefile
CC      := gcc
CFLAGS  := -std=c11 -ffreestanding -fno-stack-protector -fno-pic -mno-red-zone -mno-sse -mno-mmx -mno-80387 \
           -mcmodel=small -fno-omit-frame-pointer -O2 -g -Wall -Wextra -Werror -Iinclude $(EXTRA_CFLAGS)
UCFLAGS := -std=c11 -ffreestanding -fno-stack-protector -fno-pic -mno-sse -mno-mmx -mno-80387 -O2 -Wall -Wextra -Werror
LDFLAGS := -n -T linker.ld -nostdlib -z noexecstack -z max-page-size=0x1000 --no-warn-rwx-segments
SRCS_C  := $(wildcard kernel/*.c)
SRCS_S  := $(wildcard arch/*.asm)
OBJS    := $(SRCS_S:.asm=.o) $(SRCS_C:.c=.o)
USER    := $(basename $(notdir $(wildcard user/*.c)))
CMDLINE ?=
TIMEOUT ?= 60
SMP     ?= 4
QEMU    := qemu-system-x86_64 -serial stdio -display none -no-reboot -m 128M -boot d \
           -smp $(SMP) -accel tcg,thread=multi \
           -drive file=disk.img,format=raw,index=0,media=disk \
           -device isa-debug-exit,iobase=0xf4,iosize=0x04 $(QEMU_EXTRA)

all: kernel.elf disk.img

kernel.elf: $(OBJS) linker.ld
	ld $(LDFLAGS) -o $@ $(OBJS)

arch/trampoline.bin: arch/trampoline.nasm
	nasm -f bin $< -o $@
arch/tramp_blob.o: arch/trampoline.bin

%.o: %.asm
	nasm -f elf64 -g $< -o $@
%.o: %.c $(wildcard include/*.h)
	$(CC) $(CFLAGS) -c $< -o $@

user/%.elf: user/%.c user/ulib.h user/user.ld
	$(CC) $(UCFLAGS) -c $< -o user/$*.o
	ld -T user/user.ld -nostdlib -z noexecstack -z max-page-size=0x1000 --no-warn-rwx-segments -o $@ user/$*.o

tools/mkhobbyfs: tools/mkhobbyfs.c include/hobbyfs.h
	gcc -std=c11 -D_DEFAULT_SOURCE -O2 -Wall -Wextra -Werror -Iinclude -o $@ $<

disk.img: tools/mkhobbyfs $(USER:%=user/%.elf) tools/make_evil.py
	rm -rf rootfs && mkdir -p rootfs/bin rootfs/etc rootfs/docs
	for p in $(USER); do cp user/$$p.elf rootfs/bin/$$p; done
	python3 tools/make_evil.py rootfs/bin/hello rootfs/bin/evil
	printf 'Welcome to hobby os.\nPrograms live in /bin; try: ls /bin\n' > rootfs/etc/motd
	seq 1 5000 > rootfs/docs/numbers.txt
	cp include/hobbyfs.h rootfs/docs/
	tools/mkhobbyfs $@ 8192 rootfs

os.iso: kernel.elf
	rm -rf iso && mkdir -p iso/boot/grub && cp kernel.elf iso/boot/
	printf 'set timeout=0\nset default=0\nmenuentry "hobby os" {\n  multiboot2 /boot/kernel.elf %s\n  boot\n}\n' \
	    "$(CMDLINE)" > iso/boot/grub/grub.cfg
	grub-mkrescue -o $@ iso > /dev/null 2>&1

run: kernel.elf disk.img
	rm -f os.iso && $(MAKE) -s os.iso CMDLINE="$(CMDLINE)"
	timeout $(TIMEOUT) $(QEMU) -cdrom os.iso; status=$$?; echo "[qemu exit status $$status]"

clean:
	rm -rf $(OBJS) kernel.elf os.iso iso grub.cfg user/*.o user/*.elf user/*.bin tools/mkhobbyfs rootfs disk.img \
	       arch/trampoline.bin
.PHONY: all run clean os.iso
.PRECIOUS: user/%.elf
```

```console
$ cd os && make -s 2>&1 | grep -v "^disk.img" ; make -s run CMDLINE="test=smp" 2>&1 | grep -v "^disk.img"
hobby os: 127 MiB usable, command line "test=smp"
disk: "QEMU HARDDISK", 16384 sectors (8192 KiB)
hobbyfs: mounted
== test smp ==
smp: MADT lists 4 CPUs, local APICs at 0xfee00000; BSP is APIC 0
4 CPUs online
  AP 1: LAPIC id 1, stack near 0xffff80000000821c
  AP 2: LAPIC id 2, stack near 0xffff80000000c23c
  AP 3: LAPIC id 3, stack near 0xffff80000001025c
3 APs each increment a shared counter 1000000 times:
  volatile   2399559 of  3000000 ( 20% lost) in   1 ticks
  atomic     3000000 of  3000000 (  0% lost) in   4 ticks
  spinlock   3000000 of  3000000 (  0% lost) in  69 ticks
  the spinlock was contended in 14010 of 3000000 acquisitions
== test smp passed ==
[qemu exit status 33]
```

Three APs started, each on its own stack. With three CPUs incrementing the same `volatile` counter, a large part of the updates is lost; the `lock`-prefixed increment and the spinlock both get every one. The spinlock is much slower than the atomic add — every increment now includes taking and releasing a lock whose cache line moves between CPUs — and it was contended thousands of times. (The exact numbers change from run to run; that is the nature of a race.)

### Pattern

**P-8. For every piece of shared data, write down what protects it — "atomic", "lock X", "interrupts off", "only CPU n touches it" — and choose the weakest tool that is correct: a single counter needs one atomic instruction, not a lock; data shared with an interrupt handler needs an irq-safe lock; a lock must not be held across anything that can sleep.** *(Practice; Linux documents this next to each structure and enforces much of it at run time with lockdep, `CONFIG_PROVE_LOCKING`)* Our spinlock's owner check is a tiny lockdep: it turns "it hangs sometimes" into a message with two addresses. *Trade-off:* the debug fields (`owner`, `taken_at`, `contended`) cost memory and a few instructions per acquisition; Linux compiles such checks out unless a debug option is enabled.

### Mastery check

1. **Predict.** Why does `spin_lock` spin on a plain load and retry the `xchg` only when the lock looks free, instead of spinning on `xchg` directly?
2. **Predict.** In `ticket_unlock`, the expression `l->serving + 1` reads `serving` non-atomically. Why is that correct?
3. **Predict.** The BSP maps the local APIC with caching disabled. What could go wrong if it were mapped write-back like normal RAM?

## Chapter 7 Project — Lock fairness and scalability

**Goal:** measure, on four CPUs, how a test-and-set spinlock and a ticket lock divide a contended lock among CPUs. For a fixed time (one second of timer ticks), each AP repeatedly takes the lock, does a short piece of work inside it, releases it, and counts its acquisitions. Report the per-CPU counts, the total throughput, and the ratio between the luckiest and unluckiest CPU.

```c
// file: proj/kernel/test_fairness.c
#include <smp.h>

static spinlock_t tas = SPINLOCK_INIT("tas");
static ticketlock_t ticket;
static volatile bool stop;
static uint64_t counts[16];
static volatile uint64_t shared_work;

static void critical_section(void) { for (int i = 0; i < 20; i++) shared_work++; }   /* protected by the lock */
static void outside_work(void) { for (volatile int i = 0; i < 20; i++) { } }

static void run_tas(int index) {
    while (!stop) { spin_lock(&tas); critical_section(); spin_unlock(&tas); counts[index]++; outside_work(); }
}
static void run_ticket(int index) {
    while (!stop) { ticket_lock(&ticket); critical_section(); ticket_unlock(&ticket); counts[index]++; outside_work(); }
}

static void measure(const char *name, void (*fn)(int)) {
    memset(counts, 0, sizeof counts);
    shared_work = 0;
    stop = false;
    smp_start(fn);
    sleep_ticks(100);
    stop = true;
    smp_wait();
    uint64_t total = 0, min = ~0UL, max = 0;
    kprintf("%-7s", name);
    for (int i = 1; i < ncpus; i++) {
        kprintf("  AP%d %7lu", i, counts[i]);
        total += counts[i];
        if (counts[i] < min) min = counts[i];
        if (counts[i] > max) max = counts[i];
    }
    kprintf("  | total %8lu  max/min %lu.%02lu\n", total, max / min, max % min * 100 / min);
    KASSERT(shared_work == total * 20);                 /* the lock really excluded */
}

KTEST(fairness) {
    KASSERT(smp_init() >= 3);
    for (int round = 0; round < 3; round++) {
        measure("tas", run_tas);
        measure("ticket", run_ticket);
    }
}
```

```console
$ rm -rf final && cp -r os final && cp -r proj/. final/ && make -s -C final clean && make -s -C final 2>&1 | grep -v "^disk.img"; echo "final = os + proj"
final = os + proj
$ cd final && make -s run CMDLINE="test=fairness" 2>&1 | grep -v "^disk.img" | sed -n 5,11p
smp: MADT lists 4 CPUs, local APICs at 0xfee00000; BSP is APIC 0
tas      AP1 1033480  AP2 1034246  AP3 1260089  | total  3327815  max/min 1.21
ticket   AP1     391  AP2    6177  AP3    5854  | total    12422  max/min 15.79
tas      AP1 1295203  AP2  859684  AP3 1133169  | total  3288056  max/min 1.50
ticket   AP1   10199  AP2    3316  AP3    6807  | total    20322  max/min 3.07
tas      AP1 1097455  AP2  955049  AP3 1176256  | total  3228760  max/min 1.23
ticket   AP1   10391  AP2    4399  AP3    4693  | total    19483  max/min 2.36
$ cd final && make -s run CMDLINE="test=fairness" SMP=3 2>&1 | grep -v "^disk.img" | sed -n 5,11p
smp: MADT lists 3 CPUs, local APICs at 0xfee00000; BSP is APIC 0
tas      AP1 1920033  AP2 1882523  | total  3802556  max/min 1.01
ticket   AP1 2154215  AP2 2163777  | total  4317992  max/min 1.00
tas      AP1 1896944  AP2 1967177  | total  3864121  max/min 1.03
ticket   AP1 1925302  AP2 1966586  | total  3891888  max/min 1.02
tas      AP1 1915603  AP2 1933631  | total  3849234  max/min 1.00
ticket   AP1 1949206  AP2 2097480  | total  4046686  max/min 1.07
```

The two runs tell different stories, and the difference is the lesson.

With **two** APs (`SMP=3`), each virtual CPU has a host core to itself. Both locks are fair (max/min close to 1.00) and their throughput is similar. In this emulator the test-and-set lock does not show the unfairness it often shows on real hardware — QEMU does not model which core "owns" a cache line, so the CPU that just released the lock gets no advantage. On real multi-socket machines that advantage is large, and it is why ticket locks were introduced.

With **three** APs on a host that has only two cores, the ticket lock collapses: its throughput falls by a factor of about 200, and it becomes *less* fair than test-and-set. The reason is **lock-waiter preemption**. A ticket lock hands the lock to one specific waiter — the next number. If the host has descheduled that virtual CPU, the lock is free but nobody else may take it; every other CPU spins until the host runs the chosen one again, for a whole host time slice. A test-and-set lock gives the lock to whichever CPU is running and asks first. This is a real production problem for virtual machines: Linux's **paravirtual spinlocks** let a waiting virtual CPU tell the hypervisor to run the lock holder (or the next waiter) instead of spinning. Linux used ticket spinlocks on x86 from 2.6.25 and replaced them in 4.2 with **queued spinlocks** (fair, each waiter spins on its own cache line), with a paravirtual variant for guests.

### Review checklist

- `smp_printf` takes an irq-safe lock. Why does it need the irq-safe version when APs run with interrupts disabled anyway?
- What would `cpu_id()` return on an AP before `lapic` is set? Is that a problem in this design?
- How many cache lines does `counts[16]` occupy, and how does that affect the measurement? (Look up *false sharing*.)

## Answers to the Chapter 7 prediction questions

**7.4 Q1.** An `xchg` is a write: every attempt takes the cache line in exclusive state, so several spinning CPUs bounce the line between them continuously and slow down the holder too. A plain load can be served from each CPU's own shared copy of the line; only when the line changes (the holder released) do the waiters try the expensive `xchg`. This is "test-and-test-and-set".

**7.4 Q2.** Only the lock holder writes `serving`, and only in `ticket_unlock`. So when the holder reads `l->serving`, nobody else can be changing it; the store must be a release store (so the critical section's writes become visible first), but the read before it needs no atomicity.

**7.4 Q3.** The APIC registers are not memory: reading a register returns the device's current value, and writing one (such as the ICR) triggers an action. With caching, reads could return a stale cached copy, and writes could sit in the cache instead of reaching the device — an IPI might never be sent. Device memory must be mapped uncached (or with a memory type made for devices).

# Chapter 8 — Building and Booting Linux

*Part II of this book moves from our own kernel to the real one. Everything you built in Part I has a counterpart in Linux — a boot path, interrupt handling, a page allocator, a scheduler, a VFS, an ELF loader, spinlocks, SMP bring-up — only larger, older, and far more carefully engineered. This chapter sets up the workbench used for the rest of the book: the Linux 6.12 source, a small configuration with the debugging features we want, a build, and a boot in QEMU into a tiny user space made from BusyBox, scripted so that one command builds, boots, tests, and reports.*

**What you will be able to do after this chapter**

- Find your way around the kernel source tree and connect its directories to the ideas of Part I.
- Read Kconfig and Kbuild files; create, inspect, and reproduce a configuration.
- Build out of tree, understand what `vmlinux`, `bzImage`, and `System.map` are, and rebuild incrementally.
- Make an initramfs by hand, boot it in QEMU on a serial console, and recognise the two classic ways early boot fails.
- Script the whole build–boot–test loop so that its result is an exit status.

**The setup used in this book.** The source is the `v6.12` tag of the mainline kernel (a shallow clone: `git clone --depth 1 --branch v6.12 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git`). The build goes to a separate directory with `O=`, so the source tree stays clean. In this chapter's code directory, `linux` and `build` are symbolic links to those two directories; BusyBox comes from the distribution's static package (`busybox-static` on Debian and Ubuntu).

```console
$ ln -sfn /home/claude/kernelsrc/linux linux && ln -sfn /home/claude/kernelsrc/build build && cd linux && git --no-pager log --oneline -1 && make -s kernelversion
adc218676 (grafted, HEAD, tag: v6.12) Linux 6.12
6.12.0
$ cd linux && git ls-files | wc -l && git ls-files | grep -cE '\.[ch]$'
86680
59973
```

## Topic 8A — The source tree and the configuration

## Unit 8.1 — A map of the tree

**One-line summary:** The top-level directories split the kernel by job: `arch/` (everything specific to one CPU architecture — boot code, page tables, interrupt entry), `kernel/` (scheduler, locking, signals, time, tracing), `mm/` (memory management), `fs/` (the VFS and every file system), `drivers/` (more than half of all the code), `net/` (protocols), `block/`, `ipc/`, `security/`, `include/` (shared headers; `include/uapi/` is the stable interface to user space), `init/` (the generic start-up code), `lib/` (helpers), `scripts/` and `tools/` (build machinery and user-space tools), and `Documentation/`.

```console
$ cd linux && for d in arch drivers fs kernel mm net include init; do f=$(git ls-files "$d" | grep -E '\.[ch]$'); printf "%-8s %6d C files  %9d lines\n" "$d" "$(echo "$f" | wc -l)" "$(echo "$f" | xargs cat | wc -l)"; done
arch       9211 C files    2157617 lines
drivers   31590 C files   24252003 lines
fs         2301 C files    1642809 lines
kernel      532 C files     494960 lines
mm          185 C files     199137 lines
net        1820 C files    1286629 lines
include    6204 C files    1306533 lines
init         11 C files       4104 lines
```

`drivers/` alone is over half of the kernel. The parts this book reads most are small by comparison — and every one of them has a counterpart in Part I:

| Part I (hobby kernel) | Linux 6.12 |
|---|---|
| `boot.asm`, long-mode switch | `arch/x86/boot/`, `arch/x86/kernel/head_64.S` |
| `kmain` | `start_kernel()` in `init/main.c` |
| IDT, `isr.asm`, `interrupt_dispatch` | `arch/x86/kernel/idt.c`, `arch/x86/entry/entry_64.S`, `kernel/irq/` |
| frame bitmap, `kmalloc` | `mm/memblock.c`, `mm/page_alloc.c` (buddy allocator), `mm/slub.c` |
| `vmm.c`, demand paging | `arch/x86/mm/fault.c`, `mm/memory.c` |
| `sched.c`, `switch_to` | `kernel/sched/core.c`, `fair.c`; `arch/x86/kernel/process_64.c` |
| `syscall.c` | `arch/x86/entry/syscall_64.c`, `entry_64.S` |
| hobbyfs, file descriptors | `fs/`: `namei.c`, `read_write.c`, `file.c`; `fs/ext2/` |
| ELF loader | `fs/binfmt_elf.c` |
| spinlocks, SMP bring-up | `kernel/locking/`, `arch/x86/kernel/smpboot.c` |

`start_kernel` is a long list of initialisation calls in the order our `kmain` grew them:

```console
$ cd linux && grep -nE "^\s+(setup_arch|trap_init|mm_core_init|sched_init|init_IRQ|time_init|console_init|rest_init|smp_init|do_basic_setup)\(" init/main.c
925:	setup_arch(&command_line);
963:	trap_init();
964:	mm_core_init();
976:	sched_init();
1008:	init_IRQ();
1016:	time_init();
1040:	console_init();
1105:	rest_init();
1572:	smp_init();
1580:	do_basic_setup();
```

## Unit 8.2 — Kconfig and the configuration

**One-line summary:** Linux has over 18,000 configuration options. Each is declared in a `Kconfig` file (type, prompt, dependencies, default, help); the chosen values live in `.config`; the build turns them into `include/generated/autoconf.h` (`#define CONFIG_X 1`) for C code and into `CONFIG_X=y` variables for the Makefiles, where `obj-$(CONFIG_X) += x.o` decides whether a file is built in (`y`), built as a module (`m`), or skipped. `make defconfig` gives a sensible default; `make tinyconfig` gives the smallest kernel that boots; a **defconfig file** (made by `make savedefconfig`) stores only the options that differ from the defaults, so it is small enough to read and keep in version control.

```console
$ cd linux && sed -n '/^config E1000$/,/^config /p' drivers/net/ethernet/intel/Kconfig | head -n -1
config E1000
	tristate "Intel(R) PRO/1000 Gigabit Ethernet support"
	depends on PCI && HAS_IOPORT
	help
	  This driver supports Intel(R) PRO/1000 gigabit ethernet family of
	  adapters.  For more information on how to identify your adapter, go
	  to the Adapter & Driver ID Guide that can be located at:

	  <http://support.intel.com>

	  More specific information on configuring the driver is in
	  <file:Documentation/networking/device_drivers/ethernet/intel/e1000.rst>.

	  To compile this driver as a module, choose M here. The module
	  will be called e1000.
$ cd linux && grep -n "E1000)" drivers/net/ethernet/intel/Makefile
10:obj-$(CONFIG_E1000) += e1000/
```

This book's kernel is built from the following defconfig. It started from `tinyconfig` (so there is no block layer, no USB, no sound — nothing we do not use) and adds what the next chapters need: serial console, initramfs, devtmpfs, ACPI (to find both CPUs and to power off), PCI and the e1000 network card, networking, modules, BPF, kprobes and ftrace, `/proc/config.gz`, full debug information, and three debugging tools that slow the kernel down a lot and catch bugs early — **KASAN** (memory errors), **lockdep** (`PROVE_LOCKING`: locking errors), and `DEBUG_ATOMIC_SLEEP` (sleeping where you must not).

```text
# file: book_defconfig
CONFIG_LOCALVERSION="-book"
# CONFIG_LOCALVERSION_AUTO is not set
CONFIG_KERNEL_XZ=y
CONFIG_SYSVIPC=y
# CONFIG_CROSS_MEMORY_ATTACH is not set
CONFIG_HIGH_RES_TIMERS=y
CONFIG_BPF_SYSCALL=y
CONFIG_BPF_JIT=y
# CONFIG_PREEMPT_DYNAMIC is not set
CONFIG_IKCONFIG=y
CONFIG_IKCONFIG_PROC=y
CONFIG_CGROUPS=y
CONFIG_BLK_DEV_INITRD=y
# CONFIG_INITRAMFS_PRESERVE_MTIME is not set
CONFIG_CC_OPTIMIZE_FOR_SIZE=y
CONFIG_EXPERT=y
# CONFIG_SGETMASK_SYSCALL is not set
# CONFIG_SYSFS_SYSCALL is not set
# CONFIG_FHANDLE is not set
# CONFIG_PCSPKR_PLATFORM is not set
# CONFIG_IO_URING is not set
# CONFIG_ADVISE_SYSCALLS is not set
# CONFIG_MEMBARRIER is not set
# CONFIG_RSEQ is not set
# CONFIG_CACHESTAT_SYSCALL is not set
CONFIG_SMP=y
# CONFIG_X86_EXTENDED_PLATFORM is not set
# CONFIG_SCHED_OMIT_FRAME_POINTER is not set
# CONFIG_DMI is not set
CONFIG_NR_CPUS=2
# CONFIG_X86_MCE is not set
# CONFIG_PERF_EVENTS_AMD_UNCORE is not set
# CONFIG_X86_VSYSCALL_EMULATION is not set
# CONFIG_X86_IOPL_IOPERM is not set
# CONFIG_X86_5LEVEL is not set
# CONFIG_MTRR is not set
# CONFIG_X86_UMIP is not set
# CONFIG_X86_KERNEL_IBT is not set
# CONFIG_X86_INTEL_MEMORY_PROTECTION_KEYS is not set
# CONFIG_RELOCATABLE is not set
# CONFIG_MODIFY_LDT_SYSCALL is not set
# CONFIG_CPU_MITIGATIONS is not set
# CONFIG_SUSPEND is not set
# CONFIG_ISA_DMA_API is not set
# CONFIG_VIRTUALIZATION is not set
CONFIG_KPROBES=y
# CONFIG_SECCOMP is not set
# CONFIG_STACKPROTECTOR is not set
# CONFIG_RANDOMIZE_KSTACK_OFFSET is not set
CONFIG_MODULES=y
CONFIG_MODULE_UNLOAD=y
# CONFIG_BLOCK is not set
# CONFIG_COREDUMP is not set
# CONFIG_COMPAT_BRK is not set
# CONFIG_SPARSEMEM_VMEMMAP is not set
# CONFIG_COMPACTION is not set
# CONFIG_ZONE_DMA is not set
# CONFIG_VM_EVENT_COUNTERS is not set
# CONFIG_SECRETMEM is not set
CONFIG_NET=y
CONFIG_PACKET=y
CONFIG_UNIX=y
CONFIG_INET=y
CONFIG_PCI=y
CONFIG_DEVTMPFS=y
CONFIG_DEVTMPFS_MOUNT=y
# CONFIG_STANDALONE is not set
# CONFIG_PREVENT_FIRMWARE_BUILD is not set
# CONFIG_ALLOW_DEV_COREDUMP is not set
# CONFIG_FIRMWARE_MEMMAP is not set
CONFIG_NETDEVICES=y
CONFIG_E1000=y
CONFIG_SERIAL_8250=y
CONFIG_SERIAL_8250_CONSOLE=y
# CONFIG_HW_RANDOM is not set
# CONFIG_DEVMEM is not set
# CONFIG_DEVPORT is not set
# CONFIG_HWMON is not set
# CONFIG_USB_SUPPORT is not set
# CONFIG_VIRTIO_MENU is not set
# CONFIG_VHOST_MENU is not set
# CONFIG_SURFACE_PLATFORMS is not set
# CONFIG_X86_PLATFORM_DEVICES is not set
# CONFIG_IOMMU_SUPPORT is not set
# CONFIG_DNOTIFY is not set
# CONFIG_INOTIFY_USER is not set
CONFIG_TMPFS=y
CONFIG_TMPFS_POSIX_ACL=y
# CONFIG_MISC_FILESYSTEMS is not set
CONFIG_PRINTK_TIME=y
# CONFIG_SYMBOLIC_ERRNAME is not set
# CONFIG_DEBUG_MISC is not set
CONFIG_DEBUG_INFO_DWARF_TOOLCHAIN_DEFAULT=y
# CONFIG_SECTION_MISMATCH_WARN_ONLY is not set
CONFIG_MAGIC_SYSRQ=y
CONFIG_DEBUG_FS=y
CONFIG_KASAN=y
CONFIG_PROVE_LOCKING=y
CONFIG_DEBUG_ATOMIC_SLEEP=y
# CONFIG_RCU_TRACE is not set
CONFIG_FUNCTION_TRACER=y
# CONFIG_X86_VERBOSE_BOOTUP is not set
# CONFIG_X86_DEBUG_FPU is not set
# CONFIG_RUNTIME_TESTING_MENU is not set
```

A defconfig is expanded into a full `.config` by `make olddefconfig`, which gives every option not mentioned its default. The test below does that in a scratch directory and compares the result with the configuration of the kernel we built — they are identical, so the defconfig *is* the configuration:

```console
$ rm -rf /tmp/cfgcheck && mkdir /tmp/cfgcheck && cp book_defconfig /tmp/cfgcheck/.config && make -s -C linux O=/tmp/cfgcheck ARCH=x86_64 olddefconfig && diff <(grep '^CONFIG' /tmp/cfgcheck/.config) <(grep '^CONFIG' build/.config) && echo "identical: $(grep -c '^CONFIG_.*=' build/.config) options set"
identical: 900 options set
$ linux/scripts/config --file build/.config -s KASAN -s PROVE_LOCKING -s DEBUG_INFO -s BLOCK -s ACPI -s NR_CPUS -s HZ -s PREEMPT_NONE
y
y
y
n
y
2
250
y
$ grep -c '^CONFIG_.*=' /tmp/cfgcheck/.config && make -s -C linux O=/tmp/cfgcheck ARCH=x86_64 x86_64_defconfig && echo "x86_64_defconfig sets $(grep -c '^CONFIG_.*=' /tmp/cfgcheck/.config)"
900
x86_64_defconfig sets 1699
```

Kconfig resolved our ~100 lines into well over a thousand settings, following `depends on`, `select`, and defaults. The distribution kernels you run every day enable several thousand.

## Topic 8B — Build and boot

## Unit 8.3 — Kbuild: building, rebuilding, and what comes out

**One-line summary:** `make O=../build -j$(nproc)` builds out of tree. Kbuild walks the directories through their Makefiles (`obj-y += foo.o` adds `foo.o` to that directory's `built-in.a`), compiles every selected file once, links everything into **`vmlinux`** (an ordinary ELF file: the kernel with all symbols — and, with `DEBUG_INFO`, full debug information), and wraps a compressed copy plus real-mode setup code into **`arch/x86/boot/bzImage`**, the file a bootloader loads. **`System.map`** lists every symbol's address. Kbuild records, for every object, the exact command line and every header and CONFIG option it depended on (`.cmd` files), so after an edit only what is affected is rebuilt.

```console
$ cat build/build-time.txt; ls -l build/vmlinux build/arch/x86/boot/bzImage build/System.map | awk '{printf "%10d  %s\n", $5, $9}'
Full build of this configuration (make -j3 bzImage modules, 2 CPU cores, GCC 13.3.0):
  wall-clock time 5m12.933s
   2458458  build/System.map
   5051392  build/arch/x86/boot/bzImage
 159147464  build/vmlinux
$ file build/vmlinux | cut -d, -f1-2; size build/vmlinux; grep -E " (start_kernel|schedule|do_sys_open|kmalloc_trace_noprof)$" build/System.map
build/vmlinux: ELF 64-bit LSB executable, x86-64
   text	   data	    bss	    dec	    hex	filename
20005388	9402706	12988420	42396514	286eb62	build/vmlinux
ffffffff8149d771 T do_sys_open
ffffffff81c2e7fe T schedule
ffffffff82cc7e66 T start_kernel
$ cat build/kernel/sched/.core.o.cmd | head -1 | tr ' ' '\n' | grep -E "^-(O|f(no-)?sanitize|fsanitize|m|std)" | head -12 | tr '\n' ' '; echo
-std=gnu11 -mno-sse -mno-mmx -mno-sse2 -mno-3dnow -mno-avx -m64 -mno-80387 -mno-fp-ret-in-387 -mpreferred-stack-boundary=3 -mskip-rax-setup -mtune=generic 
```

The `.cmd` file shows the real compiler flags for `kernel/sched/core.c`: `-std=gnu11`, `-mno-red-zone`, `-mno-sse` and friends — the same decisions our hobby kernel made in Chapter 2, for the same reasons — plus the KASAN instrumentation options. An incremental rebuild after touching one file:

```console
$ touch linux/kernel/sched/core.c && cd linux && { time make -s O=../build ARCH=x86_64 -j3 bzImage; } 2>&1 | grep real
real	0m20.254s
```

Only `core.o` is recompiled; most of the time goes to relinking `vmlinux` and recompressing `bzImage`. That is the loop you will live in when changing the kernel itself — which is why the next chapters work with **modules**, which need no relink and no reboot.

## Unit 8.4 — An initramfs and the first boot

**One-line summary:** After initialising itself, the kernel needs a first user-space program: it looks for `/init` in the **initramfs** (a `cpio` archive loaded next to the kernel and unpacked into a RAM file system) or else mounts a root file system from a disk. Our initramfs contains static BusyBox (one binary that is `sh`, `mount`, `ls`, `dmesg`, … depending on the name it is called by) and an `/init` shell script. QEMU boots the kernel directly with `-kernel` and `-initrd`; `console=ttyS0` sends kernel messages to the serial port, which `-nographic` connects to our terminal.

```sh
# file: mkinitramfs.sh
#!/bin/sh
# mkinitramfs.sh OUT.cpio INIT [EXTRA_DIR]: BusyBox + an init script (+ optional extra files) -> newc cpio archive
set -e
out=$(realpath -m "$1"); init=$2; extra=$3
root=$(mktemp -d)
mkdir -p "$root/bin" "$root/sbin" "$root/proc" "$root/sys" "$root/dev" "$root/tmp" "$root/etc" "$root/root"
cp /usr/bin/busybox "$root/bin/busybox"
for applet in $(/usr/bin/busybox --list); do
    [ -e "$root/bin/$applet" ] || ln -s busybox "$root/bin/$applet"
done
{ echo '#!/bin/sh'; cat "$init"; } > "$root/init"      # the kernel executes /init directly
chmod +x "$root/init"
[ -n "$extra" ] && cp -r "$extra"/. "$root/"
(cd "$root" && find . | sort | cpio --quiet -o -H newc) > "$out"
rm -rf "$root"
echo "$out: $(wc -c < "$out") bytes"
```

The kernel starts `/init` with `execve`, exactly as a shell starts a program, so a script needs a `#!/bin/sh` first line. `mkinitramfs.sh` writes that line itself; the init scripts in this book are plain lists of commands.

```sh
# file: init-hello.sh
# The first user-space program: PID 1.
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev
echo "init: hello from PID $$, running $(uname -sr) on $(nproc) CPUs"
echo "init: command line: $(cat /proc/cmdline)"
echo "init: memory: $(grep MemTotal /proc/meminfo)"
echo "init: kernel config has $(zcat /proc/config.gz | grep -c '=y') options set to y"
echo "init: debugging features:" $(zcat /proc/config.gz | grep -oE '^CONFIG_(KASAN|PROVE_LOCKING|DEBUG_ATOMIC_SLEEP)=y' | cut -c8-)
echo "init: uptime at this point: $(cut -d' ' -f1 /proc/uptime) s"
reboot -f                                   # with -no-reboot, the reboot makes QEMU exit
```

```sh
# file: boot.sh
# boot.sh INITRAMFS [EXTRA KERNEL ARGS...]: boot the book's kernel in QEMU on the serial console.
# Run with bash. Firmware chatter (SeaBIOS, the network card's iPXE ROM) and terminal escape codes are filtered out.
set -o pipefail
initrd=$1; shift
timeout 300 qemu-system-x86_64 -kernel build/arch/x86/boot/bzImage -initrd "$initrd" \
    -append "console=ttyS0 panic=-1 $*" -m 512M -smp 2 -nographic -no-reboot $QEMU_EXTRA < /dev/null 2>&1 |
    sed -u -e 's/\x1b\[[0-9;?]*[a-zA-Z]//g' -e 's/\x1bc//g' -e 's/\r$//' -e 's/^Booting from ROM\.*//' |
    grep -av --line-buffered -E '^(SeaBIOS|iPXE|Press Ctrl-B|$)|reboot: Restarting system'
```

```console
$ sh mkinitramfs.sh hello.cpio init-hello.sh && bash boot.sh hello.cpio quiet; echo "[qemu exit status $?]"
hello.cpio: 2162176 bytes
init: hello from PID 1, running Linux 6.12.0-book on 2 CPUs
init: command line: console=ttyS0 panic=-1 quiet
init: memory: MemTotal:         386272 kB
init: kernel config has 819 options set to y
init: debugging features: KASAN=y PROVE_LOCKING=y DEBUG_ATOMIC_SLEEP=y
init: uptime at this point: 12.40 s
[qemu exit status 0]
$ bash boot.sh hello.cpio 2>&1 | grep -E "Linux version|Command line|Memory:|smpboot: Total|KernelAddressSanitizer|Unpacking initramfs|Freeing unused|Run /init|init: hello" | sed -E 's/\(gcc[^)]*\)[^#]*//' | cut -c1-110
[    0.000000] Linux version 6.12.0-book (root@vm) #7 SMP Sun Nov 15 23:45:02 UTC 2026
[    0.000000] Command line: console=ttyS0 panic=-1 
[    0.092872] kasan: KernelAddressSanitizer initialized
[    0.830652] smpboot: Total of 2 processors activated (9201.24 BogoMIPS)
[    0.842761] Memory: 377696K/523768K available (16384K kernel code, 4013K rwdata, 4460K rodata, 1420K init, 
[    2.133907] Unpacking initramfs...
[   11.724022] Freeing unused kernel image (initmem) memory: 1420K
[   11.733745] Freeing unused kernel image (rodata/data gap) memory: 1684K
[   11.734435] Run /init as init process
init: hello from PID 1, running Linux 6.12.0-book on 2 CPUs
```

The second boot, without `quiet`, shows the kernel's own milestones: the version line, the command line, the memory map summary, both CPUs online, KASAN initialised, the initramfs unpacked, the kernel's init memory freed, and finally `Run /init as init process` — the point where `start_kernel`'s work ends and user space begins. Everything before that is the kernel's equivalent of our `kmain`; with KASAN and lockdep compiled in, it takes several seconds under emulation.

### Deep & Tricky

**DT-18. PID 1 must never exit. If `/init` returns — even successfully — the kernel panics with "Attempted to kill init!", because orphaned processes are re-parented to PID 1 and it is the reaper of last resort. A real `init` loops forever (or execs into a program that does); a test `init` must end by rebooting or powering off.** *(Linux `kernel/exit.c`, `do_exit()` → `exit_notify` path: `panic("Attempted to kill init! exitcode=0x%08x\n", ...)` when the exiting task is the child reaper of the initial PID namespace)*

```sh
# file: init-exits.sh
echo "init: I am PID $$ and I am about to return"
exit 0
```

```console
$ sh mkinitramfs.sh exits.cpio init-exits.sh > /dev/null && bash boot.sh exits.cpio 2>&1 | grep -E "init: I am|Kernel panic|Rebooting in" | cut -c1-90
init: I am PID 1 and I am about to return
[   11.835110] Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000000
```

**DT-19. The initramfs must be a cpio archive in the "newc" (or "crc") format. GNU `cpio`'s default output format is the old binary format, which the kernel does not accept: it reports that unpacking failed, has no `/init`, and — with no block devices to mount a root from — cannot start any user space at all.** *(Linux `Documentation/driver-api/early-userspace/buffer-format.rst`: the format "is a SVR4 'newc' (ASCII header) or 'crc'"; `init/initramfs.c` rejects anything else)*

```console
$ d=$(mktemp -d) && cpio --quiet -i -D "$d" < hello.cpio && (cd "$d" && find . | cpio --quiet -o) > oldformat.cpio && rm -rf "$d" && od -c oldformat.cpio | head -1 && od -c hello.cpio | head -1
0000000 307   q  \0 376   H  \0 300   A  \0  \0  \0  \0  \n  \0  \0  \0
0000000   0   7   0   7   0   1   0   0   1   7   0   0   7   A   0   0
$ bash boot.sh oldformat.cpio 2>&1 | grep -iE "initramfs|cpio|kernel panic|no working init" | cut -c1-100
[    2.150166] Unpacking initramfs...
[    2.150731] Initramfs unpacking failed: invalid magic at start of compressed archive
[   11.760230] Kernel panic - not syncing: No working init found.  Try passing init= option to kerne
```

The first bytes tell the formats apart: a newc archive begins with the ASCII magic `070701`; the old binary format begins with the 16-bit magic 070707 written as binary (`307 q` in `od -c`). The kernel does not recognise it — it even tries to treat it as a *compressed* archive, hence "invalid magic at start of compressed archive" — and ten seconds later, having found no `/init` and no root device, it panics with "No working init found". The first message says exactly what went wrong, but only to someone who reads the boot log.

### Pattern

**P-9. Make the kernel development loop a single scripted command with a pass/fail exit status: build the kernel (incrementally), build the initramfs (with the tests inside), boot under QEMU with a timeout, let the guest run the tests and print a machine-readable result, and let the host script turn that into an exit code.** *(Practice; the kernel's own tools follow this design: `tools/testing/selftests/` with `make kselftest`, KUnit's `tools/testing/kunit/kunit.py run`, and `virtme-ng`)* Booting by hand teaches; booting by script makes every later experiment cheap enough to repeat, bisectable (`git bisect run`), and safe to run in CI. *Trade-off:* QEMU without hardware acceleration is slow, and a debug kernel is slower still — keep the guest's work small, and keep a second, non-debug build for performance measurements.

### Mastery check

1. **Predict.** If you delete `CONFIG_E1000=y` from `book_defconfig` and run `olddefconfig`, will the e1000 driver be built? Why?
2. **Predict.** Why can the kernel's `bzImage` be loaded by GRUB, by QEMU's `-kernel`, and by UEFI firmware directly?
3. **Predict.** The `init-hello.sh` script mounts `devtmpfs` itself, although the configuration has `DEVTMPFS_MOUNT=y`. Why is that needed with an initramfs?

## Chapter 8 Project — `ktest`: one command from source to test results

**Goal:** a host script `ktest.sh` that (1) rebuilds the kernel if needed, (2) compiles a static test program for the guest, (3) builds an initramfs whose `/init` runs every test in `/tests` with a time limit and prints `TEST <name>: PASS` or `FAIL`, then `KTEST SUMMARY passed=N failed=M`, then reboots, (4) boots it, and (5) exits 0 only if the summary line appeared and nothing failed. A test is any executable in the `tests/` directory; this project ships four — shell tests of `/proc` and `/sys`, and a compiled C test of system calls.

```c
// file: guest/syscalls.c
/* Runs inside the guest: a few system calls and their observable results. */
#define _GNU_SOURCE
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/utsname.h>
#include <sys/wait.h>
#include <time.h>
#include <unistd.h>

static int failures;
#define CHECK(cond) do { if (!(cond)) { printf("  check failed: %s (errno %d)\n", #cond, errno); failures++; } } while (0)

int main(void) {
    struct utsname u;
    CHECK(uname(&u) == 0 && strstr(u.release, "-book") != NULL);
    printf("  uname: %s %s\n", u.sysname, u.release);

    pid_t child = fork();
    if (child == 0) _exit(42);
    int status;
    CHECK(waitpid(child, &status, 0) == child && WIFEXITED(status) && WEXITSTATUS(status) == 42);

    char *p = mmap(NULL, 1 << 20, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    CHECK(p != MAP_FAILED);
    for (int i = 0; i < (1 << 20); i += 4096) p[i] = 1;          /* demand paging: 256 page faults */
    CHECK(munmap(p, 1 << 20) == 0);

    CHECK(open("/no/such/file", O_RDONLY) == -1 && errno == ENOENT);
    errno = 0;
    const char *volatile bad = NULL;                              /* volatile: hide the NULL from the compiler */
    CHECK(write(1, bad, 1) == -1 && errno == EFAULT);             /* the kernel validates user pointers */

    struct timespec a, b;
    clock_gettime(CLOCK_MONOTONIC, &a);
    usleep(20000);
    clock_gettime(CLOCK_MONOTONIC, &b);
    long ms = (b.tv_sec - a.tv_sec) * 1000 + (b.tv_nsec - a.tv_nsec) / 1000000;
    CHECK(ms >= 20 && ms < 1000);
    printf("  usleep(20 ms) took %ld ms\n", ms);
    return failures ? 1 : 0;
}
```

```sh
# file: tests/10-proc.sh
# /proc reflects the running kernel
[ "$(cat /proc/sys/kernel/osrelease)" = "$(uname -r)" ] || exit 1
grep -q "^processor" /proc/cpuinfo || exit 1
[ "$(grep -c ^processor /proc/cpuinfo)" -eq 2 ] || exit 1
[ -r /proc/self/maps ] && grep -q "\[stack\]" /proc/self/maps
```

```sh
# file: tests/20-sysfs.sh
# the e1000 card is visible as a PCI device and a network interface
ls /sys/bus/pci/drivers/e1000/ | grep -q "0000:" || exit 1
[ -d /sys/class/net/eth0 ] || exit 1
echo "  eth0 MAC: $(cat /sys/class/net/eth0/address)"
```

```sh
# file: tests/30-syscalls.sh
exec /bin/syscalls
```

```sh
# file: tests/40-expected-failure.sh
# Shows that failures are reported. Remove it to get a green run.
[ -e /this/file/does/not/exist ]
```

```sh
# file: init-ktest.sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev
passed=0; failed=0
for t in /tests/*; do
    name=$(basename "$t")
    if timeout 60 sh "$t"; then echo "TEST $name: PASS"; passed=$((passed + 1))
    else echo "TEST $name: FAIL"; failed=$((failed + 1)); fi
done
echo "KTEST SUMMARY passed=$passed failed=$failed"
reboot -f
```

```sh
# file: ktest.sh
#!/bin/sh
# ktest.sh [TESTDIR]: rebuild the kernel if needed, build the guest, boot, run the tests, report.
set -e
tests=${1:-tests}
here=$(pwd)
(cd linux && make -s O=../build ARCH=x86_64 -j"$(nproc)" bzImage)
work=$(mktemp -d)
mkdir -p "$work/bin" "$work/tests"
gcc -static -O2 -Wall -Wextra -Werror -o "$work/bin/syscalls" guest/syscalls.c
cp "$tests"/* "$work/tests/" && chmod +x "$work/tests/"*
sh mkinitramfs.sh "$here/ktest.cpio" init-ktest.sh "$work" > /dev/null
rm -rf "$work"
bash boot.sh ktest.cpio quiet > ktest.log 2>&1 || true
grep -E "^(TEST|KTEST|  )" ktest.log | tr -d '\r'
summary=$(grep -a "KTEST SUMMARY" ktest.log | tr -d '\r') || { echo "ktest: no summary - the guest did not finish"; exit 2; }
case "$summary" in *"failed=0"*) exit 0 ;; *) exit 1 ;; esac
```

```console
$ sh ktest.sh; echo "ktest exit status: $?"
TEST 10-proc.sh: PASS
  eth0 MAC: 52:54:00:12:34:56
TEST 20-sysfs.sh: PASS
  uname: Linux 6.12.0-book
  usleep(20 ms) took 20 ms
TEST 30-syscalls.sh: PASS
TEST 40-expected-failure.sh: FAIL
KTEST SUMMARY passed=3 failed=1
ktest exit status: 1
$ mkdir -p tests-green && cp tests/10-proc.sh tests/20-sysfs.sh tests/30-syscalls.sh tests-green/ && sh ktest.sh tests-green; echo "ktest exit status: $?"
TEST 10-proc.sh: PASS
  eth0 MAC: 52:54:00:12:34:56
TEST 20-sysfs.sh: PASS
  uname: Linux 6.12.0-book
  usleep(20 ms) took 20 ms
TEST 30-syscalls.sh: PASS
KTEST SUMMARY passed=3 failed=0
ktest exit status: 0
```

From now on, every experiment in Part II is "write a test, run `ktest.sh`". The exit status makes it usable with `git bisect run` — the tool that finds the commit that broke something by booting a few dozen kernels for you.

### Review checklist

- What happens if the guest hangs (a test spins forever, or the kernel deadlocks)? Which line of which script limits the damage?
- Why is `guest/syscalls.c` linked statically?
- `ktest.sh` greps the log for `KTEST SUMMARY`. What could make a broken kernel print a fake summary, and how would you make the protocol robust?

## Answers to the Chapter 8 prediction questions

**8.4 Q1.** No. In a defconfig file a missing line means "use the default", and the `E1000` entry printed in Unit 8.2 has no `default` line at all — so its default is `n`. (Its dependencies, `PCI` and `HAS_IOPORT`, would still be met; they only decide whether the option may be enabled.) This is why `savedefconfig` wrote `CONFIG_E1000=y` into the file: it records exactly the values that differ from the defaults.

**8.4 Q2.** `bzImage` is a multi-format file: it starts with the legacy boot sector and the x86 **boot protocol** header (`Documentation/arch/x86/boot.rst`) that GRUB and QEMU understand, and it also carries a PE/COFF header, so UEFI firmware can start it directly as an EFI application (the "EFI stub").

**8.4 Q3.** `DEVTMPFS_MOUNT` mounts devtmpfs on `/dev` only after the kernel mounts a root file system from a device. With an initramfs, the kernel never does that mount — `/init` runs directly from the RAM file system — so `/init` must mount `/dev` itself.

# Chapter 9 — Kernel Modules

*Goal: write code that runs inside the Linux kernel without rebuilding or rebooting it. A **loadable kernel module** is an object file that the running kernel links into itself on request — the mechanism behind most device drivers on a normal system. This chapter covers the module life cycle, parameters, the kernel's own library (memory allocation, linked lists, kernel threads), how modules share symbols, the licence rules attached to that, and what an "oops" — a crash inside kernel code — looks like and how to read it.*

**What you will be able to do after this chapter**

- Build an out-of-tree module against a configured kernel tree and load, inspect, and unload it.
- Use module parameters, `printk` log levels, and the `/sys/module` view of a module.
- Allocate memory with the right GFP flags, use `struct list_head`, and run and stop a kernel thread.
- Export symbols between modules and explain why GPL-only exports refuse a non-GPL module.
- Unwind a failed initialisation correctly with `goto`, and read an oops down to the faulting source line.

The chapter reuses Chapter 8's `mkinitramfs.sh` and `boot.sh`; each module is built on the host and copied into the initramfs, where `/init` loads it.

```console
$ ln -sfn /home/claude/kernelsrc/linux linux && ln -sfn /home/claude/kernelsrc/build build && cp ../ch08/mkinitramfs.sh ../ch08/boot.sh . && echo ready
ready
```

## Topic 9A — The module life cycle

## Unit 9.1 — Hello, kernel

**One-line summary:** A module has an **init** function, called when it is loaded (`insmod`), and an **exit** function, called when it is removed (`rmmod`). It declares them with `module_init()` and `module_exit()`, and declares its licence with `MODULE_LICENSE()`. It is built by the kernel's own build system: a one-line `Makefile` (`obj-m += hello.o`) and `make -C <kernel build dir> M=$PWD modules`, which compiles it with exactly the flags, configuration, and headers of that kernel and produces `hello.ko`. Messages go to the kernel log with `printk` (or `pr_info`, `pr_err`, …), read with `dmesg`.

```c
// file: hello/hello.c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/printk.h>
#include <linux/sched.h>
#include <linux/utsname.h>

static int __init hello_init(void)
{
	pr_info("hello: loaded into %s, by process %s (pid %d)\n",
		init_utsname()->release, current->comm, current->pid);
	return 0;		/* 0: success; a negative errno makes insmod fail */
}

static void __exit hello_exit(void)
{
	pr_info("hello: unloaded by %s (pid %d)\n", current->comm, current->pid);
}

module_init(hello_init);
module_exit(hello_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Linux & OS Development book");
MODULE_DESCRIPTION("The smallest useful module");
```

```text
# file: hello/Makefile
obj-m += hello.o
```

The code uses the kernel's coding style (tabs, braces on their own line for functions) — the style the tools in Chapter 14 check. `current` is the kernel's pointer to the running task: the process that called `insmod`.

```console
$ make -s -C build M=$PWD/hello modules && ls -l hello/hello.ko | awk '{print $5, $9}' && modinfo hello/hello.ko | grep -vE "^(srcversion|filename)"
137456 hello/hello.ko
description:    The smallest useful module
author:         Linux & OS Development book
license:        GPL
depends:        
name:           hello
vermagic:       6.12.0-book SMP mod_unload 
```

`vermagic` records the exact kernel version and the configuration features that affect binary compatibility (SMP, preemption model, `mod_unload`); the kernel refuses a module whose vermagic does not match. A module is *not* a portable binary: it is built for one kernel build.

```sh
# file: init-modules.sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev
dmesg -n 1                       # keep the console quiet; we print what we want ourselves
for script in /run-*.sh; do sh "$script"; done
reboot -f
```

```sh
# file: guest-hello/run-hello.sh
cd /modules
insmod hello.ko
grep hello /proc/modules
ls /sys/module/hello/
cat /sys/module/hello/initstate
rmmod hello
dmesg | grep "hello:" | sed 's/^\[[ 0-9.]*\] //'
```

```console
$ mkdir -p g1/modules && cp hello/hello.ko g1/modules/ && cp guest-hello/run-hello.sh g1/ && sh mkinitramfs.sh g1.cpio init-modules.sh g1 > /dev/null && bash boot.sh g1.cpio quiet 2>&1 | grep -v "^reboot"
hello 12288 0 - Live 0xffffffffa0208000 (O)
coresize   initsize   notes      sections   uevent
holders    initstate  refcnt     taint
live
hello: loading out-of-tree module taints kernel.
hello: loaded into 6.12.0-book, by process insmod (pid 46)
hello: unloaded by rmmod (pid 50)
```

`/proc/modules` shows the size in memory and the use count; `/sys/module/hello/` is its directory in sysfs; `initstate` says `live`. The log shows that `insmod` and `rmmod` were ordinary processes making system calls (`finit_module` and `delete_module`); the module's code ran in their context.

## Unit 9.2 — Parameters, `__init`, and failing cleanly

**One-line summary:** `module_param(name, type, perm)` makes a static variable settable at load time (`insmod m.ko name=value`) and, if `perm` is not 0, visible (and possibly writable) as `/sys/module/<m>/parameters/<name>`. Functions and data marked `__init` / `__initdata` are put in a separate section that the kernel **frees** after the init function returns. If init fails, it must return a negative errno and undo everything it did so far — in reverse order, usually with a chain of `goto` labels.

```c
// file: params/params.c
#include <linux/module.h>
#include <linux/moduleparam.h>
#include <linux/slab.h>

static int count = 3;
module_param(count, int, 0644);			/* rw-r--r--: root may change it at run time */
MODULE_PARM_DESC(count, "how many buffers to allocate (1-16)");

static char *label = "default";
module_param(label, charp, 0444);			/* read-only in sysfs */
MODULE_PARM_DESC(label, "a name to print");

static int fail_at = -1;
module_param(fail_at, int, 0);				/* not visible in sysfs at all */
MODULE_PARM_DESC(fail_at, "simulate an allocation failure at this buffer");

static void **buffers;

static int __init params_init(void)
{
	int i, err;

	if (count < 1 || count > 16) {
		pr_err("params: count=%d is out of range\n", count);
		return -EINVAL;
	}
	buffers = kcalloc(count, sizeof(*buffers), GFP_KERNEL);
	if (!buffers)
		return -ENOMEM;
	for (i = 0; i < count; i++) {
		buffers[i] = (i == fail_at) ? NULL : kmalloc(4096, GFP_KERNEL);
		if (!buffers[i]) {
			err = -ENOMEM;
			goto free_buffers;
		}
	}
	pr_info("params: label=%s, %d buffers allocated\n", label, count);
	return 0;

free_buffers:					/* undo in reverse order */
	pr_err("params: allocation %d failed, freeing %d earlier ones\n", i, i);
	while (--i >= 0)
		kfree(buffers[i]);
	kfree(buffers);
	return err;
}

static void __exit params_exit(void)
{
	int i;

	for (i = 0; i < count; i++)
		kfree(buffers[i]);
	kfree(buffers);
	pr_info("params: %d buffers freed\n", count);
}

module_init(params_init);
module_exit(params_exit);
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Module parameters and error unwinding");
```

```text
# file: params/Makefile
obj-m += params.o
```

```sh
# file: guest-params/run-params.sh
cd /modules
insmod params.ko count=4 label=first && cat /sys/module/params/parameters/count /sys/module/params/parameters/label
ls -l /sys/module/params/parameters/ | awk '{print $1, $NF}'
rmmod params
insmod params.ko count=99; echo "insmod exit status $?"
insmod params.ko count=5 fail_at=3; echo "insmod exit status $?"
grep -c params /proc/modules
dmesg | grep "params:" | sed 's/^\[[ 0-9.]*\] //'
```

```console
$ make -s -C build M=$PWD/params modules && mkdir -p g2/modules && cp params/params.ko g2/modules/ && cp guest-params/run-params.sh g2/ && sh mkinitramfs.sh g2.cpio init-modules.sh g2 > /dev/null && bash boot.sh g2.cpio quiet 2>&1 | grep -v "^reboot"
4
first
total 0
-rw-r--r-- count
-r--r--r-- label
insmod: can't insert 'params.ko': Invalid argument
insmod exit status 22
insmod: can't insert 'params.ko': Cannot allocate memory
insmod exit status 12
0
params: loading out-of-tree module taints kernel.
params: label=first, 4 buffers allocated
params: 4 buffers freed
params: count=99 is out of range
params: count=99 is out of range
params: allocation 3 failed, freeing 3 earlier ones
params: allocation 3 failed, freeing 3 earlier ones
```

`fail_at` has permission 0, so it has no file in `/sys`. Each failure message appears twice: BusyBox's `insmod` first tries the `finit_module` system call and, when that fails, retries with `init_module` — so the module's init runs and fails twice. A failed init leaves no trace: the module is not in `/proc/modules`, and its partial allocations were freed — with KASAN and the slab debugging that comes with it, a leak or a double free on this path would have been reported.

### Pattern

**P-10. Initialise in steps, and on failure jump to a label that undoes exactly the steps already done, in reverse order; the exit function undoes all of them in the same reverse order.** *(Linux `Documentation/process/coding-style.rst`, §7 "Centralized exiting of functions": "The goto statement comes in handy when a function exits from multiple locations and some common work such as cleanup has to be done")* Every driver's `probe()` function in the kernel follows this shape, and kernel reviewers check it first. Newer code can also use scope-based cleanup (`__free(kfree)`, `guard(mutex)` from `<linux/cleanup.h>`, added in 6.5), which runs the cleanup automatically when a variable goes out of scope. *Trade-off:* goto ladders are explicit and easy to review but repetitive and error-prone to extend; scope-based cleanup is shorter but hides control flow and is only gradually being adopted.

## Topic 9B — Using the kernel's library

## Unit 9.3 — Lists, kernel threads, and sleeping

**One-line summary:** The kernel's generic doubly-linked list is **intrusive**: a `struct list_head` is embedded *inside* your structure, and `list_entry()` / `container_of()` gets from the embedded link back to the containing object by subtracting the member's offset. A **kernel thread** (`kthread_run`) is a task with no user address space; it runs a function until it returns, checking `kthread_should_stop()` so that `kthread_stop()` can end it. Memory is allocated with `kmalloc(size, GFP_KERNEL)` in contexts that may sleep, and `GFP_ATOMIC` in contexts that may not (interrupt handlers, while holding a spinlock).

```c
// file: sampler/sampler.c
#include <linux/delay.h>
#include <linux/jiffies.h>
#include <linux/kthread.h>
#include <linux/list.h>
#include <linux/module.h>
#include <linux/mutex.h>
#include <linux/sched/signal.h>
#include <linux/slab.h>

struct sample {
	struct list_head node;			/* embedded link: the list does not own a pointer to us */
	unsigned long jiffies;
	unsigned int nr_tasks;
};

static LIST_HEAD(samples);
static DEFINE_MUTEX(samples_lock);
static struct task_struct *worker;
static unsigned int period_ms = 50;
module_param(period_ms, uint, 0444);

static unsigned int count_tasks(void)
{
	struct task_struct *p;
	unsigned int n = 0;

	rcu_read_lock();			/* the task list is protected by RCU for readers (Chapter 12) */
	for_each_process(p)
		n++;
	rcu_read_unlock();
	return n;
}

static int sampler_fn(void *unused)
{
	while (!kthread_should_stop()) {
		struct sample *s = kmalloc(sizeof(*s), GFP_KERNEL);	/* may sleep: fine in a thread */

		if (s) {
			s->jiffies = jiffies;
			s->nr_tasks = count_tasks();
			mutex_lock(&samples_lock);
			list_add_tail(&s->node, &samples);
			mutex_unlock(&samples_lock);
		}
		msleep(period_ms);
	}
	return 0;
}

static int __init sampler_init(void)
{
	worker = kthread_run(sampler_fn, NULL, "sampler");
	if (IS_ERR(worker))
		return PTR_ERR(worker);
	pr_info("sampler: thread pid %d started, period %u ms\n", worker->pid, period_ms);
	return 0;
}

static void __exit sampler_exit(void)
{
	struct sample *s, *tmp;
	unsigned int n = 0, min = UINT_MAX, max = 0;
	unsigned long first = 0, last = 0;

	kthread_stop(worker);			/* waits until sampler_fn has returned */
	list_for_each_entry_safe(s, tmp, &samples, node) {	/* _safe: we free while walking */
		if (!n++)
			first = s->jiffies;
		last = s->jiffies;
		min = min(min, s->nr_tasks);
		max = max(max, s->nr_tasks);
		list_del(&s->node);
		kfree(s);
	}
	pr_info("sampler: %u samples over %u ms, tasks min %u max %u (HZ=%d)\n", n,
		jiffies_to_msecs(last - first), min, max, HZ);
}

module_init(sampler_init);
module_exit(sampler_exit);
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("A kernel thread that samples the number of tasks into a list");
```

```text
# file: sampler/Makefile
obj-m += sampler.o
```

```sh
# file: guest-sampler/run-sampler.sh
cd /modules
insmod sampler.ko period_ms=20
ps -o pid,ppid,comm | grep -E "PID|sampler|kthreadd"
sleep 1
for i in 1 2 3 4 5; do sleep 10 & done           # five more tasks for a while
sleep 1
rmmod sampler
dmesg | grep "sampler:" | sed 's/^\[[ 0-9.]*\] //'
```

```console
$ make -s -C build M=$PWD/sampler modules && mkdir -p g3/modules && cp sampler/sampler.ko g3/modules/ && cp guest-sampler/run-sampler.sh g3/ && sh mkinitramfs.sh g3.cpio init-modules.sh g3 > /dev/null && bash boot.sh g3.cpio quiet 2>&1 | grep -v "^reboot"
PID   PPID  COMMAND
    2     0 kthreadd
   47     2 sampler
sampler: loading out-of-tree module taints kernel.
sampler: thread pid 47 started, period 20 ms
sampler: 89 samples over 2120 ms, tasks min 42 max 47 (HZ=250)
```

The thread appears in `ps` as a child of `kthreadd` (PID 2), the parent of every kernel thread. About two seconds at 20 ms gives roughly 100 samples — fewer if the sleeps run long, because `msleep` rounds up to whole jiffies (4 ms at `HZ=250`) and the loop's own work adds to each period. The task count rises by five while the `sleep` processes exist.

## Unit 9.4 — Exported symbols, licences, and module dependencies

**One-line summary:** A module can use only kernel symbols that are **exported**: `EXPORT_SYMBOL(f)` makes `f` available to every module, `EXPORT_SYMBOL_GPL(f)` only to modules whose licence is GPL-compatible. Modules can export symbols to each other, which creates a **dependency**: the provider must be loaded first, and cannot be unloaded while a user holds it (its reference count is visible in `/proc/modules`). `modprobe` resolves dependencies automatically from `modules.dep`; `insmod` does not.

```c
// file: provider/provider.h
/* The provider's interface: included by the provider itself and by its users. */
#ifndef PROVIDER_H
#define PROVIDER_H
int provider_add(int a, int b);
#endif
```

```c
// file: provider/provider.c
#include <linux/module.h>
#include "provider.h"

static atomic_t calls = ATOMIC_INIT(0);

int provider_add(int a, int b)
{
	atomic_inc(&calls);
	return a + b;
}
EXPORT_SYMBOL_GPL(provider_add);			/* GPL-compatible modules only */

static int __init provider_init(void) { pr_info("provider: ready\n"); return 0; }
static void __exit provider_exit(void) { pr_info("provider: unloaded after %d calls\n", atomic_read(&calls)); }
module_init(provider_init);
module_exit(provider_exit);
MODULE_LICENSE("GPL");
```

```c
// file: consumer/consumer.c
#include <linux/module.h>
#include "../provider/provider.h"

static int __init consumer_init(void)
{
	pr_info("consumer: provider_add(40, 2) = %d\n", provider_add(40, 2));
	return 0;
}
static void __exit consumer_exit(void) { }
module_init(consumer_init);
module_exit(consumer_exit);
#ifdef CONSUMER_PROPRIETARY
MODULE_LICENSE("Proprietary");
#else
MODULE_LICENSE("GPL");
#endif
```

```c
// file: closed/closed.c
/* A non-GPL module that uses only symbols exported to everyone (printk). */
#include <linux/module.h>

static int __init closed_init(void) { pr_info("closed: a proprietary module says hello\n"); return 0; }
static void __exit closed_exit(void) { }
module_init(closed_init);
module_exit(closed_exit);
MODULE_LICENSE("Proprietary");
```

```text
# file: provider/Makefile
obj-m += provider.o
```

```text
# file: consumer/Makefile
obj-m += consumer.o
ccflags-$(PROPRIETARY) += -DCONSUMER_PROPRIETARY
KBUILD_EXTRA_SYMBOLS := $(src)/../provider/Module.symvers
```

```text
# file: closed/Makefile
obj-m += closed.o
```

`KBUILD_EXTRA_SYMBOLS` tells the build where the provider's export list (`Module.symvers`) is, so that the consumer's build can check the symbol and record the dependency.

```console
$ make -s -C build M=$PWD/provider modules && make -s -C build M=$PWD/consumer modules && cp consumer/consumer.ko consumer-gpl.ko && modinfo -F depends consumer-gpl.ko && make -s -C build M=$PWD/closed modules && echo "built provider, consumer, closed"
provider
built provider, consumer, closed
$ make -s -C build M=$PWD/consumer clean && make -s -C build M=$PWD/consumer PROPRIETARY=y modules 2>&1 | grep -E "ERROR|error" | sed "s|$PWD/||"; ls consumer/*.ko 2>&1 | sed "s|$PWD/||"
ERROR: modpost: GPL-incompatible module consumer.ko uses GPL-only symbol 'provider_add'
ls: cannot access 'consumer/*.ko': No such file or directory
```

The licence is checked twice. At build time, modpost refuses to produce a non-GPL module that uses a GPL-only symbol — the "Proprietary" consumer was never built. At load time the kernel's module loader applies the same rule to modules built by other means. A non-GPL module that only uses ordinary exports, such as `closed.ko`, loads — and marks the kernel as tainted.

```sh
# file: guest-symbols/run-symbols.sh
cd /modules
insmod consumer.ko; echo "consumer without provider: exit status $?"
insmod provider.ko
insmod consumer.ko && grep -E "^(provider|consumer) " /proc/modules | cut -d' ' -f1-4
rmmod provider; echo "rmmod provider while in use: exit status $?"
rmmod consumer && rmmod provider
echo "tainted before closed.ko: $(cat /proc/sys/kernel/tainted)"
insmod closed.ko
echo "tainted after closed.ko:  $(cat /proc/sys/kernel/tainted)"
dmesg | grep -E "consumer|provider|closed|taint" | sed 's/^\[[ 0-9.]*\] //'
```

```console
$ mkdir -p g4/modules && cp provider/provider.ko closed/closed.ko g4/modules/ && cp consumer-gpl.ko g4/modules/consumer.ko && cp guest-symbols/run-symbols.sh g4/ && sh mkinitramfs.sh g4.cpio init-modules.sh g4 > /dev/null && bash boot.sh g4.cpio quiet 2>&1 | grep -v "^reboot"
insmod: can't insert 'consumer.ko': unknown symbol in module, or unknown parameter
consumer without provider: exit status 2
consumer 12288 0 -
provider 12288 1 consumer,
rmmod: can't unload module 'provider': Resource temporarily unavailable
rmmod provider while in use: exit status 1
tainted before closed.ko: 4096
tainted after closed.ko:  4097
consumer: loading out-of-tree module taints kernel.
consumer: Unknown symbol provider_add (err -2)
consumer: Unknown symbol provider_add (err -2)
provider: ready
consumer: provider_add(40, 2) = 42
provider: unloaded after 1 calls
closed: module license 'Proprietary' taints kernel.
Disabling lock debugging due to kernel taint
closed: module license taints kernel.
closed: a proprietary module says hello
```

### Deep & Tricky

**DT-20. The licence string is not decoration: modpost refuses to build, and the module loader refuses to link, a module without a GPL-compatible licence against `EXPORT_SYMBOL_GPL` symbols; and loading any non-GPL module taints the kernel (flag `P`, value 1 in `/proc/sys/kernel/tainted`), which kernel developers read as "bug reports from this kernel are not ours to debug".** *(Linux `Documentation/process/license-rules.rst`, "MODULE_LICENSE"; `Documentation/admin-guide/tainted-kernels.rst`, bit 0 'P' and bit 12 'O'; `scripts/mod/modpost.c`: "GPL-incompatible module … uses GPL-only symbol")* The taint values in the guest decode bit by bit: 4096 (`O`, an out-of-tree module was loaded — every module in this book) before, and 4097 = 4096 + 1 (`P`) after `closed.ko`. Taint flags never clear until reboot. Notice one more line in the log: "Disabling lock debugging due to kernel taint" — once a proprietary module is loaded, lockdep switches itself off, because its reports could no longer be trusted. (The consumer's "Unknown symbol" message appears twice for the same BusyBox reason as in Unit 9.2.)

## Topic 9C — When a module crashes

## Unit 9.5 — Reading an oops

**One-line summary:** When kernel code faults — a NULL pointer dereference, an access to freed memory — Linux does not reboot; it prints an **oops**: the kind of fault and the address, the CPU and task, the instruction pointer as `function+offset/length [module]`, the registers, the code bytes around RIP, and the **call trace**. The faulting task is killed; the kernel continues, but is now tainted (`D`, died) and possibly holding locks forever. `scripts/decode_stacktrace.sh` turns each `function+offset` into a source file and line using the kernel's (and module's) debug information.

```c
// file: buggy/buggy.c
#include <linux/module.h>
#include <linux/slab.h>

struct config {
	int verbose;
	char name[16];
};

static int mode;
module_param(mode, int, 0);

static noinline int read_verbose(const struct config *cfg)
{
	return cfg->verbose;				/* mode 1: cfg is NULL */
}

static noinline void use_after_free(void)
{
	struct config *cfg = kmalloc(sizeof(*cfg), GFP_KERNEL);

	if (!cfg)
		return;
	cfg->verbose = 1;
	kfree(cfg);
	pr_info("buggy: verbose was %d\n", cfg->verbose);	/* mode 2: read after kfree */
}

static int __init buggy_init(void)
{
	struct config *cfg = NULL;

	if (mode == 1)
		pr_info("buggy: verbose=%d\n", read_verbose(cfg));
	if (mode == 2)
		use_after_free();
	return 0;
}

static void __exit buggy_exit(void) { }
module_init(buggy_init);
module_exit(buggy_exit);
MODULE_LICENSE("GPL");
```

```text
# file: buggy/Makefile
obj-m += buggy.o
```

```sh
# file: guest-buggy/run-buggy.sh
cd /modules
dmesg -n 8                                       # show the oops on the console as it happens
insmod buggy.ko mode=1
echo "insmod returned $? - and the system is still running"
cat /proc/sys/kernel/tainted
```

```console
$ make -s -C build M=$PWD/buggy modules && mkdir -p g5/modules && cp buggy/buggy.ko g5/modules/ && cp guest-buggy/run-buggy.sh g5/ && sh mkinitramfs.sh g5.cpio init-modules.sh g5 > /dev/null && bash boot.sh g5.cpio quiet 2>&1 | sed 's/^\[[ 0-9.]*\] //' > oops.txt; grep -vE "^(R[0-9A-HJ-Z][0-9A-Z]|RSP|FS|CS|CR2|DR[0-9]|Code):|^ \? " oops.txt | head -16 | cut -c1-100
buggy: loading out-of-tree module taints kernel.
Oops: general protection fault, maybe for address 0x1: 0000 [#1] SMP KASAN
CPU: 1 UID: 0 PID: 46 Comm: insmod Tainted: G           O       6.12.0-book #7
Tainted: [O]=OOT_MODULE
RIP: 0010:read_verbose.constprop.0+0x5/0x25 [buggy]
Call Trace:
 <TASK>
 buggy_init+0x13/0x1000 [buggy]
 do_one_initcall+0x152/0x2f8
 do_init_module+0x229/0x5b1
 load_module+0x37bc/0x3bc0
 init_module_from_file+0xc4/0xe6
 idempotent_init_module+0x253/0x488
 __x64_sys_finit_module+0xb0/0xcc
 do_syscall_64+0x6a/0xc5
 entry_SYSCALL_64_after_hwframe+0x4b/0x53
$ grep -E "^(RIP|Oops)|^ [a-z_]+[a-z_.0-9]*\+0x" oops.txt | head -5 > trace.txt && linux/scripts/decode_stacktrace.sh build/vmlinux /home/claude/kernelsrc/linux $PWD/buggy < trace.txt | sed "s|$PWD/||"
Oops: general protection fault, maybe for address 0x1: 0000 [#1] SMP KASAN
RIP: 0010:read_verbose.constprop.0 (buggy/buggy.c:15) buggy
buggy_init (include/linux/slab.h:878 buggy/buggy.c:20) buggy
do_one_initcall (init/main.c:1269) 
do_init_module (kernel/module/main.c:2543) 
```

Read the oops from the top. The first line names the fault — and holds a surprise: not the "BUG: kernel NULL pointer dereference" a normal kernel prints, but a **general protection fault**. In a KASAN kernel, every memory access is preceded by a check of its *shadow* byte at `(address >> 3) + 0xdffffc0000000000`; for address 0 that is the non-canonical address `0xdffffc0000000000` itself, and touching a non-canonical address raises #GP before the real load is attempted (CR2, R15 and the KASAN offset in the configuration all show that constant). The `Tainted:` line says which taint flags were set (`O`: out-of-tree module). `RIP: read_verbose.constprop.0+0x5/0x25 [buggy]` names the function, the offset into it and its length, and the module — `.constprop.0` is GCC's copy of `read_verbose` specialised for the constant NULL argument. The call trace shows how we got there: `buggy_init`, called by `do_one_initcall`, called by the module loader on behalf of `insmod` (entries marked `?` were found on the stack but are not proven to be part of the chain; they were filtered out above). `decode_stacktrace.sh` adds file and line from the debug information — `buggy.c` line 15 is `return cfg->verbose;` (the files in this book's code directory carry a one-line `// file:` header, so their line numbers are one higher than in the listing). `insmod` was killed (its init call never returned), and the kernel lived on, now tainted with `D` (died).

### Deep & Tricky

**DT-21. A use-after-free usually does *not* crash: the freed memory is still mapped and often still holds the old value, so the bug reads plausible data and survives testing. Only a tool that tracks the state of every allocation — KASAN (shadow memory), or slab poisoning — turns it into a report, and the report says where the object was allocated, where it was freed, and where it was used afterwards.** *(Linux `Documentation/dev-tools/kasan.rst`: "KASAN is a dynamic memory safety error detector designed to find out-of-bounds and use-after-free bugs"; report sections "Allocated by task", "Freed by task")* Mode 2 of the module reads a field after `kfree`. On this kernel, built with KASAN, the read is caught at once:

```sh
# file: guest-uaf/run-uaf.sh
cd /modules
dmesg -n 8
insmod buggy.ko mode=2
echo "insmod returned $?"
```

```console
$ mkdir -p g6/modules && cp buggy/buggy.ko g6/modules/ && cp guest-uaf/run-uaf.sh g6/ && sh mkinitramfs.sh g6.cpio init-modules.sh g6 > /dev/null && bash boot.sh g6.cpio quiet 2>&1 | tr -d '\r' | sed 's/^\[[ 0-9.]*\] //' | grep -E "BUG: KASAN|Read of size|Allocated by|Freed by|^ (use_after_free|kmalloc|kfree|__kasan|kasan_save|buggy_init)|insmod returned|located [0-9]+ bytes" | cut -c1-100
BUG: KASAN: slab-use-after-free in use_after_free+0x9e/0xaf [buggy]
Read of size 4 at addr ffff888000b7dc40 by task insmod/47
 use_after_free+0x9e/0xaf [buggy]
 buggy_init+0x2f/0x1000 [buggy]
Allocated by task 47:
 kasan_save_stack+0x2e/0x49
 kasan_save_track+0x10/0x25
 __kasan_kmalloc+0x3a/0x43
 use_after_free+0x42/0xaf [buggy]
 buggy_init+0x2f/0x1000 [buggy]
Freed by task 47:
 kasan_save_stack+0x2e/0x49
 kasan_save_track+0x10/0x25
 kasan_save_free_info+0x3b/0x41
 __kasan_slab_free+0x32/0x3f
 kfree+0x1bd/0x222
 use_after_free+0x7b/0xaf [buggy]
 buggy_init+0x2f/0x1000 [buggy]
The buggy address is located 0 bytes inside of
insmod returned 0
```

The report names the bug class (slab-use-after-free), the access (a 4-byte read — `int verbose`), and then the two stories that matter: allocated by `use_after_free` and freed by `kfree` from `use_after_free`, with stack traces for both. Without KASAN this code prints "verbose was 1" and passes every test. KASAN reports and continues by default (`kasan.fault=report`); `insmod` succeeded.

### Mastery check

1. **Predict.** Why does a module's `exit` function carry `__exit` — and what happens to it when the module is built into the kernel instead (`obj-y`)?
2. **Predict.** In `sampler_exit`, why must `kthread_stop()` come *before* the list is freed?
3. **Predict.** Why is the list in `sampler` protected by a mutex and not by a spinlock?

## Chapter 9 Project — A self-testing module

**Goal:** turn the module into its own test: `selftest.ko` exercises kernel APIs used in this chapter — lists (insert, sort, remove), `kmalloc`/`krealloc`/`kfree`, a kernel thread, and `container_of` — and reports `selftest: N passed, M failed` from its init function; init returns `-EINVAL` if anything failed, so `insmod`'s exit status *is* the test result. A `run-selftest.sh` script then plugs into Chapter 8's `ktest.sh` as one more test. This is the idea of **KUnit**, the kernel's unit-test framework, in miniature.

```c
// file: selftest/selftest.c
#include <linux/completion.h>
#include <linux/kthread.h>
#include <linux/list.h>
#include <linux/list_sort.h>
#include <linux/module.h>
#include <linux/slab.h>

static int passed, failed;

#define CHECK(cond)								\
	do {									\
		if (cond) {							\
			passed++;						\
		} else {							\
			failed++;						\
			pr_err("selftest: FAILED %s at line %d\n", #cond, __LINE__); \
		}								\
	} while (0)

struct item {
	int key;
	struct list_head node;
};

static int cmp_items(void *priv, const struct list_head *a, const struct list_head *b)
{
	return list_entry(a, struct item, node)->key - list_entry(b, struct item, node)->key;
}

static void test_list(void)
{
	static const int keys[] = { 42, 7, 19, 3, 88, 7 };
	LIST_HEAD(head);
	struct item *it, *tmp;
	int i, prev = INT_MIN, n = 0;

	for (i = 0; i < ARRAY_SIZE(keys); i++) {
		it = kmalloc(sizeof(*it), GFP_KERNEL);
		if (!it) {
			failed++;
			goto out;
		}
		it->key = keys[i];
		list_add(&it->node, &head);		/* at the front: the list is reversed */
	}
	CHECK(list_first_entry(&head, struct item, node)->key == 7);
	list_sort(NULL, &head, cmp_items);
	list_for_each_entry(it, &head, node) {
		CHECK(it->key >= prev);
		prev = it->key;
		n++;
	}
	CHECK(n == ARRAY_SIZE(keys));
	CHECK(list_last_entry(&head, struct item, node)->key == 88);
out:
	list_for_each_entry_safe(it, tmp, &head, node) {
		list_del(&it->node);
		kfree(it);
	}
	CHECK(list_empty(&head));
}

static void test_container_of(void)
{
	struct item it = { .key = 5 };
	struct list_head *link = &it.node;

	CHECK(container_of(link, struct item, node) == &it);
	CHECK(offsetof(struct item, node) == 8);	/* int key + 4 bytes of padding */
}

static void test_alloc(void)
{
	char *p = kmalloc(10, GFP_KERNEL), *q;

	CHECK(p != NULL);
	if (!p)
		return;
	CHECK(ksize(p) >= 10);
	memcpy(p, "kernel!", 8);
	q = krealloc(p, 5000, GFP_KERNEL);	/* contents are preserved */
	CHECK(q && strcmp(q, "kernel!") == 0);
	kfree(q ? q : p);
	CHECK(kmalloc(0, GFP_KERNEL) == ZERO_SIZE_PTR);	/* not NULL: a special non-dereferenceable pointer */
}

static DECLARE_COMPLETION(thread_done);
static int thread_result;

static int thread_fn(void *arg)
{
	thread_result = *(int *)arg * 2;
	complete(&thread_done);
	return 0;
}

static void test_kthread(void)
{
	static int input = 21;
	struct task_struct *t = kthread_run(thread_fn, &input, "selftest-%d", 1);

	CHECK(!IS_ERR(t));
	if (IS_ERR(t))
		return;
	CHECK(wait_for_completion_timeout(&thread_done, HZ) > 0);
	CHECK(thread_result == 42);
}

static int __init selftest_init(void)
{
	test_list();
	test_container_of();
	test_alloc();
	test_kthread();
	pr_info("selftest: %d passed, %d failed\n", passed, failed);
	return failed ? -EINVAL : 0;
}

static void __exit selftest_exit(void) { }
module_init(selftest_init);
module_exit(selftest_exit);
MODULE_LICENSE("GPL");
```

```text
# file: selftest/Makefile
obj-m += selftest.o
```

```sh
# file: guest-selftest/run-selftest.sh
cd /modules
if insmod selftest.ko; then echo "selftest module: PASS"; rmmod selftest; else echo "selftest module: FAIL"; fi
dmesg | grep "selftest:" | sed 's/^\[[ 0-9.]*\] //'
```

```console
$ make -s -C build M=$PWD/selftest modules && mkdir -p g7/modules && cp selftest/selftest.ko g7/modules/ && cp guest-selftest/run-selftest.sh g7/ && sh mkinitramfs.sh g7.cpio init-modules.sh g7 > /dev/null && bash boot.sh g7.cpio quiet 2>&1 | grep -v "^reboot"
selftest module: PASS
selftest: loading out-of-tree module taints kernel.
selftest: 19 passed, 0 failed
```

`kmalloc(0)` returning `ZERO_SIZE_PTR` (the value 16) instead of NULL is one of the kernel's quiet surprises: it is not an error, and dereferencing it faults on purpose. KUnit (`CONFIG_KUNIT`, `tools/testing/kunit/kunit.py`) provides the same structure — test suites, assertions, a machine-readable result (KTAP) — and runs thousands of such tests in every kernel CI.

### Review checklist

- `test_list` jumps to `out` when an allocation fails. Is the cleanup loop correct for a half-built list?
- `thread_done` is a static completion. What happens if the module is loaded twice in a row without a reboot? (Hint: `rmmod` and `insmod` re-run static initialisers?)
- Which of the tests would behave differently on a 32-bit kernel?

## Answers to the Chapter 9 prediction questions

**9.5 Q1.** `__exit` puts the function in the `.exit.text` section. For a loadable module it is kept and called by `rmmod`. When the code is built into the kernel, it can never be unloaded, so the build discards `.exit.text` entirely — the function costs nothing.

**9.5 Q2.** The thread adds samples to the list. If the list were freed first, the still-running thread could `list_add_tail` onto a list whose nodes were just freed (a use-after-free that KASAN would report). `kthread_stop()` waits until the thread function has returned; after that, nobody else touches the list.

**9.5 Q3.** The thread holds the lock while calling `list_add_tail` only, but the code around it (`kmalloc(GFP_KERNEL)`, `msleep`) may sleep, and the exit path frees memory while walking the list. A mutex may be held across code that sleeps; a spinlock may not (with `DEBUG_ATOMIC_SLEEP`, "BUG: sleeping function called from invalid context"). Here no interrupt handler uses the list, so there is no reason for a spinlock.

# Chapter 10 — A Character Device Driver

*Goal: build the most common kind of Linux driver interface — a **character device**, a file in `/dev` whose `read`, `write`, `poll`, and `ioctl` calls go to your code. Ours, `/dev/ring`, is a byte ring buffer shared by every process that opens it: writers block when it is full, readers block when it is empty, `poll` and `select` work, `ioctl` returns statistics and resizes it, and its state is visible in `/proc` and `/sys`. Along the way: the rules for touching user memory, sleeping and waking, and the contract between a driver and user space.*

**What you will be able to do after this chapter**

- Register a misc character device and implement `file_operations`.
- Move data across the user–kernel boundary with `copy_to_user`/`copy_from_user`, and handle their return values correctly.
- Block and wake processes with wait queues, honour `O_NONBLOCK`, implement `poll`, and handle signals (`-ERESTARTSYS`).
- Design `ioctl` commands with `_IOR`/`_IOW` and a shared header.
- Publish state through `/proc` (seq_file) and sysfs attributes.

```console
$ ln -sfn /home/claude/kernelsrc/linux linux && ln -sfn /home/claude/kernelsrc/build build && cp ../ch08/mkinitramfs.sh ../ch08/boot.sh ../ch09/init-modules.sh . && echo ready
ready
```

## Topic 10A — The device and its file operations

## Unit 10.1 — A misc device: open, read, write

**One-line summary:** A character device is identified by a **major:minor** number pair; the file in `/dev` is just a name for that pair. A **misc device** (major 10) is the easiest way to get one: fill a `struct miscdevice` with a name and a `struct file_operations`, call `misc_register()`, and devtmpfs creates `/dev/<name>`. When a process calls `read(fd, buf, n)` on it, the VFS calls your `.read(file, buf, n, &pos)`; `buf` is a **user** address, which the kernel must never dereference directly — it copies with `copy_to_user()`, which checks the address and handles faults.

The interface shared with user space lives in its own header — the same role `include/uapi/` plays for the kernel:

```c
// file: ring/ring_uapi.h
/* The user-space interface of /dev/ring. Included by the driver and by programs. */
#ifndef RING_UAPI_H
#define RING_UAPI_H
#include <linux/ioctl.h>
#include <linux/types.h>

struct ring_stats {
	__u64 bytes_in, bytes_out;
	__u32 reads, writes, opens, capacity, used, reserved;	/* fixed size: 40 bytes on every architecture */
};

#define RING_IOC_MAGIC		'r'
#define RING_IOC_GET_STATS	_IOR(RING_IOC_MAGIC, 1, struct ring_stats)
#define RING_IOC_RESET		_IO(RING_IOC_MAGIC, 2)
#define RING_IOC_RESIZE		_IOW(RING_IOC_MAGIC, 3, __u32)
#endif
```

```c
// file: ring/ring.c
#include <linux/miscdevice.h>
#include <linux/module.h>
#include <linux/mutex.h>
#include <linux/poll.h>
#include <linux/proc_fs.h>
#include <linux/seq_file.h>
#include <linux/slab.h>
#include <linux/uaccess.h>
#include <linux/wait.h>
#include "ring_uapi.h"

static unsigned int capacity = 4096;
module_param(capacity, uint, 0444);

static struct {
	struct mutex lock;		/* protects everything below */
	char *buf;
	size_t cap, head, len;		/* data is buf[head .. head+len), wrapping at cap */
	wait_queue_head_t readq, writeq;
	struct ring_stats st;
} ring;

static int ring_open(struct inode *inode, struct file *file)
{
	mutex_lock(&ring.lock);
	ring.st.opens++;
	mutex_unlock(&ring.lock);
	return stream_open(inode, file);	/* a stream: no file position, no lseek */
}

static ssize_t ring_read(struct file *file, char __user *ubuf, size_t n, loff_t *pos)
{
	size_t first;

	if (mutex_lock_interruptible(&ring.lock))
		return -ERESTARTSYS;
	while (ring.len == 0) {			/* empty: wait (Unit 10.2) */
		mutex_unlock(&ring.lock);
		if (file->f_flags & O_NONBLOCK)
			return -EAGAIN;
		if (wait_event_interruptible(ring.readq, READ_ONCE(ring.len) > 0))
			return -ERESTARTSYS;	/* a signal arrived */
		if (mutex_lock_interruptible(&ring.lock))
			return -ERESTARTSYS;
	}
	n = min(n, ring.len);
	first = min(n, ring.cap - ring.head);	/* the data may wrap around the end of buf */
#ifdef COPY_UNDER_SPINLOCK			/* experiment switch for DT-23 */
	{
		static DEFINE_SPINLOCK(demo);

		spin_lock(&demo);
		n -= copy_to_user(ubuf, ring.buf + ring.head, first);
		spin_unlock(&demo);
		first = n;
	}
#endif
#ifdef RETURN_COPY_RESULT			/* experiment switch for DT-22 */
	{
		unsigned long left = copy_to_user(ubuf, ring.buf + ring.head, first);

		if (left) {
			mutex_unlock(&ring.lock);
			return left;		/* BUG: a byte count, not an error */
		}
	}
#endif
	if (copy_to_user(ubuf, ring.buf + ring.head, first) ||
	    copy_to_user(ubuf + first, ring.buf, n - first)) {
		mutex_unlock(&ring.lock);
		return -EFAULT;
	}
	ring.head = (ring.head + n) % ring.cap;
	ring.len -= n;
	ring.st.bytes_out += n;
	ring.st.reads++;
	mutex_unlock(&ring.lock);
	wake_up_interruptible(&ring.writeq);	/* there is room now */
	return n;
}

static ssize_t ring_write(struct file *file, const char __user *ubuf, size_t n, loff_t *pos)
{
	size_t tail, first;

	if (n == 0)
		return 0;
	if (mutex_lock_interruptible(&ring.lock))
		return -ERESTARTSYS;
	while (ring.len == ring.cap) {		/* full: wait for a reader */
		mutex_unlock(&ring.lock);
		if (file->f_flags & O_NONBLOCK)
			return -EAGAIN;
		if (wait_event_interruptible(ring.writeq, READ_ONCE(ring.len) < READ_ONCE(ring.cap)))
			return -ERESTARTSYS;
		if (mutex_lock_interruptible(&ring.lock))
			return -ERESTARTSYS;
	}
	n = min(n, ring.cap - ring.len);	/* a partial write is allowed, like a pipe */
	tail = (ring.head + ring.len) % ring.cap;
	first = min(n, ring.cap - tail);
	if (copy_from_user(ring.buf + tail, ubuf, first) ||
	    copy_from_user(ring.buf, ubuf + first, n - first)) {
		mutex_unlock(&ring.lock);
		return -EFAULT;
	}
	ring.len += n;
	ring.st.bytes_in += n;
	ring.st.writes++;
	mutex_unlock(&ring.lock);
	wake_up_interruptible(&ring.readq);	/* there is data now */
	return n;
}

static __poll_t ring_poll(struct file *file, poll_table *wait)
{
	__poll_t mask = 0;

	poll_wait(file, &ring.readq, wait);	/* register: wake this poller when either queue is woken */
	poll_wait(file, &ring.writeq, wait);
	mutex_lock(&ring.lock);
	if (ring.len > 0)
		mask |= EPOLLIN | EPOLLRDNORM;
	if (ring.len < ring.cap)
		mask |= EPOLLOUT | EPOLLWRNORM;
	mutex_unlock(&ring.lock);
	return mask;
}

static long ring_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
	struct ring_stats st;
	__u32 newcap;
	char *newbuf;
	long ret = 0;

	switch (cmd) {
	case RING_IOC_GET_STATS:
		mutex_lock(&ring.lock);
		st = ring.st;
		st.capacity = ring.cap;
		st.used = ring.len;
		mutex_unlock(&ring.lock);
		return copy_to_user((void __user *)arg, &st, sizeof(st)) ? -EFAULT : 0;
	case RING_IOC_RESET:
		mutex_lock(&ring.lock);
		ring.head = ring.len = 0;
		memset(&ring.st, 0, sizeof(ring.st));
		mutex_unlock(&ring.lock);
		wake_up_interruptible(&ring.writeq);
		return 0;
	case RING_IOC_RESIZE:
		if (get_user(newcap, (__u32 __user *)arg))
			return -EFAULT;
		if (newcap < 16 || newcap > 1 << 20)
			return -EINVAL;
		newbuf = kmalloc(newcap, GFP_KERNEL);
		if (!newbuf)
			return -ENOMEM;
		mutex_lock(&ring.lock);
		if (ring.len) {
			ret = -EBUSY;		/* only an empty ring may be resized */
		} else {
			swap(ring.buf, newbuf);
			ring.cap = newcap;
			ring.head = 0;
		}
		mutex_unlock(&ring.lock);
		kfree(newbuf);			/* the old buffer, or the unused new one */
		return ret;
	default:
		return -ENOTTY;			/* "inappropriate ioctl for device" */
	}
}

static const struct file_operations ring_fops = {
	.owner		= THIS_MODULE,		/* pins the module while a file is open */
	.open		= ring_open,
	.read		= ring_read,
	.write		= ring_write,
	.poll		= ring_poll,
	.unlocked_ioctl	= ring_ioctl,
};

/* sysfs: /sys/class/misc/ring/used and .../capacity */
static ssize_t used_show(struct device *dev, struct device_attribute *attr, char *buf)
{
	return sysfs_emit(buf, "%zu\n", READ_ONCE(ring.len));
}
static DEVICE_ATTR_RO(used);

static ssize_t capacity_show(struct device *dev, struct device_attribute *attr, char *buf)
{
	return sysfs_emit(buf, "%zu\n", READ_ONCE(ring.cap));
}
static DEVICE_ATTR_RO(capacity);

static struct attribute *ring_attrs[] = { &dev_attr_used.attr, &dev_attr_capacity.attr, NULL };
ATTRIBUTE_GROUPS(ring);

static struct miscdevice ring_misc = {
	.minor	= MISC_DYNAMIC_MINOR,
	.name	= "ring",
	.fops	= &ring_fops,
	.groups	= ring_groups,
	.mode	= 0666,
};

/* procfs: /proc/ring, a human-readable summary */
static int ring_proc_show(struct seq_file *m, void *v)
{
	mutex_lock(&ring.lock);
	seq_printf(m, "capacity %zu\nused %zu\nopens %u\nreads %u\nwrites %u\nbytes_in %llu\nbytes_out %llu\n",
		   ring.cap, ring.len, ring.st.opens, ring.st.reads, ring.st.writes,
		   ring.st.bytes_in, ring.st.bytes_out);
	mutex_unlock(&ring.lock);
	return 0;
}

static int __init ring_init(void)
{
	int err;

	if (capacity < 16 || capacity > 1 << 20)
		return -EINVAL;
	mutex_init(&ring.lock);
	init_waitqueue_head(&ring.readq);
	init_waitqueue_head(&ring.writeq);
	ring.cap = capacity;
	ring.buf = kmalloc(ring.cap, GFP_KERNEL);
	if (!ring.buf)
		return -ENOMEM;
	if (!proc_create_single("ring", 0444, NULL, ring_proc_show)) {
		err = -ENOMEM;
		goto free_buf;
	}
	err = misc_register(&ring_misc);
	if (err)
		goto remove_proc;
	pr_info("ring: /dev/ring is %d:%d, capacity %zu\n", MISC_MAJOR, ring_misc.minor, ring.cap);
	return 0;

remove_proc:
	remove_proc_entry("ring", NULL);
free_buf:
	kfree(ring.buf);
	return err;
}

static void __exit ring_exit(void)
{
	misc_deregister(&ring_misc);
	remove_proc_entry("ring", NULL);
	kfree(ring.buf);
}

module_init(ring_init);
module_exit(ring_exit);
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("A blocking byte ring buffer character device");
```

```text
# file: ring/Makefile
obj-m += ring.o
ccflags-y += $(RING_CFLAGS)
```

The first test needs nothing but the shell: `echo` and `cat` are ordinary programs, and `/dev/ring` is an ordinary file to them.

```sh
# file: guest-basic/run-basic.sh
insmod /modules/ring.ko
ls -l /dev/ring
echo "hello through the kernel" > /dev/ring
cat /sys/class/misc/ring/used
dd if=/dev/ring bs=64 count=1 2>/dev/null
cat /proc/ring
exec 3< /dev/ring
rmmod ring; echo "rmmod while /dev/ring is open: exit status $?"
exec 3<&-
rmmod ring && echo "rmmod after closing: ok"
```

```console
$ make -s -C build M=$PWD/ring modules && mkdir -p g1/modules && cp ring/ring.ko g1/modules/ && cp guest-basic/run-basic.sh g1/ && sh mkinitramfs.sh g1.cpio init-modules.sh g1 > /dev/null && bash boot.sh g1.cpio quiet 2>&1
crw-rw-rw-    1 0        0          10, 125 Nov 15 23:50 /dev/ring
25
hello through the kernel
capacity 4096
used 0
opens 2
reads 1
writes 1
bytes_in 25
bytes_out 25
rmmod: can't unload module 'ring': Resource temporarily unavailable
rmmod while /dev/ring is open: exit status 1
rmmod after closing: ok
```

The device got major 10 (misc) and a dynamic minor; devtmpfs created the node with mode 0666 as requested. `cat` was not used for reading because it would block forever once the ring is empty — `dd ... count=1` reads once. `.owner = THIS_MODULE` makes every open file hold a reference to the module, so `rmmod` refuses while descriptor 3 is open: unloading code that an open file still points to would be a use-after-free of the *code*.

### Deep & Tricky

**DT-22. `copy_to_user()` and `copy_from_user()` do not return an error code: they return the number of bytes that could *not* be copied (0 on success). Returning that number from `read` tells user space that `n` bytes were read — successfully — when nothing was copied.** *(Linux `include/linux/uaccess.h`: "Returns number of bytes that could not be copied. On success, this will be zero."; `Documentation/kernel-hacking/hacking.rst`, "copy_to_user() … return the amount of uncopied data")* The test program reads into an invalid address. The correct driver returns `-EFAULT`; the variant built with `-DRETURN_COPY_RESULT` returns 16, and the program believes it got 16 bytes.

```c
// file: user/badread.c
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int main(void) {
    int fd = open("/dev/ring", O_RDWR);
    char *volatile bad = (char *)1;                       /* address 1: never mapped (volatile: hide it from the compiler) */
    if (write(fd, "0123456789abcdef", 16) != 16) return 1;
    ssize_t r = read(fd, bad, 16);
    printf("read into a bad pointer returned %zd%s%s\n", r, r < 0 ? ", errno " : "", r < 0 ? strerror(errno) : " (claims success!)");
    return 0;
}
```

```sh
# file: guest-badread/run-badread.sh
insmod /modules/ring.ko && /bin/badread && rmmod ring
insmod /modules/ring-buggy.ko && /bin/badread && rmmod ring
```

```console
$ make -s -C build M=$PWD/ring clean && make -s -C build M=$PWD/ring RING_CFLAGS=-DRETURN_COPY_RESULT modules && cp ring/ring.ko ring-buggy.ko && make -s -C build M=$PWD/ring clean && make -s -C build M=$PWD/ring modules && gcc -static -O2 -o badread user/badread.c && mkdir -p g2/modules g2/bin && cp ring/ring.ko ring-buggy.ko g2/modules/ && cp badread g2/bin/ && cp guest-badread/run-badread.sh g2/ && sh mkinitramfs.sh g2.cpio init-modules.sh g2 > /dev/null && bash boot.sh g2.cpio quiet 2>&1
read into a bad pointer returned -1, errno Bad address
read into a bad pointer returned 16 (claims success!)
```

## Unit 10.2 — Blocking, waking, `O_NONBLOCK`, and `poll`

**One-line summary:** A process that must wait for a condition sleeps on a **wait queue**: `wait_event_interruptible(q, condition)` puts it to sleep until someone calls `wake_up_interruptible(&q)` *and* the condition is true (it re-checks after every wake-up — wake-ups can be spurious), or until a signal arrives, in which case it returns non-zero and the driver returns `-ERESTARTSYS` (the kernel then restarts the system call or returns `EINTR`). Never sleep holding a spinlock; drop the mutex before sleeping, retake it after, and re-check. With `O_NONBLOCK`, return `-EAGAIN` instead of sleeping. `poll()` asks the driver two things: which wait queues to sleep on (`poll_wait`) and which operations would not block right now (the returned mask).

```c
// file: user/ringtest.c
/* Functional tests for /dev/ring, run inside the guest. */
#include <errno.h>
#include <fcntl.h>
#include <poll.h>
#include <signal.h>
#include <stdio.h>
#include <string.h>
#include <sys/ioctl.h>
#include <sys/wait.h>
#include <time.h>
#include <unistd.h>
#include "../ring/ring_uapi.h"

static int failures;
#define CHECK(c) do { if (!(c)) { printf("  FAIL line %d: %s (errno %d)\n", __LINE__, #c, errno); failures++; } } while (0)

static long ms_since(struct timespec *a) {
    struct timespec b;
    clock_gettime(CLOCK_MONOTONIC, &b);
    return (b.tv_sec - a->tv_sec) * 1000 + (b.tv_nsec - a->tv_nsec) / 1000000;
}

static void on_alarm(int sig) { (void)sig; }

int main(void) {
    int fd = open("/dev/ring", O_RDWR);
    CHECK(fd >= 0);
    char buf[256];
    struct ring_stats st;
    struct timespec t0;

    /* 1. round trip */
    CHECK(write(fd, "abc", 3) == 3);
    CHECK(read(fd, buf, sizeof buf) == 3 && memcmp(buf, "abc", 3) == 0);

    /* 2. non-blocking read of an empty ring */
    int nb = open("/dev/ring", O_RDWR | O_NONBLOCK);
    errno = 0;
    CHECK(read(nb, buf, 1) == -1 && errno == EAGAIN);

    /* 3. ioctl: resize, partial write, full ring, unknown command */
    unsigned int cap = 64;
    CHECK(ioctl(fd, RING_IOC_RESIZE, &cap) == 0);
    memset(buf, 'x', 100);
    CHECK(write(fd, buf, 100) == 64);                       /* only 64 fit */
    errno = 0;
    CHECK(write(nb, buf, 1) == -1 && errno == EAGAIN);      /* full */
    CHECK(ioctl(fd, RING_IOC_RESIZE, &cap) == -1 && errno == EBUSY);
    CHECK(ioctl(fd, RING_IOC_GET_STATS, &st) == 0 && st.used == 64 && st.capacity == 64);
    CHECK(ioctl(fd, _IO('r', 99)) == -1 && errno == ENOTTY);
    CHECK(ioctl(fd, RING_IOC_RESET) == 0);
    printf("  ioctl numbers: GET_STATS=0x%08lx RESET=0x%08lx RESIZE=0x%08lx\n",
           (unsigned long)RING_IOC_GET_STATS, (unsigned long)RING_IOC_RESET, (unsigned long)RING_IOC_RESIZE);

    /* 4. a blocked reader is woken by a writer in another process */
    clock_gettime(CLOCK_MONOTONIC, &t0);
    if (fork() == 0) { usleep(200000); _exit(write(fd, "wake", 4) == 4 ? 0 : 1); }
    CHECK(read(fd, buf, sizeof buf) == 4);
    long waited = ms_since(&t0);
    CHECK(waited >= 190);
    printf("  blocking read waited %ld ms for the writer\n", waited);
    wait(NULL);

    /* 5. poll: not readable now; readable after a delayed write */
    struct pollfd p = { fd, POLLIN, 0 };
    CHECK(poll(&p, 1, 0) == 0);
    clock_gettime(CLOCK_MONOTONIC, &t0);
    if (fork() == 0) { usleep(100000); _exit(write(fd, "p", 1) == 1 ? 0 : 1); }
    CHECK(poll(&p, 1, 2000) == 1 && (p.revents & POLLIN));
    printf("  poll returned POLLIN after %ld ms\n", ms_since(&t0));
    CHECK(read(fd, buf, 1) == 1);
    wait(NULL);

    /* 6. a signal interrupts a blocked read: EINTR */
    struct sigaction sa = { 0 };
    sa.sa_handler = on_alarm;                                /* no SA_RESTART */
    sigaction(SIGALRM, &sa, NULL);
    alarm(1);
    errno = 0;
    CHECK(read(fd, buf, 1) == -1 && errno == EINTR);

    CHECK(ioctl(fd, RING_IOC_GET_STATS, &st) == 0);
    printf("  stats: opens %u reads %u writes %u in %llu out %llu\n", st.opens, st.reads, st.writes,
           (unsigned long long)st.bytes_in, (unsigned long long)st.bytes_out);
    printf("ringtest: %s\n", failures ? "FAILED" : "all checks passed");
    return failures != 0;
}
```

```sh
# file: guest-ringtest/run-ringtest.sh
insmod /modules/ring.ko && /bin/ringtest
```

```console
$ gcc -static -O2 -Wall -Wextra -o ringtest user/ringtest.c && mkdir -p g3/modules g3/bin && cp ring/ring.ko g3/modules/ && cp ringtest g3/bin/ && cp guest-ringtest/run-ringtest.sh g3/ && sh mkinitramfs.sh g3.cpio init-modules.sh g3 > /dev/null && bash boot.sh g3.cpio quiet 2>&1
  ioctl numbers: GET_STATS=0x80287201 RESET=0x00007202 RESIZE=0x40047203
  blocking read waited 206 ms for the writer
  poll returned POLLIN after 104 ms
  stats: opens 0 reads 2 writes 2 in 5 out 5
ringtest: all checks passed
```

The ioctl numbers show the encoding of `_IOR`/`_IOW`: bits 0–7 are the command number, 8–15 the "magic" type `'r'` (0x72), 16–29 the size of the argument (0x28 = 40 = `sizeof(struct ring_stats)`; 4 for the `__u32`), and the top two bits the direction (read 2, write 1, none 0). The size inside the number means that changing the struct changes the command — an old binary cannot silently pass a structure of the wrong size.

### Deep & Tricky

**DT-23. `copy_to_user()` may sleep: if the user page is not resident, the copy triggers a page fault, and handling it may require reading from disk. So user memory must never be touched while holding a spinlock, with interrupts disabled, or in an interrupt handler. Kernels built with `CONFIG_DEBUG_ATOMIC_SLEEP` check this on every call (`might_fault()`), whether or not a fault actually happens.** *(Linux `include/linux/uaccess.h`: copy_to_user "Context: User context only. This function may sleep if pagefaults are enabled."; `kernel/sched/core.c`: `__might_sleep` prints "BUG: sleeping function called from invalid context")* The variant built with `-DCOPY_UNDER_SPINLOCK` copies inside a spinlock. The page is resident, the copy succeeds, and nothing goes wrong in this run — but the check fires anyway, which is the point: it reports the bug on the first execution, not on the rare day when the page happens to be swapped out.

```sh
# file: guest-atomic/run-atomic.sh
dmesg -n 8
insmod /modules/ring-atomic.ko
echo hi > /dev/ring
dd if=/dev/ring bs=16 count=1 2>/dev/null
```

```console
$ make -s -C build M=$PWD/ring clean && make -s -C build M=$PWD/ring RING_CFLAGS=-DCOPY_UNDER_SPINLOCK modules && cp ring/ring.ko ring-atomic.ko && make -s -C build M=$PWD/ring clean && make -s -C build M=$PWD/ring modules && mkdir -p g4/modules && cp ring-atomic.ko g4/modules/ && cp guest-atomic/run-atomic.sh g4/ && sh mkinitramfs.sh g4.cpio init-modules.sh g4 > /dev/null && bash boot.sh g4.cpio quiet 2>&1 | sed 's/^\[[ 0-9.]*\] //' | grep -E "^BUG|in_atomic|preempt_count|^ (ring_read|vfs_read|ksys_read|__might_fault)|^hi|Preemption disabled|_raw_spin_lock" | head -7 | cut -c1-100
BUG: sleeping function called from invalid context at include/linux/uaccess.h:192
in_atomic(): 1, irqs_disabled(): 0, non_block: 0, pid: 47, name: dd
preempt_count: 1, expected: 0
 __might_fault+0x6a/0xfb
 ring_read+0x1b1/0x2b3 [ring]
 vfs_read+0x13d/0x27e
 ksys_read+0xf0/0x155
```

The report says *where* (the `copy_to_user` in `ring_read`) and *why* (`in_atomic(): 1` — the spinlock raised the preemption count); the full report, cut short above, goes on to say where preemption was disabled. The data still arrived: this is a report of a latent bug, not a crash.

### Pattern

**P-11. Treat the user–kernel interface as a published contract: one header shared by both sides with fixed-size types and explicit padding, ioctl numbers that encode direction and size, `-ENOTTY` for unknown commands, `-EFAULT` for bad pointers, `-EINVAL` for bad values, `-EAGAIN`/`-ERESTARTSYS` for non-blocking and interrupted waits — and validate every argument before acting on it.** *(Linux `Documentation/driver-api/ioctl.rst` and `Documentation/process/botching-up-ioctls.rst`: "Always use fixed-size types", "pad the entire struct to a multiple of 64 bits", "Check all unused fields and flags")* Once a program in the field uses an interface, the kernel's rule is that it must keep working ("we don't break user space"). *Trade-off:* a carefully designed interface takes longer to add and is harder to change; new kernel interfaces therefore usually prefer extensible forms (a size field in the struct, flags that must be zero, or a newer mechanism such as netlink or a dedicated system call) over yet another ioctl.

### Mastery check

1. **Predict.** In `ring_read`, why is the condition in `wait_event_interruptible` written with `READ_ONCE(ring.len)` and evaluated without the mutex?
2. **Predict.** Why does `RING_IOC_RESIZE` allocate the new buffer *before* taking the mutex?
3. **Predict.** A program calls `select()` on `/dev/ring` for writing, and the ring is full. Walk through what wakes it up.

## Chapter 10 Project — Atomic records and a stress test

**Goal:** pipes promise that a write of at most `PIPE_BUF` bytes is never interleaved with other writers' data. Give `/dev/ring` the same guarantee for writes up to its capacity: such a write either goes in whole or blocks until it fits (or returns `-EAGAIN` in non-blocking mode). Then prove it with a stress test: several writer processes send numbered, checksummed records of random lengths; a reader checks that every record arrives whole, uncorrupted, in each writer's order.

The change to `ring_write` is in the waiting condition and the length: wait until `n` bytes fit (when `n <= cap`), instead of until one byte fits.

```c
// file: proj/ring_atomic.patch
--- a/ring/ring.c
+++ b/ring/ring.c
@@ -98,22 +98,26 @@
 static ssize_t ring_write(struct file *file, const char __user *ubuf, size_t n, loff_t *pos)
 {
-	size_t tail, first;
+	size_t tail, first, need;
 
 	if (n == 0)
 		return 0;
 	if (mutex_lock_interruptible(&ring.lock))
 		return -ERESTARTSYS;
-	while (ring.len == ring.cap) {		/* full: wait for a reader */
+	/* writes that fit in the ring are atomic: wait until all of them fits */
+	need = n <= ring.cap ? n : 1;
+	while (ring.cap - ring.len < need) {
 		mutex_unlock(&ring.lock);
 		if (file->f_flags & O_NONBLOCK)
 			return -EAGAIN;
-		if (wait_event_interruptible(ring.writeq, READ_ONCE(ring.len) < READ_ONCE(ring.cap)))
+		if (wait_event_interruptible(ring.writeq,
+					     READ_ONCE(ring.cap) - READ_ONCE(ring.len) >= need))
 			return -ERESTARTSYS;
 		if (mutex_lock_interruptible(&ring.lock))
 			return -ERESTARTSYS;
+		need = n <= ring.cap ? n : 1;	/* the ring may have been resized */
 	}
-	n = min(n, ring.cap - ring.len);	/* a partial write is allowed, like a pipe */
+	n = min(n, ring.cap - ring.len);	/* only writes larger than the ring are split */
 	tail = (ring.head + ring.len) % ring.cap;
 	first = min(n, ring.cap - tail);
 	if (copy_from_user(ring.buf + tail, ubuf, first) ||
```

```c
// file: user/stress.c
/* stress WRITERS RECORDS: writers send framed records; the reader verifies them. */
#include <fcntl.h>
#include <stdint.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/wait.h>
#include <unistd.h>

static pid_t pids[256];
static int nwriters;

static int finish(int status) {                     /* stop any writer still blocked in write(), then reap */
    for (int w = 0; w < nwriters; w++) kill(pids[w], SIGKILL);
    while (wait(NULL) > 0) { }
    return status;
}

struct hdr { uint16_t len; uint8_t writer; uint8_t check; uint32_t seq; };   /* 8 bytes, then len payload bytes */

static uint8_t sum(const uint8_t *p, size_t n) { uint8_t s = 0; while (n--) s = (uint8_t)(s * 31 + *p++); return s; }

int main(int argc, char **argv) {
    int writers = atoi(argv[1]), records = atoi(argv[2]);
    nwriters = writers;
    for (int w = 0; w < writers; w++) {
        if ((pids[w] = fork()) == 0) {
            int fd = open("/dev/ring", O_WRONLY);
            srand(w + 1);
            uint8_t rec[8 + 600];
            for (int i = 0; i < records; i++) {
                struct hdr h = { (uint16_t)(1 + rand() % 600), (uint8_t)w, 0, (uint32_t)i };
                for (int k = 0; k < h.len; k++) rec[8 + k] = (uint8_t)(rand() & 0xFF);
                h.check = sum(rec + 8, h.len);
                memcpy(rec, &h, 8);
                ssize_t wr = write(fd, rec, 8 + h.len);
                if (wr != 8 + h.len) { printf("writer %d: short write, %zd of %d bytes\n", w, wr, 8 + h.len); _exit(1); }
            }
            _exit(0);
        }
    }
    int fd = open("/dev/ring", O_RDONLY);
    uint32_t next[256] = { 0 };
    long total = 0, bad = 0, bytes = 0;
    uint8_t stream[8192];
    size_t have = 0;
    while (total < (long)writers * records) {
        ssize_t r = read(fd, stream + have, sizeof stream - have);
        if (r <= 0) { perror("read"); return finish(1); }
        have += (size_t)r;
        bytes += r;
        for (;;) {                                         /* consume every complete record */
            struct hdr h;
            if (have < 8) break;
            memcpy(&h, stream, 8);
            if (h.len == 0 || h.len > 600 || h.writer >= writers) { printf("garbage header at record %ld\n", total); return finish(1); }
            if (have < 8u + h.len) break;
            if (sum(stream + 8, h.len) != h.check || h.seq != next[h.writer]) bad++;
            next[h.writer] = h.seq + 1;
            memmove(stream, stream + 8 + h.len, have - 8 - h.len);
            have -= 8 + h.len;
            total++;
        }
    }
    finish(0);
    printf("stress: %d writers x %d records = %ld records, %ld bytes, %ld bad\n", writers, records, total, bytes, bad);
    return bad != 0;
}
```

```sh
# file: guest-stress/run-stress.sh
insmod /modules/$RING.ko capacity=1024
echo "== $RING driver =="
/bin/stress 4 2000
cat /proc/ring | tr '\n' ' '; echo
sleep 0.5
rmmod ring
```

```sh
# file: guest-stress/init-stress.sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev
dmesg -n 1
RING=ring-unit sh /run-stress.sh
RING=ring-atomic sh /run-stress.sh
reboot -f
```

```console
$ cp ring/ring.c ring/ring.c.unit && patch -s -p1 < proj/ring_atomic.patch && make -s -C build M=$PWD/ring clean && make -s -C build M=$PWD/ring modules && cp ring/ring.ko ring-atomic-records.ko && cp ring/ring.c.unit ring/ring.c && make -s -C build M=$PWD/ring clean && make -s -C build M=$PWD/ring modules && echo "built both drivers"
built both drivers
$ gcc -static -O2 -Wall -o stress user/stress.c && mkdir -p g5/modules g5/bin && cp ring/ring.ko g5/modules/ring-unit.ko && cp ring-atomic-records.ko g5/modules/ring-atomic.ko && cp stress g5/bin/ && cp guest-stress/run-stress.sh g5/ && sh mkinitramfs.sh g5.cpio guest-stress/init-stress.sh g5 > /dev/null && bash boot.sh g5.cpio quiet 2>&1
== ring-unit driver ==
garbage header at record 2
writer 0: short write, 191 of 365 bytes
capacity 1024 used 0 opens 5 reads 2 writes 5 bytes_in 2048 bytes_out 2048 
== ring-atomic driver ==
stress: 4 writers x 2000 records = 8000 records, 2450174 bytes, 0 bad
capacity 1024 used 0 opens 5 reads 3097 writes 8000 bytes_in 2450174 bytes_out 2450174 
```

With the unit's driver, a record that does not fit in the remaining space is written partially; another writer's record can then land in the middle of it, and the reader sees a corrupt record or a garbage header and stops. With the atomic-record driver, all 8,000 records arrive intact and in order per writer. The same reasoning is behind `PIPE_BUF` in POSIX and behind the message boundaries of `SOCK_SEQPACKET` sockets.

### Review checklist

- With the atomic-write rule, can a writer that wants to write 1,000 bytes into a 1,024-byte ring starve while other writers keep writing 100-byte records? What would fix it?
- `RING_IOC_RESIZE` may change `cap` while a writer is waiting with a `need` computed from the old capacity. Why does the patch recompute `need` after waking?
- The ring is a single global buffer. What would change to give every *open* of `/dev/ring` its own buffer (hint: `file->private_data`)?

## Answers to the Chapter 10 prediction questions

**10.2 Q1.** The condition is evaluated by `wait_event_interruptible` itself, repeatedly, before and after sleeping — it cannot take our mutex (that would sleep inside the wait loop's own state handling and could deadlock with the waker). A racy read is fine here because the result is only a hint: after waking, the code takes the mutex and checks `ring.len` again in the `while` loop. `READ_ONCE` makes the compiler perform a real load each time, so the check cannot be optimised into a stale value.

**10.2 Q2.** `kmalloc(GFP_KERNEL)` may sleep for a long time (reclaiming memory). Doing it before taking the mutex keeps the critical section short, so readers and writers are not blocked during the allocation. If the resize turns out to be impossible (`-EBUSY`), the unused buffer is simply freed.

**10.2 Q3.** `select` calls `ring_poll`, which registers the caller on both wait queues via `poll_wait` and returns a mask without `EPOLLOUT` (the ring is full). The process sleeps. A reader later removes data and calls `wake_up_interruptible(&ring.writeq)`; that wakes the selecting process, `select` calls `ring_poll` again, which now returns `EPOLLOUT`, and `select` returns.

# Chapter 11 — Processes, Memory, and Scheduling in Linux

*Goal: look inside the running kernel with experiments instead of reading code first. Every claim in this chapter is checked by a small program run in our guest, reading the numbers Linux publishes in `/proc`: how `fork` shares memory with copy-on-write, what a process's address space looks like, how much memory "allocated" memory really uses, what happens when memory runs out, how the scheduler divides a CPU between processes of different priority, and why a real-time process cannot quite take all of it. The corresponding kernel source is named at each step, so the experiments are also a reading guide.*

**What you will be able to do after this chapter**

- Explain `fork`, threads, and copy-on-write, and measure their cost in page faults.
- Read `/proc/<pid>/maps`, `status`, `stat`, and `smaps_rollup`, and `/proc/meminfo`, `buddyinfo`, and `slabinfo`.
- Predict and observe demand paging, memory overcommit, and the OOM killer.
- Predict the CPU split between `nice` levels under the EEVDF scheduler, and the effect of real-time scheduling and its throttling.

```console
$ ln -sfn /home/claude/kernelsrc/linux linux && ln -sfn /home/claude/kernelsrc/build build && cp ../ch08/mkinitramfs.sh ../ch08/boot.sh ../ch09/init-modules.sh . && echo ready
ready
```

Every experiment is a static C program; one helper header reads values from `/proc`:

```c
// file: lab/proc.h
#pragma once
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/resource.h>

/* The value of "Key:" in a /proc file such as /proc/self/status or /proc/meminfo (first number, usually kB). */
static inline long proc_value(const char *file, const char *key) {
    FILE *f = fopen(file, "r");
    char line[256];
    long v = -1;
    size_t n = strlen(key);
    while (f && fgets(line, sizeof line, f))
        if (strncmp(line, key, n) == 0 && line[n] == ':') { v = atol(line + n + 1); break; }
    if (f) fclose(f);
    return v;
}

static inline long minor_faults(void) { struct rusage r; getrusage(RUSAGE_SELF, &r); return r.ru_minflt; }
```

## Topic 11A — Processes

## Unit 11.1 — `fork`, threads, and copy-on-write

**One-line summary:** Every process and every thread is a `struct task_struct` (`include/linux/sched.h`). `fork()` creates a new task with a *copy* of the parent's address space — but copying is lazy: the kernel duplicates the page tables, marks every private writable page **read-only in both** processes, and copies a page only when one of them writes to it (a **copy-on-write** fault; `do_wp_page()` in `mm/memory.c`). A **thread** is a task created with `clone(CLONE_VM | CLONE_FILES | ...)`: it shares the same `struct mm_struct` (address space), so nothing is copied at all.

```c
// file: lab/cow.c
/* Copy-on-write, measured in page faults. */
#include <sys/mman.h>
#include <sys/wait.h>
#include <unistd.h>
#include "proc.h"

#define SIZE (64L << 20)                 /* 64 MiB = 16384 pages of 4 KiB */

int main(void) {
    char *p = mmap(NULL, SIZE, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    long f0 = minor_faults();
    memset(p, 1, SIZE);
    printf("parent: first touch of 64 MiB:   %6ld faults (demand paging: one per page)\n", minor_faults() - f0);

    pid_t pid = fork();
    if (pid == 0) {
        long f = minor_faults();         /* fork() resets the child's counters to 0 */
        volatile long sum = 0;
        for (long i = 0; i < SIZE; i += 4096) sum += p[i];
        printf("child:  read every page:         %6ld faults (pages are shared)\n", minor_faults() - f);
        f = minor_faults();
        for (long i = 0; i < SIZE; i += 4096) p[i] = 2;
        printf("child:  write every page:        %6ld faults (copy-on-write: one copy per page)\n", minor_faults() - f);
        printf("child:  VmRSS %ld kB\n", proc_value("/proc/self/status", "VmRSS"));
        _exit(0);
    }
    waitpid(pid, NULL, 0);
    f0 = minor_faults();
    for (long i = 0; i < SIZE; i += 4096) p[i] = 3;
    printf("parent: write after child exited: %5ld faults (still write-protected since fork)\n", minor_faults() - f0);
    printf("parent: p[0] = %d (the child's writes went to its own copies)\n", p[0]);
    return 0;
}
```

```c
// file: lab/threads.c
/* Threads share one address space: same maps, one VmRSS, different TIDs. */
#define _GNU_SOURCE
#include <pthread.h>
#include <sys/wait.h>
#include <unistd.h>
#include "proc.h"

static int shared = 0;

static void *worker(void *arg) {
    shared += 1;
    printf("thread: pid %d tid %d, sees shared=%d at %p; Threads: %ld\n", getpid(), gettid(), shared, (void *)&shared,
           proc_value("/proc/self/status", "Threads"));
    return arg;
}

int main(void) {
    pthread_t t;
    printf("main:   pid %d tid %d, shared at %p\n", getpid(), gettid(), (void *)&shared);
    pthread_create(&t, NULL, worker, NULL);
    pthread_join(t, NULL);
    printf("main:   shared=%d after the thread ran; Threads: %ld after it exited\n", shared,
           proc_value("/proc/self/status", "Threads"));
    pid_t pid = fork();
    if (pid == 0) { shared += 100; _exit(0); }
    waitpid(pid, NULL, 0);
    printf("main:   shared=%d after a forked child added 100 to its copy\n", shared);
    return 0;
}
```

```sh
# file: guest-proc/run-proc.sh
/bin/cow
/bin/threads
```

```console
$ mkdir -p g1/bin && for p in cow threads; do gcc -static -O2 -Wall -pthread -o g1/bin/$p lab/$p.c; done && cp guest-proc/run-proc.sh g1/ && sh mkinitramfs.sh g1.cpio init-modules.sh g1 > /dev/null && bash boot.sh g1.cpio quiet 2>&1
parent: first touch of 64 MiB:    16384 faults (demand paging: one per page)
child:  read every page:              0 faults (pages are shared)
child:  write every page:         16384 faults (copy-on-write: one copy per page)
child:  VmRSS 66020 kB
parent: write after child exited: 16384 faults (still write-protected since fork)
parent: p[0] = 3 (the child's writes went to its own copies)
main:   pid 48 tid 48, shared at 0x4c9b50
thread: pid 48 tid 49, sees shared=1 at 0x4c9b50; Threads: 2
main:   shared=1 after the thread ran; Threads: 2 after it exited
main:   shared=1 after a forked child added 100 to its copy
```

### Deep & Tricky

**DT-24. After `fork()`, *both* processes take a page fault on the first write to each private page — the parent too, even if the child never writes and has already exited. `fork` write-protects the parent's own page-table entries; the fault handler then finds the page no longer shared and reuses it without copying, but the fault itself (a trap into the kernel per page) has already happened.** *(Linux `mm/memory.c`: `copy_present_pte()` — "If it's a COW mapping, write protect it both in the parent and the child"; `do_wp_page()` → `wp_can_reuse_anon_folio()` reuses an exclusive page)* This is why a large process that forks (a database taking a snapshot, a shell spawning a command from a big program) pays page faults across its whole working set, and why `vfork()` and `posix_spawn()` exist: they avoid copying the page tables at all.

## Topic 11B — Memory

## Unit 11.2 — The address space and demand paging

**One-line summary:** A process's address space is a set of **VMAs** (virtual memory areas, `struct vm_area_struct`), each a range with permissions and a backing: a file (the program's code and data, shared libraries) or nothing (**anonymous** memory: heap, stack, `mmap(MAP_ANONYMOUS)`). `/proc/<pid>/maps` lists them. Creating a VMA allocates no memory; pages are allocated by the page-fault handler the first time each one is touched (`handle_mm_fault()`), exactly like our hobby kernel's lazy zone. `VmSize` counts the VMAs; `VmRSS` (resident set size) counts the pages actually present.

```c
// file: lab/vm.c
#include <errno.h>
#include <sys/mman.h>
#include "proc.h"

static void show(const char *when) {
    printf("%-30s VmSize %7ld kB  VmRSS %6ld kB  Rss %6ld kB  faults %6ld\n", when,
           proc_value("/proc/self/status", "VmSize"), proc_value("/proc/self/status", "VmRSS"),
           proc_value("/proc/self/smaps_rollup", "Rss"), minor_faults());
}

int main(void) {
    FILE *f = fopen("/proc/self/maps", "r");
    char line[256];
    while (fgets(line, sizeof line, f)) {               /* the address space of a static program */
        char range[64], perms[8], path[128] = "";
        sscanf(line, "%63s %7s %*s %*s %*s %127s", range, perms, path);
        printf("  %-34s %s  %s\n", range, perms, path);
    }
    fclose(f);
    show("start");
    char *p = mmap(NULL, 256 << 20, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    show("after mmap(256 MiB)");
    for (long i = 0; i < (64 << 20); i += 4096) p[i] = 1;
    show("after touching 64 MiB of it");
    int r = madvise(p, 32 << 20, MADV_DONTNEED);          /* try to give 32 MiB back to the kernel */
    printf("madvise(MADV_DONTNEED) returned %d (%s)\n", r, r ? strerror(errno) : "ok");
    munmap(p, 32 << 20);                                  /* unmap the first 32 MiB instead */
    show("after munmap of 32 MiB");
    munmap(p + (32 << 20), 224 << 20);
    show("after munmap of the rest");
    return 0;
}
```

```sh
# file: guest-vm/run-vm.sh
/bin/vm
```

```console
$ mkdir -p g2/bin && gcc -static -O2 -Wall -o g2/bin/vm lab/vm.c && cp guest-vm/run-vm.sh g2/ && sh mkinitramfs.sh g2.cpio init-modules.sh g2 > /dev/null && bash boot.sh g2.cpio quiet 2>&1
  00400000-00401000                  r--p  /bin/vm
  00401000-0049e000                  r-xp  /bin/vm
  0049e000-004c5000                  r--p  /bin/vm
  004c5000-004ca000                  r--p  /bin/vm
  004ca000-004cc000                  rw-p  /bin/vm
  004cc000-004d2000                  rw-p  
  229ee000-22a10000                  rw-p  [heap]
  7fa80a252000-7fa80a256000          r--p  [vvar]
  7fa80a256000-7fa80a258000          r-xp  [vdso]
  7ffdda3e4000-7ffdda405000          rw-p  [stack]
start                          VmSize    1132 kB  VmRSS    716 kB  Rss    856 kB  faults     52
after mmap(256 MiB)            VmSize  263276 kB  VmRSS    716 kB  Rss    856 kB  faults     52
after touching 64 MiB of it    VmSize  263276 kB  VmRSS  66252 kB  Rss  66392 kB  faults  16436
madvise(MADV_DONTNEED) returned -1 (Function not implemented)
after munmap of 32 MiB         VmSize  230508 kB  VmRSS  33536 kB  Rss  33624 kB  faults  16436
after munmap of the rest       VmSize    1132 kB  VmRSS    768 kB  Rss    856 kB  faults  16436
```

The map shows the program's code (`r-xp`), read-only data, data and BSS (`rw-p`; the BSS part has no file), the heap, the stack, and two entries the kernel adds to every process: `[vvar]` (kernel data readable by user space) and `[vdso]` (a small shared library of kernel code, which lets `clock_gettime` run without entering the kernel). The 256 MiB `mmap` grew `VmSize` immediately and `VmRSS` not at all; touching 64 MiB cost 16,384 page faults — one per 4 KiB page — and that much RSS.

`madvise` failed with `ENOSYS`, "function not implemented": this kernel started from `tinyconfig`, and the `madvise` and `fadvise` system calls are behind `CONFIG_ADVISE_SYSCALLS`, which is not enabled. A system call that a configuration leaves out simply does not exist — programs must handle `ENOSYS`, and it is one more reason to know your kernel's configuration (`/proc/config.gz`). Unmapping the first 32 MiB released it at once. The two resident-size columns come from different places: `VmRSS` from counters the kernel updates on every fault and unmap, `Rss` from `smaps_rollup`, computed by walking the page tables when the file is read; they agree closely.

## Unit 11.3 — Running out: overcommit and the OOM killer

**One-line summary:** Because allocation only reserves address space, Linux by default **overcommits**: it grants more memory than it has, betting that not all of it will be touched (`vm.overcommit_memory = 0`, a heuristic that refuses only obviously impossible single requests). When processes then touch more pages than RAM (and swap) can hold, page reclaim fails and the **OOM killer** (`mm/oom_kill.c`) chooses a process — by `oom_score`, roughly its memory use — and kills it with `SIGKILL`. `malloc` does not return `NULL`; the process simply dies later, at a page fault.

```c
// file: lab/oom.c
#include <signal.h>
#include <sys/wait.h>
#include <unistd.h>
#include "proc.h"

int main(void) {
    setvbuf(stdout, NULL, _IONBF, 0);                   /* parent and child share the output: no buffering */
    printf("MemTotal %ld kB, MemAvailable %ld kB, overcommit_memory=%ld\n", proc_value("/proc/meminfo", "MemTotal"),
           proc_value("/proc/meminfo", "MemAvailable"), (long)(fgetc(fopen("/proc/sys/vm/overcommit_memory", "r")) - '0'));
    char *huge = malloc(64L << 30);
    printf("malloc(64 GiB) on a 512 MiB machine: %s\n", huge ? "succeeded" : "NULL (refused by the heuristic)");
    pid_t pid = fork();
    if (pid == 0) {
        long got = 0;
        for (;;) {                                 /* allocate and touch 32 MiB at a time */
            char *p = malloc(32 << 20);
            if (!p) { printf("child: malloc returned NULL after %ld MiB\n", got); _exit(1); }
            memset(p, 1, 32 << 20);
            got += 32;
            printf("child: %ld MiB allocated and touched\n", got);
        }
    }
    int status;
    waitpid(pid, &status, 0);
    if (WIFSIGNALED(status)) printf("parent: child was killed by signal %d (%s)\n", WTERMSIG(status), strsignal(WTERMSIG(status)));
    return 0;
}
```

```sh
# file: guest-oom/run-oom.sh
/bin/oom | tail -4
dmesg | grep -E "invoked oom-killer|Out of memory: Killed" | sed 's/^\[[ 0-9.]*\] //' | cut -c1-100
```

```console
$ mkdir -p g3/bin && gcc -static -O2 -Wall -o g3/bin/oom lab/oom.c && cp guest-oom/run-oom.sh g3/ && sh mkinitramfs.sh g3.cpio init-modules.sh g3 > /dev/null && bash boot.sh g3.cpio quiet 2>&1
child: 256 MiB allocated and touched
child: 288 MiB allocated and touched
child: 320 MiB allocated and touched
parent: child was killed by signal 9 (Killed)
oom invoked oom-killer: gfp_mask=0x140dca(GFP_HIGHUSER_MOVABLE|__GFP_COMP|__GFP_ZERO), order=0, oom_
Out of memory: Killed process 48 (oom) total-vm:361504kB, anon-rss:350336kB, file-rss:0kB, shmem-rss
```

### Deep & Tricky

**DT-25. With the default overcommit policy, a successful `malloc` (or `mmap`) is not a promise of memory. The failure arrives later, at the page fault that finds no free page, and it does not arrive as an error return: the OOM killer sends `SIGKILL`, which cannot be caught — and the victim may be a *different* process than the one that allocated last.** *(Linux `Documentation/mm/overcommit-accounting.rst`: mode 0 "Heuristic overcommit handling. Obvious overcommits of address space are refused"; `Documentation/admin-guide/sysctl/vm.rst`, `oom_kill_allocating_task`; `mm/oom_kill.c` selects by `oom_badness()`)* Code that must handle memory exhaustion gracefully (databases, servers) sets `vm.overcommit_memory=2` (strict accounting, so allocation itself fails), pre-touches or `mlock`s its memory, or runs inside a memory cgroup with its own limit. Note that the 64 GiB request above was refused only because a *single* request was obviously impossible; the child's many 32 MiB requests all "succeeded".

## Unit 11.4 — Where kernel memory goes: buddy, slab, meminfo

**One-line summary:** Physical memory is managed by the **buddy allocator** (`mm/page_alloc.c`): free memory is kept in blocks of 2^order pages (order 0 = 4 KiB up to order 10 = 4 MiB); a request is served from the smallest block that fits, splitting larger ones, and freed blocks merge with their "buddy". `/proc/buddyinfo` shows how many free blocks of each order exist. Small kernel objects come from **slab caches** (SLUB, `mm/slub.c`), each holding objects of one size or type (`task_struct`, `dentry`, `kmalloc-64`); `/proc/slabinfo` lists them. `/proc/meminfo` adds it all up.

```sh
# file: guest-mem/run-mem.sh
echo "== /proc/buddyinfo (free blocks per order 0..10) =="
cat /proc/buddyinfo
echo "== largest slab caches by memory =="
awk 'NR > 2 { printf "%6d kB  %-24s %7d objects x %5d bytes\n", $3 * $4 / 1024, $1, $3, $4 }' /proc/slabinfo | sort -n -r | head -8
echo "== task_struct =="
grep "^task_struct" /proc/slabinfo | awk '{ print "  " $3 " task_structs of " $4 " bytes each (" $5 " per slab)" }'
echo "== /proc/meminfo (selected) =="
grep -E "^(MemTotal|MemFree|MemAvailable|Cached|AnonPages|Slab|SReclaimable|KernelStack|PageTables|Committed_AS)" /proc/meminfo
for i in 1 2 3 4 5 6 7 8; do sleep 30 & done
echo "== after starting 8 processes =="
grep "^task_struct" /proc/slabinfo | awk '{ print "  " $3 " task_structs" }'
grep KernelStack /proc/meminfo
```

```console
$ mkdir -p g4 && cp guest-mem/run-mem.sh g4/ && sh mkinitramfs.sh g4.cpio init-modules.sh g4 > /dev/null && bash boot.sh g4.cpio quiet 2>&1
== /proc/buddyinfo (free blocks per order 0..10) ==
Node 0, zone    DMA32      1      2      3      2      7      7      5      5      1      3     83 
== largest slab caches by memory ==
  7075 kB  Acpi-State                 64692 objects x   112 bytes
  2271 kB  kernfs_node_cache           9690 objects x   240 bytes
  2048 kB  kmalloc-4k                   256 objects x  8192 bytes
  1632 kB  kmalloc-1k                   816 objects x  2048 bytes
  1333 kB  shmem_inode_cache            903 objects x  1512 bytes
   848 kB  kmalloc-512                  848 objects x  1024 bytes
   768 kB  kmalloc-2k                   192 objects x  4096 bytes
   637 kB  Acpi-Operand                6279 objects x   104 bytes
== task_struct ==
  55 task_structs of 5952 bytes each (5 per slab)
== /proc/meminfo (selected) ==
MemTotal:         386272 kB
MemFree:          351864 kB
MemAvailable:     348656 kB
Cached:             2084 kB
AnonPages:           148 kB
Slab:              21376 kB
SReclaimable:        984 kB
KernelStack:        1344 kB
PageTables:          416 kB
Committed_AS:       3104 kB
== after starting 8 processes ==
  65 task_structs
KernelStack:        1600 kB
```

A `task_struct` in this kernel is several kilobytes — it holds scheduling state, credentials, signal state, accounting, and, with lockdep enabled, a record of every lock the task holds. Each new process adds one, plus a kernel stack: `KernelStack` grows by 32 kB per process, because KASAN doubles the normal 16 KiB x86-64 kernel stack (its instrumentation makes stack frames larger). The KASAN shadow and lockdep's tables are a large part of the kernel's own memory use in this debug build.

## Topic 11C — Scheduling

## Unit 11.5 — Fair shares, `nice`, and real time

**One-line summary:** Normal tasks (`SCHED_OTHER`) are scheduled by **EEVDF** (Earliest Eligible Virtual Deadline First; `kernel/sched/fair.c`, replacing CFS in Linux 6.6): each task's runtime is scaled by a **weight**, and the CPU is divided in proportion to the weights. The weight comes from the **nice** value through a table (`sched_prio_to_weight` in `kernel/sched/core.c`): nice 0 = 1024, and each nice step changes the weight by about 1.25×, so one step is roughly a 10% difference in CPU share between two competing tasks. **Real-time** tasks (`SCHED_FIFO`, `SCHED_RR`) always run before normal tasks — except that by default real-time tasks together may use only 950 ms of every second (`sched_rt_runtime_us`), leaving 5% for everything else.

```c
// file: lab/share.c
/* share NICE_B [fifo]: two CPU-bound tasks on CPU 0 for 3 s. Task A has nice 0; task B has NICE_B (or SCHED_FIFO). */
#define _GNU_SOURCE
#include <sched.h>
#include <signal.h>
#include <sys/wait.h>
#include <unistd.h>
#include "proc.h"

static pid_t spawn(int nice_value, int fifo) {
    pid_t pid = fork();
    if (pid == 0) {
        cpu_set_t set;
        CPU_ZERO(&set);
        CPU_SET(0, &set);
        sched_setaffinity(0, sizeof set, &set);           /* both on CPU 0: they must share it */
        if (fifo) { struct sched_param sp = { .sched_priority = 10 }; sched_setscheduler(0, SCHED_FIFO, &sp); }
        else if (nice(nice_value) == -1) _exit(1);
        for (volatile unsigned long n = 0;; n++) { }
    }
    return pid;
}

static double cpu_seconds(pid_t pid) {                   /* utime + stime from /proc/PID/stat, in clock ticks */
    char path[64], buf[1024];
    snprintf(path, sizeof path, "/proc/%d/stat", pid);
    FILE *f = fopen(path, "r");
    if (!f) return 0;
    size_t n = fread(buf, 1, sizeof buf - 1, f);
    fclose(f);
    buf[n] = 0;
    char *p = strrchr(buf, ')');                           /* fields after "(comm)": state is field 3 */
    unsigned long utime = 0, stime = 0;
    if (p) sscanf(p + 2, "%*c %*d %*d %*d %*d %*d %*u %*u %*u %*u %*u %lu %lu", &utime, &stime);
    return (double)(utime + stime) / (double)sysconf(_SC_CLK_TCK);
}

static long ctxt(pid_t pid, const char *key) {
    char path[64];
    snprintf(path, sizeof path, "/proc/%d/status", pid);
    return proc_value(path, key);
}

int main(int argc, char **argv) {
    int nice_b = atoi(argv[1]), fifo = argc > 2;
    pid_t a = spawn(0, 0), b = spawn(nice_b, fifo);
    sleep(3);
    double ca = cpu_seconds(a), cb = cpu_seconds(b);
    char label[16];
    if (fifo) snprintf(label, sizeof label, "SCHED_FIFO");
    else snprintf(label, sizeof label, "nice %d", nice_b);
    printf("A nice 0: %5.2f s CPU (%4.1f%%) | B %-10s: %5.2f s CPU (%4.1f%%) | preempted: A %ld times, B %ld times\n",
           ca, 100 * ca / (ca + cb), label, cb, 100 * cb / (ca + cb),
           ctxt(a, "nonvoluntary_ctxt_switches"), ctxt(b, "nonvoluntary_ctxt_switches"));
    kill(a, SIGKILL);
    kill(b, SIGKILL);
    while (wait(NULL) > 0) { }
    return 0;
}
```

```sh
# file: guest-sched/run-sched.sh
for n in 0 1 5 10 19; do /bin/share $n; done
/bin/share 0 fifo
echo "sched_rt_runtime_us=$(cat /proc/sys/kernel/sched_rt_runtime_us) sched_rt_period_us=$(cat /proc/sys/kernel/sched_rt_period_us)"
```

```console
$ mkdir -p g5/bin && gcc -static -O2 -Wall -o g5/bin/share lab/share.c && cp guest-sched/run-sched.sh g5/ && sh mkinitramfs.sh g5.cpio init-modules.sh g5 > /dev/null && bash boot.sh g5.cpio quiet 2>&1
A nice 0:  1.50 s CPU (50.0%) | B nice 0    :  1.50 s CPU (50.0%) | preempted: A 365 times, B 365 times
A nice 0:  1.66 s CPU (55.5%) | B nice 1    :  1.33 s CPU (44.5%) | preempted: A 335 times, B 334 times
A nice 0:  2.26 s CPU (75.6%) | B nice 5    :  0.73 s CPU (24.4%) | preempted: A 187 times, B 184 times
A nice 0:  2.70 s CPU (90.3%) | B nice 10   :  0.29 s CPU ( 9.7%) | preempted: A 76 times, B 72 times
A nice 0:  2.95 s CPU (98.7%) | B nice 19   :  0.04 s CPU ( 1.3%) | preempted: A 16 times, B 11 times
A nice 0:  0.14 s CPU ( 4.7%) | B SCHED_FIFO:  2.85 s CPU (95.3%) | preempted: A 4 times, B 3 times
sched_rt_runtime_us=950000 sched_rt_period_us=1000000
```

The prediction from the weight table: nice 1 has weight 820, so B should get 820 / (1024 + 820) = 44.5%; nice 5 (335) → 24.7%; nice 10 (110) → 9.7%; nice 19 (15) → 1.4%. The measurements follow the table closely. Against a `SCHED_FIFO` task, the normal task gets almost nothing — what it does get is the 5% that real-time throttling reserves: 950,000 µs of every 1,000,000 µs for real-time tasks.

### Deep & Tricky

**DT-26. A `SCHED_FIFO` task runs until it blocks or yields; no time slice and no `nice` value of a normal task can preempt it. The only reason a spinning real-time task does not freeze a CPU completely is RT throttling, a safety net that limits all real-time tasks together to `sched_rt_runtime_us` per `sched_rt_period_us` (950 ms per second by default) — a limit that real-time applications with genuine deadlines often disable, at which point one bug can hang a CPU.** *(Linux `Documentation/scheduler/sched-rt-group.rst`: "the default values … are 1000000 and 950000 … 0.05s to be used by SCHED_OTHER"; `man 7 sched`)* Setting `sched_rt_runtime_us` to -1 removes the limit. Since Linux 6.12 the kernel can also run normal tasks through a **deadline server** (`fair_server`) instead of relying only on throttling, but the observable default — about 5% for everyone else — is the same.

### Pattern

**P-12. Before changing kernel behaviour or tuning it, measure it from the counters the kernel already publishes: `/proc/<pid>/status`, `stat`, `schedstat` (with `CONFIG_SCHED_INFO`), `maps`, `smaps`; `/proc/meminfo`, `vmstat`, `buddyinfo`, `slabinfo`; and `getrusage`. Write the prediction down first (from the source or documentation), then compare.** *(Practice; this is how kernel performance work is argued on the mailing lists — a patch that changes behaviour comes with before/after numbers)* Every result in this chapter was a prediction from the source checked by a counter: one fault per page, 44.5% at nice 1, 5% under a FIFO task. *Trade-off:* `/proc` counters are coarse and sampled; for "why" questions (which code path, how long each step takes) you need tracing — ftrace, perf, and BPF — which Chapter 12 and Chapter 13 introduce.

### Mastery check

1. **Predict.** After `fork()`, the child calls `exec()` immediately. How many copy-on-write faults does the child take, and what happens to the parent's write-protected pages?
2. **Predict.** Two nice-0 tasks share CPU 0 and a third nice-0 task runs on CPU 1. What share of a CPU does each get?
3. **Predict.** `VmRSS` of a process is 100 MB. Its child (after `fork`, no writes) also shows `VmRSS` 100 MB. How much physical memory do the two use together?

## Chapter 11 Project — `ptop`: a tiny `top` from `/proc`

**Goal:** a program that samples every process twice, one second apart, and prints a table sorted by CPU use: PID, command, state, CPU % (from `utime + stime` in `/proc/<pid>/stat`, in clock ticks), RSS, minor and major faults per second, and voluntary/involuntary context switches. Run it in the guest while a mix of workloads is running: a CPU spinner, a niced spinner, a sleeper, and a process that keeps touching new memory.

```c
// file: lab/ptop.c
#include <ctype.h>
#include <dirent.h>
#include <time.h>
#include <unistd.h>
#include "proc.h"

static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec / 1e9; }

struct sample { int pid; char comm[20], state; unsigned long utime, stime, minflt, majflt; long rss_pages; };

static int read_stat(int pid, struct sample *s) {
    char path[64], buf[1024];
    snprintf(path, sizeof path, "/proc/%d/stat", pid);
    FILE *f = fopen(path, "r");
    if (!f) return -1;
    size_t n = fread(buf, 1, sizeof buf - 1, f);
    fclose(f);
    buf[n] = 0;
    char *open = strchr(buf, '('), *close = strrchr(buf, ')');   /* comm may contain spaces and ')' */
    if (!open || !close) return -1;
    s->pid = pid;
    snprintf(s->comm, sizeof s->comm, "%.*s", (int)(close - open - 1), open + 1);
    /* fields after comm: 3 state ... 10 minflt 12 majflt 14 utime 15 stime ... 24 rss */
    unsigned long v[30] = {0};
    char *p = close + 2;
    s->state = *p;
    for (int i = 4; i <= 24 && p; i++) { p = strchr(p, ' '); if (p) v[i] = strtoul(++p, NULL, 10); }
    s->minflt = v[10]; s->majflt = v[12]; s->utime = v[14]; s->stime = v[15]; s->rss_pages = (long)v[24];
    return 0;
}

static int snapshot(struct sample *out, int max) {
    DIR *d = opendir("/proc");
    struct dirent *e;
    int n = 0;
    while ((e = readdir(d)) && n < max)
        if (isdigit((unsigned char)e->d_name[0]) && read_stat(atoi(e->d_name), &out[n]) == 0) n++;
    closedir(d);
    return n;
}

int main(void) {
    static struct sample a[512], b[512];
    long hz = sysconf(_SC_CLK_TCK);
    double t0 = now();
    int na = snapshot(a, 512);
    sleep(1);
    double elapsed = now() - t0;                          /* sleep(1) plus the time to read /proc: measure it */
    int nb = snapshot(b, 512);
    struct row { struct sample *s; double cpu; long minf; } rows[512];
    int nr = 0;
    for (int i = 0; i < nb; i++)
        for (int j = 0; j < na; j++)
            if (a[j].pid == b[i].pid) {
                rows[nr].s = &b[i];
                rows[nr].cpu = 100.0 * (double)(b[i].utime + b[i].stime - a[j].utime - a[j].stime) / (double)hz / elapsed;
                rows[nr].minf = (long)((double)(b[i].minflt - a[j].minflt) / elapsed);
                nr++;
            }
    for (int i = 0; i < nr; i++)                          /* sort by CPU, descending */
        for (int j = i + 1; j < nr; j++)
            if (rows[j].cpu > rows[i].cpu) { struct row t = rows[i]; rows[i] = rows[j]; rows[j] = t; }
    printf("  PID COMMAND          S   CPU%%   RSS(kB)  minflt/s  ctxsw vol/invol\n");
    for (int i = 0; i < nr && i < 10; i++) {
        struct sample *s = rows[i].s;
        char path[64];
        snprintf(path, sizeof path, "/proc/%d/status", s->pid);
        printf("%5d %-16s %c %6.1f %9ld %9ld  %6ld/%ld\n", s->pid, s->comm, s->state, rows[i].cpu, s->rss_pages * 4,
               rows[i].minf, proc_value(path, "voluntary_ctxt_switches"), proc_value(path, "nonvoluntary_ctxt_switches"));
    }
    return 0;
}
```

```c
// file: lab/toucher.c
/* Keep touching new memory, then give it back: a steady stream of page faults. */
#include <sys/mman.h>
#include <unistd.h>

int main(void) {
    for (;;) {
        char *p = mmap(NULL, 8 << 20, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
        for (long i = 0; i < (8 << 20); i += 4096) p[i] = 1;
        munmap(p, 8 << 20);
        usleep(10000);
    }
}
```

```sh
# file: guest-ptop/run-ptop.sh
spin() { while :; do :; done; }
spin & spin & sh -c 'renice -n 10 -p $$ > /dev/null; while :; do :; done' &
sleep 1000 & /bin/toucher &
sleep 2
/bin/ptop
```

```console
$ mkdir -p g6/bin && for p in ptop toucher; do gcc -static -O2 -Wall -o g6/bin/$p lab/$p.c; done && cp guest-ptop/run-ptop.sh g6/ && sh mkinitramfs.sh g6.cpio init-modules.sh g6 > /dev/null && bash boot.sh g6.cpio quiet 2>&1
  PID COMMAND          S   CPU%   RSS(kB)  minflt/s  ctxsw vol/invol
   46 sh               R   73.3       700         0       0/323
   50 toucher          R   54.3      8144     13124      22/324
   47 sh               R   53.4       724         0       0/338
   48 sh               R   10.3      1544         0       1/94
   52 ptop             R    4.3       652         0       1/43
   15 rcu_sched        I    2.6         0         0     331/0
    7 kworker/0:0-events_ I    0.0         0         0      16/8
    8 kworker/0:0H-events I    0.0         0         0       5/0
    9 kworker/0:1      I    0.0         0         0       5/0
   10 kworker/u8:0-eval_m I    0.0         0         0       4/1
```

Four busy tasks share two CPUs: two spinners at nice 0, the spinner at nice 10, and the toucher. The niced spinner gets about a tenth of a CPU, as the weights predict. The toucher takes about 13,000 minor faults per second (2,048 per 8 MiB round) — and a large share of a CPU: in this debug kernel under emulation each fault is expensive (KASAN, lockdep, emulated page-table walks), much more than on real hardware. Its *voluntary* switches (it sleeps 10 ms per round) set it apart from the spinners, which are only ever preempted. This is the same arithmetic `top` and `htop` do.

### Review checklist

- `ptop` reads `/proc/<pid>/stat` with `strrchr(buf, ')')`. Why not `sscanf("%d (%s) %c ...")`?
- A process exits between the two snapshots. What does `ptop` do with it, and is that right?
- CPU% is computed from clock ticks (`utime`, `stime`). What is their resolution here, and what does `/proc/<pid>/schedstat` (available with `CONFIG_SCHED_INFO`, which this kernel lacks) offer instead?

## Answers to the Chapter 11 prediction questions

**11.5 Q1.** Usually none or very few: `exec` throws away the child's copy of the address space and builds a new one, so the child does not write to the inherited pages (a few writes on the stack before `exec` cost a few faults). The parent's pages stay write-protected until it writes to each of them once more; then each costs a fault, which reuses the page without copying because the child's mapping is gone (DT-24).

**11.5 Q2.** The load balancer puts two tasks on one CPU and one on the other, so two tasks get 50% of a CPU each and one gets 100% — unless the balancer migrates tasks around, in which case each gets about 2/3 of a CPU on average. Fair shares are per CPU run queue; the balancer works to equalise *load* across CPUs, not to give every task an exactly equal share.

**11.5 Q3.** About 100 MB plus a little: the pages are shared copy-on-write, and RSS counts every page mapped into the process even if another process maps the same page. `/proc/<pid>/smaps` gives `Pss` (proportional set size), which divides each shared page among its users: about 50 MB each here.

# Chapter 12 — Synchronization, RCU, and Debugging the Kernel

*Goal: learn the kernel's locking toolbox and, above all, the tools that find concurrency and memory bugs for you. Linux has the same problems our hobby kernel met in Chapter 7 — on hundreds of CPUs — and it answers them with a set of primitives (spinlocks, mutexes, read-write semaphores, atomics, per-CPU data, sequence locks, and **RCU**) and with run-time checkers that turn "it hangs once a month" into a report with stack traces: **lockdep**, **KASAN**, and the sleep-in-atomic check. Then we look inside the running kernel without changing it: **ftrace** and **kprobes**.*

**What you will be able to do after this chapter**

- Choose between spinlock, mutex, rwsem, atomic, and RCU for a given piece of shared data.
- Read a lockdep report and explain a deadlock that did not happen — yet.
- Write an RCU-protected pointer update: publish, read, wait for a grace period, free.
- Trace the call graph of a system call with ftrace, watch scheduler events, and add a probe on any kernel function with a kprobe event.

```console
$ ln -sfn /home/claude/kernelsrc/linux linux && ln -sfn /home/claude/kernelsrc/build build && cp ../ch08/mkinitramfs.sh ../ch08/boot.sh ../ch09/init-modules.sh . && echo ready
ready
```

## Topic 12A — Locks and lockdep

## Unit 12.1 — The kernel's locking toolbox

**One-line summary:** Use a **spinlock** (`spinlock_t`, `spin_lock_irqsave`) for short critical sections that never sleep, especially those shared with interrupt handlers. Use a **mutex** (`struct mutex`) when the critical section may sleep (allocate with `GFP_KERNEL`, copy to user, wait for I/O); it puts waiters to sleep instead of spinning. A **rw_semaphore** allows many readers or one writer. **`atomic_t`/`refcount_t`** cover single counters. **Per-CPU** variables avoid sharing altogether. A **seqlock** lets readers retry instead of blocking a frequent writer. **RCU** lets readers run with no lock and no atomic instruction at all, at the cost of more work for writers (Unit 12.3).

| Primitive | May sleep inside? | Use from IRQ? | Readers in parallel? | Typical use |
|---|---|---|---|---|
| `spinlock_t` | no | yes (`_irqsave`) | no | short, IRQ-shared state |
| `rwlock_t` | no | yes | yes | rarely now; RCU is preferred |
| `struct mutex` | yes | no | no | most sleeping-context state |
| `struct rw_semaphore` | yes | no | yes | `mmap_lock`, read-mostly trees |
| `atomic_t`, `refcount_t` | — | yes | — | counters, reference counts |
| `seqlock_t` | no (readers retry) | yes | yes (lock-free) | time, small hot read-mostly data |
| RCU | readers: no (classic RCU) | yes | yes (lock-free) | lists and pointers read far more than written |

## Unit 12.2 — lockdep: deadlocks that have not happened

**One-line summary:** **lockdep** (`CONFIG_PROVE_LOCKING`, `kernel/locking/lockdep.c`) watches every lock acquisition at run time. It groups locks into **classes** (all instances initialised at the same place), records every "class A was held while class B was taken" edge, and reports as soon as a new edge would close a cycle — the first time the *possibility* of a deadlock appears, even if the timing that would actually deadlock never occurs. It also checks irq-safety (Chapter 7's DT-17, in general form) and RCU usage.

```c
// file: lockorder/lockorder.c
#include <linux/module.h>
#include <linux/mutex.h>

static DEFINE_MUTEX(config_lock);
static DEFINE_MUTEX(stats_lock);
static int config, stats;

static noinline void update_config(void)	/* takes config, then stats */
{
	mutex_lock(&config_lock);
	config++;
	mutex_lock(&stats_lock);
	stats++;
	mutex_unlock(&stats_lock);
	mutex_unlock(&config_lock);
}

static noinline void report_stats(void)	/* takes stats, then config: the opposite order */
{
	mutex_lock(&stats_lock);
	mutex_lock(&config_lock);
	pr_info("lockorder: config=%d stats=%d\n", config, stats);
	mutex_unlock(&config_lock);
	mutex_unlock(&stats_lock);
}

static int __init lockorder_init(void)
{
	update_config();		/* one after the other, in one thread: no deadlock can happen here */
	report_stats();
	return 0;
}

static void __exit lockorder_exit(void) { }
module_init(lockorder_init);
module_exit(lockorder_exit);
MODULE_LICENSE("GPL");
```

```text
# file: lockorder/Makefile
obj-m += lockorder.o
```

```sh
# file: guest-lockdep/run-lockdep.sh
dmesg -n 8
insmod /modules/lockorder.ko && echo "insmod finished normally"
```

```console
$ make -s -C build M=$PWD/lockorder modules && mkdir -p g1/modules && cp lockorder/lockorder.ko g1/modules/ && cp guest-lockdep/run-lockdep.sh g1/ && sh mkinitramfs.sh g1.cpio init-modules.sh g1 > /dev/null && bash boot.sh g1.cpio quiet 2>&1 | sed 's/^\[[ 0-9.]*\] //' | grep -E "WARNING|circular|trying to acquire|already holding|^ *\((config|stats)_lock|at: (update_config|report_stats)|existing dependency|Possible unsafe|CPU0|CPU1|lock\(|DEADLOCK|insmod finished|lockorder:" | head -30
lockorder: loading out-of-tree module taints kernel.
WARNING: possible circular locking dependency detected
insmod/47 is trying to acquire lock:
ffffffffa0210128 (config_lock){+.+.}-{3:3}, at: report_stats+0x21/0x51 [lockorder]
but task is already holding lock:
ffffffffa0210068 (stats_lock){+.+.}-{3:3}, at: report_stats+0x13/0x51 [lockorder]
the existing dependency chain (in reverse order) is:
 Possible unsafe locking scenario:
       CPU0                    CPU1
  lock(stats_lock);
                               lock(config_lock);
                               lock(stats_lock);
  lock(config_lock);
 *** DEADLOCK ***
 #0: ffffffffa0210068 (stats_lock){+.+.}-{3:3}, at: report_stats+0x13/0x51 [lockorder]
 print_circular_bug+0x363/0x37f
 check_noncircular+0x184/0x1ad
 ? print_circular_bug+0x37f/0x37f
lockorder: config=1 stats=1
insmod finished normally
```

### Deep & Tricky

**DT-27. A lock-order inversion is a bug even if every test passes: two code paths that take the same two locks in opposite orders deadlock only when they run at the same moment on two CPUs, which may take months in production. lockdep reports it the first time both orders have been *seen*, in any thread, at any time — here in a single thread with no concurrency at all.** *(Linux `Documentation/locking/lockdep-design.rst`: "the validator … proves that a deadlock cannot happen" by tracking lock-class dependencies; "even if the actual deadlock never occurs")* The report has three parts: the new acquisition (`config_lock` taken in `report_stats` while holding `stats_lock`), the existing dependency chain that it closes into a cycle (`config_lock` → `stats_lock`, recorded in `update_config`), and the two-CPU scenario that would deadlock. The fix is a documented lock order (always `config_lock` first) — which is why kernel source files often begin with a comment listing their lock order (see the top of `mm/rmap.c` or `mm/filemap.c`).

## Topic 12B — RCU

## Unit 12.3 — Read-Copy-Update

**One-line summary:** RCU protects a pointer to data that is read very often and changed rarely. A **reader** wraps its access in `rcu_read_lock()`/`rcu_read_unlock()` (which, without kernel preemption, costs nothing at all) and loads the pointer with `rcu_dereference()`. A **writer** builds a new copy, publishes it with `rcu_assign_pointer()` (a release store, so readers never see a half-built object), and must not free the old copy until every reader that might still hold it has finished: it waits for a **grace period** with `synchronize_rcu()` (blocking) or hands the free to RCU with `kfree_rcu()` (asynchronous). A grace period ends once every CPU has passed through a quiescent state — a context switch, idle, or user mode — because a classic RCU reader may not sleep inside its critical section.

```c
// file: rcudemo/rcudemo.c
#include <linux/delay.h>
#include <linux/kthread.h>
#include <linux/ktime.h>
#include <linux/module.h>
#include <linux/rcupdate.h>
#include <linux/slab.h>

struct config {
	int a, b;			/* invariant: a + b == 100 */
	struct rcu_head rcu;
};

static struct config __rcu *cfg;
static DEFINE_MUTEX(update_lock);	/* writers serialise among themselves; readers take no lock */
static atomic_long_t reads, bad;
static int mode;			/* 0: synchronize_rcu, 1: kfree_rcu, 2: BROKEN - kfree at once */
module_param(mode, int, 0);

static int reader_fn(void *unused)
{
	while (!kthread_should_stop()) {
		struct config *c;
		int a;

		rcu_read_lock();
		c = rcu_dereference(cfg);
		a = c->a;
		udelay(20);		/* a slow reader: keeps the old version in use for a while */
		if (a + c->b != 100)
			atomic_long_inc(&bad);
		rcu_read_unlock();
		atomic_long_inc(&reads);
		cond_resched();		/* a quiescent state: lets grace periods end */
	}
	return 0;
}

static void update(int i)
{
	struct config *new = kmalloc(sizeof(*new), GFP_KERNEL), *old;

	if (!new)
		return;
	new->a = i;
	new->b = 100 - i;
	mutex_lock(&update_lock);
	old = rcu_replace_pointer(cfg, new, lockdep_is_held(&update_lock));	/* publish */
	mutex_unlock(&update_lock);
	if (mode == 0) {
		synchronize_rcu();	/* wait until no reader can still see old */
		kfree(old);
	} else if (mode == 1) {
		kfree_rcu(old, rcu);	/* free it after a grace period, without waiting */
	} else {
		kfree(old);		/* BROKEN: a reader may still be using old */
	}
}

static int __init rcudemo_init(void)
{
	struct task_struct *reader;
	struct config *c = kmalloc(sizeof(*c), GFP_KERNEL);
	int i, updates = mode == 0 ? 100 : 2000;
	ktime_t t0;

	if (!c)
		return -ENOMEM;
	c->a = 0;
	c->b = 100;
	RCU_INIT_POINTER(cfg, c);
	reader = kthread_run(reader_fn, NULL, "rcu-reader");
	if (IS_ERR(reader))
		return PTR_ERR(reader);
	t0 = ktime_get();
	for (i = 1; i <= updates; i++)
		update(i % 100);
	pr_info("rcudemo: mode %d: %d updates in %lld ms (%lld us each), %ld reads, %ld inconsistent\n", mode, updates,
		ktime_ms_delta(ktime_get(), t0), ktime_us_delta(ktime_get(), t0) / updates,
		atomic_long_read(&reads), atomic_long_read(&bad));
	kthread_stop(reader);
	synchronize_rcu();
	kfree(rcu_dereference_protected(cfg, 1));
	return 0;
}

static void __exit rcudemo_exit(void) { }
module_init(rcudemo_init);
module_exit(rcudemo_exit);
MODULE_LICENSE("GPL");
```

```text
# file: rcudemo/Makefile
obj-m += rcudemo.o
```

```sh
# file: guest-rcu/run-rcu.sh
cd /modules
insmod rcudemo.ko mode=0 && rmmod rcudemo
insmod rcudemo.ko mode=1 && rmmod rcudemo
dmesg | grep "rcudemo:" | sed 's/^\[[ 0-9.]*\] //'
dmesg -n 8
insmod rcudemo.ko mode=2
```

```console
$ make -s -C build M=$PWD/rcudemo modules && mkdir -p g2/modules && cp rcudemo/rcudemo.ko g2/modules/ && cp guest-rcu/run-rcu.sh g2/ && sh mkinitramfs.sh g2.cpio init-modules.sh g2 > /dev/null && bash boot.sh g2.cpio quiet 2>&1 | sed 's/^\[[ 0-9.]*\] //' | grep -E "^rcudemo:|BUG: KASAN|Read of size|^ (reader_fn|update|kfree|rcudemo_init)\+|Freed by task|Allocated by task" | head -14 | cut -c1-110
rcudemo: loading out-of-tree module taints kernel.
rcudemo: mode 0: 100 updates in 1363 ms (13639 us each), 52806 reads, 0 inconsistent
rcudemo: mode 1: 2000 updates in 82 ms (41 us each), 3451 reads, 0 inconsistent
BUG: KASAN: slab-use-after-free in reader_fn+0x124/0x1df [rcudemo]
Read of size 4 at addr ffff888000bcfd84 by task rcu-reader/57
 reader_fn+0x124/0x1df [rcudemo]
Allocated by task 56:
 rcudemo_init+0x171/0x1000 [rcudemo]
Freed by task 56:
 kfree+0x1bd/0x222
 rcudemo_init+0x271/0x1000 [rcudemo]
rcudemo: mode 2: 2000 updates in 62 ms (31 us each), 2239 reads, 575 inconsistent
```

Modes 0 and 1 keep the invariant on every read — the reader never sees a half-updated or freed object, without taking any lock. `synchronize_rcu()` is expensive (each update waits for a whole grace period — milliseconds); `kfree_rcu()` makes updates cheap by batching the frees. Mode 2 frees immediately, and KASAN catches the reader touching the freed object. KASAN reports only the first such bug by default (`kasan_multi_shot` changes that); the run went on, and the reader saw hundreds of inconsistent versions — memory that had been freed and reused for the next update while the reader was still looking at it.

### Deep & Tricky

**DT-28. Publishing a new version is only half of an RCU update; the old version must outlive every reader that may hold it. Freeing it immediately is a use-after-free that, without KASAN, usually reads stale-but-plausible data and goes unnoticed — the bug RCU is famous for being easy to make. The rule is: unlink or replace, *then* wait for a grace period (or `call_rcu`/`kfree_rcu`), *then* free.** *(Linux `Documentation/RCU/whatisRCU.rst`, §3 "the synchronize_rcu() … waits for all pre-existing RCU read-side critical sections"; `Documentation/RCU/checklist.rst`, item: "the updater … must not free … until after a grace period has elapsed")* With `CONFIG_PROVE_RCU` (enabled here through lockdep), the kernel also checks the other side: `rcu_dereference()` outside `rcu_read_lock()` — or a `rcu_dereference_protected()` whose lockdep condition is false — prints "suspicious RCU usage".

## Topic 12C — Looking inside: ftrace and kprobes

## Unit 12.4 — ftrace: function graphs and trace events

**One-line summary:** **ftrace** (`kernel/trace/`) is the tracer built into the kernel, controlled through files in **tracefs** (`/sys/kernel/tracing`). The `function_graph` tracer records every kernel function entry and exit with durations, filtered to the call graph below chosen functions (`set_graph_function`) and limited in depth. **Trace events** are static trace points placed in the source (`sched_switch`, `sys_enter_openat`, …); enabling one costs almost nothing when off and records structured fields when on. With `DYNAMIC_FTRACE`, the instrumentation is patched in only while tracing, so a kernel with ftrace compiled in runs at full speed otherwise.

```sh
# file: guest-ftrace/run-ftrace.sh
mount -t tracefs tracefs /sys/kernel/tracing
echo guest > /etc/hostname
cd /sys/kernel/tracing
echo "available tracers: $(cat available_tracers)"
echo "functions that can be traced: $(wc -l < available_filter_functions)"
echo do_sys_openat2 > set_graph_function
echo 2 > max_graph_depth
echo function_graph > current_tracer
cat /etc/hostname > /dev/null
echo 0 > tracing_on                         # stop recording; changing the tracer would clear the buffer
echo "== function_graph of do_sys_openat2, depth 2 =="
grep -vE "^#|^ *$" trace | head -24
echo nop > current_tracer
echo 1 > tracing_on
echo 1 > events/sched/sched_switch/enable
(i=0; while [ $i -lt 30000 ]; do i=$((i + 1)); done) &
sleep 0.3
echo 0 > events/sched/sched_switch/enable
echo "== sched_switch events: $(grep -c sched_switch trace) recorded; first three =="
grep sched_switch trace | head -3 | sed -E 's/ +/ /g' | cut -c1-140
```

```console
$ mkdir -p g3 && cp guest-ftrace/run-ftrace.sh g3/ && sh mkinitramfs.sh g3.cpio init-modules.sh g3 > /dev/null && bash boot.sh g3.cpio quiet 2>&1 | cut -c1-120
available tracers: function_graph function nop
functions that can be traced: 23251
== function_graph of do_sys_openat2, depth 2 ==
 1)               |  do_sys_openat2() {
 1) # 1373.792 us |    getname();
 1) + 92.580 us   |    get_unused_fd_flags();
 1) # 1337.320 us |    do_filp_open();
 1) + 35.446 us   |    fd_install();
 1) ! 128.196 us  |    putname();
 1) # 5280.366 us |  }
 0)               |  do_sys_openat2() {
 0) ! 175.605 us  |    getname();
 0) + 39.082 us   |    get_unused_fd_flags();
 0) ! 263.548 us  |    do_filp_open();
 0) + 17.377 us   |    fd_install();
 0) + 97.626 us   |    putname();
 0) ! 680.381 us  |  }
 1)               |  do_sys_openat2() {
 1) ! 134.038 us  |    getname();
 1) + 24.179 us   |    get_unused_fd_flags();
 1) ! 780.348 us  |    do_filp_open();
 1) + 17.821 us   |    fd_install();
 1) ! 107.934 us  |    putname();
 1) # 1136.731 us |  }
== sched_switch events: 25 recorded; first three ==
 sh-44 [001] d..3. 19.753074: sched_switch: prev_comm=sh prev_pid=44 prev_prio=120 prev_state=R ==> next_comm=kworker/1:
 <idle>-0 [000] d..3. 19.753074: sched_switch: prev_comm=swapper/0 prev_pid=0 prev_prio=120 prev_state=R ==> next_comm=s
 kworker/1:2-36 [001] d..3. 19.754206: sched_switch: prev_comm=kworker/1:2 prev_pid=36 prev_prio=120 prev_state=I ==> ne
```

The graph shows what opening a file costs inside the kernel, function by function. The first call was interrupted by the timer: the interrupt's entry, handler, and exit appear *inside* `do_sys_openat2`, and their time is counted in its total — a tracer shows what happened on the CPU, not only what the function did. The other calls show the steps cleanly: copying the path in from user space (`getname`), allocating a descriptor, the path walk and open (`do_filp_open`), installing the file. Durations are in microseconds — inflated here by emulation and by KASAN, which instruments every memory access. The `sched_switch` events record each context switch with the previous task's name, PID, priority and state, and the next task's.

## Unit 12.5 — kprobes: a probe on any function

**One-line summary:** A **kprobe** (`kernel/kprobes.c`) plants a breakpoint at almost any kernel instruction; when it is hit, a handler runs and the original instruction is then executed. Through tracefs's `kprobe_events`, a probe can be defined with a one-line description — name, function, and *fetch arguments* such as registers (`%di` is the first argument on x86-64) and memory dereferences (`+0(%si):string`) — and then used like any trace event. A `kretprobe` (`r:`) fires on function return and can record the return value (`$retval`). BPF programs attach to the same mechanism (Chapter 13).

```sh
# file: guest-kprobe/run-kprobe.sh
mount -t tracefs tracefs /sys/kernel/tracing
echo guest > /etc/hostname
cd /sys/kernel/tracing
echo 'p:myopen do_sys_openat2 dfd=%di:s32 path=+0(%si):string' >> kprobe_events
echo 'r:myopen_ret do_sys_openat2 fd=$retval:s32' >> kprobe_events
cat kprobe_events
echo 1 > events/kprobes/enable
cat /etc/hostname > /dev/null
cat /no/such/file 2> /dev/null
echo 0 > events/kprobes/enable
grep -E "myopen" trace | grep -vE '"/dev/null"|events/kprobes' | sed -E 's/^ +//; s/ +/ /g' | cut -d' ' -f1,5- | grep -A1 -E 'hostname|no/such' | grep -v "^--"
```

```console
$ mkdir -p g4 && cp guest-kprobe/run-kprobe.sh g4/ && sh mkinitramfs.sh g4.cpio init-modules.sh g4 > /dev/null && bash boot.sh g4.cpio quiet 2>&1 | cut -c1-120
p:kprobes/myopen do_sys_openat2 dfd=%di:s32 path=+0(%si):string
r10:kprobes/myopen_ret do_sys_openat2 fd=$retval:s32
cat-48 myopen: (do_sys_openat2+0x0/0x124) dfd=-100 path="/etc/hostname"
cat-48 myopen_ret: (do_sys_open+0x71/0x8c <- do_sys_openat2) fd=3
cat-49 myopen: (do_sys_openat2+0x0/0x124) dfd=-100 path="/no/such/file"
cat-49 myopen_ret: (do_sys_open+0x71/0x8c <- do_sys_openat2) fd=-2
```

Each `open` shows up twice: the entry probe with the directory descriptor (`-100` is `AT_FDCWD`, "relative to the current directory") and the path string read from user memory, and the return probe with the result — a descriptor number, or `-2` (`ENOENT`) for the missing file. No module was written and nothing was rebuilt; this works on any production kernel with kprobes enabled, which is why it is the first tool to reach for when asking "is this function even called, and with what?".

### Pattern

**P-13. Develop and test kernel code on a debug kernel — KASAN, lockdep (`PROVE_LOCKING`, which includes `PROVE_RCU`), `DEBUG_ATOMIC_SLEEP`, and for data races KCSAN — and investigate running systems with tracing, not by adding `printk`s and rebuilding.** *(Linux `Documentation/process/submit-checklist.rst`: test "with CONFIG_PREEMPT, CONFIG_DEBUG_PREEMPT, CONFIG_SLUB_DEBUG, CONFIG_DEBUG_PAGEALLOC, CONFIG_DEBUG_MUTEXES, CONFIG_DEBUG_SPINLOCK, CONFIG_DEBUG_ATOMIC_SLEEP, CONFIG_PROVE_RCU and CONFIG_DEBUG_OBJECTS_RCU_HEAD all simultaneously enabled")* The checkers find bug classes whose symptoms are rare and far from their cause; the tracers answer questions about a kernel you cannot rebuild. *Trade-off:* a debug kernel is several times slower and uses much more memory (KASAN's shadow alone is 1/8 of all memory), so performance must be measured on a separate, production-like build.

### Mastery check

1. **Predict.** Why can a classic RCU reader not call `msleep()` inside `rcu_read_lock()`?
2. **Predict.** lockdep groups locks into classes by where they were initialised. Why would giving every `struct inode`'s lock its own class make lockdep useless?
3. **Predict.** A kprobe on `do_sys_openat2` with `path=+0(%si):string`: why `%si`, and what would `+0(%di):string` print?

## Chapter 12 Project — Find and fix three bugs

**Goal:** `cache.ko` is a small key-value cache module with three bugs, each one invisible in a normal run and each one caught by a different checker in our debug kernel. Run it, read each report, find the bug in the source, and fix it; the fixed module must load and unload with a clean log. The three tests are selected with the module parameter `test=1..3`.

```c
// file: cache/cache.c
#include <linux/module.h>
#include <linux/rculist.h>
#include <linux/slab.h>
#include <linux/spinlock.h>

struct entry {
	struct list_head node;
	int key, value;
	struct rcu_head rcu;
};

static LIST_HEAD(entries);
static DEFINE_SPINLOCK(entries_lock);
static int test;
module_param(test, int, 0);

/* Test 1: insert under the spinlock */
static int cache_insert(int key, int value)
{
	struct entry *e;

	spin_lock(&entries_lock);
	e = kmalloc(sizeof(*e), GFP_KERNEL);
	if (!e) {
		spin_unlock(&entries_lock);
		return -ENOMEM;
	}
	e->key = key;
	e->value = value;
	list_add_rcu(&e->node, &entries);
	spin_unlock(&entries_lock);
	return 0;
}

/* Test 2: lockless lookup, with a pointer to the last entry found */
static struct entry __rcu *last_hit;

static int cache_lookup(int key)
{
	struct entry *e = rcu_dereference(last_hit);

	if (e && e->key == key)
		return e->value;
	list_for_each_entry_rcu(e, &entries, node) {
		if (e->key == key) {
			rcu_assign_pointer(last_hit, e);
			return e->value;
		}
	}
	return -1;
}

/* Test 3: removal */
static void cache_remove(int key)
{
	struct entry *e;

	spin_lock(&entries_lock);
	list_for_each_entry(e, &entries, node) {
		if (e->key == key) {
			list_del_rcu(&e->node);
			if (rcu_access_pointer(last_hit) == e)
				RCU_INIT_POINTER(last_hit, NULL);
			kfree(e);
			break;
		}
	}
	spin_unlock(&entries_lock);
}

static int __init cache_init(void)
{
	int i, found = 0;

	for (i = 0; i < 10; i++)
		if (cache_insert(i, i * i))
			return -ENOMEM;
	if (test >= 2)
		found = cache_lookup(7);
	if (test >= 3) {
		struct entry *e;

		rcu_read_lock();
		e = list_first_or_null_rcu(&entries, struct entry, node);
		cache_remove(e->key);		/* remove the entry we are looking at ... */
		found += e->value;		/* ... and keep using it, as an RCU reader may */
		rcu_read_unlock();
	}
	pr_info("cache: test %d done, found %d\n", test, found);
	return 0;
}

static void __exit cache_exit(void)
{
	struct entry *e, *tmp;

	list_for_each_entry_safe(e, tmp, &entries, node) {
		list_del(&e->node);
		kfree(e);
	}
}

module_init(cache_init);
module_exit(cache_exit);
MODULE_LICENSE("GPL");
```

```text
# file: cache/Makefile
obj-m += cache.o
```

```sh
# file: guest-cache/run-cache.sh
dmesg -n 1
for t in 1 2 3; do
    dmesg -c > /dev/null
    insmod /modules/cache.ko test=$t && rmmod cache
    echo "== test $t: $(dmesg | grep -cE '^(\[[ 0-9.]*\] )?(BUG|WARNING):') report(s) =="
    dmesg | sed 's/^\[[ 0-9.]*\] //' | grep -E "^BUG|^WARNING|suspicious|^cache:|at .*cache\.c:|^ (cache_[a-z]+)\+|lockdep_rcu|Read of size|in_atomic|kmalloc|Freed by" | grep -v "^ *$" | head -8
done
```

```console
$ make -s -C build M=$PWD/cache modules && mkdir -p g5/modules && cp cache/cache.ko g5/modules/ && cp guest-cache/run-cache.sh g5/ && sh mkinitramfs.sh g5.cpio init-modules.sh g5 > /dev/null && bash boot.sh g5.cpio quiet 2>&1 | cut -c1-110
== test 1: 1 report(s) ==
cache: loading out-of-tree module taints kernel.
BUG: sleeping function called from invalid context at include/linux/sched/mm.h:321
in_atomic(): 1, irqs_disabled(): 0, non_block: 0, pid: 47, name: insmod
 __kmalloc_cache_noprof+0x2b/0x15c
 cache_init+0x9a/0x1000 [cache]
cache: test 1 done, found 0
== test 2: 1 report(s) ==
WARNING: suspicious RCU usage
cache/cache.c:41 suspicious rcu_dereference_check() usage!
 lockdep_rcu_suspicious+0x1b7/0x1e8
 cache_init+0x1cf/0x1000 [cache]
cache: test 2 done, found 49
== test 3: 2 report(s) ==
WARNING: suspicious RCU usage
cache/cache.c:41 suspicious rcu_dereference_check() usage!
 lockdep_rcu_suspicious+0x1b7/0x1e8
 cache_init+0x1cf/0x1000 [cache]
BUG: KASAN: slab-use-after-free in cache_init+0x4e0/0x1000 [cache]
Read of size 4 at addr ffff88800483d094 by task insmod/69
 cache_init+0x4e0/0x1000 [cache]
 __kasan_kmalloc+0x3a/0x43
```

The three reports, and the bugs behind them:

1. **"BUG: sleeping function called from invalid context"** in `cache_insert`: `kmalloc(GFP_KERNEL)` may sleep, and it is called while holding a spinlock. Fix: allocate before taking the lock (or use `GFP_ATOMIC`, which may fail more easily — allocating first is better).
2. **"suspicious RCU usage"** in `cache_lookup`: `rcu_dereference()` outside `rcu_read_lock()`. Lockdep's RCU checking (`PROVE_RCU`) reports it even though nothing is freed concurrently in this run. (The unprotected `list_for_each_entry_rcu` loop is the same bug, but checking list iterators needs one more option, `CONFIG_PROVE_RCU_LIST`, which this kernel does not enable — one checker, one blind spot.) Fix: wrap the whole lookup in `rcu_read_lock()`/`rcu_read_unlock()`, and copy the value out before unlocking.
3. **"BUG: KASAN: slab-use-after-free"**: `cache_remove` frees the entry right after unlinking it, while an RCU reader (the caller, here) still holds a pointer to it. Fix: `kfree_rcu(e, rcu)` (DT-28).

```diff
// file: proj/cache-fix.patch
--- a/cache/cache.c
+++ b/cache/cache.c
@@ -17,14 +17,11 @@
 /* Test 1: insert under the spinlock */
 static int cache_insert(int key, int value)
 {
-	struct entry *e;
+	struct entry *e = kmalloc(sizeof(*e), GFP_KERNEL);	/* may sleep: allocate before locking */
 
+	if (!e)
+		return -ENOMEM;
 	spin_lock(&entries_lock);
-	e = kmalloc(sizeof(*e), GFP_KERNEL);
-	if (!e) {
-		spin_unlock(&entries_lock);
-		return -ENOMEM;
-	}
 	e->key = key;
 	e->value = value;
 	list_add_rcu(&e->node, &entries);
@@ -37,17 +34,25 @@
 
 static int cache_lookup(int key)
 {
-	struct entry *e = rcu_dereference(last_hit);
+	struct entry *e;
+	int value = -1;
 
-	if (e && e->key == key)
-		return e->value;
+	rcu_read_lock();			/* readers must be inside a read-side critical section */
+	e = rcu_dereference(last_hit);
+	if (e && e->key == key) {
+		value = e->value;
+		goto out;
+	}
 	list_for_each_entry_rcu(e, &entries, node) {
 		if (e->key == key) {
 			rcu_assign_pointer(last_hit, e);
-			return e->value;
+			value = e->value;
+			break;
 		}
 	}
-	return -1;
+out:
+	rcu_read_unlock();
+	return value;
 }
 
 /* Test 3: removal */
@@ -61,7 +66,7 @@
 			list_del_rcu(&e->node);
 			if (rcu_access_pointer(last_hit) == e)
 				RCU_INIT_POINTER(last_hit, NULL);
-			kfree(e);
+			kfree_rcu(e, rcu);	/* readers may still hold e */
 			break;
 		}
 	}
```

```console
$ cp cache/cache.c cache/cache.c.orig && patch -s -p1 < proj/cache-fix.patch && make -s -C build M=$PWD/cache modules && cp cache/cache.ko g5/modules/ && sh mkinitramfs.sh g5.cpio init-modules.sh g5 > /dev/null && bash boot.sh g5.cpio quiet 2>&1 | cut -c1-110; cp cache/cache.c.orig cache/cache.c
== test 1: 0 report(s) ==
cache: loading out-of-tree module taints kernel.
cache: test 1 done, found 0
== test 2: 0 report(s) ==
cache: test 2 done, found 49
== test 3: 0 report(s) ==
cache: test 3 done, found 130
```

All three tests now run without a single report. None of the three bugs changed the output of the buggy version — `found` was right every time. That is the case for the checkers: they report the rule that was broken, not the rare failure that breaking it causes.

### Review checklist

- The fixed `cache_insert` holds the spinlock only around `list_add_rcu`. Could it use a mutex instead? Would that change anything for the readers?
- `cache_exit` frees entries with plain `kfree`. Is that safe? What guarantees that no reader is still running at module exit?
- Which of the three bugs would also be found by a non-debug kernel, eventually, and how would it show up?

## Answers to the Chapter 12 prediction questions

**12.5 Q1.** A grace period ends only when every CPU has passed through a quiescent state; a context switch *is* one. If a reader slept inside its critical section, the CPU would switch to another task and report a quiescent state while the reader still holds a pointer to the old data — the writer could then free it under the sleeping reader. (Sleepable RCU, **SRCU**, exists for the cases where readers must sleep; it tracks readers explicitly.)

**12.5 Q2.** lockdep's ordering rules are per class: "inode lock before page lock" must hold for *every* inode. Giving each inode its own class would mean each lock is seen only a few times, so lockdep could never connect the order used in one code path with the opposite order in another path on a different inode — and it would also run out of class slots. Where two instances of the same class are legitimately nested (a parent and child directory), the code tells lockdep with `mutex_lock_nested()` and a subclass.

**12.5 Q3.** On x86-64 the first arguments are passed in RDI, RSI, RDX, RCX, R8, R9. `do_sys_openat2(int dfd, const char __user *filename, struct open_how *how)` has `filename` in RSI, so `+0(%si):string` reads the string at that address. `+0(%di):string` would treat the directory descriptor (-100) as a pointer and fail to read — tracefs prints `(fault)`.

# Chapter 13 — Networking and BPF

*Goal: follow data through the Linux network stack — from a socket in a program, through TCP/IP, to the e1000 driver and the emulated wire — and then run our own code inside the kernel without writing a module, using **BPF**. The guest gets a real network connection through QEMU's user-mode networking, talks to a web server on the host, captures its own packets with a packet socket, counts them with a BPF program loaded through the raw `bpf()` system call, and finally serves HTTP to the host.*

**What you will be able to do after this chapter**

- Configure an interface by hand and connect a guest to the host through QEMU's user-mode network.
- Name the layers a packet crosses in Linux and see each one in `/proc`, `/sys`, and a packet capture.
- Use stream sockets correctly — including the fact that TCP has no message boundaries.
- Capture raw frames with `AF_PACKET`, and decode Ethernet, IPv4, TCP, and UDP headers.
- Write, load, and attach an eBPF program with the raw `bpf()` system call, use a BPF map, and read the verifier's log.

```console
$ ln -sfn /home/claude/kernelsrc/linux linux && ln -sfn /home/claude/kernelsrc/build build && cp ../ch08/mkinitramfs.sh ../ch08/boot.sh ../ch09/init-modules.sh . && echo ready
ready
```

## Topic 13A — The stack

## Unit 13.1 — From socket to wire

**One-line summary:** A `send()` on a TCP socket copies data into the socket's send buffer (`net/ipv4/tcp.c`); TCP cuts it into segments and adds its header; IP (`net/ipv4/ip_output.c`) adds the IP header and chooses a route and a device; the neighbour layer resolves the next hop's MAC address with ARP; the queueing discipline (`net/sched/`) hands the frame, as a **`struct sk_buff`**, to the driver's `ndo_start_xmit` (`drivers/net/ethernet/intel/e1000/e1000_main.c`), which places it in a DMA ring and tells the card. Receiving runs the other way, starting from the card's interrupt and NAPI polling. In QEMU's **user-mode networking** ("slirp"), the "wire" ends in a small TCP/IP stack inside QEMU that turns the guest's connections into ordinary sockets on the host: the guest is `10.0.2.15`, and `10.0.2.2` is the host.

```sh
# file: guest-net/net-up.sh
# Bring eth0 up by hand: what a DHCP client would do.
ip link set lo up
ip link set eth0 up
ip addr add 10.0.2.15/24 dev eth0
ip route add default via 10.0.2.2
```

```sh
# file: guest-net/run-net.sh
. /net-up.sh
ip addr show eth0 | grep -E "link/ether|inet "
ip route
echo "e1000 driver: $(readlink /sys/class/net/eth0/device/driver | xargs basename), PCI $(basename $(readlink /sys/class/net/eth0/device))"
grep -E "eth0|CPU0" /proc/interrupts | sed -E 's/ +/ /g'
wget -q -O - http://10.0.2.2:8099/hello.txt
cat /proc/net/arp
grep eth0 /proc/net/dev | awk '{ print "eth0: received " $2 " bytes in " $3 " packets, sent " $10 " bytes in " $11 " packets" }'
grep -E "eth0" /proc/interrupts | sed -E 's/ +/ /g'
```

```console
$ mkdir -p www && echo "Hello from the host, served over TCP through e1000 and slirp." > www/hello.txt && mkdir -p g1 && cp guest-net/net-up.sh guest-net/run-net.sh g1/ && sh mkinitramfs.sh g1.cpio init-modules.sh g1 > /dev/null && (python3 -m http.server 8099 --bind 127.0.0.1 --directory www > /dev/null 2>&1 & echo $! > http.pid) && sleep 1 && bash boot.sh g1.cpio quiet 2>&1 | grep -v "run-net.sh"; kill $(cat http.pid)
    link/ether 52:54:00:12:34:56 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 scope global eth0
default via 10.0.2.2 dev eth0 
10.0.2.0/24 dev eth0 scope link  src 10.0.2.15 
e1000 driver: e1000, PCI 0000:00:03.0
 CPU0 CPU1 
 11: 0 3 IO-APIC 11-fasteoi eth0
Hello from the host, served over TCP through e1000 and slirp.
IP address       HW type     Flags       HW address            Mask     Device
10.0.2.2         0x1         0x2         52:55:0a:00:02:02     *        eth0
eth0: received 607 bytes in 6 packets, sent 719 bytes in 9 packets
 11: 0 13 IO-APIC 11-fasteoi eth0
```

The e1000's interrupt count rose by 9 while 16 packets went in and out: one interrupt can announce several packets, and NAPI then handles them all in one polling pass — under heavy load it keeps polling and leaves the interrupt off. The ARP table holds the host side's MAC address (`52:55:0a:00:02:02`, invented by slirp). The web page came from `python3 -m http.server` on the host, reached through `10.0.2.2`.

## Unit 13.2 — Sockets: TCP is a byte stream

**One-line summary:** A TCP socket delivers a **stream of bytes**, not messages: the sizes of `write()` calls on one side have no relation to the sizes of `read()` results on the other. The kernel may combine several writes into one segment (Nagle's algorithm, `tcp_sendmsg`) and the receiver gets whatever is in its receive buffer when it reads — so a protocol on top of TCP must frame its own messages (a length prefix, a delimiter), and every reader must loop until it has a whole message. UDP (`SOCK_DGRAM`) is the opposite: each `sendto` is one datagram and each `recvfrom` returns exactly one.

```c
// file: lab/stream.c
/* Three writes on a TCP connection, one read on the other side; then the same with UDP. */
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <unistd.h>

static void put(int fd, const char *s) { if (write(fd, s, strlen(s)) < 0) _exit(1); }

int main(void) {
    struct sockaddr_in a = { .sin_family = AF_INET, .sin_port = htons(5555), .sin_addr.s_addr = htonl(INADDR_LOOPBACK) };
    int ls = socket(AF_INET, SOCK_STREAM, 0), one = 1;
    setsockopt(ls, SOL_SOCKET, SO_REUSEADDR, &one, sizeof one);
    bind(ls, (struct sockaddr *)&a, sizeof a);
    listen(ls, 1);
    if (fork() == 0) {
        int c = socket(AF_INET, SOCK_STREAM, 0);
        connect(c, (struct sockaddr *)&a, sizeof a);
        put(c, "one;");
        put(c, "two;");
        put(c, "three;");
        close(c);
        _exit(0);
    }
    int s = accept(ls, NULL, NULL);
    usleep(200000);                                          /* let all three writes arrive */
    char buf[64];
    ssize_t n = read(s, buf, sizeof buf);
    printf("TCP: 3 writes (4+4+6 bytes) -> first read returned %zd bytes: \"%.*s\"\n", n, (int)n, buf);
    wait(NULL);

    int u = socket(AF_INET, SOCK_DGRAM, 0);
    a.sin_port = htons(5556);
    bind(u, (struct sockaddr *)&a, sizeof a);
    if (fork() == 0) {
        int c = socket(AF_INET, SOCK_DGRAM, 0);
        sendto(c, "one;", 4, 0, (struct sockaddr *)&a, sizeof a);
        sendto(c, "two;", 4, 0, (struct sockaddr *)&a, sizeof a);
        sendto(c, "three;", 6, 0, (struct sockaddr *)&a, sizeof a);
        _exit(0);
    }
    usleep(200000);
    for (int i = 0; i < 3; i++) {
        n = recv(u, buf, sizeof buf, 0);
        printf("UDP: recv %d returned %zd bytes: \"%.*s\"\n", i + 1, n, (int)n, buf);
    }
    wait(NULL);
    return 0;
}
```

```sh
# file: guest-stream/run-stream.sh
ip link set lo up
/bin/stream
```

```console
$ mkdir -p g2/bin && gcc -static -O2 -Wall -o g2/bin/stream lab/stream.c && cp guest-stream/run-stream.sh g2/ && sh mkinitramfs.sh g2.cpio init-modules.sh g2 > /dev/null && bash boot.sh g2.cpio quiet 2>&1
TCP: 3 writes (4+4+6 bytes) -> first read returned 14 bytes: "one;two;three;"
UDP: recv 1 returned 4 bytes: "one;"
UDP: recv 2 returned 4 bytes: "two;"
UDP: recv 3 returned 6 bytes: "three;"
```

### Deep & Tricky

**DT-29. `read()` on a TCP socket returns *up to* the requested number of bytes — whatever has arrived — and message boundaries are not preserved: several writes may arrive as one read, and one write may arrive in several reads. A protocol that assumes "one `write`, one `read`" works on a quiet loopback in testing and fails under load or over a real network.** *(RFC 9293 (TCP), §3.8.4 and the definition of the TCP service as a "reliable, in-order, byte-stream service"; `man 7 tcp`; POSIX `read()`: "The value returned may be less than nbyte")* Here all three writes were merged into one 14-byte read; with a slower sender they could just as well have been split. Every real protocol frames its messages: HTTP with headers and `Content-Length` or chunked encoding, TLS with record lengths, our Chapter 10 stress test with a length header.

## Unit 13.3 — Packet sockets: seeing the frames

**One-line summary:** An `AF_PACKET` socket of type `SOCK_RAW` with protocol `ETH_P_ALL` receives a copy of every frame that passes through the stack — in both directions — with the link-layer header included; `recvfrom` also reports the direction (`sll_pkttype`: `PACKET_HOST` for incoming, `PACKET_OUTGOING`). It is what `tcpdump` uses. Decoding is a matter of reading the headers in order: Ethernet (14 bytes: destination MAC, source MAC, EtherType), then IPv4 (20+ bytes: version and header length, total length, TTL, protocol, addresses), then TCP or UDP (ports, and for TCP sequence numbers and flags). All multi-byte fields are big-endian ("network byte order").

```c
// file: lab/sniff.c
/* sniff N: print N frames seen on any interface. */
#include <arpa/inet.h>
#include <linux/if_packet.h>
#include <net/ethernet.h>
#include <netinet/ip.h>
#include <netinet/tcp.h>
#include <netinet/udp.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>

int main(int argc, char **argv) {
    int count = atoi(argv[1]);
    int s = socket(AF_PACKET, SOCK_RAW, htons(ETH_P_ALL));
    if (s < 0) { perror("socket"); return 1; }
    unsigned char f[2048];
    for (int i = 0; i < count; i++) {
        struct sockaddr_ll from;
        socklen_t fl = sizeof from;
        ssize_t n = recvfrom(s, f, sizeof f, 0, (struct sockaddr *)&from, &fl);
        if (n < 14 || from.sll_ifindex == 1) { i--; continue; }   /* skip loopback */
        struct ether_header *eh = (struct ether_header *)f;
        printf("%s %4zd B  ", from.sll_pkttype == PACKET_OUTGOING ? "out" : "in ", n);
        if (ntohs(eh->ether_type) == ETHERTYPE_ARP) { printf("ARP\n"); continue; }
        if (ntohs(eh->ether_type) != ETHERTYPE_IP) { printf("ethertype 0x%04x\n", ntohs(eh->ether_type)); continue; }
        struct iphdr *ip = (struct iphdr *)(f + 14);
        char src[16], dst[16];
        inet_ntop(AF_INET, &ip->saddr, src, sizeof src);
        inet_ntop(AF_INET, &ip->daddr, dst, sizeof dst);
        unsigned char *l4 = f + 14 + ip->ihl * 4;
        if (ip->protocol == IPPROTO_TCP) {
            struct tcphdr *t = (struct tcphdr *)l4;
            int payload = ntohs(ip->tot_len) - ip->ihl * 4 - t->doff * 4;
            printf("TCP %s:%u > %s:%u [%s%s%s%s] seq %u len %d\n", src, ntohs(t->source), dst, ntohs(t->dest),
                   t->syn ? "S" : "", t->fin ? "F" : "", t->psh ? "P" : "", t->ack ? "." : "", ntohl(t->seq), payload);
        } else if (ip->protocol == IPPROTO_UDP) {
            struct udphdr *u = (struct udphdr *)l4;
            printf("UDP %s:%u > %s:%u len %u\n", src, ntohs(u->source), dst, ntohs(u->dest), ntohs(u->len) - 8);
        } else {
            printf("IP proto %u %s > %s\n", ip->protocol, src, dst);
        }
        fflush(stdout);
    }
    return 0;
}
```

```sh
# file: guest-sniff/run-sniff.sh
. /net-up.sh
/bin/sniff 12 &
sleep 1
wget -q -O /dev/null http://10.0.2.2:8099/hello.txt
wait
```

```console
$ mkdir -p g3/bin && gcc -static -O2 -Wall -o g3/bin/sniff lab/sniff.c && cp guest-net/net-up.sh guest-sniff/run-sniff.sh g3/ && sh mkinitramfs.sh g3.cpio init-modules.sh g3 > /dev/null && (python3 -m http.server 8099 --bind 127.0.0.1 --directory www > /dev/null 2>&1 & echo $! > http.pid) && sleep 1 && bash boot.sh g3.cpio quiet 2>&1; kill $(cat http.pid)
out   90 B  ethertype 0x86dd
out   86 B  ethertype 0x86dd
out   42 B  ARP
in    64 B  ARP
out   74 B  TCP 10.0.2.15:50740 > 10.0.2.2:8099 [S] seq 3028052269 len 0
in    60 B  TCP 10.0.2.2:8099 > 10.0.2.15:50740 [S.] seq 64001 len 0
out   54 B  TCP 10.0.2.15:50740 > 10.0.2.2:8099 [.] seq 3028052270 len 0
out  139 B  TCP 10.0.2.15:50740 > 10.0.2.2:8099 [P.] seq 3028052270 len 85
in    60 B  TCP 10.0.2.2:8099 > 10.0.2.15:50740 [.] seq 64002 len 0
in   303 B  TCP 10.0.2.2:8099 > 10.0.2.15:50740 [P.] seq 64002 len 249
in    60 B  TCP 10.0.2.2:8099 > 10.0.2.15:50740 [F.] seq 64251 len 0
out   54 B  TCP 10.0.2.15:50740 > 10.0.2.2:8099 [.] seq 3028052355 len 0
```

Read the capture as a TCP conversation. The first two frames are IPv6 (EtherType `0x86dd`): the kernel announcing itself on the newly configured link. Then ARP request and reply (who is 10.0.2.2?), the three-way handshake (`S`, `S.`, `.`), the HTTP request (`P.` with the request's length), the response data, and the closing `F` segments. Every line is a frame that went through the e1000 driver.

## Topic 13B — BPF

## Unit 13.4 — A BPF program, by hand

**One-line summary:** **eBPF** is a small 64-bit instruction set (11 registers, 512 bytes of stack) that the kernel runs at hook points — sockets, kprobes, tracepoints, network devices, scheduler and security hooks. A program is loaded with the `bpf(BPF_PROG_LOAD, ...)` system call, checked by the **verifier** (`kernel/bpf/verifier.c`) — which proves that it terminates, never reads uninitialised registers or memory, and stays in bounds — and then JIT-compiled to native code. Programs keep state in **maps** (here an array: key = IP protocol number, value = packet count), which user space reads with `bpf(BPF_MAP_LOOKUP_ELEM, ...)`. Normally a compiler (clang `-target bpf`) and a library (libbpf) do the work; writing the instructions by hand once shows exactly what they produce.

```c
// file: lab/bpfprog.h
/* A socket-filter eBPF program, written as raw instructions, that counts packets by IP protocol. */
#pragma once
#include <linux/bpf.h>
#include <string.h>
#include <sys/syscall.h>
#include <unistd.h>

#define INSN(c, d, s, o, i) ((struct bpf_insn){ .code = (c), .dst_reg = (d), .src_reg = (s), .off = (o), .imm = (i) })
#define MOV64_REG(d, s)      INSN(BPF_ALU64 | BPF_MOV | BPF_X, d, s, 0, 0)
#define MOV64_IMM(d, i)      INSN(BPF_ALU64 | BPF_MOV | BPF_K, d, 0, 0, i)
#define ADD64_IMM(d, i)      INSN(BPF_ALU64 | BPF_ADD | BPF_K, d, 0, 0, i)
#define LD_ABS(size, off)    INSN(BPF_LD | BPF_ABS | (size), 0, 0, 0, off)     /* r0 = packet[off], big-endian */
#define STX_MEM(size, d, s, o) INSN(BPF_STX | BPF_MEM | (size), d, s, o, 0)
#define JEQ_IMM(d, i, o)     INSN(BPF_JMP | BPF_JEQ | BPF_K, d, 0, o, i)
#define JNE_IMM(d, i, o)     INSN(BPF_JMP | BPF_JNE | BPF_K, d, 0, o, i)
#define JA(o)                INSN(BPF_JMP | BPF_JA, 0, 0, o, 0)
#define CALL(f)              INSN(BPF_JMP | BPF_CALL, 0, 0, 0, f)
#define EXIT()               INSN(BPF_JMP | BPF_EXIT, 0, 0, 0, 0)
#define ATOMIC_ADD64(d, s, o) INSN(BPF_STX | BPF_ATOMIC | BPF_DW, d, s, o, BPF_ADD)
#define LD_MAP_FD(d, fd)     INSN(BPF_LD | BPF_IMM | BPF_DW, d, BPF_PSEUDO_MAP_FD, 0, fd), INSN(0, 0, 0, 0, 0)

static inline long bpf(int cmd, union bpf_attr *attr) { return syscall(SYS_bpf, cmd, attr, sizeof(*attr)); }

static inline int create_counter_map(void) {
    union bpf_attr a;
    memset(&a, 0, sizeof a);
    a.map_type = BPF_MAP_TYPE_ARRAY;
    a.key_size = 4;
    a.value_size = 8;
    a.max_entries = 256;                     /* index 0-254: IP protocol number; 255: not IPv4 */
    return (int)bpf(BPF_MAP_CREATE, &a);
}

/* Returns a program fd, or -1 with the verifier's log in log[]. */
static inline int load_counter_prog(int map_fd, int with_null_check, char *log, unsigned log_size) {
    struct bpf_insn prog[] = {
        MOV64_REG(BPF_REG_6, BPF_REG_1),            /* r6 = skb context (needed by LD_ABS) */
        LD_ABS(BPF_H, 12),                          /* r0 = EtherType */
        MOV64_IMM(BPF_REG_7, 255),                  /* key = 255 ("not IPv4") */
        JNE_IMM(BPF_REG_0, 0x0800, 2),              /* not IPv4? skip the next 2 instructions */
        LD_ABS(BPF_B, 14 + 9),                      /* r0 = IPv4 protocol byte */
        MOV64_REG(BPF_REG_7, BPF_REG_0),            /* key = protocol */
        STX_MEM(BPF_W, BPF_REG_10, BPF_REG_7, -4),  /* *(u32 *)(fp - 4) = key */
        LD_MAP_FD(BPF_REG_1, map_fd),               /* r1 = map */
        MOV64_REG(BPF_REG_2, BPF_REG_10),
        ADD64_IMM(BPF_REG_2, -4),                   /* r2 = &key */
        CALL(BPF_FUNC_map_lookup_elem),             /* r0 = &value, or NULL */
        with_null_check ? JEQ_IMM(BPF_REG_0, 0, 2) : JA(0),  /* if (r0 == NULL) goto out */
        MOV64_IMM(BPF_REG_1, 1),
        ATOMIC_ADD64(BPF_REG_0, BPF_REG_1, 0),      /* __sync_fetch_and_add(value, 1) */
        MOV64_IMM(BPF_REG_0, 0),                    /* out: return 0 = the socket gets no copy */
        EXIT(),
    };
    union bpf_attr a;
    memset(&a, 0, sizeof a);
    a.prog_type = BPF_PROG_TYPE_SOCKET_FILTER;
    a.insns = (unsigned long)prog;
    a.insn_cnt = sizeof prog / sizeof prog[0];
    a.license = (unsigned long)"GPL";
    a.log_buf = (unsigned long)log;
    a.log_size = log_size;
    a.log_level = 1;
    log[0] = 0;
    return (int)bpf(BPF_PROG_LOAD, &a);
}

static inline unsigned long map_get(int map_fd, unsigned key) {
    unsigned long value = 0;
    union bpf_attr a;
    memset(&a, 0, sizeof a);
    a.map_fd = (unsigned)map_fd;
    a.key = (unsigned long)&key;
    a.value = (unsigned long)&value;
    bpf(BPF_MAP_LOOKUP_ELEM, &a);
    return value;
}
```

```c
// file: lab/bpfcount.c
/* bpfcount SECONDS: count every frame on every interface by protocol, in the kernel, with eBPF. */
#include <arpa/inet.h>
#include <linux/if_ether.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>
#include "bpfprog.h"

int main(int argc, char **argv) {
    static char log[65536];
    int map = create_counter_map();
    int prog = load_counter_prog(map, 1, log, sizeof log);
    if (prog < 0) { printf("load failed:\n%s", log); return 1; }
    printf("program loaded: fd %d, verifier says: %s", prog, strrchr(log, '\n') ? "" : "\n");
    char *last = log + strlen(log) - 1;                          /* print the verifier's final statistics line */
    while (last > log && last[-1] != '\n') last--;
    printf("%s", last);
    int s = socket(AF_PACKET, SOCK_RAW, htons(ETH_P_ALL));
    if (setsockopt(s, SOL_SOCKET, SO_ATTACH_BPF, &prog, sizeof prog) < 0) { perror("SO_ATTACH_BPF"); return 1; }
    sleep((unsigned)atoi(argv[1]));
    static const char *names[256] = { [1] = "ICMP", [6] = "TCP", [17] = "UDP", [255] = "not IPv4 (ARP, ...)" };
    for (unsigned k = 0; k < 256; k++) {
        unsigned long v = map_get(map, k);
        if (v) printf("  %-20s %lu packets\n", names[k] ? names[k] : "other", v);
    }
    return 0;
}
```

```c
// file: lab/verifier.c
/* Load the same program without the NULL check and show why the verifier refuses it. */
#include <errno.h>
#include <stdio.h>
#include "bpfprog.h"

int main(void) {
    static char log[65536];
    int map = create_counter_map();
    int prog = load_counter_prog(map, 0, log, sizeof log);
    printf("BPF_PROG_LOAD without the NULL check: %s (errno %d, EACCES: refused by the verifier)\n", prog < 0 ? "rejected" : "accepted", errno);
    char *p = log, *line;
    while ((line = strsep(&p, "\n")))                            /* the last few lines tell the story */
        if (strstr(line, "map_lookup") || strstr(line, "invalid mem") || strstr(line, "R0=") || strstr(line, "processed"))
            printf("  %s\n", line);
    return 0;
}
```

```sh
# file: guest-bpf/run-bpf.sh
. /net-up.sh
echo "bpf_jit_enable=$(cat /proc/sys/net/core/bpf_jit_enable)"
/bin/bpfcount 4 &
sleep 1
wget -q -O /dev/null http://10.0.2.2:8099/hello.txt
nslookup example.org 10.0.2.3 > /dev/null 2>&1
ping -c 2 -W 1 10.0.2.2 > /dev/null 2>&1
wait
/bin/verifier
```

```console
$ mkdir -p g4/bin && for p in bpfcount verifier; do gcc -static -O2 -Wall -o g4/bin/$p lab/$p.c; done && cp guest-net/net-up.sh guest-bpf/run-bpf.sh g4/ && sh mkinitramfs.sh g4.cpio init-modules.sh g4 > /dev/null && (python3 -m http.server 8099 --bind 127.0.0.1 --directory www > /dev/null 2>&1 & echo $! > http.pid) && sleep 1 && bash boot.sh g4.cpio quiet 2>&1 | cut -c1-110; kill $(cat http.pid)
bpf_jit_enable=1
program loaded: fd 4, verifier says: processed 24 insns (limit 1000000) max_states_per_insn 0 total_states 1 p
  ICMP                 4 packets
  TCP                  10 packets
  UDP                  4 packets
  not IPv4 (ARP, ...)  11 packets
BPF_PROG_LOAD without the NULL check: rejected (errno 13, EACCES: refused by the verifier)
  11: (85) call bpf_map_lookup_elem#1   ; R0=map_value_or_null(id=2,ks=4,vs=8)
  R0 invalid mem access 'map_value_or_null'
  processed 14 insns (limit 1000000) max_states_per_insn 0 total_states 1 peak_states 1 mark_read 1
```

The counts were made inside the kernel: for every frame, the JIT-compiled program read two header fields straight from the `sk_buff` and incremented a map slot with an atomic add; user space read the map once at the end. Nothing was copied to user space per packet — this is why BPF-based monitoring tools can watch millions of packets per second where a packet socket would drown.

### Deep & Tricky

**DT-30. The BPF verifier tracks the *type* of every register, including "pointer to a map value *or NULL*" after `bpf_map_lookup_elem`. Dereferencing that register before a branch has proved it non-NULL is rejected at load time — even for an array map, where the lookup of an in-range key cannot actually fail — because the verifier reasons about types, not about the values a particular program happens to use.** *(Linux `Documentation/bpf/verifier.rst`: "the verifier … will reject … if R0 is not checked for NULL"; the log reports `R0 invalid mem access 'map_value_or_null'`)* The rejection happens before the program ever runs: an unsafe BPF program cannot crash the kernel, which is the property that lets BPF programs be loaded into production kernels where a buggy module would be unacceptable. The cost is that correct programs sometimes need extra checks the verifier can follow.

### Pattern

**P-14. When you need custom logic inside the kernel for observation, filtering, or policy, reach for BPF before a kernel module: a verified program attached to an existing hook (kprobe, tracepoint, socket, XDP, cgroup, LSM), with state in maps, controlled from user space.** *(Practice; Linux `Documentation/bpf/`; tools such as `bpftrace`, `bcc`, Cilium, and `systemd`'s BPF-based firewalling are built this way)* A module can do anything — including crash the machine and break on the next kernel version; a BPF program can only do what the verifier and the hook allow, and (with BTF and CO-RE) keeps working across kernel versions. *Trade-off:* the verifier limits program size and complexity and forbids unbounded loops and arbitrary memory access, and debugging a rejected program means reading verifier logs; for a new device, a new file system, or deep changes to core behaviour, a module or a kernel patch is still the tool.

### Mastery check

1. **Predict.** In the capture, the guest's first packet to `10.0.2.2` was not TCP. Why, and what would happen to the TCP SYN if no ARP reply came?
2. **Predict.** Why must the BPF program copy `r1` into `r6` before using `LD_ABS`?
3. **Predict.** The socket filter returns 0. What would returning the frame length (`skb->len`) change for `bpfcount`?

## Chapter 13 Project — A status server, fetched from the host

**Goal:** a small HTTP server running in the guest on port 80 that answers `/` with a summary (kernel, uptime, load), `/meminfo` with `/proc/meminfo`, and `/net` with live per-protocol packet counts from the BPF program of Unit 13.4 — and exits when asked for `/quit`. QEMU forwards host port 8098 to guest port 80 (`hostfwd`), so the *host* fetches the pages with `curl`: the request crosses the host's TCP stack, slirp, the emulated e1000, and the guest's stack.

```c
// file: lab/statusd.c
#include <arpa/inet.h>
#include <linux/if_ether.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <sys/utsname.h>
#include "bpfprog.h"

static char out[16384];

static void add_file(size_t *len, const char *path, int max_lines) {
    FILE *f = fopen(path, "r");
    char line[256];
    for (int i = 0; f && i < max_lines && fgets(line, sizeof line, f); i++)
        *len += (size_t)snprintf(out + *len, sizeof out - *len, "%s", line);
    if (f) fclose(f);
}

int main(void) {
    static char log[65536];
    int map = create_counter_map(), prog = load_counter_prog(map, 1, log, sizeof log);
    int ps = socket(AF_PACKET, SOCK_RAW, htons(ETH_P_ALL));
    if (prog < 0 || setsockopt(ps, SOL_SOCKET, SO_ATTACH_BPF, &prog, sizeof prog) < 0) { puts("bpf setup failed"); return 1; }
    struct sockaddr_in a = { .sin_family = AF_INET, .sin_port = htons(80), .sin_addr.s_addr = htonl(INADDR_ANY) };
    int ls = socket(AF_INET, SOCK_STREAM, 0), one = 1;
    setsockopt(ls, SOL_SOCKET, SO_REUSEADDR, &one, sizeof one);
    if (bind(ls, (struct sockaddr *)&a, sizeof a) < 0 || listen(ls, 8) < 0) { perror("bind/listen"); return 1; }
    puts("statusd: listening on port 80");
    fflush(stdout);
    for (int quit = 0; !quit; ) {
        struct sockaddr_in peer;
        socklen_t pl = sizeof peer;
        int c = accept(ls, (struct sockaddr *)&peer, &pl);
        char req[1024];
        size_t got = 0;
        ssize_t n;
        while (got < sizeof req - 1 && (n = read(c, req + got, sizeof req - 1 - got)) > 0) {   /* DT-29: loop */
            got += (size_t)n;
            req[got] = 0;
            if (strstr(req, "\r\n\r\n")) break;              /* end of the request headers */
        }
        char path[64] = "/";
        sscanf(req, "GET %63s", path);
        size_t len = 0;
        if (strcmp(path, "/quit") == 0) { quit = 1; len = (size_t)snprintf(out, sizeof out, "bye\n"); }
        else if (strcmp(path, "/meminfo") == 0) add_file(&len, "/proc/meminfo", 6);
        else if (strcmp(path, "/net") == 0) {
            static const char *names[256] = { [1] = "icmp", [6] = "tcp", [17] = "udp", [255] = "non-ip" };
            for (unsigned k = 0; k < 256; k++) {
                unsigned long v = map_get(map, k);
                if (v) len += (size_t)snprintf(out + len, sizeof out - len, "%s %lu\n", names[k] ? names[k] : "other", v);
            }
        } else {
            struct utsname u;
            uname(&u);
            len += (size_t)snprintf(out, sizeof out, "%s %s on %s\nuptime: ", u.sysname, u.release, u.machine);
            add_file(&len, "/proc/uptime", 1);
            len += (size_t)snprintf(out + len, sizeof out - len, "loadavg: ");
            add_file(&len, "/proc/loadavg", 1);
        }
        char hdr[128];
        int hl = snprintf(hdr, sizeof hdr, "HTTP/1.0 200 OK\r\nContent-Type: text/plain\r\nContent-Length: %zu\r\n\r\n", len);
        if (write(c, hdr, (size_t)hl) != hl || write(c, out, len) != (ssize_t)len) perror("write");
        close(c);
        printf("statusd: served %s to %s:%u\n", path, inet_ntoa(peer.sin_addr), ntohs(peer.sin_port));
        fflush(stdout);
    }
    return 0;
}
```

```sh
# file: guest-statusd/run-statusd.sh
. /net-up.sh
/bin/statusd
```

```console
$ mkdir -p g5/bin && gcc -static -O2 -Wall -o g5/bin/statusd lab/statusd.c && cp guest-net/net-up.sh guest-statusd/run-statusd.sh g5/ && sh mkinitramfs.sh g5.cpio init-modules.sh g5 > /dev/null && echo built
built
$ QEMU_EXTRA="-nic user,model=e1000,hostfwd=tcp:127.0.0.1:8098-:80" bash boot.sh g5.cpio quiet > guest.log 2>&1 & guest=$!; for i in $(seq 120); do curl -s -m 5 http://127.0.0.1:8098/ > page1.txt && break; sleep 1; done; curl -s http://127.0.0.1:8098/meminfo; for k in 1 2; do echo "--- /net, fetch $k ---"; curl -s http://127.0.0.1:8098/net; done; curl -s http://127.0.0.1:8098/quit; wait $guest; echo "--- first page ---"; cat page1.txt; echo "--- guest console ---"; cat guest.log
MemTotal:         386272 kB
MemFree:          350964 kB
MemAvailable:     347776 kB
Buffers:               0 kB
Cached:             3028 kB
SwapCached:            0 kB
--- /net, fetch 1 ---
tcp 53
non-ip 11
--- /net, fetch 2 ---
tcp 64
non-ip 11
bye
--- first page ---
Linux 6.12.0-book on x86_64
uptime: 17.43 22.08
loadavg: 0.42 0.10 0.03 1/42 50
--- guest console ---
statusd: listening on port 80
statusd: served / to 10.0.2.2:36258
statusd: served / to 10.0.2.2:38092
statusd: served / to 10.0.2.2:33950
statusd: served /meminfo to 10.0.2.2:33966
statusd: served /net to 10.0.2.2:33982
statusd: served /net to 10.0.2.2:33986
statusd: served /quit to 10.0.2.2:34002
```

The host's `curl` connected to `127.0.0.1:8098`; QEMU accepted the connection and opened a TCP connection inside the guest network to `10.0.2.15:80`, so the guest sees the requests coming from slirp's gateway address `10.0.2.2`. The two `/net` requests show the BPF counters growing between them — the second fetch counts the packets of the first. The guest console may show more requests for `/` than the one `curl` that succeeded: while the guest was still booting, the host's first attempts reached QEMU's forwarder and were dropped or retried. The server reads each request in a loop until the blank line that ends the HTTP headers: DT-29 applied.

### Review checklist

- `statusd` serves one client at a time. What happens to a second client that connects while the first is slow? Which socket call's argument limits how many can wait?
- The response is written with two `write` calls. Could the client see them as one read, or the second before the first?
- The BPF program counts frames on every interface. Where would you change it to count only `eth0`, and why might it be better done in the attachment than in the program?

## Answers to the Chapter 13 prediction questions

**13.4 Q1.** Before IP can send a frame to `10.0.2.2`, the neighbour layer needs the gateway's MAC address, so it sends an ARP request and queues the SYN. If no reply came, the kernel would retry ARP a few times and then drop the queued packet; TCP would retransmit the SYN later, triggering ARP again, until `connect` times out (or fails with `EHOSTUNREACH`).

**13.4 Q2.** `LD_ABS` and `LD_IND` are legacy instructions from classic BPF, defined to read from the packet held in the implicit context register **R6**, and they clobber R1–R5 (they may call a helper internally). The program receives the context in R1, so it must move it to R6 first; the verifier rejects `LD_ABS` if R6 does not hold the context.

**13.4 Q3.** A socket filter's return value is the number of bytes of the frame to deliver to the socket (0 = drop). Returning the length would make the packet socket receive a copy of every frame — counting would still work, but `bpfcount` never reads from the socket, so its receive buffer would fill up and further frames would be dropped (and counted as drops), wasting memory and CPU for nothing.

# Chapter 14 — Capstone: Testing, Style, and Sending a Patch

*Goal: finish both halves of the book the way professional kernel work is finished. For the hobby kernel, that means a regression suite: one command that boots it once per test and checks every result, including the tests that are *supposed* to crash. For Linux, it means turning a driver into an upstream-style patch series: kernel coding style checked by `checkpatch.pl`, a Kconfig entry and a Makefile line in the right places, commits with proper messages and a `Signed-off-by`, a selftest in `tools/testing/selftests/`, a series in which every commit builds, `git format-patch` with a cover letter, and `get_maintainer.pl` to find out who should review it.*

**What you will be able to do after this chapter**

- Write a regression runner for a kernel that encodes expected passes *and* expected failures.
- Read and fix `checkpatch.pl` reports, and know what the tool cannot check.
- Add a driver to the kernel tree with Kconfig and Kbuild, and verify that the configuration system sees it.
- Write a commit message in the kernel's format, sign it off, and explain what the sign-off certifies.
- Produce a bisectable patch series, test every commit, and prepare it for the mailing list.

```console
$ ln -sfn /home/claude/kernelsrc/linux linux && ln -sfn /home/claude/kernelsrc/build build && cp ../ch08/mkinitramfs.sh ../ch08/boot.sh ../ch09/init-modules.sh . && echo ready
ready
```

## Topic 14A — The hobby kernel's regression suite

## Unit 14.1 — Every test, every time

**One-line summary:** Each `KTEST` in our kernel (Chapters 3–7) ran when we wrote it. A **regression suite** runs all of them after every change, because a change in one place (the scheduler, the heap, the page tables) can break a test written chapters earlier. The runner asks the kernel which tests exist, boots once per test with a time limit, and compares QEMU's exit status with the *expected* one: 33 for "passed", 35 for "kernel panicked" — which is the expected result for the three tests that deliberately crash the kernel.

```sh
# file: check.sh
# check.sh OSDIR: boot the hobby kernel once per test and compare each result with the expectation.
cd "$1" || exit 2
make -s > /dev/null 2>&1 || { echo "build failed"; exit 2; }
tests=$(make -s run CMDLINE="test=list" TIMEOUT=60 2>/dev/null | tr -d '\r' | sed -n 's/.*available: //p')
expect_panic=" pagefault divide invalid_opcode "         # these crash the kernel on purpose
pass=0; fail=0
printf "%-16s %-9s %-9s %s\n" TEST EXPECTED RESULT SECONDS
for t in $(echo "$tests" | tr ' ' '\n' | sort); do
    start=$(date +%s)
    (sleep 4; printf 'regression input\n') | make -s run CMDLINE="test=$t" TIMEOUT=120 > "/tmp/check-$t.log" 2>&1
    status=$(sed -n 's/.*qemu exit status \([0-9]*\).*/\1/p' "/tmp/check-$t.log")
    case "$expect_panic" in *" $t "*) want=35; wname=panic ;; *) want=33; wname=pass ;; esac
    case "$status" in 33) got=pass ;; 35) got=panic ;; 124) got=timeout ;; *) got="exit-$status" ;; esac
    if [ "$status" = "$want" ]; then pass=$((pass + 1)); mark=""; else fail=$((fail + 1)); mark="   <-- REGRESSION"; fi
    printf "%-16s %-9s %-9s %3d%s\n" "$t" "$wname" "$got" $(( $(date +%s) - start )) "$mark"
done
echo "check: $pass as expected, $fail regressions"
[ "$fail" -eq 0 ]
```

```console
$ rm -rf os && cp -r ../ch07/final os && make -s -C os clean && sh check.sh os; echo "check.sh exit status: $?"
TEST             EXPECTED  RESULT    SECONDS
breakpoint       pass      pass        4
divide           panic     panic       4
echo             pass      pass        4
elf              pass      pass        4
fairness         pass      pass        7
fs               pass      pass        4
heap             pass      panic       4   <-- REGRESSION
invalid_opcode   panic     panic       4
irqlock          pass      pass        4
isolation        pass      pass        4
lazy             pass      pass        4
pagefault        panic     panic       4
paging           pass      panic       4   <-- REGRESSION
pmm              pass      pass        4
preempt          pass      pass        4
smp              pass      pass        4
threads          pass      pass        4
timer            pass      pass        4
tlb              pass      pass        4
user             pass      pass        4
check: 18 as expected, 2 regressions
check.sh exit status: 1
```

The suite's first run found **two regressions**, in tests that passed when they were written in Chapter 4. Neither is a kernel bug; both are tests whose assumptions a later chapter broke:

- `heap` asserted that after freeing everything the heap is a single free block. Since Chapter 5, `sched_init` allocates the `kmain` and idle task structures and stacks from the heap *before* any test runs, so those blocks are still in use. The test must compare with the heap's state when it started, not with an empty heap.
- `paging` mapped a test page at `0xFFFF800000003000` — inside the kernel heap's region (`HEAP_START` is `0xFFFF800000000000`). In Chapter 4 nothing had used the heap yet, so the address happened to be free; once the scheduler allocates at boot, the heap's first pages are mapped and `vmm_map` refuses. The test must use an address that nothing else owns.

The fixes:

```c
// file: fix/kernel/test_heap.c
#include <mm.h>

static uint32_t rng = 12345;
static uint32_t rnd(void) { rng = rng * 1103515245 + 12345; return rng >> 8; }

KTEST(heap) {
    struct heap_stats start, s;
    kheap_check(&start);                                   /* other code may already own heap blocks */
    char *a = kmalloc(100), *b = kmalloc(200), *c = kmalloc(300);
    kprintf("kmalloc(100)=%p kmalloc(200)=%p kmalloc(300)=%p\n", (void *)a, (void *)b, (void *)c);
    kfree(b);
    char *d = kmalloc(150);
    kprintf("after kfree(b), kmalloc(150)=%p: %s\n", (void *)d, d == b ? "reuses b's block (first fit)" : "new block");
    kfree(a); kfree(c); kfree(d);

    static char *ptr[500];
    static size_t len[500];
    for (int op = 0; op < 20000; op++) {
        int i = (int)(rnd() % 500);
        if (ptr[i]) {
            for (size_t k = 0; k < len[i]; k++) KASSERT(ptr[i][k] == (char)i);
            kfree(ptr[i]);
            ptr[i] = NULL;
        } else {
            len[i] = 1 + rnd() % (rnd() % 8 == 0 ? 16384 : 256);
            ptr[i] = kmalloc(len[i]);
            KASSERT(ptr[i] && (uintptr_t)ptr[i] % 16 == 0);
            memset(ptr[i], i, len[i]);
        }
        if (op % 5000 == 0) kheap_check(&s);
    }
    for (int i = 0; i < 500; i++) kfree(ptr[i]);
    kheap_check(&s);
    kprintf("used bytes before %zu, after %zu; blocks before %zu, after %zu\n", start.used_bytes, s.used_bytes,
            start.blocks, s.blocks);
    KASSERT(s.used_bytes == start.used_bytes);          /* everything the test allocated was returned */
    KASSERT(s.free_blocks <= start.free_blocks + 1);    /* and coalesced (the heap may have grown once) */
}
```

```c
// file: fix/kernel/test_paging.c
#include <mm.h>

KTEST(paging) {
    vmm_walk((uint64_t)&pmm_alloc);

    uint64_t frame = pmm_alloc();
    uint64_t a = 0xFFFFE00000003000UL, b = 0xFFFFA00000000000UL;   /* a: outside the heap's region */
    KASSERT(vmm_map(a, frame, PTE_WRITABLE) && vmm_map(b, frame, PTE_WRITABLE));
    vmm_walk(a);
    char *pa = (char *)a, *pb = (char *)b;
    memcpy(pa, "written through a", 18);
    kprintf("read through b: \"%s\" (two virtual pages, one frame)\n", pb);
    KASSERT(vmm_unmap(a) == frame && vmm_unmap(b) == frame);
    uint64_t phys;
    kprintf("after unmap: translate(a) %s\n", vmm_translate(a, &phys) ? "succeeds" : "fails");
    pmm_free(frame);
}

KTEST(lazy) {
    uint64_t free_before = pmm_free_count();
    volatile uint64_t *zone = (volatile uint64_t *)LAZY_START;
    kprintf("lazy zone at %p: first read returns %lu\n", (void *)zone, zone[0]);
    zone[1] = 42;
    zone[512 * 5] = 7;
    for (int page = 100; page < 110; page++) zone[512 * page] = (uint64_t)page;
    kprintf("page faults handled: %lu; frames used: %lu (data pages + new page tables)\n", lazy_faults,
            free_before - pmm_free_count());
    KASSERT(lazy_faults == 12 && zone[1] == 42 && zone[512 * 105] == 105);
}
```

```console
$ cp -r fix/. os/ && sh check.sh os; echo "check.sh exit status: $?"
TEST             EXPECTED  RESULT    SECONDS
breakpoint       pass      pass        4
divide           panic     panic       4
echo             pass      pass        4
elf              pass      pass        4
fairness         pass      pass        7
fs               pass      pass        4
heap             pass      pass        4
invalid_opcode   panic     panic       4
irqlock          pass      pass        4
isolation        pass      pass        4
lazy             pass      pass        4
pagefault        panic     panic       4
paging           pass      pass        4
pmm              pass      pass        4
preempt          pass      pass        4
smp              pass      pass        4
threads          pass      pass        4
timer            pass      pass        4
tlb              pass      pass        4
user             pass      pass        4
check: 20 as expected, 0 regressions
check.sh exit status: 0
```

Every test now behaves as expected. This is what a regression suite is for: the two broken tests had been silently wrong since Chapter 5, and nobody had run them again. (`test=list` in `check.sh` is not a test: asking for an unknown name makes `run_test` print the list of registered tests — the `.ktests` linker section put to one more use.)

### Pattern

**P-15. Encode expectations, not just successes: a regression suite must check that failures still fail in the expected way — a fault handler still catches the fault, a validation still rejects bad input — as well as that working things still work.** *(Practice; Linux selftests report `ok`, `not ok`, and `# SKIP` in KTAP format, and many kernel tests assert that an operation fails with a specific errno)* A test that only checks "no crash" passes when the protection it was written for disappears. Our three panic tests would turn into silent passes if a change made the kernel ignore page faults; the suite would report them as regressions. *Trade-off:* expected-failure lists must be maintained by hand, and a test that "fails as expected" for the wrong reason (a panic, but a different one) still passes — the stronger form checks the *message*, not only the exit code.

## Topic 14B — Kernel style

## Unit 14.2 — `checkpatch.pl` and what it cannot see

**One-line summary:** The kernel has one coding style (`Documentation/process/coding-style.rst`): tabs of 8 columns, 80-column lines preferred (100 accepted), K&R braces with function braces on their own line, no `typedef` for structs, short lower-case names, comments that say *why*. `scripts/checkpatch.pl` checks style and many common mistakes in a patch or a file; `--strict` adds the checks some subsystems demand. Every source file starts with an **SPDX** licence identifier (`// SPDX-License-Identifier: GPL-2.0` in a C file). checkpatch finds style problems and some bug patterns — it does not find logic errors, races, or bad design.

```console
$ for m in ch09/hello/hello.c ch09/sampler/sampler.c ch10/ring/ring.c ch12/cache/cache.c; do tail -n +2 ../$m > /tmp/$(basename $m) && printf "%-24s " $m && linux/scripts/checkpatch.pl --no-tree --strict --terse -f /tmp/$(basename $m) | tail -1; done
ch09/hello/hello.c       total: 0 errors, 1 warnings, 0 checks, 23 lines checked
ch09/sampler/sampler.c   total: 0 errors, 1 warnings, 3 checks, 83 lines checked
ch10/ring/ring.c         total: 0 errors, 1 warnings, 3 checks, 260 lines checked
ch12/cache/cache.c       total: 0 errors, 1 warnings, 0 checks, 105 lines checked
$ tail -n +2 ../ch10/ring/ring.c > ring.c && linux/scripts/checkpatch.pl --no-tree --strict --terse -f ring.c
ring.c:1: WARNING: Missing or malformed SPDX-License-Identifier tag in line 1
ring.c:94: CHECK: line length of 101 exceeds 100 columns
ring.c:119: CHECK: line length of 107 exceeds 100 columns
ring.c:147: CHECK: multiple assignments should be avoided
total: 0 errors, 1 warnings, 3 checks, 260 lines checked
```

(The listings in this book start with a `// file:` line that names the file; it is not part of the source, so `tail -n +2` drops it before checking.) The reports are short because the code was written in kernel style from the start: a missing SPDX line in every file, a few lines over 100 columns, and one "multiple assignments" (`ring.head = ring.len = 0`). None of them is a bug, and checkpatch did not notice the bug that Chapter 10's project fixed (partial writes interleaving records) — that needed a stress test.

## Topic 14C — An upstream-style patch series

## Unit 14.3 — Putting a driver in the tree

**One-line summary:** An in-tree driver needs three things besides its source: a **Kconfig** entry (so it can be selected, with dependencies and help text), a **Makefile** line (`obj-$(CONFIG_X) += ...`), and a place in the right subsystem directory. Our ring buffer becomes `drivers/misc/hobbyring/`, selectable as `CONFIG_HOBBYRING`. The work happens on a **branch** — here in a separate `git worktree`, so the main source tree used by the other chapters stays untouched.

The driver is Chapter 10's ring with the atomic-record change of its project, renamed, without the experiment switches, and with its SPDX line:

```c
// file: series/drivers/misc/hobbyring/hobbyring.c
// SPDX-License-Identifier: GPL-2.0
/*
 * hobbyring: a blocking byte ring buffer shared by all openers of /dev/hobbyring.
 *
 * Writes of at most the ring's capacity are atomic: they are never interleaved
 * with other writers' data. Readers block while the ring is empty, writers while
 * the data does not fit. The ring can be inspected and resized with ioctl().
 */
#include <linux/miscdevice.h>
#include <linux/module.h>
#include <linux/mutex.h>
#include <linux/poll.h>
#include <linux/slab.h>
#include <linux/uaccess.h>
#include <linux/wait.h>
#include "hobbyring.h"

static unsigned int capacity = 4096;
module_param(capacity, uint, 0444);
MODULE_PARM_DESC(capacity, "initial ring size in bytes (16 - 1048576)");

static struct {
	struct mutex lock;		/* protects everything below */
	char *buf;
	size_t cap, head, len;
	wait_queue_head_t readq, writeq;
	struct hobbyring_stats st;
} ring;

static int hobbyring_open(struct inode *inode, struct file *file)
{
	mutex_lock(&ring.lock);
	ring.st.opens++;
	mutex_unlock(&ring.lock);
	return stream_open(inode, file);
}

static ssize_t hobbyring_read(struct file *file, char __user *ubuf, size_t n, loff_t *pos)
{
	size_t first;

	if (mutex_lock_interruptible(&ring.lock))
		return -ERESTARTSYS;
	while (ring.len == 0) {
		mutex_unlock(&ring.lock);
		if (file->f_flags & O_NONBLOCK)
			return -EAGAIN;
		if (wait_event_interruptible(ring.readq, READ_ONCE(ring.len) > 0))
			return -ERESTARTSYS;
		if (mutex_lock_interruptible(&ring.lock))
			return -ERESTARTSYS;
	}
	n = min(n, ring.len);
	first = min(n, ring.cap - ring.head);
	if (copy_to_user(ubuf, ring.buf + ring.head, first) ||
	    copy_to_user(ubuf + first, ring.buf, n - first)) {
		mutex_unlock(&ring.lock);
		return -EFAULT;
	}
	ring.head = (ring.head + n) % ring.cap;
	ring.len -= n;
	ring.st.bytes_out += n;
	ring.st.reads++;
	mutex_unlock(&ring.lock);
	wake_up_interruptible(&ring.writeq);
	return n;
}

static ssize_t hobbyring_write(struct file *file, const char __user *ubuf, size_t n, loff_t *pos)
{
	size_t tail, first, need;

	if (n == 0)
		return 0;
	if (mutex_lock_interruptible(&ring.lock))
		return -ERESTARTSYS;
	need = n <= ring.cap ? n : 1;		/* writes that fit are atomic */
	while (ring.cap - ring.len < need) {
		mutex_unlock(&ring.lock);
		if (file->f_flags & O_NONBLOCK)
			return -EAGAIN;
		if (wait_event_interruptible(ring.writeq,
					     READ_ONCE(ring.cap) - READ_ONCE(ring.len) >= need))
			return -ERESTARTSYS;
		if (mutex_lock_interruptible(&ring.lock))
			return -ERESTARTSYS;
		need = n <= ring.cap ? n : 1;
	}
	n = min(n, ring.cap - ring.len);
	tail = (ring.head + ring.len) % ring.cap;
	first = min(n, ring.cap - tail);
	if (copy_from_user(ring.buf + tail, ubuf, first) ||
	    copy_from_user(ring.buf, ubuf + first, n - first)) {
		mutex_unlock(&ring.lock);
		return -EFAULT;
	}
	ring.len += n;
	ring.st.bytes_in += n;
	ring.st.writes++;
	mutex_unlock(&ring.lock);
	wake_up_interruptible(&ring.readq);
	return n;
}

static __poll_t hobbyring_poll(struct file *file, poll_table *wait)
{
	__poll_t mask = 0;

	poll_wait(file, &ring.readq, wait);
	poll_wait(file, &ring.writeq, wait);
	mutex_lock(&ring.lock);
	if (ring.len > 0)
		mask |= EPOLLIN | EPOLLRDNORM;
	if (ring.len < ring.cap)
		mask |= EPOLLOUT | EPOLLWRNORM;
	mutex_unlock(&ring.lock);
	return mask;
}

static long hobbyring_resize(__u32 __user *arg)
{
	char *newbuf;
	__u32 newcap;
	long ret = 0;

	if (get_user(newcap, arg))
		return -EFAULT;
	if (newcap < 16 || newcap > 1 << 20)
		return -EINVAL;
	newbuf = kmalloc(newcap, GFP_KERNEL);
	if (!newbuf)
		return -ENOMEM;
	mutex_lock(&ring.lock);
	if (ring.len) {
		ret = -EBUSY;
	} else {
		swap(ring.buf, newbuf);
		ring.cap = newcap;
		ring.head = 0;
	}
	mutex_unlock(&ring.lock);
	kfree(newbuf);
	return ret;
}

static long hobbyring_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
	struct hobbyring_stats st;

	switch (cmd) {
	case HOBBYRING_IOC_GET_STATS:
		mutex_lock(&ring.lock);
		st = ring.st;
		st.capacity = ring.cap;
		st.used = ring.len;
		mutex_unlock(&ring.lock);
		return copy_to_user((void __user *)arg, &st, sizeof(st)) ? -EFAULT : 0;
	case HOBBYRING_IOC_RESET:
		mutex_lock(&ring.lock);
		ring.head = 0;
		ring.len = 0;
		memset(&ring.st, 0, sizeof(ring.st));
		mutex_unlock(&ring.lock);
		wake_up_interruptible(&ring.writeq);
		return 0;
	case HOBBYRING_IOC_RESIZE:
		return hobbyring_resize((__u32 __user *)arg);
	default:
		return -ENOTTY;
	}
}

static const struct file_operations hobbyring_fops = {
	.owner		= THIS_MODULE,
	.open		= hobbyring_open,
	.read		= hobbyring_read,
	.write		= hobbyring_write,
	.poll		= hobbyring_poll,
	.unlocked_ioctl	= hobbyring_ioctl,
};

static struct miscdevice hobbyring_misc = {
	.minor	= MISC_DYNAMIC_MINOR,
	.name	= "hobbyring",
	.fops	= &hobbyring_fops,
	.mode	= 0666,
};

static int __init hobbyring_init(void)
{
	int err;

	if (capacity < 16 || capacity > 1 << 20)
		return -EINVAL;
	mutex_init(&ring.lock);
	init_waitqueue_head(&ring.readq);
	init_waitqueue_head(&ring.writeq);
	ring.cap = capacity;
	ring.buf = kmalloc(ring.cap, GFP_KERNEL);
	if (!ring.buf)
		return -ENOMEM;
	err = misc_register(&hobbyring_misc);
	if (err)
		kfree(ring.buf);
	return err;
}

static void __exit hobbyring_exit(void)
{
	misc_deregister(&hobbyring_misc);
	kfree(ring.buf);
}

module_init(hobbyring_init);
module_exit(hobbyring_exit);
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Blocking byte ring buffer with atomic writes");
```

```c
// file: series/drivers/misc/hobbyring/hobbyring.h
/* SPDX-License-Identifier: GPL-2.0 WITH Linux-syscall-note */
/*
 * User-space interface of /dev/hobbyring.
 *
 * A driver meant for upstream would put this file in include/uapi/linux/ so
 * that it is installed with the kernel headers.
 */
#ifndef _HOBBYRING_H
#define _HOBBYRING_H

#include <linux/ioctl.h>
#include <linux/types.h>

struct hobbyring_stats {
	__u64 bytes_in;
	__u64 bytes_out;
	__u32 reads;
	__u32 writes;
	__u32 opens;
	__u32 capacity;
	__u32 used;
	__u32 reserved;
};

#define HOBBYRING_IOC_MAGIC	'r'
#define HOBBYRING_IOC_GET_STATS	_IOR(HOBBYRING_IOC_MAGIC, 1, struct hobbyring_stats)
#define HOBBYRING_IOC_RESET	_IO(HOBBYRING_IOC_MAGIC, 2)
#define HOBBYRING_IOC_RESIZE	_IOW(HOBBYRING_IOC_MAGIC, 3, __u32)

#endif /* _HOBBYRING_H */
```

```text
# file: series/drivers/misc/hobbyring/Kconfig
# SPDX-License-Identifier: GPL-2.0-only
config HOBBYRING
	tristate "Blocking byte ring buffer device (/dev/hobbyring)"
	help
	  A character device that holds a ring buffer of bytes shared by
	  every process that opens /dev/hobbyring. Readers block while it is
	  empty; writes that fit are atomic. Useful as a teaching example.

	  To compile this driver as a module, choose M here: the module
	  will be called hobbyring.

	  If unsure, say N.
```

```text
# file: series/drivers/misc/hobbyring/Makefile
# SPDX-License-Identifier: GPL-2.0-only
obj-$(CONFIG_HOBBYRING)	+= hobbyring.o
```

The commit message follows `Documentation/process/submitting-patches.rst`: a subject line prefixed with the subsystem, a body that explains the *problem* and *why* this is the solution (not a list of what the diff does), and a `Signed-off-by` line. The identity used here is a placeholder; a real patch carries the author's real name.

```text
# file: series/msg1.txt
misc: add hobbyring, a blocking ring buffer character device

Teaching material and test setups often need a simple byte channel that
any number of processes can open by name, with pipe-like blocking but
without the open-ended lifetime of a FIFO.

Add a misc device, /dev/hobbyring, backed by a single ring buffer.
Readers sleep while it is empty and writers while their data does not
fit; writes no larger than the ring are atomic, so concurrent writers
never interleave records. O_NONBLOCK, poll() and three ioctls (statistics,
reset, resize) are supported.

Signed-off-by: Book Reader <reader@example.org>
```

```console
$ git -C linux worktree remove --force ../linux-wt > /dev/null 2>&1; git -C linux branch -D hobbyring hobbyring-badsplit > /dev/null 2>&1; git -C linux worktree add -q ../linux-wt -b hobbyring && ln -sfn /home/claude/kernelsrc/linux-wt wt && git -C wt config user.name "Book Reader" && git -C wt config user.email reader@example.org && git -C wt --no-pager log --oneline -1
adc218676 (grafted, HEAD -> hobbyring, tag: v6.12) Linux 6.12
$ for f in $(cd series && find drivers -type f | sort); do mkdir -p wt/$(dirname $f) && tail -n +2 series/$f > wt/$f; done && echo 'obj-$(CONFIG_HOBBYRING)		+= hobbyring/' >> wt/drivers/misc/Makefile && sed -i 's|^source "drivers/misc/c2port/Kconfig"|source "drivers/misc/hobbyring/Kconfig"\n&|' wt/drivers/misc/Kconfig && git -C wt add -A && git -C wt --no-pager diff --cached --stat
 drivers/misc/Kconfig               |   1 +
 drivers/misc/Makefile              |   1 +
 drivers/misc/hobbyring/Kconfig     |  12 ++++
 drivers/misc/hobbyring/Makefile    |   2 +
 drivers/misc/hobbyring/hobbyring.c | 217 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 drivers/misc/hobbyring/hobbyring.h |  30 ++++++++
 6 files changed, 263 insertions(+)
$ git -C wt --no-pager diff --cached -- drivers/misc/Makefile drivers/misc/Kconfig
diff --git a/drivers/misc/Kconfig b/drivers/misc/Kconfig
index 3fe7e2a9b..d9c6a4e34 100644
--- a/drivers/misc/Kconfig
+++ b/drivers/misc/Kconfig
@@ -610,6 +610,7 @@ config MARVELL_CN10K_DPI
 	  To compile this driver as a module, choose M here: the module
 	  will be called mrvl_cn10k_dpi.
 
+source "drivers/misc/hobbyring/Kconfig"
 source "drivers/misc/c2port/Kconfig"
 source "drivers/misc/eeprom/Kconfig"
 source "drivers/misc/cb710/Kconfig"
diff --git a/drivers/misc/Makefile b/drivers/misc/Makefile
index a9f94525e..44e09db8f 100644
--- a/drivers/misc/Makefile
+++ b/drivers/misc/Makefile
@@ -72,3 +72,4 @@ obj-$(CONFIG_TPS6594_PFSM)	+= tps6594-pfsm.o
 obj-$(CONFIG_NSM)		+= nsm.o
 obj-$(CONFIG_MARVELL_CN10K_DPI)	+= mrvl_cn10k_dpi.o
 obj-y				+= keba/
+obj-$(CONFIG_HOBBYRING)		+= hobbyring/
```

Two existing files changed by one line each; the rest is new. Before committing, prove that the configuration system and the build both accept it:

```console
$ rm -rf /tmp/cfg14 && mkdir /tmp/cfg14 && cp build/.config /tmp/cfg14/ && wt/scripts/config --file /tmp/cfg14/.config --module HOBBYRING && make -s -C wt O=/tmp/cfg14 ARCH=x86_64 olddefconfig && grep HOBBYRING /tmp/cfg14/.config
CONFIG_HOBBYRING=m
$ make -s -C build M=$PWD/wt/drivers/misc/hobbyring CONFIG_HOBBYRING=m modules && ls wt/drivers/misc/hobbyring/*.ko | sed "s|$PWD/||"
wt/drivers/misc/hobbyring/hobbyring.ko
$ git -C wt commit -q -F <(tail -n +2 series/msg1.txt) && git -C wt --no-pager log --oneline -1 && (cd wt && ./scripts/checkpatch.pl --strict -g HEAD)
ef450a65c (HEAD -> hobbyring) misc: add hobbyring, a blocking ring buffer character device
WARNING: added, moved or deleted file(s), does MAINTAINERS need updating?
#52: 
new file mode 100644

total: 0 errors, 1 warnings, 0 checks, 272 lines checked

NOTE: For some of the reported defects, checkpatch may be able to
      mechanically convert to the typical style using --fix or --fix-inplace.

Commit ef450a65cd2d ("misc: add hobbyring, a blocking ring buffer character device") has style problems, please review.

NOTE: If any of the errors are false positives, please report
      them to the maintainer, see CHECKPATCH in MAINTAINERS.
```

The module was built from the worktree against the book's kernel build (as an external build of that one directory — a full in-tree build would recompile the kernel). checkpatch, run on the commit itself with `-g HEAD`, checks the message too: subject format, line lengths, the sign-off; it also runs `scripts/spdxcheck.py` on the SPDX lines (which needs the Python modules `ply` and `git`). Its one warning — new files were added, does `MAINTAINERS` need updating? — is a fair question that the chapter project answers. checkpatch exits with status 1 whenever it reports anything, which is why scripts treat its output, not its status, as the result.

### Deep & Tricky

**DT-31. `Signed-off-by:` is not a politeness: it certifies the Developer's Certificate of Origin — that you wrote the change or have the right to submit it under the project's licence, and that you understand it will be public and kept forever. A patch without one is not accepted, and a sign-off added on someone else's behalf is a false statement. It must carry a real name (since 2023, "a known identity"; anonymous contributions are not accepted).** *(Linux `Documentation/process/submitting-patches.rst`, §"Sign your work - the Developer's Certificate of Origin"; the DCO 1.1 text, clauses (a)–(d))* checkpatch only checks that the line is there and well-formed. Other trailers have precise meanings too: `Reviewed-by:` and `Acked-by:` are given by *other* people and may be added only when they gave them; `Fixes: <12-char sha> ("subject")` marks a bug fix for the stable trees; `Reported-by:` and `Closes:` credit and link a report. The placeholder identity in this chapter is fine for practice and wrong for a real submission.

## Unit 14.4 — A bisectable series, a selftest, and the cover letter

**One-line summary:** A change that has several logical parts becomes a **series**: one commit per part, in an order where *every* commit builds and works, so that `git bisect` can land on any of them. Tests for a user-visible interface go to `tools/testing/selftests/<name>/` and report in **KTAP/TAP** format through the `kselftest.h` helpers. `git format-patch --cover-letter` produces one mail per commit plus an introduction (`0000-cover-letter.patch`); `scripts/get_maintainer.pl` reads `MAINTAINERS` and tells you which maintainers and lists to send it to. Sending is done with `git send-email`, as plain text — never as an attachment or HTML.

```c
// file: series/tools/testing/selftests/hobbyring/hobbyring_test.c
// SPDX-License-Identifier: GPL-2.0
/* Tests for /dev/hobbyring: round trip, non-blocking behaviour, ioctls, atomic writes. */
#include <errno.h>
#include <fcntl.h>
#include <string.h>
#include <sys/ioctl.h>
#include <sys/wait.h>
#include <unistd.h>
#include "../kselftest.h"
#include "../../../../drivers/misc/hobbyring/hobbyring.h"

static int fd, nbfd;

static void test_round_trip(void)
{
	char buf[16];

	ksft_test_result(write(fd, "abc", 3) == 3 && read(fd, buf, sizeof(buf)) == 3 &&
			 !memcmp(buf, "abc", 3), "round trip\n");
}

static void test_nonblock_empty(void)
{
	char c;

	errno = 0;
	ksft_test_result(read(nbfd, &c, 1) == -1 && errno == EAGAIN,
			 "non-blocking read of empty ring: EAGAIN\n");
}

static void test_resize_and_full(void)
{
	__u32 cap = 64;
	char buf[64] = { 0 };
	struct hobbyring_stats st;
	int ok = ioctl(fd, HOBBYRING_IOC_RESIZE, &cap) == 0 && write(fd, buf, 64) == 64;

	errno = 0;
	ok = ok && write(nbfd, buf, 1) == -1 && errno == EAGAIN;
	ok = ok && ioctl(fd, HOBBYRING_IOC_GET_STATS, &st) == 0;
	ok = ok && st.used == 64 && st.capacity == 64;
	ok = ok && ioctl(fd, HOBBYRING_IOC_RESIZE, &cap) == -1 && errno == EBUSY;
	ok = ok && ioctl(fd, HOBBYRING_IOC_RESET) == 0;
	ksft_test_result(ok, "resize, full ring, statistics, busy resize, reset\n");
}

static void test_unknown_ioctl(void)
{
	errno = 0;
	ksft_test_result(ioctl(fd, _IO('r', 99)) == -1 && errno == ENOTTY,
			 "unknown ioctl: ENOTTY\n");
}

static void test_atomic_write(void)
{
	char rec[48], buf[64];
	int status, ok = 1;

	/* ring of 64 bytes holding 40: a 48-byte write must wait, not split */
	memset(rec, 'w', sizeof(rec));
	ok = write(fd, rec, 40) == 40;
	if (fork() == 0)
		_exit(write(fd, rec, 48) == 48 ? 0 : 1);
	usleep(200000);
	ok = ok && ioctl(fd, HOBBYRING_IOC_GET_STATS, buf) == 0 &&
	     ((struct hobbyring_stats *)buf)->used == 40;	/* nothing of the 48 went in yet */
	ok = ok && read(fd, buf, 40) == 40;
	wait(&status);
	ok = ok && WIFEXITED(status) && WEXITSTATUS(status) == 0 && read(fd, buf, 64) == 48;
	ksft_test_result(ok, "a write that does not fit waits and goes in whole\n");
}

int main(void)
{
	__u32 cap = 64;

	ksft_print_header();
	fd = open("/dev/hobbyring", O_RDWR);
	if (fd < 0)
		ksft_exit_skip("/dev/hobbyring not available (needs CONFIG_HOBBYRING)\n");
	nbfd = open("/dev/hobbyring", O_RDWR | O_NONBLOCK);
	ksft_set_plan(5);
	test_round_trip();
	test_nonblock_empty();
	test_resize_and_full();
	test_unknown_ioctl();
	ioctl(fd, HOBBYRING_IOC_RESIZE, &cap);
	test_atomic_write();
	ksft_finished();
}
```

```text
# file: series/tools/testing/selftests/hobbyring/Makefile
# SPDX-License-Identifier: GPL-2.0
TEST_GEN_PROGS := hobbyring_test

include ../lib.mk
```

```text
# file: series/tools/testing/selftests/hobbyring/config
CONFIG_HOBBYRING=m
```

```text
# file: series/msg2.txt
selftests: add tests for hobbyring

Cover the user-visible behaviour of /dev/hobbyring: a round trip,
EAGAIN for non-blocking reads and writes, the three ioctls including
EBUSY for resizing a non-empty ring and ENOTTY for unknown commands,
and the atomicity of a write that has to wait for space.

The test skips when the device is not present.

Signed-off-by: Book Reader <reader@example.org>
```

```console
$ for f in $(cd series && find tools -type f | sort); do mkdir -p wt/$(dirname $f) && tail -n +2 series/$f > wt/$f; done && git -C wt add tools/testing/selftests/hobbyring && git -C wt commit -q -F <(tail -n +2 series/msg2.txt) && git -C wt --no-pager log --oneline -3
f2d36efd6 (HEAD -> hobbyring) selftests: add tests for hobbyring
ef450a65c misc: add hobbyring, a blocking ring buffer character device
adc218676 (grafted, tag: v6.12) Linux 6.12
$ gcc -static -O2 -Wall -Iwt/tools/testing/selftests -o hobbyring_test wt/tools/testing/selftests/hobbyring/hobbyring_test.c && mkdir -p g1/modules g1/bin && cp wt/drivers/misc/hobbyring/hobbyring.ko g1/modules/ && cp hobbyring_test g1/bin/ && printf 'insmod /modules/hobbyring.ko\n/bin/hobbyring_test\necho "selftest exit status: $?"\n' > g1/run-selftest.sh && sh mkinitramfs.sh g1.cpio init-modules.sh g1 > /dev/null && bash boot.sh g1.cpio quiet
TAP version 13
1..5
ok 1 round trip
ok 2 non-blocking read of empty ring: EAGAIN
ok 3 resize, full ring, statistics, busy resize, reset
ok 4 unknown ioctl: ENOTTY
ok 5 a write that does not fit waits and goes in whole
# Totals: pass:5 fail:0 xfail:0 xpass:0 skip:0 error:0
selftest exit status: 0
```

The selftest ran in the guest against the module built from the branch and reported in TAP: a plan line (`1..5`), one `ok` line per test, and a totals line; its exit status follows the kselftest convention (0 = all passed, 4 = skipped). In the kernel tree it would run with `make -C tools/testing/selftests TARGETS=hobbyring run_tests`.

### Deep & Tricky

**DT-32. Every commit in a series must build and work on its own, not just the last one: `git bisect` checks out commits in the middle of the history, and a commit that does not build stops a bisection dead (or, worse, sends it the wrong way). Splitting a change "by file type" — Makefile in one commit, the source it refers to in the next — is the classic mistake.** *(Linux `Documentation/process/submitting-patches.rst`, §"Separate your changes": "make sure that the kernel builds and runs properly after each patch in the series. Developers using git bisect to track down a problem can end up splitting your patch series at any point")* `git rebase --exec` runs a command after replaying each commit and stops at the first failure. (The command runs at the top of the worktree, which this chapter created next to the build directory — so `../build` is the kernel build.) Here it checks a badly split copy of our driver commit — the build files first, the source second — and stops at the broken first commit:

```console
$ cd wt && git checkout -q -b hobbyring-badsplit HEAD~2 && git checkout -q hobbyring~1 -- drivers/misc && git rm -q --cached drivers/misc/hobbyring/hobbyring.c && git commit -q -m "misc: hobbyring build files (bad split)" && git add drivers/misc/hobbyring/hobbyring.c && git commit -q -m "misc: hobbyring source (bad split)" && git --no-pager log --oneline -2
e4b4bf32a (HEAD -> hobbyring-badsplit) misc: hobbyring source (bad split)
dde62159b misc: hobbyring build files (bad split)
$ cd wt && git rebase -q HEAD~2 --exec 'git clean -fdxq drivers/misc/hobbyring; make -s -C ../build M=$PWD/drivers/misc/hobbyring CONFIG_HOBBYRING=m modules > /dev/null 2>&1 && echo "  $(git log -1 --format=%s): builds" || { echo "  $(git log -1 --format=%s): DOES NOT BUILD"; false; }' 2>&1 | grep -E "^  .*(builds|BUILD)$|execution failed" | cut -c1-100; git rebase --abort; git checkout -q hobbyring && git branch -D -q hobbyring-badsplit && git status --short | head -3
  misc: hobbyring build files (bad split): DOES NOT BUILD
warning: execution failed: git clean -fdxq drivers/misc/hobbyring; make -s -C ../build M=$PWD/driver
```

The good branch passes the same check (next command), so `format-patch` can produce the series:

```console
$ cd wt && git rebase -q HEAD~2 --exec 'git clean -fdxq drivers/misc/hobbyring; make -s -C ../build M=$PWD/drivers/misc/hobbyring CONFIG_HOBBYRING=m modules > /dev/null 2>&1 && echo "  $(git log -1 --format=%s): builds" || { echo "  $(git log -1 --format=%s): DOES NOT BUILD"; false; }' 2>&1 | grep -E "^  .*(builds|BUILD)$"
  misc: add hobbyring, a blocking ring buffer character device: builds
  selftests: add tests for hobbyring: builds
$ B=$PWD; rm -rf outgoing && cd wt && git format-patch -q --cover-letter -2 -o $B/outgoing && ls $B/outgoing && sed -n '2,3p;/^Subject/,/^$/p' $B/outgoing/0001-*.patch | cut -c1-90
0000-cover-letter.patch
0001-misc-add-hobbyring-a-blocking-ring-buffer-character-.patch
0002-selftests-add-tests-for-hobbyring.patch
From: Book Reader <reader@example.org>
Date: Mon, 16 Nov 2026 00:45:38 +0000
Subject: [PATCH 1/2] misc: add hobbyring, a blocking ring buffer character
 device
$ B=$PWD; cd wt && ./scripts/checkpatch.pl --strict --terse $B/outgoing/0001-*.patch $B/outgoing/0002-*.patch | grep -E "^total|ERROR|WARNING"
outgoing/0001-misc-add-hobbyring-a-blocking-ring-buffer-character-.patch:53: WARNING: added, moved or deleted file(s), does MAINTAINERS need updating?
total: 0 errors, 1 warnings, 0 checks, 272 lines checked
outgoing/0002-selftests-add-tests-for-hobbyring.patch:24: WARNING: added, moved or deleted file(s), does MAINTAINERS need updating?
total: 0 errors, 1 warnings, 0 checks, 95 lines checked
$ B=$PWD; cd wt && ./scripts/get_maintainer.pl --no-git --no-git-fallback --norolestats $B/outgoing/0001-*.patch | sed -E 's/ <[^>]*>//'
Derek Kiernan
Dragan Cvetic
Arnd Bergmann
Greg Kroah-Hartman
linux-kernel@vger.kernel.org
```

`get_maintainer.pl` read the `MAINTAINERS` file and listed everyone whose entries cover the files the patch touches — the char/misc maintainers (Arnd Bergmann and Greg Kroah-Hartman) and the people listed for other `drivers/misc/` entries that match `drivers/misc/Kconfig` — plus the kernel mailing list; the e-mail addresses are removed from the output above. A new driver would also add itself to `MAINTAINERS`, so that future patches to it reach its author — a third commit, or part of the first. Before sending, the cover letter's `*** SUBJECT HERE ***` and `*** BLURB HERE ***` placeholders must be replaced by a summary of the series, and the series is sent with `git send-email --to=... --cc=... outgoing/*.patch` after a test run to yourself.

### Pattern

**P-16. A patch is a message to reviewers: make each one small, self-contained, bisectable, style-clean, tested, and explained — the subject says what, the body says why — and send it to the people `get_maintainer.pl` names, then respond to review with a new version (`[PATCH v2]`, with a changelog below the `---` line).** *(Linux `Documentation/process/submitting-patches.rst` and `Documentation/process/5.Posting.rst`)* Reviewers' time is the scarcest resource in the kernel community; everything in this list exists to save it. *Trade-off:* the process is slow by design — a series often goes through several versions over weeks — and for a driver like ours the honest first question from a reviewer would be "why not a pipe, a FIFO, or a UNIX socket?". A real submission must answer that question before the code matters.

### Mastery check

1. **Predict.** `check.sh` treats exit status 124 as "timeout". Which part of which script produces 124, and why is a hang reported differently from a panic?
2. **Predict.** Why does the selftest include the driver's header through a long relative path, and what would change if the header lived in `include/uapi/linux/`?
3. **Predict.** After `git rebase --exec` stopped on the bad split, what state was the repository in, and what did `git rebase --abort` restore?

## Chapter 14 Project — Your own series

**Goal:** apply the whole workflow to a change of your choice and keep the evidence. Pick one: (a) add a `MAINTAINERS` entry for hobbyring as a third patch; (b) add a sysfs attribute (`/sys/class/misc/hobbyring/used`) as Chapter 10 did, with a selftest case for it; (c) port one improvement from the review checklists of Chapters 9–13 into the driver. For the chosen change, the series must pass: `checkpatch.pl --strict` with no errors or warnings, `git rebase --exec` building every commit, the selftest in the guest, `check.sh`-style automation that runs both, and a cover letter that explains *why*. The repository steps of this chapter are the template; the commands below are the acceptance check for the two patches already made.

```console
$ cd wt && ./scripts/checkpatch.pl --strict -g HEAD~2..HEAD | grep -E "^total|WARNING|ERROR" ; git --no-pager log --format="%h %s%n    %(trailers:key=Signed-off-by,valueonly,separator=%x2C )" HEAD~2..HEAD
WARNING: added, moved or deleted file(s), does MAINTAINERS need updating?
total: 0 errors, 1 warnings, 0 checks, 272 lines checked
WARNING: added, moved or deleted file(s), does MAINTAINERS need updating?
total: 0 errors, 1 warnings, 0 checks, 95 lines checked
f2d36efd6 selftests: add tests for hobbyring
    Book Reader <reader@example.org>
ef450a65c misc: add hobbyring, a blocking ring buffer character device
    Book Reader <reader@example.org>
```

The series has no errors, every commit builds, and the test passes in the guest. checkpatch still raises one warning per patch: new files were added, so does `MAINTAINERS` need an entry? For a new driver the answer is yes — which is exactly option (a) above. With that patch added, the series has reached the point where a real one would be sent — and where the real work, review, begins.

### Review checklist

- Which tests in `check.sh` depend on timing (the scheduler's fairness, the timer's rate), and how could they become flaky? What would you do about a flaky test?
- `hobbyring` keeps its state in one global structure. What would a reviewer say about a driver that cannot have two instances?
- In the cover letter, how would you justify a new character device instead of an existing kernel mechanism — or would you withdraw the series?

## Answers to the Chapter 14 prediction questions

**14.4 Q1.** The `timeout` command in the hobby kernel's `Makefile` (`timeout $(TIMEOUT) $(QEMU) ...`) kills QEMU and exits with status 124 when the time limit expires. A hang is a different failure from a panic — the kernel never reached a verdict — so the runner reports it separately; mixing the two would hide deadlocks (Chapter 5's DT-10) behind the expected panics.

**14.4 Q2.** The selftests are built as ordinary user programs, and the header lives inside `drivers/`, which is not on any user-space include path. If it were in `include/uapi/linux/hobbyring.h`, it would be installed by `make headers_install` and the test would `#include <linux/hobbyring.h>` with the kselftest build's `-isystem` pointing at the installed headers — the way real selftests include new uapi headers.

**14.4 Q3.** `git rebase --exec` stopped after replaying the first commit, with the repository in the middle of a rebase: HEAD detached at that commit, the remaining commit still to be applied, and a `.git/rebase-merge` directory recording the plan. `git rebase --abort` threw the in-progress rebase away and restored the branch (`hobbyring-badsplit`) to exactly where it was before the rebase started.

# Appendix A — Index of Deep & Tricky entries and Patterns

Every *Deep & Tricky* entry (DT) and *Pattern* (P) in the book, in number order, with the unit where it is explained. Use it for revision: read each title and explain the entry before looking it up.

## Deep & Tricky

| # | Entry | Where |
| --- | --- | --- |
| DT-1 | The BIOS guarantees that the boot sector is loaded at physical address 0x7C00, but not which segment:offset pair points there — some BIOSes jump to `0000:7C00`, others to `07C0:0000` — so a boot sector that uses absolute addresses must first reload CS with a far jump; and the only register value it can rely on is `DL` (the boot drive). | Unit 0.2 |
| DT-2 | Kernel code must be compiled with `-mno-red-zone`: an interrupt arriving while a leaf function keeps data in the 128-byte red zone below `rsp` would push the interrupt frame onto that same memory and corrupt it — user code is safe only because the CPU switches to a different (kernel) stack on interrupts from ring 3. | Unit 1.3 |
| DT-3 | Inline assembly that reads or writes memory the compiler cannot see must declare a `"memory"` clobber (or pass the memory as an operand); otherwise the optimiser may keep values in registers across the statement, reorder it, or delete stores it believes are dead — the classic cause of "works at -O0, broken at -O2" kernel bugs. | Unit 1.5 |
| DT-4 | `-ffreestanding` does not stop GCC from emitting calls to `memcpy`, `memmove`, `memset`, and `memcmp`: the GCC manual states that a freestanding environment must still provide them, and the compiler turns large struct copies and initialisations into calls — so a kernel without these functions fails to link. In hosted mode the optimiser goes further and recognises loops as `memset`/`memcpy`, which can turn the body of your own `memset` into a call to itself. | Unit 2.1 |
| DT-5 | The CPU pushes an error code for only some exceptions — 8 (double fault), 10–14, 17, 21, 29, 30 — and never for hardware interrupts or `int n`. A handler that assumes the wrong layout pops one word too many or too few before `iretq`, which then loads RIP from the wrong slot. | Unit 3.1 |
| DT-6 | The 8259 does not deliver a second interrupt of the same or lower priority until it receives EOI: the in-service bit for the current IRQ stays set. Forget the EOI and the timer fires exactly once — the kernel keeps running, but time has stopped. | Unit 3.2 |
| DT-7 | `sti` enables interrupts only after the *next* instruction completes (the "interrupt shadow"), so `sti; hlt` is atomic: an interrupt that is already pending is taken after `hlt` begins and wakes it up. Written as two separate steps — check the buffer with interrupts on, then `hlt` — an IRQ that arrives between the check and `hlt` is handled, and then the CPU sleeps anyway, with data sitting in the buffer, until some unrelated interrupt happens. | Unit 3.3 |
| DT-8 | The memory map says "usable", but at boot some of that usable RAM is still in use: the bootloader places its boot-information structure (and any modules) in ordinary usable memory. The kernel must not overwrite it until it has finished reading it — which includes every pointer it kept into it, such as our `kernel_cmdline`. | Unit 4.1 |
| DT-9 | The CPU caches translations in the TLB (translation lookaside buffer) and does not watch the page tables for changes. After changing or removing a present entry, the kernel must invalidate that translation (`invlpg`, or reload CR3 for everything non-global) — otherwise the CPU may keep using the old frame. | Unit 4.2 |
| DT-10 | When the timer handler switches to another thread, it must already have sent EOI. A thread preempted inside its timer handler is suspended until it is scheduled again — and if it still owes the PIC an EOI, the PIC blocks IRQ 0 (and all lower-priority lines) until then. The next thread then runs with no timer: if it never yields, nothing ever preempts it, and the suspended thread never runs again to send the EOI. | Unit 5.2 |
| DT-11 | A page is accessible from ring 3 only if the user bit is set in the entry at *every* level of the walk — PML4, PDPT, PD and PT. The effective permission is the AND of all levels. A kernel that sets `PTE_USER` only on the last-level entry gets page faults on perfectly "mapped" user pages. | Unit 5.4 |
| DT-12 | A system call that uses a user-supplied pointer without checking it turns the kernel into a "confused deputy": the user cannot read kernel memory directly (DT-11 stops it), but it can ask the kernel to read it on its behalf — and the kernel, running in ring 0, is allowed to. | Unit 5.4 |
| DT-13 | A C struct is not a file format: the compiler may insert padding between members and at the end (only "implementation-defined" alignment rules decide how much), and the byte order of multi-byte integers is the CPU's. Data that crosses a machine boundary — a disk image built on one machine and read on another, a network packet — must use fixed-width types, an explicit byte order, and a layout whose size is verified. | Unit 6.2 |
| DT-14 | An executable file is input like any other, and a loader that trusts its headers lets the file decide where the kernel writes. `p_vaddr`, `p_memsz`, `p_offset`, `p_filesz`, and the entry point must all be checked against the address space before anything is mapped or copied. | Unit 6.4 |
| DT-15 | In C, two threads accessing the same non-atomic object, at least one of them writing, without synchronisation, is a *data race*, and a data race is undefined behaviour — not merely "an unpredictable value". `volatile` does not change that: it constrains the compiler's treatment of accesses, not their atomicity or ordering with respect to other threads. | Unit 7.1 |
| DT-16 | x86 is not sequentially consistent: "Loads may be reordered with older stores to different locations". Two cores that each store a flag and then read the other's flag can both miss each other — which breaks every hand-written mutual-exclusion algorithm (Dekker, Peterson) that does not use a fence. | Unit 7.2 |
| DT-17 | On a single CPU, a plain spinlock shared with an interrupt handler is a deadlock waiting for the right timing: the thread takes the lock, the interrupt arrives, the handler spins on the lock — and the thread that would release it cannot run until the handler returns. The fix is to disable local interrupts *before* taking such a lock, everywhere outside the handler. | Unit 7.3 |
| DT-18 | PID 1 must never exit. If `/init` returns — even successfully — the kernel panics with "Attempted to kill init!", because orphaned processes are re-parented to PID 1 and it is the reaper of last resort. A real `init` loops forever (or execs into a program that does); a test `init` must end by rebooting or powering off. | Unit 8.4 |
| DT-19 | The initramfs must be a cpio archive in the "newc" (or "crc") format. GNU `cpio`'s default output format is the old binary format, which the kernel does not accept: it reports that unpacking failed, has no `/init`, and — with no block devices to mount a root from — cannot start any user space at all. | Unit 8.4 |
| DT-20 | The licence string is not decoration: modpost refuses to build, and the module loader refuses to link, a module without a GPL-compatible licence against `EXPORT_SYMBOL_GPL` symbols; and loading any non-GPL module taints the kernel (flag `P`, value 1 in `/proc/sys/kernel/tainted`), which kernel developers read as "bug reports from this kernel are not ours to debug". | Unit 9.4 |
| DT-21 | A use-after-free usually does *not* crash: the freed memory is still mapped and often still holds the old value, so the bug reads plausible data and survives testing. Only a tool that tracks the state of every allocation — KASAN (shadow memory), or slab poisoning — turns it into a report, and the report says where the object was allocated, where it was freed, and where it was used afterwards. | Unit 9.5 |
| DT-22 | `copy_to_user()` and `copy_from_user()` do not return an error code: they return the number of bytes that could *not* be copied (0 on success). Returning that number from `read` tells user space that `n` bytes were read — successfully — when nothing was copied. | Unit 10.1 |
| DT-23 | `copy_to_user()` may sleep: if the user page is not resident, the copy triggers a page fault, and handling it may require reading from disk. So user memory must never be touched while holding a spinlock, with interrupts disabled, or in an interrupt handler. Kernels built with `CONFIG_DEBUG_ATOMIC_SLEEP` check this on every call (`might_fault()`), whether or not a fault actually happens. | Unit 10.2 |
| DT-24 | After `fork()`, *both* processes take a page fault on the first write to each private page — the parent too, even if the child never writes and has already exited. `fork` write-protects the parent's own page-table entries; the fault handler then finds the page no longer shared and reuses it without copying, but the fault itself (a trap into the kernel per page) has already happened. | Unit 11.1 |
| DT-25 | With the default overcommit policy, a successful `malloc` (or `mmap`) is not a promise of memory. The failure arrives later, at the page fault that finds no free page, and it does not arrive as an error return: the OOM killer sends `SIGKILL`, which cannot be caught — and the victim may be a *different* process than the one that allocated last. | Unit 11.3 |
| DT-26 | A `SCHED_FIFO` task runs until it blocks or yields; no time slice and no `nice` value of a normal task can preempt it. The only reason a spinning real-time task does not freeze a CPU completely is RT throttling, a safety net that limits all real-time tasks together to `sched_rt_runtime_us` per `sched_rt_period_us` (950 ms per second by default) — a limit that real-time applications with genuine deadlines often disable, at which point one bug can hang a CPU. | Unit 11.5 |
| DT-27 | A lock-order inversion is a bug even if every test passes: two code paths that take the same two locks in opposite orders deadlock only when they run at the same moment on two CPUs, which may take months in production. lockdep reports it the first time both orders have been *seen*, in any thread, at any time — here in a single thread with no concurrency at all. | Unit 12.2 |
| DT-28 | Publishing a new version is only half of an RCU update; the old version must outlive every reader that may hold it. Freeing it immediately is a use-after-free that, without KASAN, usually reads stale-but-plausible data and goes unnoticed — the bug RCU is famous for being easy to make. The rule is: unlink or replace, *then* wait for a grace period (or `call_rcu`/`kfree_rcu`), *then* free. | Unit 12.3 |
| DT-29 | `read()` on a TCP socket returns *up to* the requested number of bytes — whatever has arrived — and message boundaries are not preserved: several writes may arrive as one read, and one write may arrive in several reads. A protocol that assumes "one `write`, one `read`" works on a quiet loopback in testing and fails under load or over a real network. | Unit 13.2 |
| DT-30 | The BPF verifier tracks the *type* of every register, including "pointer to a map value *or NULL*" after `bpf_map_lookup_elem`. Dereferencing that register before a branch has proved it non-NULL is rejected at load time — even for an array map, where the lookup of an in-range key cannot actually fail — because the verifier reasons about types, not about the values a particular program happens to use. | Unit 13.4 |
| DT-31 | `Signed-off-by:` is not a politeness: it certifies the Developer's Certificate of Origin — that you wrote the change or have the right to submit it under the project's licence, and that you understand it will be public and kept forever. A patch without one is not accepted, and a sign-off added on someone else's behalf is a false statement. It must carry a real name (since 2023, "a known identity"; anonymous contributions are not accepted). | Unit 14.3 |
| DT-32 | Every commit in a series must build and work on its own, not just the last one: `git bisect` checks out commits in the middle of the history, and a commit that does not build stops a bisection dead (or, worse, sends it the wrong way). Splitting a change "by file type" — Makefile in one commit, the source it refers to in the next — is the classic mistake. | Unit 14.4 |

## Patterns

| # | Pattern | Where |
| --- | --- | --- |
| P-1 | Make every boot experiment automatic and observable: output on the serial port, a programmatic exit (QEMU's `isa-debug-exit`), a timeout around every run, and the exit status checked. Then a kernel test is just a command that passes or fails. | Unit 0.4 |
| P-2 | Confine architecture-specific instructions to small, named, inline functions (`outb`, `inb`, `read_cr3`, `wrmsr`, `cpu_relax`) with precise constraints and clobbers, and write everything else in portable C. Check generated code with `objdump -d` when it matters. | Unit 1.5 |
| P-3 | Keep the boot path tiny and paranoid: check the Multiboot2 magic, build the minimal page tables needed, jump to C as early as possible, and make the very first C code bring up an output channel so that everything after it can report problems. | Unit 2.4 |
| P-4 | Keep interrupt handlers short: acknowledge the device, move the data into a buffer, update a counter, return. Do the real work in normal code that takes the data out with interrupts disabled only for the moment of the check. | Unit 3.3 |
| P-5 | Build each memory layer on the one below with a narrow interface — frames (`pmm_alloc`/`pmm_free`), pages (`vmm_map`/`vmm_unmap`), bytes (`kmalloc`/`kfree`) — and give each layer a self-check that can run at any time (`kheap_check`, the frame allocator's double-free panic). | Unit 4.3 |
| P-6 | Act on a task only at safe points: anything (an interrupt, another task, the shell) may set a flag — `need_resched`, `killed` — but the flag is acted upon only at a well-defined boundary: the end of interrupt handling, just before returning to the interrupted code. | Unit 5.4 |
| P-7 | Define a persistent or shared data format once — in one header used by every program that reads or writes it — and give it a tool on the host: the image builder, a checker, a dumper. Test the reader in the kernel against images made by the host tool. | Unit 6.4 |
| P-8 | For every piece of shared data, write down what protects it — "atomic", "lock X", "interrupts off", "only CPU n touches it" — and choose the weakest tool that is correct: a single counter needs one atomic instruction, not a lock; data shared with an interrupt handler needs an irq-safe lock; a lock must not be held across anything that can sleep. | Unit 7.4 |
| P-9 | Make the kernel development loop a single scripted command with a pass/fail exit status: build the kernel (incrementally), build the initramfs (with the tests inside), boot under QEMU with a timeout, let the guest run the tests and print a machine-readable result, and let the host script turn that into an exit code. | Unit 8.4 |
| P-10 | Initialise in steps, and on failure jump to a label that undoes exactly the steps already done, in reverse order; the exit function undoes all of them in the same reverse order. | Unit 9.2 |
| P-11 | Treat the user–kernel interface as a published contract: one header shared by both sides with fixed-size types and explicit padding, ioctl numbers that encode direction and size, `-ENOTTY` for unknown commands, `-EFAULT` for bad pointers, `-EINVAL` for bad values, `-EAGAIN`/`-ERESTARTSYS` for non-blocking and interrupted waits — and validate every argument before acting on it. | Unit 10.2 |
| P-12 | Before changing kernel behaviour or tuning it, measure it from the counters the kernel already publishes: `/proc/<pid>/status`, `stat`, `schedstat` (with `CONFIG_SCHED_INFO`), `maps`, `smaps`; `/proc/meminfo`, `vmstat`, `buddyinfo`, `slabinfo`; and `getrusage`. Write the prediction down first (from the source or documentation), then compare. | Unit 11.5 |
| P-13 | Develop and test kernel code on a debug kernel — KASAN, lockdep (`PROVE_LOCKING`, which includes `PROVE_RCU`), `DEBUG_ATOMIC_SLEEP`, and for data races KCSAN — and investigate running systems with tracing, not by adding `printk`s and rebuilding. | Unit 12.5 |
| P-14 | When you need custom logic inside the kernel for observation, filtering, or policy, reach for BPF before a kernel module: a verified program attached to an existing hook (kprobe, tracepoint, socket, XDP, cgroup, LSM), with state in maps, controlled from user space. | Unit 13.4 |
| P-15 | Encode expectations, not just successes: a regression suite must check that failures still fail in the expected way — a fault handler still catches the fault, a validation still rejects bad input — as well as that working things still work. | Unit 14.1 |
| P-16 | A patch is a message to reviewers: make each one small, self-contained, bisectable, style-clean, tested, and explained — the subject says what, the body says why — and send it to the people `get_maintainer.pl` names, then respond to review with a new version (`[PATCH v2]`, with a changelog below the `---` line). | Unit 14.4 |

# Appendix B — Reference Tables

The numbers and encodings used throughout the book, in one place. Each table names the unit where the topic is taught.

## x86-64 exceptions (Unit 3.1)

| Vector | Name | Error code | Kind | Typical cause |
| --- | --- | --- | --- | --- |
| 0 | Divide error (#DE) | no | fault | `div` by zero, quotient overflow |
| 3 | Breakpoint (#BP) | no | trap | `int3` |
| 6 | Invalid opcode (#UD) | no | fault | `ud2`, unsupported instruction |
| 8 | Double fault (#DF) | yes (0) | abort | exception while delivering an exception |
| 13 | General protection (#GP) | yes | fault | privileged instruction in ring 3, non-canonical address, bad segment |
| 14 | Page fault (#PF) | yes | fault | not-present page or access-rights violation; address in CR2 |

Page-fault error code bits: 0 P (page was present: protection violation), 1 W/R (write), 2 U/S (user mode), 3 RSVD (reserved bit set), 4 I/D (instruction fetch, with NX or SMEP enabled).

## Page-table entry bits (Units 4.2, 5.3)

| Bit | Name | Meaning |
| --- | --- | --- |
| 0 | P | present |
| 1 | R/W | writable (AND across all levels) |
| 2 | U/S | user-accessible (AND across all levels — DT-11) |
| 3 | PWT | write-through |
| 4 | PCD | cache disable (device memory — Unit 7.4) |
| 5 | A | accessed (set by the CPU) |
| 6 | D | dirty (set by the CPU, last level only) |
| 7 | PS | page size: 2 MiB in a PD entry, 1 GiB in a PDPT entry |
| 12–51 | address | physical address of the next table or page |
| 63 | XD/NX | no-execute (when EFER.NXE = 1) |

Virtual address split (4-level paging): bits 47–39 PML4 index, 38–30 PDPT, 29–21 PD, 20–12 PT, 11–0 offset; bits 63–48 copy bit 47 (canonical form).

## The hobby kernel's GDT and system calls (Units 5.3, 5.4, 6.4)

| Selector | Descriptor | | Number | System call | Arguments → result |
| --- | --- | --- | --- | --- | --- |
| 0x08 | kernel code (64-bit) | | 0 | write | buf, len → len or -errno |
| 0x10 | kernel data | | 1 | exit | code → does not return |
| 0x18 (0x1B) | user data, RPL 3 | | 2 | getpid | → pid |
| 0x20 (0x23) | user code, RPL 3 | | 3 | sleep | ms → 0 |
| 0x28 | TSS (two slots) | | 4–5 | yield, ticks | → 0, ticks |
| | | | 6–9 | open, read, close, readdir | Unit 6.4 |
| | | | 10–11 | spawn, wait | path, args → pid; pid → status |

System calls enter through `int 0x80` (gate DPL 3); number in RAX, arguments in RDI, RSI, RDX, result in RAX.

## Legacy PC I/O ports used (Chapters 2–7)

| Port(s) | Device | Used for |
| --- | --- | --- |
| 0x20–0x21, 0xA0–0xA1 | 8259 PICs | remap, mask, EOI (Unit 3.2) |
| 0x40–0x43 | 8254 PIT | 100 Hz timer (Unit 3.2) |
| 0x70–0x71 | CMOS RTC | clock check, periodic IRQ 8 (Units 3.2, 7.3) |
| 0x80 | POST port | I/O delay |
| 0x1F0–0x1F7, 0x3F6 | primary ATA channel | PIO disk access (Unit 6.1) |
| 0x3F8–0x3FF | COM1 (16550 UART) | console output and input (Units 2.3, 3.3) |
| 0xF4 | QEMU `isa-debug-exit` | end the run with status (v << 1) \| 1 |

## Linux kernel taint flags (Units 9.4, 9.5)

| Bit | Value | Letter | Meaning |
| --- | --- | --- | --- |
| 0 | 1 | P | proprietary (non-GPL) module loaded |
| 7 | 128 | D | kernel died recently (oops or BUG) |
| 9 | 512 | W | a warning was issued |
| 12 | 4096 | O | out-of-tree module loaded |
| 13 | 8192 | E | unsigned module loaded (with module signing) |

`/proc/sys/kernel/tainted` holds the sum; `Documentation/admin-guide/tainted-kernels.rst` lists all flags.

## ioctl number encoding (Unit 10.2)

| Bits | Field | Example: `_IOR('r', 1, struct ring_stats)` = 0x80287201 |
| --- | --- | --- |
| 0–7 | command number | 0x01 |
| 8–15 | type ("magic") | 0x72 = `'r'` |
| 16–29 | argument size | 0x028 = 40 bytes |
| 30–31 | direction: 0 none, 1 write (user → kernel), 2 read (kernel → user) | 2 |

## Choosing a lock in Linux (Unit 12.1)

| Situation | Use |
| --- | --- |
| a single counter or flag | `atomic_t`, `refcount_t` (for object lifetimes) |
| short section, no sleeping, not used from IRQ | `spin_lock` |
| shared with an interrupt handler | `spin_lock_irqsave` outside the handler (DT-17) |
| section may sleep (allocation, user copies, I/O) | `struct mutex` |
| many readers, few writers, readers may sleep | `struct rw_semaphore` |
| very many readers, rare updates, readers must be fast | RCU (`rcu_read_lock`, `rcu_assign_pointer`, `kfree_rcu`) |
| per-CPU statistics | per-CPU variables (`this_cpu_inc`) |

## Debug options used by this book's kernel (Chapters 8, 9, 12)

| Option | Finds | Report starts with |
| --- | --- | --- |
| `CONFIG_KASAN` | out-of-bounds and use-after-free accesses | `BUG: KASAN: slab-use-after-free in …` |
| `CONFIG_PROVE_LOCKING` (lockdep) | lock-order inversions, irq-safety violations | `WARNING: possible circular locking dependency detected` |
| `CONFIG_PROVE_RCU` (with lockdep) | RCU dereferences outside read-side sections | `WARNING: suspicious RCU usage` |
| `CONFIG_DEBUG_ATOMIC_SLEEP` | sleeping (or possibly sleeping) in atomic context | `BUG: sleeping function called from invalid context` |
| `CONFIG_FTRACE`, `CONFIG_KPROBES` | (tracing, not checking) | tracefs: `/sys/kernel/tracing` |

## Further reading

- Intel® 64 and IA-32 Architectures Software Developer's Manual, Volume 3 (System Programming Guide) — the reference for Part I.
- The Multiboot2 Specification; the UEFI Specification; System V AMD64 ABI; ELF-64 Object File Format.
- Linux `Documentation/`: `process/` (submitting patches, coding style), `core-api/`, `locking/`, `RCU/`, `dev-tools/` (KASAN, KUnit, kselftest), `trace/`, `bpf/`, `driver-api/`.
- *Linux Device Drivers* (3rd ed.) for driver concepts — read with the current kernel source at hand, as its APIs are from 2.6.10.
- *Is Parallel Programming Hard, And, If So, What Can You Do About It?* (Paul E. McKenney) — memory ordering and RCU in depth.
- *Operating Systems: Three Easy Pieces* (Arpaci-Dusseau) — the theory behind Part I, free online.
