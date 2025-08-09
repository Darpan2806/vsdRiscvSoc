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
<img width="755" height="131" alt="Screenshot from 2025-08-02 21-55-57" src="https://github.com/user-attachments/assets/f98096dc-45b7-4447-9f04-697dfb72bf95" />

### Task 6 — Build and Install Spike (ISA Simulator)

```bash
git clone https://github.com/riscv/riscv-isa-sim.git
cd riscv-isa-sim
mkdir -p build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14
make -j$(nproc)
sudo make install
```


### Task 7 — Build and Install Proxy Kernel

```bash
cd $pwd/riscv_toolchain
git clone https://github.com/riscv/riscv-pk.git
cd riscv-pk
mkdir -p build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc8.3.0-2019.08.0-x86_64-linux-ubuntu14 --host=riscv64-unknown-elf
make -j$(nproc)
sudo make install
```

### Task 8 — Add Cross Bin Directory to PATH

**Current shell:**

```bash
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH
```

**Persistent:**

```bash
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```
## Task 9 — (Optional) Install Icarus Verilog

```bash
cd $pwd/riscv_toolchain
git clone https://github.com/steveicarus/iverilog.git
cd iverilog
git checkout --track -b v10-branch origin/v10-branch
git pull
chmod +x autoconf.sh
./autoconf.sh
./configure
make -j$(nproc)
sudo make install
```

### Task 10 — Sanity Checks

```bash
which riscv64-unknown-elf-gcc
riscv64-unknown-elf-gcc -v
which spike
spike --version
which pk
```
<img width="1851" height="424" alt="image" src="https://github.com/user-attachments/assets/664d99f5-9bdd-4f70-bb84-f85fe8fecd3e" />

## Final Deliverable: Unique C Test

### 1. Create `unique_test.c`
```c
#include <stdint.h>
#include <stdio.h>

#ifndef USERNAME
#define USERNAME "unknown_user"
#endif

#ifndef HOSTNAME
#define HOSTNAME "unknown_host"
#endif

// 64-bit FNV-1a
static uint64_t fnv1a64(const char *s) {
    const uint64_t FNV_OFFSET = 1469598103934665603ULL;
    const uint64_t FNV_PRIME = 1099511628211ULL;
    uint64_t h = FNV_OFFSET;
    for (const unsigned char *p = (const unsigned char*)s; *p; ++p) {
        h ^= (uint64_t)(*p);
        h *= FNV_PRIME;
    }
    return h;
}

int main(void) {
    const char *user = USERNAME;
    const char *host = HOSTNAME;
    char buf[256];
    int n = snprintf(buf, sizeof(buf), "%s@%s", user, host);
    if (n <= 0) return 1;
    uint64_t uid = fnv1a64(buf);
    printf("RISC-V Uniqueness Check\n");
    printf("User: %s\n", user);
    printf("Host: %s\n", host);
    printf("UniqueID: 0x%016llx\n", (unsigned long long)uid);
#ifdef __VERSION__
    unsigned long long vlen = (unsigned long long)sizeof(__VERSION__) - 1;
    printf("GCC_VLEN: %llu\n", vlen);
#endif
    return 0;
}
```

### 2. Compile with Injected Identity

```bash
echo "Username: '$(id -un)'"
echo "Hostname: '$(hostname -s)'"
```

```bash
riscv64-unknown-elf-gcc -O2 -Wall -march=rv64imac -mabi=lp64 \
-DUSERNAME='"darpan"' -DHOSTNAME='"UBUNTU"' \
unique_test.c -o unique_test
```

### 3. Run on Spike + Proxy Kernel

```bash
spike pk ./unique_test
```
---
**Output**:-

<img width="930" height="206" alt="Screenshot from 2025-08-02 21-42-00" src="https://github.com/user-attachments/assets/215d3a6a-74d1-4162-a90e-675b570843ef" />

## Conclusion

This task offered a comprehensive, hands-on introduction to the bare-metal RISC-V development workflow. It involved setting up the necessary toolchain, writing and compiling C programs targeted for the RISC-V architecture, and running them using an ISA-level simulator. Each of these steps was performed on an Ubuntu system, which provided an ideal open-source environment for low-level hardware and software experimentation. Through this process, I gained practical experience with essential components of the RISC-V ecosystem — including compiler toolchains, simulators, and debugging utilities.

Completing this initial setup significantly deepened my understanding of open-source hardware development, as well as the intricacies of low-level Linux-based toolchains. Moreover, this foundational knowledge lays the groundwork for more advanced stages of the lab, such as Register Transfer Level (RTL) simulation, digital logic synthesis, and the design and integration of complete System-on-Chip (SoC) architectures.







