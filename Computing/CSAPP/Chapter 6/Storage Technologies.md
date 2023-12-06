```toc
```

## Random Access Memory
* RAM comes in two varieties - static and dynamic.
* SRAM is faster and more expensive than DRAM.
* SRAM is used for cache memories, both on and off the CPU chip.
* DRAM is used for main memory.

#### SRAM
* SRAM stores each bit in a **bistable** memory cell. 
* There are two stable states. Any other state is unstable.
* This means its value is robust to electrical disturbances.
* Each cell is implemented with a six-transistor circuit.

#### DRAM
* DRAM stores each bit as charge on a capacitor.
* DRAM storage can be made very dense - it consists of a capacitor and a single transistor.
* But it's super sensitive.
* Various sources of leakage current cause a DRAM cell to lose its charge within ~100 ms. 
* But this is a long time for machines operating with clock cycles measured in ns.
* The memory system must periodically refresh every bit of memory by reading it out and then rewriting it. These are called **refresh cycles**.

This table summarises SRAM and DRAM:

![](_attachments/Screenshot%202023-03-19%20at%2018.02.33.png)

> [!NOTE]
> I skipped sections on Conventional DRAMs, Memory Modules, and Enhanced DRAMs.

#### Nonvolatile Memory
* DRAM and SRAM are [volatile](https://en.wikipedia.org/wiki/Volatile_memory).
* There also exists a variety of [nonvolatile](https://en.wikipedia.org/wiki/Non-volatile_memory) memories. There are collectively referred to as **read-only memories (ROMs)**[^fn1], even though some types of ROMs can be written to.
* ROMs are distinguished by the number of times they can be reprogrammed (written to) and by the mechanism for reprogramming them:
	* A *programmable* ROM (PROM) can be programmed exactly once.
	* An *erasable programmable* ROM (EPROM) can be programmed ~$10^3$ times.
	* An *electrically erasable* PROM (EEPROM) can be reprogrammed ~$10^5$ times.
	* *Flash memory* is based on EEPROMs. SSDs are a flash-based disk drive. ^fd6059
* Programs stored in ROM devices are often called **firmware**. When a computer system is powered up, it runs firmware stored in a ROM.

#### Accessing Main Memory
* Data flows between the processor and DRAM main memory over electrical conduits called **buses**. 
* A bus is a collection of parallel wires that carry address, data, and control signals.
* A data transfer is accomplished with a series of steps called a **bus transaction**.
* A **read transaction** transfers data from CPU to main memory.
* A **write transaction** does the opposite.
* Figure 6.6 shows the configuration of an example computer system:

![](_attachments/Screenshot%202023-03-19%20at%2018.18.54.png)

* The main components are the CPU chip, a chipset called the **I/O bridge**, and DRAM main memory.
* A **system bus** connects the CPU to I/O bridge.
* A **memory bus** connects the I/O bridge to main memory.
* The system bus "translates the electrical signals of the system bus into electrical signals of the memory bus".
* The I/O bridge also connects the system & memory buses to the **I/O bus**. 

But for now, we'll focus on the memory bus.

* Consider what happens when the CPU performs a load operation like `movq A,%rax`, where the contents of address `A` are loaded into register `%rax`.
* Circuitry on the CPU chip called the **bus interface** initiates a read transaction on the bus. The read transaction consists of three stages:

![](_attachments/Screenshot%202023-03-19%20at%2018.24.38.png)

* First, the CPU places  `A` on the system bus.
* Second, the I/O bridge passes the signal to the memory bus. The main memory senses the address signal, reads the address from the memory bus, fetches the data from DRAM, and writes the data to the memory bus. The I/O bridge translates the memory bus signal into a system bus signal.
* Third, the CPU senses the data from the system bus, reads the data from the bus, and copies the data to `%rax`.
* A similar set of steps are followed during a write transaction.

## Disk Storage 
Disks are workhorse storage devices that hold enormous amounts of data. But it takes on the order of milliseconds to read information from a disk; ~$10^5$ times longer than from DRAM.

#### Disk Geometry
* Disks are constructed from **platters**. Each platter consists of two **surfaces**.
* A rotating **spindle** in the centre spins the platter at a fixed **rotational rate**.
* A disk typically contains one or more platters in a sealed container.

Here's a diagram:

![](_attachments/Screenshot%202023-03-20%20at%2021.15.14.png)

* Each surface consists of concentric rings, called **tracks**.
* Each track is partitioned into a collection of **sectors**.
* Each sector contains an equal number of bits encoded in the magnetic material on the sector.
* Sectors are separated by **gaps**. These store formatting bits that identify sectors.

#### Disk Capacity
* Disk capacity is determined by **recording density** (number of bits per inch of track), and **track density** (number of tracks per inch of radius).

#### Disk Operation
* Disks read/write bits using a **read/write head**. 
* By moving the head along its radial axis, the drive can position the head over any track.

Here's another diagram:

![](_attachments/Screenshot%202023-03-20%20at%2021.20.18.png)

* Disks with multiple platters possess a separate head for each surface.

> [!FUN FACT]
> The head flies (literally) on a thin cushion of air over the disk surface, at a height of ~0.1 micros and a speed of ~ 80 km/h!

* Disks read/write data in sector-size blocks. The **access time** has three components:
	* **Seek time**: time taken to move the head to the correct track.
	* **Rotational latency**: time spent waiting for the first bit of the target sector to pass under the head.
	* **Transfer time**: Time spent reading a single sector.
* Seek time and rotational latency usually dominate, and are roughly equal. So twice the seek time is a reasonable rule for estimating disk access time.


#### Logical Disk Blocks
* To hide the complexity of disk geometries from the OS, modern disks present their geometry as $B$ sector-sized **logical blocks**, numbered $0,1,\dots,B-1$.
* A small hardware/firmware device in the disk package, called the **disk controller**, maintains the mapping between logical block numbers and actual (physical disk sectors).
* When the OS wants to perform an I/O operation, it sends a command to the disk controller, specifying a particular logical block number. Firmware on the controller performs a fast table lookup to translate the logical block number into a (surface, track, sector) triple that identifies its physical location.
* Hardware on the controller interprets this triple to move the heads to the appropriate cylinder, gathers up the bits sensed by the head into a memory buffer on the controller, and copies them into main memory.

#### Connecting I/O Devices
* I/O devices are connected to the CPU and main memory via an **I/O bus**:

![](_attachments/Screenshot%202023-03-20%20at%2021.38.43.png)

* The I/O bus is slower than the system and memory buses.
* But it can accommodate a variety of I/O devices.

#### Accessing Disks
* A detailed description of how I/O devices work is beyond our scope.
* The CPU issues a command to I/O devices using a technique called **memory-mapped I/O**. In a system with memory-mapped I/O, a block of addresses in VM space is reserved for communicating with I/O devices. ^e55d6d
* Each of these addresses is called an **I/O port**. Each device is associated with one or more ports when it's attached to the bus.
* For example, suppose the disk controller is mapped to port `0xa0`. The CPU might initiate a disk read by executing three store instructions to address `0xa0`:
	* The first sends a command word that tells the disk to initiate a read.
	* The second specifies the logical block number.
	* The third indicates the main memory address to store the output.
* After issuing the request, the CPU will typically do other work.
* After the disk controller receives the read command, it translates the logical block number to a sector address, reads the contents of the sector, and transfers them directly to main memory, without intervention from the CPU.
* This process, whereby a device performs a R/W bus transaction on its own, without CPU involvement, is called [direct memory access](https://en.wikipedia.org/wiki/Direct_memory_access)(DMA).
* Once the DMA transfer is complete, the controller notifies the CPU by sending an interrupt signal. 

Here's a summary diagram:

![](_attachments/Screenshot%202023-03-20%20at%2022.01.23.png)

## Solid State Disks
* **Solid State Disks** are based on [flash memory](#^fd6059). In some situations, they're better than the conventional rotating disk.
* Here's the basic idea:

![](_attachments/Screenshot%202023-03-20%20at%2022.07.15.png)

* An SSD package consists of one or more flash memory chips, and a **flash translation layer**, which is a hardware/firmware device that plays the same role as a disk controller, translating logical blocks into accesses of the underlying physical device.
* A flash memory consists of $B$ **blocks**, where each block contains $P$ **pages**. Typically, a block consists of 32-128 pages.
* A page can be written only after the entire **block** has been erased. However, once a block is erased, each page can be written once with no further erasing.
* A block wears out after ~$10^5$ writes.
* This table shows the performance characteristics of a typical SSD:

![](_attachments/Screenshot%202023-03-20%20at%2022.10.28.png)

* Note the reading is faster than writing.
* Random writes are slower for two reasons:
	* Erasing a block takes a relatively long time.
	* If a write operation attempts to modify a page, then any pages in the same block with useful data must be copied to a new block before the write can occur.
* SSDs have a number of advantages over rotating disks:
	* Faster access times.
	* Use less power.
* They also have disadvantages:
	* Potential to wear out after many writes. **Wear-leveling** logic in the flash translation layer attempts to spread erasures evenly across all blocks. In practice, it's pretty good, so SSDs take many years to wear out.
	* They're ~30x more expensive per byte than rotating disks (although this gap is decreasing). 

 [^fn1]: I'm not convinced that all non-volatile memories are referred to as ROMs. It seems that some non-volatile memories are not ROMs. See [here](https://en.wikipedia.org/wiki/Non-volatile_memory) for more.