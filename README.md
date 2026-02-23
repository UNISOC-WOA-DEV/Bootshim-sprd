# Bootshim-sprd
Used for booting UEFI firmware on SPRD devices

[中文文档](https://github.com/UNISOC-WOA-DEV/BootShim-sprd/README_zh.md)

## Overview
Bootshim is a small bootstrap program that enables UEFI firmware to be booted on UNISOC (SPRD) devices via the existing bootloader (cboot/zboot). It acts as a trampoline that relocates the UEFI image to its expected base address and then transfers control to it.

## How It Works
1. **Bootloader loads the combined image**  
   The bootloader (cboot or zboot) reads the `boot` partition, which contains the concatenated data of `bootshim` + `UEFI` (the actual firmware), and loads it into memory at address `0x80080000`. Execution starts at the bootshim entry point.

2. **Relocation decision**  
   Bootshim compares its current runtime address (where it was loaded) with the desired UEFI base address (`UEFI_BASE`, defined at compile time).  
   - If they match, the image is already at the correct location and bootshim jumps directly to UEFI.  
   - If they differ, bootshim copies the entire UEFI payload (of size `UEFI_SIZE`) from the current location to `UEFI_BASE`.

3. **Copy loop**  
   The copy is performed in 16‑byte chunks using `ldp`/`stp` instructions for efficiency.

4. **Jump to UEFI**  
   After successful relocation (or if no relocation was needed), bootshim performs an absolute branch to `UEFI_BASE`, handing over control to the UEFI firmware.

## Build Requirements
- **ARM64 cross-compiler** (e.g., `aarch64-linux-gnu-gcc`)
- **GNU make**
- **Access to UEFI build outputs**: you need to know the base address (`FD_BASE`) and size (`FD_SIZE`) of your UEFI firmware. These values must match the memory layout expected by the platform.

## Build Instructions
The build process is typically integrated with the UEFI build system. Two key macros are passed to the assembler:

- `UEFI_BASE` – the physical address where UEFI expects to run (often the same as `FD_BASE`).
- `UEFI_SIZE` – the size of the UEFI firmware binary (usually `FD_SIZE`).

## Usage
Concatenate the BootShim binary in front of the UEFI firmware binary to form a kernel image, then replace the kernel in the boot partition with this combined image.

`cat BootShim.bin UEFI_FD.bin > kernel`
