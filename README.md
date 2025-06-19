# riscv_workshop
## Introduction:
The "Microprocessor for You in Thirty Hours" (MYTH) workshop, based on the RISC-V architecture, offers an immersive and practical introduction to microprocessor design. Conducted over five days, it guides participants through the complete journey from high-level programming to low-level hardware concepts. Starting with C programming, GCC toolchain usage, and Spike simulation, learners gradually move into number systems and RISC-V assembly. The course covers essential topics like combinational and sequential logic, instruction decoding, ALU design, and control flow handling. It culminates in building a single-cycle RISC-V CPU, exploring memory systems, load-store mechanics, and testbench development—offering a complete hands-on experience in CPU microarchitecture and verification.
## What is RISC-V?
RISC-V is an open-source Instruction Set Architecture (ISA) based on the principles of Reduced Instruction Set Computing. Unlike proprietary ISAs like x86 or ARM, RISC-V is freely available for anyone to use, modify, and implement — making it ideal for education, research, and industry applications. Its modular and minimalist design allows developers to build efficient and scalable processors across a wide range of devices, from microcontrollers to high-performance CPUs.
# DAY 1: Introduction to RISC-V Instruction Set Architecture & GNU Toolchain:
The first day introduced us to the VSD-IAT platform and lab environment. We learned how high-level code is translated into assembly and then into machine language. An overview of the RISC-V instruction set architecture was provided, covering base integer (RV64I / RV32I), multiply (RV64M), and floating-point (RV64F / RV64D) extensions.

We also covered number systems, including the difference between unsigned and signed binary numbers. In signed representation, the most significant bit indicates the sign (0 for positive, 1 for negative).

Additionally, we explored integer representation:

32-bit word and 64-bit doubleword

Unsigned range for RV64: 0 to 2⁶⁴−1

Signed range: −2⁶³ to 2⁶³−1

The day concluded with an overview of the hardware design flow—from RISC-V ISA to RTL (e.g., PicoRV32), synthesis, and physical design using tools like Qflow.
## DAY 1 Lab:
1. C program for adding numbers from 1 to n:

   - Create a file, paste the below code and save it on your computer
```c
#include <stdio.h>

int main() {
    int i, sum = 0, n = 100;
    for (i = 1; i <= n; ++i) {
        sum += i;
    }
    printf("Sum of numbers from 1 to %d is %d\n", n, sum);
    return 0;
}
```
Command used to compile the C program is `gcc <filename.c>` or `gcc -o <binary_file_name> <filename.c>`  
To run the program, use `./a.out` or `./<binary_file_name>`
 
![D1_sum _1ton](https://github.com/user-attachments/assets/0a6ae92e-abf9-4cfb-8113-fcb0b0864beb)

2. C program for adding numbers from 1 to n using RISC-V toolchain:

   The same C program is now compiled using the RISC-V toolchain.  
    - Command used to compile the C program is `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c`.  
    - To disassemble and view the object file in readable format, we use the command `riscv64-unknown-elf-objdump -d sum1ton.o`.  
    - To run the program, we use Spike, a RISC-V simulator, with the command `spike pk sum1ton.o`.  
    - Spike also has a debug mode for step-by-step execution, which can be invoked using `spike -d pk sum1ton.o`.

  

   
