# Interrupts

Spurv's interrupt model is designed to be as simplistic as possible without compromising on standard features. Spurv defines 64 interrupt vectors addresses, all of which are stored in internal registers within the CPU. By default, Spurv starts with all of these addresses initiated to 0. Spurv defines two interrupt classes, which have inherently different behaviours. The first, 'exception interrupts', are interrupts triggered internally by the CPU upon events such as page faults, permission errors, arithmetic errors, and other such errors. Please note that the syscall interrupt vector is considered an exception vector since it is raised internally. The second, 'I/O interrupts', are triggered upon I/O events from external devices. These devices may happen to sit on the same chip as the main CPU, but are considered to be external for all intents and purpose. Examples include timer events, I/O bus events, etc.

Spurv also defines 64 internal registers that point to the address immediately after the top of the stack that should be used for each interrupt vector.

## Interrupt Calls

Interrupt calls are not equivalent to standard branch instructions, and have a fixed ABI. They cause the jump to the interrupt vector address and interrupt stack previously specified, then push the following items onto the stack in this order:

- return address (previous value of the instruction pointer)
- stack address (previous value of the stack pointer)
- base address (previous value of the stack pointer)
- previous mode (previous value of the mode register)

## Exception Interrupts

Exception interrupts set both the 'exception' and 'interrupt' bits in the mode register, then perform an interrupt call on the exception address.

There are 32 exception interrupt vectors, specified as follows:

Vector | Name                    | Description
-------|-------------------------|------------
0x00   | last_chance             | Triggered if an exception occurs while in an exception handler (i.e: the exception flag in the mode register is set)
0x01   | null_access             | Triggered when the CPU attempts to read/write from/to a physical address witin the 0x0000-0x1000 range
0x02   | illegal_register_access | Triggered when the CPU attempts to read/write from/to a register that the current mode does not have permission for
0x03   | page_fault              | Triggered when the CPU attempts to access a virtual address that does not exist or has no permission for (note: there is deliberately no distinction between 'virtual address existing' and 'virtual address being permissible')
0x04   | illegal_instruction     | Triggered when the CPU attempts to execute an instruction that the current mode does not have permission to execute

## I/O Interrupts

I/O interrupts take lower precedence than exception interrupts. If an I/O interrupt occurs during the same clock cycle as an exception interrupt, the CPU will choose to handle the exception interrupt first.

I/O interrupts are cached internally by the CPU in a FIFO buffer. After each clock cycle, the CPU checks the contents of the buffer. If it has at least one incoming exception entry, it handles it. Otherwise, execution continues as normal.

I/O interrupts may originate from a range of external devices, and the methods that these devices use to signal events is implementation-defined (TODO: clarify this more, standardise busses). Since they are implementation-defined, the use of each of the 32 I/O interrupt vectors is also implementation-defined.
