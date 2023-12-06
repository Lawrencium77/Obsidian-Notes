In binary, addition can be decomposed into a series of and/or operations. 

In turn, all other mathematical operations (e.g. multiplication, subtraction) can be broken down into addition.  

**Clock speed** is the frequency with which components can turn on/off in the processor. This, in turn, determined the number of **clock cycles** that can be done per second. A clock cycle is simply one electronic pulse of a CPU - a clock rate of 3.4 GHz corresponds to 3.4 bn clock cycles per second.

Each mathematical operation requires a certain number of clock cycles to complete.

GPUs typically have lower clock speed than CPUs (although maybe by only a factor of ~2). Hardware design engineers are now reaching a physcial limit on clock speed, primarily due to thermal losses.

We can increase the number of cores in a processing unit by increasing the number of transistors. This requires making the transistors smaller. 


### Computational Logic

As explained above, all mathematical operations can be broken down into a series of and/or statements. These and/or statements are implemented by physical electronic devices, which are designed to perform simple Boolean functions on binary inputs.

Logic gates are primarly implemented using diodes or transistors (although there are lots of alternatives). In the words of Wikipedia:

 " With amplification, logic gates can be cascaded in the same way that Boolean functions can be composed, allowing the construction of a physical model of all of [Boolean logic](https://en.wikipedia.org/wiki/Boolean_logic "Boolean logic"), and therefore, all of the algorithms and mathematics that can be described with Boolean logic ''.
 
 Logic gates are often shown in a series of diagrams. The most important of these are as follows:
 
 ![](_attachments/Screenshot%202022-03-04%20at%2015.43.06.png)

![](_attachments/Screenshot%202022-03-04%20at%2015.43.19.png)

![](_attachments/Screenshot%202022-03-04%20at%2015.43.31.png)

![](_attachments/Screenshot%202022-03-04%20at%2015.43.42.png)

A really cool point here is that NAND gates are functionally complete - any boolean function can be implemented by a combination of NAND gates. This means that CPUs are essentially a large composition of NAND gates. 

NAND gates require 4 transitors to construct. NOR gates are also functionally complete, but are more complex to build. An entire processor can be built using NAND gates alone.

One of the fundamental building blocks of the CPU is the **Arithmetic Logical Unit (ALU)**. These perform arithmetic and bitwise operations on integer binary numbers.

### Other kinds of processing unit
There exists more than just CPUs and [GPUs](GPUs/GPUs.md)! CPUs are the most generaly type of processing unti we have. An example of anothe unit type is the **Field-Programmable Gate Array (FPGA)**, which essentially hard-code the user's specified operations into silicon.

![](_attachments/Screenshot%202022-03-04%20at%2015.34.32.png)