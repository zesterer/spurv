# Register Model

Spurv defines 16 32-bit registers in the default 32-bit mode. They are as follows.

Code      | Name    | Access             | Description
----------|---------|--------------------|------------
0000-0111 | ra-rh   | k = rw, u = rw     | General-purpose registers
1000      | ip      | k = ro, u = ro     | Instruction pointer, points to start of next instruction
1001      | sp      | k = rw, u = rw     | Stack pointer, can be used as general-purpose register in most cases
1010      | bp      | k = rw, u = rw     | Base pointer, points to start of previous stack frame
1011      | sr      | k = rw, u = rw     | Status register, altered by ALU to reflect result of arithmetic operations
1100      | mr      | k = rw             | Mode register, indicates current CPU mode
1101      | fp      | k = ro             | Fault pointer, points to the address that triggered a page fault
1110      | r0      | k = ro, u = ro     | Zero register, bits are permanently 0 (used to simplify instruction set)
1111      | r1      | k = ro, u = ro     | One register, bits are permanently 1 (used to simplify instruction set)

## Register Details

### General-purpose registers, 'ra-rh'

ra, rb, rc, rd, re, rf, rg, and rh are all general-purpose 32-bit registers in the default 32-bit mode. They can hold any value, at almost any time, and exist for general-purpose computation, as decided by the programmer / compiler. They don't follow ARM's r0-rX naming scheme for two reasons: Firstly, remembering a digit tends to be more difficult for a programmer that finds themselves used to alphabetical identifiers. Secondly, 'r0' and 'r1' are used by the zero register and one register respectively.

### Instruction pointer, 'ip'

The instruction pointer contains the address of the start of the next instruction at the point of any logical operation being applied. When an instruction is decoded and the form of the instruction is found (i.e: does it contain trailing bytes, or how long is the instruction?) the instruction pointer is incremented accordingly such that it will always point to the address of the next instruction during the execution phase of an operation.

### Stack pointer, 'sp'

The stack pointer contains the starting address of the last item pushed onto the stack. All stack 'objects' are equal to the size of the registers (32 bits in the default 32-bit mode), so the stack pointer contains the address of the first of 4 bytes previously pushed to the stack.

### Base pointer, 'bp'

The base pointer contains the starting address of the previous stack frame. Upon stack initiation, the base pointer should be initiated with a value of 0, indicating that the stack goes no further back into memory. On every function call, the base pointer should be pushed to the stack, along with the instruction pointer, as indicated elsewhere in this specification.

### Status register, 'sr'

The status register describes the result of recent arithmetic operations. However, it can be artifically overwritten to trigger more interesting behaviour if necessary. The bits in the status register are as follows:

ZCNV-0000 0000-0000 0000-0000 0000-0000

(Any un-lettered bits are reserved and should remain zero for compliance with the current Spurv specification)

Bit | Name          | Description
----|---------------|------------
Z   | Zero flag     | Will be set to 1 if the last arithmetic operation gave an answer of zero
C   | Carry flag    | Will be set to 1 if the last arithmetic operation gave an answer too large to fit within a 32-bit integer
N   | Negative flag | Will be set to 1 if the last arithmetic operation gave an answer that had the sign bit (bit 31) set
V   | Overflow flag | Will be set to 1 if the last artihmetic operation gave an answer in which a signed 32-bit integer overflow occured

### Mode register, 'mr'

The mode register describes the current execution state of the CPU. It is inaccessible while in user mode, and its use in user mode will trigger an illegal_register_access exception. It defines several useful flags for the operating system to determine the permission level of the CPU and the mode of the CPU. Its bits are as follows, and may be modified by code to produce a mode change. The bits in the mode register are as follows:

MIEP-XXXX 0000-0000 0000-0000 0000-0000

Bit | Name           | Description
----|----------------|------------
M   | Mode flag      | Indicates the current permission mode. 0 = kernel, 1 = user
I   | Interrupt flag | Indicates whether the CPU is executing within an interrupt handler
E   | Exception flag | Indicates whether the CPU is executing within an exception handler (note that an exception IS a form of interrupt)
P   | Paging flag    | Indicates whether the CPU has enabled paging (address protection and translation)

### Fault pointer, 'fp'

The fault pointer points to the address that caused a page fault and is accessible only by kernel mode. Attempting to access it from user mode will trigger an illegal_register_access exception. Its value does not change until another page fault occurs.

### Zero register, 'r0'

The zero register is a read-only (in all permission modes) register that contains only 0 bits. It is used to simply arithmetic operations and CPU implementation. For example, an arithmetic operation applied to the zero register will still have its result bits set in the status register, without the need to wipe an existing general-purpose register.

### One register, 'r1'

The one register is, a read-only (in all permission modes) register that contains only 1 bits. Like the zero register, it is used to simply the instruction set. For example, an XOR operation that takes the one register as an operand as the effect of negating the bits of the other operand (i.e: simulating a NOT operation). This means that no NOT instruction is required in the instruction set.
