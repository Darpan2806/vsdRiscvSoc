🛠️ Task 1: RISC-V Toolchain Setup and Verification Using WSL

RISC-V Toolchain Ubuntu Status

A comprehensive guide for setting up a complete RISC-V development toolchain with detailed troubleshooting and solutions for common issues.
📋 Table of Contents

    Overview
    Prerequisites
    Installation
    Issues Encountered & Solutions
    Project Structure
    Contributing
    License

🎯 Overview

This project documents the complete setup process for a RISC-V development toolchain, including:

    RISC-V GCC Cross-Compiler (riscv64-unknown-elf-gcc)
    Spike RISC-V ISA Simulator
    RISC-V Proxy Kernel (pk)
    Icarus Verilog for HDL simulation
    GTKWave for waveform viewing

The setup includes comprehensive troubleshooting documentation for compatibility issues between different toolchain versions and solutions for cross-compiler interference problems.
🔧 Prerequisites
System Requirements

    OS: Ubuntu 22.04 LTS (64-bit) - native or VirtualBox
    RAM: 4-8 GB recommended
    Storage: 30 GB free space
    CPU: 2+ cores recommended

🚀 Installation

    Once you have completed the installation of the ubuntu distro, Open the terminal and run the base dependencies command shown below.

Base Dependencies

sudo apt-get install -y git vim autoconf automake autotools-dev curl \
libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex \
texinfo gperf libtool patchutils bc zlib1g-dev libexpat1-dev gtkwave

result:

<img width="723" height="436" alt="Screenshot from 2025-08-02 21-44-03" src="https://github.com/user-attachments/assets/86bdccda-0fc1-4c64-889e-5d6f38ffa0ef" />

1. Workspace Setup

    Then we need to create a workspace setpu where we will download and install all our files and tools. We begin by storing our home path which make installing, cleaning and removing files easy.
   
      cd
      pwd=$PWD
      mkdir -p riscv_toolchain

2. Install RISC-V GCC Toolchain

    Now we Download the prebuilt toolchain and extract it and verify that it is downloaded and extraced correctly. Also add it to our path and verify the path.


      wget "https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz"
      tar -xvzf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz

result:

<img width="1398" height="557" alt="image" src="https://github.com/user-attachments/assets/75b59ba0-26e3-432d-8989-b840bd4e7228" />

After downloading and extracting the file and running the verification code for it you will get some results like shown above in the image
and after adding your path you will get some result like the one you are seeing right above.
Verifying if we installed the toolchain correctly or not so for that we should run these command.


riscv64-unknown-elf-gcc --version
riscv64-unknown-elf-objdump --version
riscv64-unknown-elf-gdb --version
riscv64-unknown-elf-gcc -dumpmachine
ls -la ~/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin/ | grep riscv64



    But when i ran the code mentioned above i got error showing like (This totally depend on what version of ubuntu you are using)

riscv64-unknown-elf-gdb: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file or directory

    So to resolve it i first understood the problem that my ubuntu system had libncurses.so.6 and libtinfo.so.6 installed instead of toolchain needing version 5 as it was older and the prebuilt toolchain was compiled against libncurses.so.5 and libtinfo.so.5 , but modern Ubuntu systems have libncurses.so.6 and libtinfo.so.6. The older version is missing. So solution for it was to first download the older version and then create symbolic links to point the gbd compiler to it.


3. now insatll Device Tree Compiler 

sudo apt-get install -y device-tree-compiler

result :

<img width="755" height="131" alt="image" src="https://github.com/user-attachments/assets/7bd485fa-c0f4-405b-b9b2-90e09df18712" />

4. Build Spike (RISC-V ISA Simulator)

     cd $(pwd)/riscv_toolchain
      git clone https://github.com/riscv/riscv-isa-sim.git
      cd riscv-isa-sim
      mkdir -p build && cd build
      ../configure --prefix=$(pwd)/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14
      make -j$(nproc)
      sudo make install
   
 result:

  <img width="1855" height="642" alt="image" src="https://github.com/user-attachments/assets/0f635b89-a77a-4b3d-9ea4-edc4feef12b4" />

  
