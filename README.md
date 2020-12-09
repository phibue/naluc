# naluc
No Arithmetic Unit Computer: If you can count and have memory you can do all calculations 

Highly inspired by the projects of agp.cooper, weird CPU and CPU4, I try to design my own.

Goals:
- Use of 74xxxx Chips, except for memory
- Keep it simple
- Easy programmable and usefull
- Efficient and fast execution
- Capable of context switching. Interrupts
- Keep it as weird as possible

Implications:
- Address range 16bit at least
- Serial interface for programming, input and output. Interrupts help to make the serial interface in software
- Efficiency compromizes simplicity. Some additional chips can make it much faster

Possibilities to add some additional chips vs. simplicity:
1. No need for a PC (program counter): A programm counter can be incremented using the ALU. An instruction needs more cycles, decoding becomes complex and slow. --> not a good idea
2. Separate program memory (Harward architecture): To have just one memory could be easier, but in that case the PC using 74LS191 needs additional tristate buffers to switch the counter to the shared address bus. A Harward architecture needs some additional buffers to load the program memory. But program execution can be faster. Single cycle execution simplifies decoding. --> yes, this is the way to go
3. Databus 16bit is easier than 8bit since addresses have to be 16bit. And 16bit instructions are easier to decode. 8bit immediate arguments can be included. --> Yes, 16bit keeps things simple
4. Memory: Static RAM chips are cheap and fast up to the size of several Mbit. This is too tempting, I can't build 64kB RAM using old chips from 1970. --> These large memories use 3.3V supply. I'll use 74HCxx powered with 3.3V. 
5. Some page addressing is necessary for memory >64kWord
6. Using 16bit everywhere leads to high chip count. The number of registers must be minimized. Memory mapped registers solve this problem. Context switching on interrupt becomes more efficient.
7. Functional programming, compiler languages and recursive functions need efficient addressing modes. Local variables within functions need some data stack. Relative addressing to a data stack pointer requires lots of slow 16bit additions. Therefore a stack pointer to PUSH and POP data is easier. Register bank switching is also easier. This moves some complexity from hardware to the compiler. A hardware stack pointer speeds up stack access, it needs 4* 74HC191 and 2* 74HC573. This is the same chip count as 3 additional registers. --> The stack pointer seems to be a good idea. A stack pointer may be usefull to handle interrupts.
8. Interrupt: The interrupt handler is a program at address 0. The interrupt signal will execute a JUMP 0 instruction, which sets the PC to 0. A Register is required that keeps the former PC content, the PCR register (PC return address). Interrupt and all jump instructions including subroutine calls and return instructions, they all update the PCR. The PCR is saved at the beginning of the interrupt handler and at the beginning of all subroutines. Interrupts must be disabled during the next instruction to avoid overwriting the PCR. 
--> No need for a separate CALL instruction. Just use JUMP and the destination decides if it needs to keep the PCR or not.
9. And finally, to make it quite weird: Memory access time is 10ns. A 16bit adder using 74HC181 is much slower. The concept of the weird CPU fits well because the result from the adder can be read several instructions later. The processor clock can be much faster than the adder speed. 
However I decided to remove the adder because it is slow and needs too many chips. The hardware stack pointer is able to count up and down. This is all which is necessary to build up a lookup table for an 8 bit adder. Ok, this lookup table needs 16bit input and 8bit output. This eats up 64kB of memory. The memory is large enough. The same can be done with multiplication or any other operation. The 10ns memory is faster than everything else I could build anyway. --> Without arithmetic unit the processor will be much faster at arithmetics!  

Conclusion:
- At a closer look it becomes clear that such a design ends up in quite a large number of chips. I would prefer something small and simple.
- It turns out that instruction decoding easily becomes really complicated. The number of instructions must be kept really small.
- Especially interrupts add quite a lot of complexity. 
- Separate program memory: 
  * using RAM: Additional hardware is required to initialize this RAM. An additional instruction is necessary to write on this RAM
  * using ROM: Much simpler. In that case the program is static an could contain an interpreter language. An interpreter language could make programming easy.
    However: During development this ROM needs to be programmable somehow.
- Probably doing everything on an FPGA is the better approach. Building with 74xx can be done in a second step.	





§+Q2q