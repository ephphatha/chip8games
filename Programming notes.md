# Inlining Subroutines

When trying to eek out the last bit of performance it can be useful to remove call/return/jump instructions, especially if the subroutine/target is only ever reached from one location.
 Conversely, if you're trying to save a few more bytes it could be worthwhile turning even a really simple set of repeated instructions into a subroutine, despite now needing to
 execute call/return instructions as well as the actual work.

Where `INSTRUCTION_COUNT` is the number of instructions involved in a single instance of a routine and  `CALLS` is the number of distinct locations that call the routine the
 following formulae give the cost in terms of ROM size and executed instruction count. 
In terms of ROM size an inlined subroutine results in `INSTRUCTION_COUNT * CALLS` worth of instructions in the final rom, whereas a callable subroutine results in
 `INSTRUCTION_COUNT + 1 + CALLS` worth of instructions in the final rom.
The proportion of execution time spent on calling/returning a callable subroutine compared to an inlined subroutine is `2 / (INSTRUCTION_COUNT + 2)`.

The following examples assume that each subroutine has a single return point. Inlining a routine that has multiple return points is not impossible but becomes more costly very
 quickly. It's likely that a routine with more than a single code path is already too long to see any benefit from inlining (unless it is only used in one location).

## One instruction
Call subroutine is a single instruction already, always inline single instruction subroutines. Saves byte count, saves instruction count, can be used with every conditional skip.
 Creating a single instruction subroutine adds two bytes to the rom size and causes two extra instructions every time it is used.

## Two instructions
Two instruction subroutines start to get interesting. In terms of ROM size, if a subroutine is called from two distinct places in the ROM it still results in an extra instruction
 over inlining it twice (2 instructions, +1 for the return, +2 for the calls for a total of 5 instructions, inlining it twice would be only 4 instructions) and that's before
 factoring in the extra instructions being executed. At three call sites it is still preferable to inline as this is netural in terms of ROM size.

When calling the subroutine more than three times it starts to become a tradeoff. Calling and returning means that you are executing twice as many instructions as strictly
 necessary for the actual work effort. However you start to save an instruction from the ROM size (ie 2 bytes) every time the routine is called. If bytesize is critical then
 calling a subroutine saves an instruction from the fourth call site onward. If performance is more of a concern then be aware you will "waste" 2 bytes for each instance from the
 fourth onwards. Where performance is critical rewriting a conditional skip into a call and an inverted skip over a jump past the inlined subroutine is neutral on both byte size
 and instruction count.

## Three instructions
Subroutines with three instructions still result in 40% "wasted" instructions but the ROM size benefits are more immediate. A 3 instruction subroutine called from two distinct
 places in the rom takes up 6 instructions worth of bytes no matter whether it is inlined or a callable subroutine. It is still worth inlining at this point to avoid the wasted
 call/return instructions given it is size neutral.

For subroutines called three or more times, you save two instructions in terms of ROM size by defining a callable routine instead of inlining.

## Four or more instructions
At this point the only reason to inline is if the code path is performance critical or you only ever call it from a single location. The overhead of performing two additional
 instructions for a callable subroutine is still significant even up to routines 18 instructions long. The chart below shows the overhead involved in a callable subroutine up to
 20 instructions in length (not counting the call/return instructions) as a percentage of overall execution time (assuming no branches or conditional skips).

![Graph of 2/(x+2) for x in the range 1 to 20, where x represents the instruction count and y is the overhead involved in call/return instructions as a percentage](Call_Return%20Overhead%20vs.%20Raw%20Instruction%20Count.png)

The impact of executing a call/return into a routine is a reciprical relationship. For a routine of 1 instruction the overhead is 80%, but it drops sharply to 25% at 6
 instructions and peters out to 10% at 18 instructions.

# Multiplication
Multiplying by two, four, or 8 is always worth doing as shift instructions (*2 is one left shift, *4 is two, *8 is three). Other values can be faster (due to not needing to loop) and/or more
 space efficient if the range of operands is known and small. Multiplying by using a lookup table requires at minimum 3 instructions; initialising i, adding the offset (from a register, so
 another instruction is needed to multiply by a constant), and one more to load the value.