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

 Output:
![D1_sum _1ton](https://github.com/user-attachments/assets/0a6ae92e-abf9-4cfb-8113-fcb0b0864beb)

2. C program for adding numbers from 1 to n using RISC-V toolchain:

   The same C program is now compiled using the RISC-V toolchain.  
    - Command used to compile the C program is `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c`.  
    - To disassemble and view the object file in readable format, we use the command `riscv64-unknown-elf-objdump -d sum1ton.o`.  
    - To run the program, we use Spike, a RISC-V simulator, with the command `spike pk sum1ton.o`.  
    - Spike also has a debug mode for step-by-step execution, which can be invoked using `spike -d pk sum1ton.o`.

  Output:
  
![D1_Lab for signed and Unsigned  numbers](https://github.com/user-attachments/assets/f13f4e56-389e-4ac2-a954-410cfc19ce3c)
![D1_gcc compile and disassemble](https://github.com/user-attachments/assets/2c78960a-0fcb-4e1e-b1c2-003ae0aaf2be)

# DAY 2: Introduction to ABI and Basic Verification Flow
## What is ABI?
ABI (Application Binary Interface) defines how a program communicates with the system at the binary level.
It covers rules for data types, memory alignment, function calls, and register usage.
ABI ensures compatibility between compiled code, libraries, and the operating system.
In RISC-V, common ABIs include `ilp32` (32-bit) and `lp64` (64-bit).
Using `-mabi=lp64` means `long` and pointers are 64-bit.
It helps ensure the program runs correctly on RISC-V 64-bit systems.
   
## Registers, Memory Allocation & ABI Usage:
In RISC-V architecture, there are 32 general-purpose registers available. On RV64, each register is 64 bits wide (XLEN = 64). These registers are used to hold data and addresses during computation.

Double words (64-bit values) can either be stored directly in a register or in memory as eight consecutive bytes. RISC-V uses little-endian memory ordering, where the least significant byte (LSB) is stored at the lowest address.

For example, in an array of three double words:

Bytes 0–7 → First double word

Bytes 8–15 → Second double word

Bytes 16–23 → Third double word

The ABI (Application Binary Interface) accesses these hardware registers by mapping them to standard ABI names. These registers are used according to the instruction type:

I-type: Instructions with immediate values

R-type: Instructions with only register operands

S-type: Instructions used for storing values into memory

Every RISC-V instruction is encoded as a 32-bit binary pattern, like in the case of a `ld` (load double word) instruction which loads data into `x8` from a base address in `x23` with an offset of 16.

## DAY 2 Lab:
### 1. Review ASM function call

  - In the terminal create a new file with the name `1to9_custom.c`
  - Paste the below C program and save it.
    ```c
    #include <stdio.h>

      extern int load(int a, int b);

      int main() {
    int result = 0;
    int count = 9;
    result = load(0x0, count + 1);
    printf("Sum of number from 1 to %d is %d\n", count, result);
      } 
    ```
 ![1to9custom](https://github.com/user-attachments/assets/49299555-6d18-476e-befe-3516a6a7d6c7)

  
  - Create another file and name it `load.S`.
  - Paste the below c program and save it.
    ```c
      .section .text
      .global load
      .type load, @function

      load:
          add     a4, a0, zero     // Initialize sum register a4 with 0x0
          add     a2, a0, a1       // Store count of 10 in register a2. Register a1 is loaded with 0xa (decimal 10) from main
          add     a3, a0, zero     // Initialize intermediate sum register a3 by 0
      loop:
          add     a4, a3, a4       // Incremental addition
          addi    a3, a3, 1        // Increment intermediate register by 1
          blt     a3, a2, loop     // If a3 is less than a2, branch to label named <loop>
          add     a0, a4, zero     // Store final result to register a0 so that it can be read by main program
          ret
    ```

    
![loadS](https://github.com/user-attachments/assets/d870a197-efd9-4b25-8103-e1926cb18073)

### Simulating new C program with Function Call:

Type the below Cat command on the terminal
  ```
cat 1_to_9.c
cat load.s
```
Output console:
![D2_Simulate New C Program With Function Call-part 1](https://github.com/user-attachments/assets/bb6f9535-dbdc-4b36-bc2a-dd335dc66cf2)

- Type `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o 1to9_custom.o 1to9_custom.c load.S` to compile the program.
- Now type `spike pk 1to9_custom.o` to simulate spike function.

Output console:
![D2_Simulate New C Program With Function Call-part 2](https://github.com/user-attachments/assets/c4c4bcd2-e6e5-4d2f-a334-d7dfa508012e)

- To view to disassemble and view the object file in readable format, we use `riscv64-unknown-elf-objdump -d 1to9_custom.o | less.`

Output console:
![D2_Simulate New C Program With Function Call-part 3](https://github.com/user-attachments/assets/eff3a0af-9896-4808-b4bc-397708f1c96e)

### Program on RISC-V CPU:
- Downloading the collaterals:
  
      git clone https://github.com/kunalg123/riscv_workshop_collaterals.git
      cd riscv_workshop_collaterals/labs
