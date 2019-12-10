# HPC architectures

## Parallel architecture

### (S/M)I(S/M)D

## SIMD

-   e.g. vector processors

![image-20191210181947741](https://i.imgur.com/CVsEbOi.png)

## MIMD

#### **Shared memory** MIMD-SM

Multiple processors share a single memory space. Communication via reads/writes to memory.

-   **Simple** to program, use and maintain
-   Difficult scaling: access to **memory** simultaneously 
-   Sophisticated hardware for **cache coherency**

##### SMP Symmetric MultiProcessing

Equal access to all parts of memory i.e. **same** latency and bandwidth.

![image-20191210182741124](https://i.imgur.com/tgSxDMn.png)

-   e.g. any multicore laptop/PC/server 

##### NUMA Non Uniform Memory Access

-   cache-coherent: cc

Each processor has some fast **local** memory. Direct access to slower remote memory via global address space.

-   modern multi-core processors are NUMA to some degree. 

![image-20191210183541193](https://i.imgur.com/KRWymLO.png)

#### **Distributed memory** MIMD-DM

Each processing unit has its own memory space. Each runs its own **copy of the OS**. No interaction except via the interconnect.

-   Highly scalable
-   Adding processors increases memory bandwidth 
-   Relies on interconnection quality
-   Job’s locality
-   High system management overhead potentially

![image-20191210182403353](https://i.imgur.com/cjjhOaW.png)

#### Distributed shared memory clusters

**Dominate** architecture. Shared memory within one node, distributed memory between nodes.

Each Shared memory block is known as a ***node***. Nodes can also contain accelerators.

-   constructed as a standard distributed memory machine • but with more powerful nodes
-   hard to take advantage of mixed architecture
-   complicated to understand performance: combination of interconnect and memory system behaviour 

![Screenshot 2019-12-10 at 18.44.52](https://i.imgur.com/fCpCRqU.png)

## Batch system 

-   Front end for user, running Unix commands, editing files, compiling code etc.
-   Back end for running application codes. Not interactive.

## Compared to Cloud Computing

|                    | Cloud                                                     | HPC                                                          |
| ------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| Performance        | Extra layer of virtual machine, shared hardware resources | Exclusive on hardware                                        |
| Tight-coupling     | Not have it                                               | Requires a low-latency, high-bandwidth communication system between tasks |
| Programming models | Often based on interpreted/JIT                            | High-level compiled languages with extensive optimisation    |

## Compiler

Source code -> compiler -> object code -> linker -> executable 

### IR intermediate representation

![image-20191210181316073](https://i.imgur.com/K240Qgy.png)

# HPC hardware

## Processors

Electronic circuits designed to run programs. 

### 4 key performance factors

1.  **Parallel processing**

    Now CPUs have multiple cores, where each can process multiple instructions per cycle 

2.  **Clock frequency**

    CPUs aim to maximise clock frequency, but this has now hit a 

    limit due to power restrictions 

3.  **Memory bandwidth**

    CPUs use regular DDR memory, which has limited bandwidth 

4.  **Memory latency**

    Latency from DDR is high, but CPUs strive to **hide** the latency 

    through multithreading or OoO execution
    

### Instruction set

Each type of processor is designed to interpret a particular language the “instruction set”. Forms an abstract description of the processor.

-   RISC reduced instruction set computer, newer
-   CISC complex … earlier

#### Vector instruction set

apply the same operation to all members of the argument vector registers 

#### Out of order execution (for structural and data hazards)

Hardware chooses to reorder instructions as they are fetched.

-   Good for running legacy binary code
-   Bad for small energy efficiency implementations

#### Branch prediction (for control hazards)

would like to know whether branches are taken or not.

### ILP instruction level parallelism

#### Limits of ILP

Hard to find enough independent instructions

-   compiler techniques help 
-   checking for hazards between *n* instructions requires *n^2* comparisons 
-   branches are major obstacle
-   many programs only exhibit average ILP of between 2 and 4. 

#### Pipelining

Execution of instructions is broken down into stages. Each stage can be executed in one CPU clock cycle.

-   Allows one instruction to be completed per clock cycle, even though the instruction itself may take many cycles to complete. 

#### Superscalar processors

Instructions are divided into classes which use different function units (e.g. integer and FP). Instructions in different classes can be issued in the same cycle. Instructions of the same class can also be issued together if there are replicated function units.

-   Detection done in hardware

-   Helped by compiler techniques

#### VLIW processors (very long instruction word)

Instructions are grouped into one long instruction corresponding to available functional units.

-   Scheduling done by compiler, not hardware
-   Leads to binary compatibility issues

#### SIMD

Instructions which encode operations on multiple data items. Requires a (substantial) extension to the instruction set architecture. An example of a vector instruction. 


### Clock

Operation of the processor is driven by a clock circuit, which generates electronic signals at a fixed frequency.

-    Limited by physics 

### Components

#### Register file

Local storage in the CPU, hold most of the state of the processor finite-state-machine 

#### Caches

Small, fast memory placed between CPU and main memory, Hold copies of data and instructions.

##### why

-   Fetching an item from cache is much faster than from main memory.
-   Hold copies of data and instructions for fast re-access.

##### Cache block (line)

The minimum unit of data which can be determined to be present in or absent from the cache. 

Address tag; set index; block offset

###### Size tradeoff 

-   Large blocks result in fewer misses because they exploit spatial locality. 
-   Too large causes additional capacity/conflict misses. 
-   Larger blocks have higher miss times

##### Set associativity

Cache is divided into sets. A set is a group of blocks (typically 2 or 4) 

If there are *k* blocks in a set, the cache is said to be k-way set associative.

##### Block replace

Random, LRU (least recently used, replace the block in set which was unused for longest time)

##### write

###### Write through

Write data to cache block and to main memory. Normally do not cache on miss. 

-   Good for I/O and multiprocessor cache coherency
-   Avoid CPU waiting

###### Write back

Write data to cache block only. Copy data back to main memory only when block is replaced. Cache on miss.

-   Reduce memory bandwidth
-   Harder to implement

##### Cache miss

Compulsory or cold start; capacity; conflict

memory access cost = hit time + miss ratio x miss time 

##### Prefetching

Load data into cache before the load is issued. Reduce miss rate.

-   Hardware: consecutive ones
-   Compiler: prefetch ahead of loads

#### Cache coherency

To avoid two processors caching different values of the same memory location, caches must be kept *coherent.* A write to memory location must cause all other copies to be removed,

###### Protocols

1.  Snooping based

    every cached copy caries sharing status (extra tag fore sharing status)

    -   MSI, MESI, MOESI

2.  Directory based

    sharing status stored centrally (in a directory) 

    -   Scalability issues: 

#### Function units

Basic building blocks of processors 

-   instruction unit
-   integer unit
-   Floating point unit
-   Control unit
-   Load/store unit

### Multicore/multithread

Modern CPUs are actually collections of multiple independent processors attached to a common memory system. A full **core** replicates all components of the processor, may still **share caches** with other cores.

With hardware support, a thread switch can be done in a single clock cycle. Can simply round-robin threads on consecutive cycles, or switch when a thread stalls on a load. **Latency hiding**

Simultaneous multithreading (SMT) (Hyperthreading) tries to fill **spare instruction slots** by mixing instructions from more than one thread in the same clock cycle.

#### Considerations

-   Processor affinity
-   Initialisation optimisation 
-   Avoiding false sharing 
-   Cache blocking/data tiling 
-   Delaying
-   Holding data 

### Multiprocessors 

In a distributed memory multiprocessor, each node runs a separate copy of the operating system. Processes **cannot** migrate between nodes

In a shared memory multiprocessor, one copy of the operating system manages all the CPUs. Processes **can** migrate between CPUs

### SIMD

256 GFlop/s: 2 GHz x 4 vector instruction x 2 FMA instruction x 16 cores 



## Accelerator

-   Problem with CPU: see processors.
-   Need a chip which can perform many parallel operations every clock cycle—need more simple, low power, number-crunching cores
-   Solution: CPU + accelerator 

### Accelerators addressing performance

- **Parallel processing**
a much **higher extent** of parallelism than CPUs. Many more cores and/or operations per core 

- **Clock frequency**
Accelerators typically have **lower** clock-frequency than CPUs, and instead get performance through **parallelism** 

- **Memory bandwidth**
Accelerators use **high** bandwidth GDDR memory or HBM2 memory 

- **Memory latency**
    - Memory latency from GDDR is similar to DDR 
    - GPUs hide latency through very high levels of **multithreading** 
    - Xeon Phi hides latency in a similar way to CPUs, although the caches are smaller and no out-of-order execution

### GPU

-   CPU-GPU communication (BW/latency) presents bottleneck 
-   Added complexity of programming model 
-   Tensor cores: matrix multiply engines D=AB+C
-   GPU accelerated systems scale from simple workstations to large-scale supercomputers 

### Intel Xeon Phi

-   Peak performance and memory bandwidth similar to GPUs 
-   Vector units drive performance 

## Memory

### SRAM, DRAM (static, dynamic)

DRAM: main memory. SRAM: cache

Organised in **banks**. Different banks can service read or write requests **concurrently**. 

### **Memory Stacking**

Multiple DRAM layers manufactured separately then glued together (HBM on Intel KNL or Nvidia pascal GPUs). 

### **Virtual memory**

-   Allows memory and disk to be seamless whole
-   Can share physical memory

### Consistency models

-   **Sequential consistency** requires all orderings to be preserved
-   **Processor consistency or total store ordering**. Allows reads to occur before previous writes have completed. 
-   Partial store ordering, Weak ordering, Release consistency

## Interconnect

### Packet switching

Network bits are usually organised into packets.

-   Currently HPC networks are all packet-switched networks. 
-   Routers read header to determine where next destination.

#### Optical switching

-   Circuit-switches, not packet-switches 

### Routing and addressing 

-   Deterministic
    -   Always takes the same path
    -   Packet order preserved 
    -   Not use all available bandwidth  
-   Adaptive
    -   Path can vary according to network conditions/randomly
    -   Easier to implement fault tolerance. 
    -   Better use of available paths. 
-   Minimal: always takes the shortest path, which may not necessarily be unique
-   Non-minimal

### Serial analogue signals

Many modern technologies use **high speed serial** connections

-   USB, Infiniband, SATA, PCIe, HDMI
-   Consumes large amounts of power 

### Optical

Modern optical networks use **lasers** confined within optical fibre. 

-   Higher frequency
-   Low energy loss

### Network

More complex networks are built by linking network segments using routers. Topology of the network can affect cost/performance.

-   Routers as graph nodes
-   Links as graph edges 

#### Diameter

The largest distance between 2 nodes contained in the graph.

-   proportional to the worst case network latency

#### Degree

of a graph node: the number of edges attached to that node. 

-   corresponds to the number of **ports** needed on the router.

#### Bisection

When a graph is divided into two equal parts and the number of links connecting the two parts is the *width* of that bisection.

*Minimum bisection width* the smallest width from all possible bisections.

-   proportional to the worst case bandwidth between two halves 

#### Pattern

|               | Degree | Diameter | MBW     |
| ------------- | ------ | -------- | ------- |
| Ring, p       | 2      | p/2      | 2       |
| 3D torus      | 6      | 3m/2     | 2m^2    |
| d-D Hypercube | d      | d        | 2^(d-1) |



## Storage

-   HSM hierarchical storage management system
-   Hierarchical devices

## FPGA (Field Programmable Gate Array)\

An IC that can be configured to perform different tasks. 

- No instructions; unless you create some yourself 
- No instruction cache misses/hits to worry about
- No data prefetching; unless you write the logic
- No data cache misses/hits to worry about 
- No need to worry about 32/64 bits; you can use any number of bits (e.g. 42) 
- No need to worry about expensive vs cheap instructions (except running out of space on the FPGA) 

### Programming Steps 

Programming -> synthesis -> place -> route ->timing/validation ->generation of bitstream

### Challenges

-   **Partition** (or splitting) of a design: an FPGA has a limited amount of space; it is important to port the correct kernel. 
-   Hard to program, debug
-   Slow memory, FP calculation
-   Algorithm might not be suitable 

### Performance, advantages

-   Lower clock frequency
-   Use custom made hardware circuitry to perform **multiple** sequential and parallel operation per clock cycle
-   Several orders of magnitude of parallelism is possible (especially true for fixed-precision calculations)
-   Very deep pipelines using flip/flops to store temporary values 
-   No jitter/latency (unexpected delays) in processing data
-   Super fast BRAM and fast High bandwidth memory availability
-   Direct connection to peripherals (network and memory devices)
-   Performance per watt is stellar (ideal for IoT, embedded devices)

### Compared to CPU/GPGPU

-   Different workflow compared to CPUs or GPGPUs; 

-   Data flows across the pre-defined hardware instructions vs the instructions/data fetching of CPUs/GPGPUs; 

-   FPGAs have a constant throughput of data without any jitters; very important for specific applications (networking, safety); 

-   FPGAs use less energy than CPUs or GPGPUs; very important for specific applications (embedded, IoT); 

-   ASICs use even less energy and have better performance (than FPGAs). 

### CLB configurable logic block

-   Logic functions, implemented as LUTs (lookup tables)
-   Multiplexers
-   Flip-flops/registers
-   Encoders/Decoders 

### Other

-   I/O: to the Host via PCIe, ...
-   BRAM: ~L1 cache
-   DRAM: ~DRAM for CPU
-   HBM2

### IC integrated circuit

Dedicated chips that pack together tens/hundreds/thousands of discrete **electronic components**. 

-   Designed for a broad range of use.

#### ASIC application-Specific Integrated Circuit 

-   Designed for a specific application. 

Compare:

-   ASICs are custom made chips, very expensive and time-consuming to produce and **cannot** be modified once manufactured (the algorithms is etched on the fabric) 
-   FPGAs are generic chips that can be programmed (and **reprogrammed**) to implement several algorithms ]



## HPC hardware trends

-   Many-core
-   Complex I/O and memory 
-   More heterogeneity in processing and memory system 

## Energy

### Efficiency  (Flop/s)/Watt

Perform computation using **resources optimally** for performance. Drawing as little power as possible for the computation. **Minimising the energy** consumed for the computation.

First and foremost: choose the appropriate **hardware** for your problem.

-   Adjusting the frequency of the CPU (DVFS Dynamic Voltage Frequency Scaling )
-   Do not add parallel resources unless you can use them efficiently
-   Not all CPU operations cost the same in terms of energy
-   Try different compilers and compiler flags 