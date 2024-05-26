- Processors have a number of registers inside the processor core to perform data processing and control.
- Most of these registers are grouped in a unit called  register bank .

- **Each data processing instruction specifies.**
	- Operation required.
	- Source Register(s).
	- Destination Register(s).
#### Load-store architecture
In the ARM architecture, if data in memory is to be processed, it has to be loaded from the memory to registers in the register bank, processed inside the processor, and then written back to the memory, if needed.

By having a sufficient number of registers in the  register bank , this arrangement is easy to use, and allows efficient program code to be generated using C compilers.

For instance, a number of data variables can be stored in the register bank for a short period of time while other data processing takes place, without the need to be updated to
the system memory and read back every time they are used.
![[Pasted image 20240524225151.png]]
Cortex-M4 processor has 16 registers. 13 of them are general purpose 32-bit registers, and the other three have special uses.

## General Purpose Registers (GPRs):
#### R0 - R12:
- General Purpose Registers.
	- R0 - R7 >> Called low register. Due to the limited available space, many 16-bit instructions can only access the low registers.
	- R8 - R12 >> Can be used with the 32-bit instructions, and few 16-bit instructions, like MOV.
	- R0 - R12: initial values are undefined.
_______________________________________________________________________
Each instruction in the ARM Cortex-M4 architecture is 32 bits long and contains three parts: an opcode, operand 1, and operand 2. It might seem confusing because each part is approximately 32 bits, which would total around 96 bits, far exceeding the 32-bit instruction size. Here's how the ARM Cortex-M4 manages to fit everything into 32 bits:
#### 1. Field Length Allocation
The 32-bit instruction is divided into smaller fields. Typically, the opcode, operand 1, and operand 2 each use fewer bits than 32. For instance, the opcode might use 6-8 bits, and the remaining bits are split between the operands.

**Example:**
```
Instruction: ADD R1, R2, R3

- Opcode (ADD): 8 bits
- Operand 1 (R1 - Destination Register): 4 bits
- Operand 2 (R2 - First Source Register): 4 bits
- Operand 3 (R3 - Second Source Register): 4 bits
- Additional fields (e.g., condition codes, shift amounts): remaining 12 bits

[ opcode (8 bits) | R1 (4 bits) | R2 (4 bits) | R3 (4 bits) | additional fields (12 bits) ]
```

#### 2. Register Indexes
When operands are registers, the instruction uses register indexes instead of full 32-bit values. For example, in `ADD R1, R2, R3`, `R1`, `R2`, and `R3` are represented by their register indexes, which are typically 4-5 bits each, allowing up to 16 or 32 registers.

**Example:**
```
Instruction: ADD R1, R2, R3

- Opcode (ADD): 8 bits
- Operand 1 (R1 - Destination Register): 4 bits
- Operand 2 (R2 - First Source Register): 4 bits
- Operand 3 (R3 - Second Source Register): 4 bits
- Additional fields: remaining 12 bits

[ opcode (8 bits) | R1 (4 bits) | R2 (4 bits) | R3 (4 bits) | additional fields (12 bits) ]
```

#### 3. Memory Addresses and Immediate Values
When the operands involve memory addresses or direct values, different encoding schemes are used to fit them within the 32-bit instruction format. These schemes might use a combination of smaller fields, immediate values, and base registers.

**Examples:**
- **Memory Address Example**: `LDR R1, [R2, #4]` (Load from memory, where address is R2 + 4)
    ```
    Instruction: LDR R1, [R2, #4]

    - Opcode (LDR): 8 bits
    - Operand 1 (R1 - Destination Register): 4 bits
    - Base Register (R2): 4 bits
    - Offset (4): 12 bits
    - Additional fields: remaining 4 bits

    [ opcode (8 bits) | R1 (4 bits) | R2 (4 bits) | offset (12 bits) | additional fields (4 bits) ]
    ```

- **Direct Value Example**: `MOV R1, #10` (Move the value 10 into R1)
    ```
    Instruction: MOV R1, #10

    - Opcode (MOV): 8 bits
    - Operand 1 (R1 - Destination Register): 4 bits
    - Immediate value (10): 12 bits
    - Additional fields: remaining 8 bits

    [ opcode (8 bits) | R1 (4 bits) | immediate value (12 bits) | additional fields (8 bits) ]
    ```

General-Purpose Registers (GPRs) are essential components within a processor core, serving as fast-access storage units for data and instructions. Their significance lies in:

1. **Speed and Efficiency:** GPRs enable high-speed data processing by providing fast access to frequently used values directly within the CPU core. This minimizes the need for slower memory accesses, enhancing overall system performance and efficiency.

2. **Data Storage and Manipulation:** GPRs store operand values, critical data, and intermediate results during computation. This allows for efficient data manipulation, arithmetic operations, and logical processing within the CPU.

3. **Resource Optimization:** Efficient utilization of GPRs is crucial for optimizing resource usage within the processor. Compilers allocate and manage GPRs effectively to minimize memory access times and maximize parallel processing capabilities, leading to optimized code execution.

4. **Pipeline Efficiency:** GPRs play a key role in maintaining the efficiency of instruction pipelines within the CPU. By storing instruction operands and results, they ensure smooth instruction flow through the pipeline, preventing stalls and delays in execution.

5. **Compiler Support:** Compilers optimize code generation by efficiently utilizing available GPRs. This includes register allocation strategies to minimize register spills and maximize the use of available registers, contributing to improved performance of compiled code.
___________________________________________________________________
## Special Registers:
#### R13, Stack Pointer (SP):
It is used for accessing the stack memory via PUSH and POP operations.
- **There are two stack pointers:**
	- **Main stack pointer ( MSP or SP_main):** 
		- Is the default stack pointer. It's selected after reset or when the processor is in the Handler mode.
	- **Process stack pointer ( PSP or SP_process):** 
		- The PSP is used only in the Thread mode.
		- The selection of the stack pointer is determined by a special register called **CONTROL**.
	- Hint: In normal programs, only one of these Stack Pointers will be visible.
	- Both MSP and PSP are 32-bit.
	- The lowest two bits of these stack pointers (MSP - PSP) are always zero, and write to these bits is ignored.
	- In ARM Cortex-M processors, PUSH and POP are always 32 bit and the addresses of the transfers in stack operations must be aligned with 32-bit boundaries.
#### R14, Link Register (LR):
R14, also referred to as the Link Register (LR), functions as a storage for the return address during the execution of a program. When a function or subroutine is invoked, the LR retains the address of the instruction to which control should return after the function or subroutine completes. Upon completion, the value in LR is loaded back into the Program Counter (PC) to resume execution at the correct point in the main program. 

If a function needs to call another function or subroutine, it must first save the current value of LR onto the stack to prevent it from being overwritten. During exception handling, the LR is automatically updated to a specific value known as EXC_RETURN, which is used to facilitate the return to the main program once the exception handler has finished executing.

==Important notes:==
Although the return address values in the Cortex-M processors are always even
(bit 0 is zero because the instructions must be aligned to half-word addresses), bit
0 of LR is readable and writable. Some of the branch/call operations require that
bit zero of LR (or any register being used) be set to 1 to indicate Thumb state.
![[Pasted image 20240526004812.png]]
**Call depth** refers to the number of nested function calls at any given point during the execution of a program. It represents how deep the current execution context is in terms of function calls. Each time a function calls another function, the call depth increases by one. Conversely, when a function returns, the call depth decreases by one.

