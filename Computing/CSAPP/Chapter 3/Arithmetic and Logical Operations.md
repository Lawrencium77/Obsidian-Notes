Figure 3.10 lists some of the x86-64 integer and logic operations. These operations are split into four groups:

1. Load effective address
2. Unary (single operand)
3. Binary (two operands)
4. Shifts

![](_attachments/Screenshot%202023-09-29%20at%2017.59.59.png)

## Load Effective Address
This is actually a variant of `movq`. It copies the *effective address* of the source (i.e. the actual memory location where the variable's value is stored), to the destination. In C, this is the address-of operator `&S`. 

## Bit Shift Operations
Note that there are two kinds of **right** shift operators: **logical** and **arithmetic**. These differ according to how the deal with the leftmost bit, as they move bits to the right. Arithmetic right shift preserves the sign bit and fills empty positions with the sign bit's value. Logical right shift ignores the sign but and fills empty positions with 0s.

