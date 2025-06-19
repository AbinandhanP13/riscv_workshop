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
Output on console:
![D2_Simulate New C Program With Function Call-part 1](https://github.com/user-attachments/assets/bb6f9535-dbdc-4b36-bc2a-dd335dc66cf2)

- Type `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o 1to9_custom.o 1to9_custom.c load.S` to compile the program.
- Now type `spike pk 1to9_custom.o` to simulate spike function.

Output on console:
![D2_Simulate New C Program With Function Call-part 2](https://github.com/user-attachments/assets/c4c4bcd2-e6e5-4d2f-a334-d7dfa508012e)

- To view to disassemble and view the object file in readable format, we use `riscv64-unknown-elf-objdump -d 1to9_custom.o | less.`

Output console:
![D2_Simulate New C Program With Function Call-part 3](https://github.com/user-attachments/assets/eff3a0af-9896-4808-b4bc-397708f1c96e)

### Program on RISC-V CPU:
- Downloading the collaterals:
  
      git clone https://github.com/kunalg123/riscv_workshop_collaterals.git
      cd riscv_workshop_collaterals/labs
Output on Console:
![D2_Program on RISCV CPU-part 1](https://github.com/user-attachments/assets/a217f79a-2d17-4fe4-a1e7-f6bfcee1e1e3)

- Now type `vim picorv32.v `
Output on Console:
![D2_Program on RISCV CPU-part 2](https://github.com/user-attachments/assets/ec061a9b-1ab5-4e69-b8c7-377ad639c3f7)

# DAY 3: Digital Logic with TL-Verilog and Makerchip
### Logic Gates  
Logic gates are the basic components of digital electronics. They take binary inputs (0 or 1) and produce binary outputs based on specific logical rules. These gates implement Boolean algebra and are used to build more complex gates like NAND, NOR, XOR, and XNOR.

### Combinational Logic  
Combinational logic circuits are digital systems whose output depends only on the current input values. These circuits do not have memory or store any past inputs. They are constructed using gates like AND, OR, NOT, and others.

### Sequential Logic  
Sequential logic circuits generate output based on both current inputs and past inputs. They include memory elements that store the previous state, allowing the circuit to "remember" history. Flip-flops and latches are key components in sequential logic.

### Flip-Flops  
A flip-flop is a bistable memory element used in digital circuits to store one bit of data. It remains in one of two stable states (0 or 1) until triggered by an external control signal, such as a clock pulse. Flip-flops are used in building registers, counters, and state machines.

### Pipelined logic
Pipelined logic, or pipelining, is a technique in computer architecture and digital circuit design that increases throughput by dividing a task into multiple stages and executing them in parallel.

### Validity  
Validity ensures that only useful parts of a chip are active during operation. In many chips, unused gates remain idle but still consume power. Validity eliminates these inactive gates to save energy and improve efficiency.

##DAY 3 Lab:
1. Combinational Calculator:
   ![image](https://github.com/user-attachments/assets/589f68e0-bb2d-448e-8c55-2e8abc52a070)

2. Sequential Calculator:
   ![image](https://github.com/user-attachments/assets/5952e9a2-ed5b-4891-b5f4-9e728a7ddf73)

3. Cycle Calculator:
   ![image](https://github.com/user-attachments/assets/9170199d-b5cb-4f87-84d5-e1394421993e)

4. Total Distance Using Pythagoras:
   ![image](https://github.com/user-attachments/assets/1a9d7e24-2db8-4324-b21c-c4671c4c200b)

5. 2-Cycle calculator with Validity:
   ![image](https://github.com/user-attachments/assets/36eb1149-8f34-498e-b25e-81e086e30333)

# DAY 4: Basic RISC-V CPU Micro-architecture.   
## Single-Cycle RISC-V Processor Architecture:
![image](https://github.com/user-attachments/assets/f029a05c-036a-4945-be29-56622d769e46)

- PC (Program Counter): Holds the address of the current instruction.

- IMem Rd (Instruction Memory Read): Fetches the instruction from memory.

- Dec (Decoder): Decodes the fetched instruction and determines control signals.

- RF Rd / RF Wr (Register File Read/Write): Reads/writes data from/to CPU registers.

- ALU (Arithmetic Logic Unit): Performs arithmetic or logic operations.

- DMem Rd/Wr (Data Memory Read/Write): Used for load/store instructions to access memory.

- Muxes (Multiplexers): Control which data path is selected.

- +1 and + blocks: Handle PC update logic, for sequential or branch operations.

## DAY 4 Lab:
1. Reset
   ![image](https://github.com/user-attachments/assets/4ffc56a6-6172-4171-af6a-6b273602988e)

2. Fetch:
   ![image](https://github.com/user-attachments/assets/666cd993-36b8-48c7-a25e-768b922e077a)

3. Decode with Branches:
   ![image](https://github.com/user-attachments/assets/feeda875-7ca8-4544-be45-ae81e54d1b44)

4. Register File Read:
   ![image](https://github.com/user-attachments/assets/7c5b4eeb-9b91-4534-a043-60d430891aa4)

5. Register File write:
   ![image](https://github.com/user-attachments/assets/e7b012d7-63ce-4b16-9f7f-ef6d03e8f14e)

6. ALU Operations:
   ![image](https://github.com/user-attachments/assets/ddec97a0-3520-41bd-a6db-60e89294ad6f)
   
7. Testbench:
   ![image](https://github.com/user-attachments/assets/50069f7a-2ec4-4f72-a4f1-5283b17693f4)

# DAY 5: Pipelining the RISC-V Core
## Pipelined RISC-V Core  
On Day 5, a 4-stage pipelined RISC-V core was developed, building on the single-cycle design from Day 4. This core supports all base integer instruction sets and includes logic for both load and store operations through a dedicated data memory block.

## Pipeline Hazards and Handling  
To resolve issues caused by instruction dependencies, **register bypassing** and **squashing** techniques were implemented. These help prevent hazards like "read-after-write" and incorrect branching, which are common in pipelined architectures.

## Testing the Pipelined Design  
The pipelined CPU was tested using assembly code that included load and store operations. This validated the functional correctness and hazard handling in the new pipelined setup.

## Timing Abstraction in TL-Verilog  
The use of **Timing Abstraction** in TL-Verilog made it easier to convert the non-pipelined design into a pipelined one. This abstraction allows for retiming and restructuring of the pipeline without introducing functional bugs, simplifying the design and debugging process.

## DAY 5 Lab:
### The Final Code:
```
\m4_TLV_version 1d: tl-x.org
\SV
   // https://myth.makerchip.com/sandbox/0YEf3h2oL/0Y6h36w
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop

   m4_include_lib(['https://gist.githubusercontent.com/AbinandhanP13/d85364952ad0a60e882fea75ec8275d2/raw/66c879094159e277ce710443aedd56bfad13f134/AbinandhanP-RiscV'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)
\TLV

   // /====================\
   // | Sum 1 to 9 Program |
   // \====================/
   //
   // Program for MYTH Workshop to test RV32I
   // Add 1,2,3,...,9 (in that order).
   //
   // Regs:
   //  r10 (a0): In: 0, Out: final sum
   //  r12 (a2): 10
   //  r13 (a3): 1..10
   //  r14 (a4): Sum
   //
   // External to function:
   m4_asm(ADD, r10, r0, r0)             // Initialize r10 (a0) to 0.
   // Function:
   m4_asm(ADD, r14, r10, r0)            // Initialize sum register a4 with 0x0
   m4_asm(ADDI, r12, r10, 1010)         // Store count of 10 in register a2.
   m4_asm(ADD, r13, r10, r0)            // Initialize intermediate sum register a3 with 0
   // Loop:
   m4_asm(ADD, r14, r13, r14)           // Incremental addition
   m4_asm(ADDI, r13, r13, 1)            // Increment intermediate register by 1
   m4_asm(BLT, r13, r12, 1111111111000) // If a3 is less than a2, branch to label named <loop>
   m4_asm(ADD, r10, r14, r0)            // Store final result to register a0 so that it can be read by main program

   m4_asm(SW, r0, r10, 10000)
   m4_asm(LW, r17, r0, 10000)

   // Optional:
   // m4_asm(JAL, r7, 00000000000000000000) // Done. Jump to itself (infinite loop). (Up to 20-bit signed immediate plus implicit 0 bit (unlike JALR) provides byte address; last immediate bit should also be 0)
   m4_define_hier(['M4_IMEM'], M4_NUM_INSTRS)


   |cpu
      @0
         $reset = *reset;

         $start = >>1$reset && !$reset;
         //$valid = $reset ? 1'b0 : $start ? 1'b1 : >>3$valid;
         //$pc[31:0] = (>>1$reset == 1) ? 0 : (>>1$taken_br) ? >>1$br_tgt_pc : >>1$inc_pc;
         $pc[31:0] = >>1$reset ? 0:
                     (>>3$valid_taken_br || (>>3$is_jal && >>3$valid_jump)) ? >>3$br_tgt_pc:
                     (>>3$is_jalr && >>3$valid_jump) ? >>3$jalr_tgt_pc :
                     >>3$valid_load?  >>3$inc_pc:
                     >>1$inc_pc;

         $imem_rd_addr[M4_IMEM_INDEX_CNT - 1:0] = $pc[M4_IMEM_INDEX_CNT + 1:2];
         $imem_rd_en = !reset;
      @1
         *passed = |cpu/xreg[10]>>5$value == (1+2+3+4+5+6+7+8+9);
         $instr[31:0] = $imem_rd_data;
         $inc_pc[31:0] = $pc + 32'd4;
         $instr[31:0] = $imem_rd_data[31:0];
         $is_i_instr = $instr[6:2] ==? 5'b0000x ||
                       $instr[6:2] ==? 5'b001x0 ||
                       $instr[6:2] ==? 5'b11001;

         $is_j_instr = $instr[6:2] ==? 5'b11011;

         $is_r_instr = $instr[6:2] ==? 5'b01011 ||
                       $instr[6:2] ==? 5'b011x0 ||
                       $instr[6:2] ==? 5'b10100;

         $is_b_instr = $instr[6:2] ==? 5'b11000;

         $is_s_instr = $instr[6:2] ==? 5'b0100x;

         $is_u_instr = $instr[6:2] ==? 5'b0x101;

         $imm[31:0] = $is_i_instr ? { {21{$instr[31]}},  $instr[30:20] }:
                      $is_s_instr ? { {21{$instr[31]}}, $instr[30:25], $instr[11:7] } :
                      $is_b_instr ? { {20{$instr[31]}}, $instr[7], $instr[31:25], $instr[11:8], 1'b0 } :
                      $is_u_instr ? { $instr[31:12] , 12'b0 } :
                      $is_j_instr ? { {12{$instr[31]}}, $instr[19:12], $instr[20], $instr[30:21], 1'b0 } :
                      32'b0 ;


         $rs2_valid = $is_r_instr || $is_s_instr || $is_b_instr;
         ?$rs2_valid
            $rs2[4:0] = $instr[24:20];

         $u_or_j = $is_u_instr || $is_j_instr;

         $rs1_valid = $u_or_j ? 1'b0 : 1'b1;
         ?$rs1_valid
            $rs1[4:0] = $instr[19:15];
         $funct3_valid = $u_or_j ? 1'b0 : 1'b1;
         ?$funct3_valid
            $funct3[2:0] = $instr[14:12];

         $funct7_valid = $is_r_instr ? 1'b1 : 1'b0;
         ?$funct7_valid
            $funct7[6:0] = $instr[31:25];
         $rd_valid = ($is_s_instr || $is_b_instr) ? 1'b0 : 1'b1;
         ?$rd_valid
            $rd[4:0] = $instr[11:7];
            $opcode[6:0] = $instr[6:0];


         // decode funct7,funct3,opcode
         $dec_bits[10:0] = { $funct7[5], $funct3, $opcode };
         $is_add = $dec_bits == 11'b0_000_0110011;
         $is_addi = $dec_bits ==? 11'bx_000_0010011;
         $is_beq = $dec_bits ==? 11'bx_000_1100011;
         $is_bne = $dec_bits ==? 11'bx_001_1100011;
         $is_blt = $dec_bits ==? 11'bx_100_1100011;
         $is_bge = $dec_bits ==? 11'bx_101_1100011;
         $is_bltu = $dec_bits ==? 11'bx_110_1100011;
         $is_bgeu = $dec_bits ==? 11'bx_111_1100011;
         $is_load = $dec_bits ==? 11'bx_xxx_0000011;
         $is_lui = $dec_bits ==?   11'bx_xxx_0110111;
         $is_auipc = $dec_bits ==?   11'bx_xxx_0010111;
         $is_jal = $dec_bits ==?   11'bx_xxx_1101111;
         $is_jalr = $dec_bits ==?  11'bx_000_1100111;
         $is_sb = $dec_bits ==?  11'bx_000_0100011;
         $is_sh = $dec_bits ==?  11'bx_001_0100011;
         $is_sw = $dec_bits ==?  11'bx_010_0100011;
         $is_slti = $dec_bits ==?  11'bx_010_0010011;
         $is_sltiu = $dec_bits ==?  11'bx_011_0010011;
         $is_xori = $dec_bits ==?  11'bx_100_0010011;
         $is_ori = $dec_bits ==?  11'bx_110_0010011;
         $is_andi = $dec_bits ==?  11'bx_111_0010011;
         $is_slli = $dec_bits ==?  11'b0_001_0010011;
         $is_srli = $dec_bits ==?  11'b0_101_0010011;
         $is_srai = $dec_bits ==?  11'b1_101_0010011;
         $is_sub = $dec_bits ==?  11'b1_000_0110011;
         $is_sll = $dec_bits ==?  11'b0_001_0110011;
         $is_slt = $dec_bits ==?  11'b0_010_0110011;
         $is_sltu = $dec_bits ==?  11'b0_011_0110011;
         $is_xor = $dec_bits ==?  11'b0_100_0110011;
         $is_srl = $dec_bits ==?  11'b0_101_0110011;
         $is_sra = $dec_bits ==?  11'b1_101_0110011;
         $is_or = $dec_bits ==?  11'b0_110_0110011;
         $is_and = $dec_bits ==?  11'b0_111_0110011;


      @2
         // Read register file
         $src1_value[31:0] = (>>1$rf_wr_index == $rf_rd_index1) && >>1$rf_wr_en  ?
                              >>1$rf_wr_data : $rf_rd_data1;
         $src2_value[31:0] = (>>1$rf_wr_index == $rf_rd_index2) && >>1$rf_wr_en  ?
                              >>1$rf_wr_data : $rf_rd_data2;
         $br_tgt_pc[31:0] = $pc + $imm;

         $rf_rd_en1 = $rs1_valid;
         $rf_rd_index1[4:0] = $rs1;
         $rf_rd_en2 = $rs2_valid;
         $rf_rd_index2[4:0] = $rs2;

      @3
         $sltu_rslt = $src1_value < $src2_value;
         $sltiu_rslt = $src1_value < $imm;
         // ALU code
         $result[31:0] = $is_addi ? $src1_value + $imm:
                         $is_add ? $src1_value + $src2_value:
                         $is_load ? $src1_value + $imm:
                         $is_s_instr ? $src1_value + $imm :
                         $is_andi ? $src1_value & $imm:
                         $is_ori ? $src1_value | $imm:
                         $is_xori ? $src1_value ^ $imm:
                         $is_slli ? $src1_value << $imm[5:0]:
                         $is_srli ? $src1_value >> $imm[5:0]:
                         $is_and ? $src1_value & $src2_value:
                         $is_or ? $src1_value | $src2_value:
                         $is_xor ? $src1_value ^ $src2_value:
                         $is_sub ? $src1_value - $src2_value:
                         $is_sll ? $src1_value << $src2_value[4:0]:
                         $is_srl ? $src1_value >> $src2_value[4:0]:
                         $is_sltu ? $src1_value < $src2_value:
                         $is_sltiu ? $src1_value < $imm:
                         $is_lui ? {$imm[31:12],'0}:
                         $is_auipc ? $pc + $imm :
                         $is_jal ? $pc + 4 :
                         $is_jalr ? $pc + 4 :
                         $is_srai ? { {32{$src1_value[31]}}, $src1_value} >> $imm[4:0]:
                         $is_slt ? ($src1_value[31] == $src2_value[31]) ? $sltu_rslt : {31'b0, $src1_value[31]}:
                         $is_slti ? ($src1_value[31] == $imm[31]) ? $sltiu_rslt : {31'b0, $src1_value[31]}:
                         $is_sra ? { {32{$src1_value[31]}}, $src1_value} >> $src2_value[4:0]:
                         'x;

         //$rf_wr_en = $rd ? $rd_valid: 1'bx;
         //$rf_wr_index[4:0] = $rd;
         //$rf_wr_data[31:0] = $result[31:0];

         $taken_br = $is_beq ?($src1_value == $src2_value):
                     $is_bne ?($src1_value != $src2_value):
                     $is_blt ?(($src1_value < $src2_value) ^ ($src1_value[31] != $src2_value[31])):
                     $is_bge ?(($src1_value >= $src2_value) ^ ($src1_value[31] != $src2_value[31])):
                     $is_bltu ?($src1_value < $src2_value):
                     $is_bgeu ?($src1_value >= $src2_value):
                     1'b0;

         $valid_taken_br = $valid && $taken_br;
         $valid_load = $valid && $is_load;
         $is_jump = $is_jal || $is_jalr;
         $valid_jump = $is_jump && $valid;

         $valid = !(>>1$valid_taken_br || >>2$valid_taken_br || >>1$valid_load || >>2$valid_load || >>1$valid_jump || >>2$valid_jump) ;

         $jalr_tgt_pc[31:0] = $src1_value + $imm;
         $rf_wr_en = ($rd!='0 && $rd_valid && $valid) || >>2$valid_load;
         $rf_wr_index[4:0] = >>2$valid_load ? >>2$rd : $rd;
         $rf_wr_data[31:0] = >>2$valid_load ? >>2$ld_data: $result ;

      @4
         $dmem_wr_en = $is_s_instr && $valid;
         $dmem_addr[3:0] = $result[5:2];
         $dmem_wr_data[31:0] = $src2_value;
         $dmem_rd_en = $is_load;

      @5
         $ld_data[31:0] = $dmem_rd_data;
         //$br_tgt_pc[31:0] = $pc + $imm;
         // $valid_taken_br = $valid && $taken_br;
    // Note: Because of the magic we are using for visualisation, if visualisation is enabled below,
      //       be sure to avoid having unassigned signals (which you might be using for random inputs)
      //       other than those specifically expected in the labs. You'll get strange errors for these.


   // Assert these to end simulation (before Makerchip cycle limit).
   *passed = *cyc_cnt > 40;
   *failed = 1'b0;

   // Macro instantiations for:
   //  o instruction memory
   //  o register file
   //  o data memory
   //  o CPU visualization
   |cpu
      m4+imem(@1)    // Args: (read stage)
      m4+rf(@2, @3)  // Args: (read stage, write stage) - if equal, no register bypass is required
      m4+dmem(@4)    // Args: (read/write stage)

   m4+cpu_viz(@4)    // For visualisation, argument should be at least equal to the last stage of CPU logic. @4 would work for all labs.
\SV
   endmodule
```

![image](https://github.com/user-attachments/assets/58479bd6-6363-4751-aab2-693e3e668097)

The final Circuit:
![image](https://github.com/user-attachments/assets/3db8f54c-4485-49c9-8032-7d3991cad9dc)

Visualisation:
![image](https://github.com/user-attachments/assets/c7120231-2cf3-4bb6-a993-ac35e182f96a)

# Acknowledgement
- Kunal Ghosh, Co-founder (VSD Corp. Pvt. Ltd)
- Steve Hoover, Founder, Redwood EDA.
