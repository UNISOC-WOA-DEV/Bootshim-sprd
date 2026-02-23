# Bootshim-sprd
用于SPRD设备引导UEFI固件

## 概述
Bootshim 是一个小型引导程序，用于在 UNISOC（SPRD）设备上通过现有的 bootloader（cboot/zboot）启动 UEFI 固件。它充当跳板，将 UEFI 镜像重定位到其预期基址，然后跳转执行。

## 工作原理
1. **Bootloader 加载组合镜像**  
   Bootloader（cboot 或 zboot）读取 `boot` 分区，该分区包含 `bootshim` + `UEFI`（实际固件）的拼接数据，并将其加载到一般内存地址 `0x80080000`执行，从 bootshim 入口开始

2. **重定位判断**  
   Bootshim 比较其当前运行时地址（加载位置）与期望的 UEFI 基址（编译时定义的 `UEFI_BASE`）
   - 如果两者相同，说明镜像已在正确位置，bootshim 直接跳转到 UEFI。  
   - 如果不同，bootshim 将整个 UEFI 负载（大小为 `UEFI_SIZE`）从当前位置复制到 `UEFI_BASE`

3. **复制循环**  
   使用 `ldp`/`stp` 指令以 16 字节为单位高效复制

4. **跳转到 UEFI**  
   成功重定位（或无需重定位）后，bootshim 通过绝对跳转指令转移到 `UEFI_BASE`，将控制权交给 UEFI 固件

## 构建要求
- **ARM64 交叉编译器**（例如 `aarch64-linux-gnu-gcc`）
- **GNU make**
- **访问 UEFI 构建输出**：需要知道 UEFI 固件的基址（`FD_BASE`）和大小（`FD_SIZE`），这些值必须与平台期望的内存布局一致

## 构建方法
构建过程通常与 UEFI 构建系统集成。两个关键宏需要传递给汇编器：

- `UEFI_BASE` – UEFI 期望运行的物理地址（通常与 `FD_BASE` 相同）
- `UEFI_SIZE` – UEFI 固件二进制的大小（通常为 `FD_SIZE`）

## 使用
在产出的UEFI固件前拼接BootShim的二进制执行数据作为kernel替换进boot镜像中刷入

`cat BootShim.bin UEFI_FD.bin > kernel`
