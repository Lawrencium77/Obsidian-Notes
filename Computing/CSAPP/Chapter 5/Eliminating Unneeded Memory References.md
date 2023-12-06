The code for `combine3` ([Figure 5.9](Reducing%20Procedure%20Calls.md)) uses the memory location `dest` as an accumulator. The x86-64 machine code for each loop iteration is shown below:

![](_attachments/Screenshot%202023-11-10%20at%2017.27.55.png)

We see that the accumulated value is read from and written to memory on each iteration. We can eliminate this by rewriting the code in the style shown in Figure 5.10:

![](_attachments/Screenshot%202023-11-10%20at%2017.28.56.png)

Making this change improves CPE measurements by a factor ranging from 2.2x to 5.7x.