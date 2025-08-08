# vsdRiscvSoc

# Task 1 – RISC-V Toolchain Setup & Uniqueness Test (VSD SoC Lab)

This repository provides a detailed record of the procedures and verification results for Task 1 of the VSD RISC-V SoC Lab. The task involves setting up the complete RISC-V GCC toolchain, building the Spike ISA simulator and Proxy Kernel, and verifying the setup by compiling and executing a uniqueness test program.

---

## What's Implemented

- Installed the RISC-V toolchain (`riscv64-unknown-elf-gcc`)
- Built and tested **Spike** (ISA simulator)
- Installed and verified **Proxy Kernel (pk)**
- Ran a C-based **uniqueness test** on `spike pk`
- Platform: Rocky Linux (RHEL-based alternative to Ubuntu)

---

## Table of Contents

1. [Install base developer tools](#task-1--install-base-developer-tools)  
2. [Create a clean workspace](#task-2--create-clean-workspace)  
3. [Get a prebuilt RISC‑V GCC toolchain](#task-3--download-prebuilt-riscv-gcc-toolchain)  
4. [Add toolchain to your PATH](#task-4--add-toolchain-to-path)  
5. [Install Device Tree Compiler (DTC)](#task-5--install-device-tree-compiler-dtc)  
6. [Build and install Spike](#task-6--build-and-install-spike-isa-simulator)  
7. [Build and install Proxy Kernel (riscv-pk)](#task-7--build-and-install-proxy-kernel)  
8. [Add cross-bin directory to PATH](#task-8--add-cross-bin-directory-to-path)  
9. [Install Icarus Verilog (optional)](#task-9--optional-install-icarus-verilog)  
10. [Perform sanity checks](#task-10--sanity-checks)  
11. [Final uniqueness test and output](#final-deliverable-unique-c-test)  
12. [Conclusion](#conclusion)

    ## Results


### Task 1 — Install Base Developer Tools

```bash
sudo apt-get install -y git vim autoconf automake autotools-dev curl \
libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex \
texinfo gperf libtool patchutils bc zlib1g-dev libexpat1-dev gtkwave
```


### Task 2 — Create Clean Workspace

```bash
cd
pwd=$PWD
mkdir -p riscv_toolchain
cd riscv_toolchain
```
### Task 3 — Download Prebuilt RISC‑V GCC Toolchain

```bash
wget "https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz"
tar -xvzf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
```

<img width="1398" height="557" alt="Screenshot from 2025-08-02 21-49-07" src="https://github.com/user-attachments/assets/e4eb3c3a-8557-4220-99d1-f69c634cdb68" />


### Task 4 — Add Toolchain to PATH

**Temporary (current shell):**

```bash
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH
```


**Persistent (all terminals):**

```bash
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH' >> ~/.bashrc
```

<img width="1838" height="78" alt="Screenshot from 2025-08-08 12-13-05" src="https://github.com/user-attachments/assets/33e878bc-72d7-4b88-81f7-fd737652a516" />


### Task 5 — Install Device Tree Compiler (DTC)

```bash
sudo apt-get install -y device-tree-compiler
```
### Task 6 — Build and Install Spike (ISA Simulator)

```bash
git clone https://github.com/riscv/riscv-isa-sim.git
cd riscv-isa-sim
mkdir -p build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14
make -j$(nproc)
sudo make install
```












