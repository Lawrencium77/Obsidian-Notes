* A process shouldn't be allowed to modify its read-only code, any of the kernel code, or any VPs that are shared with other processes.
* Providing separate virtual address spaces makes it easy to isolate the private memories of different processes.
* But address translation can be extended to provide even finer-access control.
* Since address translation hardware reads a PTE each time the CPU generates an address, it's easy to control access to a VP by adding some permission bits to the PTE.
* Figure 9.10 shows the idea:

![](_attachments/Screenshot%202023-04-10%20at%2015.13.41.png)

* We have three permission bits for each PTE.
* The `SUP` bit indicates whether processes must be running in [kernel mode](https://stackoverflow.com/questions/1311402/what-is-the-difference-between-user-and-kernel-modes-in-operating-systems) to access the page. Processes running in kernel mode can access any page. But processes running in user mode can only access pages for which `SUP` is `0`.
* The `READ` and `WRITE` bits control read and write access to the page.
* If an instruction violates these permissions, the CPU triggers a protection fault that transfers control to an exception handler in the kernel. This sends a `SIGSEGV` signal to the offending process.
* Linux shells report this as a [segmentation fault](https://en.wikipedia.org/wiki/Segmentation_fault). ^30cbf1