5. Build RISC-V Proxy Kernel (Fixed Version)

         cd ../..
      git clone https://github.com/riscv/riscv-pk.git
      cd riscv-pk
      git checkout v1.0.0 
      mkdir -p build && cd build
      ../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14 --host=riscv64-unknown-elf
      make -j$(nproc)
      sudo make install
      cd $pwd/riscv_toolchain

   result:
   

<img width="1660" height="513" alt="image" src="https://github.com/user-attachments/assets/ea937fe0-4486-4ccc-bb56-5e6dc64a6145" />

6. A Unique C Test (Username & Machine Dependent)

   

cat > unique_test.c << 'EOF'
#include <stdint.h>
#include <stdio.h>

#ifndef USERNAME
#define USERNAME "unknown_user"
#endif

#ifndef HOSTNAME
#define HOSTNAME "unknown_host"
#endif

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

result:

<img width="930" height="206" alt="Screenshot from 2025-08-02 21-42-00" src="https://github.com/user-attachments/assets/01496bcf-b607-4333-8da6-856339a54b6d" />



   🔧 Detailed Problem Categories
1. Library Compatibility Issues

    Problem: Modern Ubuntu systems lack legacy library versions required by older toolchains
    Impact: GDB and other tools fail to start due to missing dependencies
    Solution: Manual installation of legacy packages with symbolic link creation

2. Version Incompatibility Between Tools

    Problem: Newer software versions use features not supported by older toolchain components
    Impact: Build failures with cryptic CSR (Control and Status Register) errors
    Solution: Use version-matched compatible releases that align with toolchain era

3. Cross-Compiler PATH Interference

    Problem: RISC-V cross-compiler tools interfere with native x86-64 compilation
    Impact: Native builds fail because wrong assembler is used for host architecture
    Solution: Temporary PATH isolation during native tool compilation

4. Shell Command Expansion Issues

    Problem: Shell command substitution creates unquoted tokens for C preprocessor
    Impact: Preprocessor sees separate identifiers instead of single string constants
    Solution: Use explicit string literals to bypass problematic shell expansion

5. Installation Path Configuration

    Problem: Tools install to non-standard locations not included in default PATH
    Impact: Successfully installed tools appear missing and unusable
    Solution: Permanent PATH updates to include all installation directories

🎯 Key Learning Points
Debugging Methodology

    Multi-stage debugging: Complex issues often require multiple investigation rounds
    Root cause isolation: Surface symptoms may not reveal the underlying problem
    Environment testing: Test individual components to identify interference sources

Toolchain Management

    Version compatibility: All components must match in feature support and era
    PATH management: Cross-compilers can pollute native build environments
    Clean environments: Isolate native and cross-compilation environments when needed

Configuration Best Practices

    Persistent configuration: Always make PATH changes permanent via .bashrc
    Verification steps: Test each installation step before proceeding
    Backup strategies: Save working configurations before making changes

🔍 Troubleshooting Tips
When Builds Fail:

    Check which tools are actually being used (which gcc, which as)
    Verify library dependencies (ldd for missing libraries)
    Examine detailed error logs rather than summary messages
    Test components individually in isolation

For PATH Issues:

    Use echo $PATH to verify current configuration
    Test with minimal clean PATH when debugging
    Check installation directories with ls -la
    Verify permanent changes with fresh shell sessions

For Compatibility Problems:

    Check tool versions and release dates for alignment
    Look for older compatible versions when needed
    Understand feature dependencies between components
    Consider using container environments for complex setups

📋 Final Verification Checklist

    All required tools found in PATH (which commands succeed)
    Tool versions compatible with each other
    Unique test program compiles without errors
    Unique test program runs and produces expected output
    PATH configuration persists across shell sessions
    No cross-compiler interference with native builds

📁 Project Structure

riscv_toolchain/
├── riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/
│   ├── bin/                    
│   ├── lib/                    
│   └── riscv64-unknown-elf/   
├── riscv-isa-sim/              
├── riscv-pk/                   
├── iverilog/                  
├── unique_test.c               
└── unique_test                



   

   










