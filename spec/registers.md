# Register Model

Spurv defines 16 32-bit registers in the default 32-bit mode. They are as follows.

Code      | Name    | Access         | Purpose
----------|---------|----------------|--------
0000-0111 | ra-rh   | k = rw, u = rw | General-purpose registers
1000      | ip      | k = ro, u = ro | Instruction pointer, points to start of next instruction
1001      | sp      | k = rw, u = rw | Stack pointer, can be used as general-purpose register in most cases
1010      | bp      | k = rw, u = rw | Base pointer, points to start of previous stack frame
1011      | sr      | k = rw, u = rw | Status register, altered by ALU to reflect result of arithmetic operations
1100      | mr      | k = rw         | Mode register, indicates current CPU mode
1101      | fp      | k = ro         | Fault pointer, points to the address that triggered a page fault
1110      | r0      | k = ro, u = ro | Zero register, bits are permanently 0 (used to simplify instruction set)
1111      | r1      | k = ro, u = ro | One register, bits are permanently 1 (used to simplify instruction set)
