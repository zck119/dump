6.172 class notes
=============

Lecture 2 Bentley Rules for Optimizing Work 
---------------
Work: amount of operations for a given input. 
\* Reducing work doesn't necessarily reducing running time. 

### Data Structures 
**Packing and Encoding** 
Packing: store more than one data value in a machine word. 
Encoding: convert data values into a representation requiring fewer bits. 

Pack data with bits: 
```C
// date struct with only 22 bits
typedef struct {
    unsigned int year: 13; 
    unsigned int month: 4; 
    unsigned int day: 5; 
} date_t;
```

**Augmentation**
Make common operations do less work. 

**Precomputation**
Perform calculations in advance. 

**Compile-Time Initialization**
Store the values of constants during compilation. 

**Caching**
Store results accessed recently so that they don't have to be calculated again. 

**Sparsity**
Avoid storing and computing on zeros. 

### Logic
**Constant folding and Propagation**
Compute and substitute constants during compilation. The constants and expressions are evaluated with sufficiently high optimization level. 

**Common subexpression elimination**
Avoid computing the same expression multiple times by storing the value after the first evaluation. 

**Algebraic Identities**
Replace expensive logical expressions with algebraic equivalents that requires less work. 

**Short-Circuiting**
When performing a series of tests, stop evaluating as soon as the answer is clear. 

**Ordering Tests and Creating a Fast Path**
Perform those more successful before those rarely successful. Inexpensive tests should precede expensive ones. 

e.g. When deciding whether two spheres collide, first check if the two bounding boxes collide. 

**Combining Tests**
Replace a sequence of tests with one test or a switch. 

### Loops
**Hoisting**
I.e. loop-invariant code motion: avoid recomputing loop-invariant code. 

**Sentinels**
Special dummy values placed in a data structure to simplify logic of boundary conditions and handling of loop-exit tests. 

e.g. when checking for overflow, force overflow by adding an extra large value. Then only overflow needs to be checked, instead of both overflow and whether index reaches end of array. 


**Partial Loop Unrolling**
Repeat the loop several times and increase loop increment. 
Take care of the corner cases!

**Loop Fusion (Jamming)**
Saving the overhead of loop control by combine multiple loops together. 

**Eliminating Wasted Iterations**
Modify loop bounds to avoid executing loop iterations. 

### Functions
**Inlining**
Avoid the overhead of function call by replacing a call to a function with the body of the function itself.

**Passing by Reference**

**Tail-Recursion Elimitation**
Replace a recursive call that occurs as the last step of a function with a branch (i.e. run it within the current function, with different parameters), saving function-call overhead. 

**Coarsening Recursion**
Increase the size of base case and handle it with more efficient that avoids function-call overhead.  


Lecture 3 - Bit hacks
-----------------
**Two's complement**
`x + ~x = -1`, i.e. `-x = ~x + 1`

**C Bitwise Operators**
`&` AND
`|` OR
`^` XOR
`~` NOT
`<<` shift left
`>>` shift right

Examples: 
- Set the k-th bit in a word x to 1: `y = x | (1 << k)`
- Clear the k-th bit in a word x: `y = x & ~(1 << k)`
- Flip the k-th bit in a word x: `y = x ^ (1 << k)`
- Extract a bit field from a word x: `(x & mask) >> shift`
- Set a bit field in a word x to a value y: `x = (x & ~mask) | ((y << shift) & mask)`
- Swap two integers x and y without temp: `x = x ^ y; y = x ^ y; x = x ^ y`
    - **Note** the performance is actually better if temp is used, since in that case two write can happen at the same time, while here it's serial
- Find the minimum r of integers x and y (with branch): `r = (x < y) ? x : y`
- Find the minimum r of integers x and y (without branch): `r = y ^ ((x ^ y) & -(x < y))`
    - However, modern compilers can do this on their own, and they work **better**...
- Compute (x+y) mod n, assuming 0 <= x,y <= n: `z = x + y; r = z - (n & -(z >= n))`
- Round up to a power of 2: flood lower bits, then +1 
```C
uint64_t n;
--n;            // in case n is already a power of 2
n |= n >> 1;
n |= n >> 2;
n |= n >> 4;
n |= n >> 8;
n |= n >> 16;
n |= n >> 32;
++n;
```
- Find the least significant 1 in a word x: `r = x & (-x)`
- Compute the log based 2: Use a deBrujin sequence (a cyclic 0-1 sequence s.t. each of the 2^k 0-1 strings of length k occurs exactly once as a substring of s). A multiplication and then a table loop-up is needed.  
- count the number of 1 in a word x: 
    - `for (r=0; x!=0; ++r) x &= x - 1; ` (works for sparse x)
    - Table loop-up
    - Parallel divide-and-conquer: use masks to parallel compute the number of 1's in consecutive 2,4,8,... bits. Remember to take care of overflowing. 
    - **Note** this is a builtin in modern machines, so don't write it yourself. 

**8 queens problem**
Representation: 
A new queen is safe if: 
For bit array `down` (vertical lines): ...
For bit array `left` (diagonal): `left & (1 << (r+c))`
For bit array `right` (diagonal): `right & (1 << (n-r+c))`

**Predictable branch**
Making branches predictable enables the program to read instructions ahead of time and runs faster. Here, *predictable* means that the branch condition has a certain predictable pattern (e.g. always true, or TF alternating). During execution, the processor predicts the outcome of the condition, and whenever a mispredicted branch is taken, all corresponding operations in the processor pipeline are invalid, leading to large delay. 

## x86_64 Architecture and Assembly
**Assembly Notation**
For a 64-bit register `a` in x86-64 architecture, `%rax`, `%eax`, `%ax`, `%al` refers to its 63-32, 31-16, 15-8, 7-0 bits, respectively. 

Suffix is added to opcodes to indicate the data type. 
For this lecture, `OP A, B, C` means `C <- A OP B`. `C` is omitted if `B` equals `C`.

Note that results of 32-bit operations are implicitly zero-extended to 64-bit values (even for negative numbers). 

**Memory Notation**
`%rax`      value in rax
0x172       contents at memory location 172 hex
$172        value 172
(%rax)      data at addr in reg (like rax is a pointer)
172(%rax)   value 172+%rax  (usually an address)
172(%rdi,%rdx,8)    value 172+8*%rdx+%rdi   (usually an address)


### Hazard: prevent further intructions running
#### Data Hazard
Two instructions depending on the same variable can't be run at once. 

**Read After Write (RAW)**: Read a register right after writing to it. Unrolling can be used to delay the read operation, thus eliminate the hazard (as long as the number of instructions in between is longer than pipeline length).

**Two Name Dependence Cases**:
1. Write After Read (WAR) (anti-dependence)
2. Write After Write (WAW) (output dependence)

Name dependence can be avoided if the name used in instructions is changed so that no such conflict arises. 

#### Control Hazard
##### Superscalar Execution
**Instruction Level Parallelism**
Instructions are run at the same time (pipeline). 

##### Vector Hardward**
**Single-instruction stream, multiple data (SIMD)**
Vector instructions operates in an elementwise fashion: i-th element of one vector register can only take part in operations with the i-th element of other vector registers. 

Vectorizable loops: the loop bounds must be known at time and execution, no complex logic (no branch), everything has the same index. 
e.g. 
`for (int i = 0; i < n; ++i) A[i] = B[i] * C[i];` is vectorizable
`for (int i = 0; i < n; ++i) A[i] = B[i-1] * C[i];` is unvectorizable

Clang can vectorize loop 
```C
#pragma clang loop vectorize(enable)
interleave(enable) interleave_count(2)
```

Speedup: 
1. The system vectorization registers bit width is fixed - the shorter the input bit width is, the more iterations can be run in parallel, the higher the speedup is. 
2. Some operators are not supported for vectorization, e.g. multiplication of *uint64_t* (it's supported for uint32_t). In this case attempts to vectorization leads to worse performance. 

**Strip Mining**
e.g. 
```C
// unvectorizable
for (int i = 0; i < n; ++i) {
    sum += A[i];
}

// vectorizing the loop: Create an array of m, setting B[k] = sum of A[j] for j % m = k. Then sum up elements of B. 
```
Note: alignment can cause problem for strip mining. 

#### Branch Predictions

### Memory System
Use locality: 
Temporal locality (i.e. in time)
Spatial locality

#### Cache Issues
Cold miss, capacity miss, conflict miss, sharing miss (true/false) (e.g. locks)


## Multicore Programming
Multicore is introduced due to limitation on increasing clock cycles (heat and power problem).

#### Cache consistence
The cache of processors need to maintain **cache consistence**
**MSI Protocol**
M: cache block has been modified. No other caches contain this block in M or S state. 
S: other caches may be sharing this block. 
I: cache block is invalid (same as missing).

For modification: make all other copies invalid (I state), modify the value and set it in M state. 

## Race conditions
**Determinacy race**: two logically parallel instructions access the same memory location and at least one of them performs a write. 

Type of races: none, read race (one write, one read), write race (two writes)

**Avoiding races**: 
1. iterations of `cilk_for` should be independent. 
2. Between a `cilk_spawn` and the corresponding `cilk_sync`, the code of the spawned child should be independent of the code of the parent.
    - Note that the argument to a spawned function are evaluated in the parent before teh spawn occurs. 

**Cilksan race detector**
The Cilksan-instrumented program is produced by compiling with `-fsanitize=cilk` flag.

**Performance Measuring**
Let T_p be the execution time on p processors
T_1 = total work
T_inf = span (longest path)
Work Law : T_p >= T_1/p
Span Law : T_p >= T_inf

Series composition: 
Work: T_1(AUB) = T_1(A) + T_1(B)
Span: T_inf(AUB) = T_inf(A) + T_inf(B)

Parallel composition: 
Work: T_1(AUB) = T_1(A) + T_1(B)
Span: T_inf(AUB) = max(T_inf(A), T_inf(B))

Speedup: T_1/T_p is speedup on p processors
Speedup <= p

**Cilkscale scalability analyzer**
Cilkscale computes work and span to give upper bound on parallel speedup. 

## Scheduling theory
Theorem: any greedy scheduler achieves T_p <= T_1/p + T_inf
Corollary: Any greedy scheduler achieves within a factor of 2 of optimal
Corollary: Any greedy scheduler achieves near-perfect linear speedup whenever T_1/T_inf ~ p

## From C to Assembly
Example of low level bug: corrent with `-O0` and broken with `-O3`

**Four states of compilation**
source ->(-E) preprocessed source ->(-S) assembly ->(-o) object file ->(-) linking multiple files

**Disassembling**
Use `objdump -S <program>` to get assembly code (has to be executable with debug symbols (`-g`))

**x86-64 calling convention**
`%rbp` and `%rsp` points to the base and top of the current stack frame
`%rip` points to the current instruction address

For function call: 
Whenever a function is called, arguments are pushed into the stack, then the return address (i.e. inst addr), then the base pointer
Note that callee needs to push all registers they use, and then restore them when the call ended. 

`%rax` is usually the return value


## Analysis of multithread programs
#### Master Theorem
For T(n) = a T(n/b) + f(n):
1. f(n) = O(n^{log_b(a) - eps}), for some eps > 0
   => T(n) = O(n^{log_b(a)})
2. f(n) = \Theta(n^{log_b(a)} lg^k(n)) for some constant k >= 0:
   => T(n) = O(n^{log_b(a)} lg^{k+1}(n))
3. f(n) = \Omega(n^{log_b(a) + eps}), for some eps > 0, and that af(n/b) <= kf(n), for some constant k < 1
   => T(n) = \Theta(f(n))   

#### Execution of `cilk_for`
`cilk_for` spawns a tree of threads, with the leaf nodes being the actual work. So for loop of n, the overhead is O(lg(n))                                                

## Measuring Performance
#### Time functions
/usr/bin/time measures times :
 - real: wall clock time
 - User: amount of CPU time in user mode
 - Sys: amount of CPU time in kernel within the process

#### Eliminate Interference
- Be the only user
- Don't have background jobs
- avoid hyperthreading: Hyperthreading make programs using unused slots in the core, making several programs sharing the same core. This improves throughput, but damages performance measurement (due to L1/L2 cache sharing).

## Storage Allocation
#### Stack allocation
```C
// allocation
sp += x;
return sp - x;

// free (only at the end)
sp -= x;
```
Should check for stack overflow & underflow

#### Fixed size heap allocation
C: `malloc`, `free`
C++: `new`, `delete`

Possible problem: memory leak (allocated memory not freed), double freeing, dangling pointers

**Free List**
Keep a blockwise list of free block addresses in a linked list. 
```C
// Allocate 1 object
x = free;
free = free->next;
return x;

// free 
x->next = free;
free = x;
```
Problem with this method: poor spatial locality due to external fragmentation
Always allocate on the fullest page to avoid increasing page lookup caching problem. 

####  Variable size Allocation
We create a binned free list, with the k-th bin containing list of memories of size 2^k. 
If bin is not large enough, find a block in hte next larbger nonempty bin, split it into arrays of blocks of size power of 2 and distribute pieces. 

**Analysis of Binned free list**
Theorem: Suppose that the max amount of heap memory in use at any time is `M`. If the heap is managed by a BFL allocator, the amount of virtual memory consumed by the heap is O(Mlg(M)). 

Coalescing: splicing together adjacent small blocks into a larger block. 

#### Garbage collector
Identifies and recycles objects that the program no longer access. 

**Terms**
Roots: objects directly accessible 
Live objects: reachable from the roots by following pointers
Dead objects: inaccessible and can be recycled

Problem: objects in a cycle never have reference count of 0. 

##### Stop-and-Copy Garbage Collection
run BFS to identify objects to recycle. We call the original memory space a FROM space, and call the new memory space a TO space (also used as a queue, with head and tail pointers pointing to addresses in the TO space). When the FROM space is full, the BFS is run, and we copy visited objects from the FROM space to the TO space, thus forming a consecutive memory space. 

**Steps**
 - When an object is copied to the TO space, store a forwarding pointer in the FROM object, which implicitly marks it as moved
 - When an object is removed from the queue in the TO space, update all pointers:
    - First enqueue all its adjacent vertices and place forwarding pointers in FROM vertices
    - Update all pointers of the dequeued object, pointing them to objects in the TO space

Linear time is used for each iteration. 

After the BFS, the FROM space can be recycled, and we rename our TO space to be the new FROM space. 

## Memory Allocators
user footprint U: maximum number of bytes used over time
allocator virtual footprint A: maximum number of bytes used of virtual memory provided to the allocator
virtual fragmentation: F=A/U

physical footprint P: maximum number of bytes of physical memory consumed by memory allocated by the allocator
physical framentation: P/U

`mmap` allocates the virtual memory space. The memory and the page table is not allocated until it is used. 


**Fragmentations**
Space overhead: space used by the allocator for bookkeeping
Internal fragmentation: waste due to allocating larger blocks than requested
External fragmentation: waste due to inability to use storage because it is not contiguous (i.e. freed memory)
Blowup: additional waste for parallel allocator compared to serial version. 

false sharing: two different objects are saved in the same cacheline, resulting in two threads competing for lock of the cacheline. 

OS page size is 4kB, so allocating as much virtual memory as you like, the actual physical memory is only a multiple of 4kB. 




## What compilers can and cannot do
#### Basic optimization
**Inlining**: replace a function call with a copy of the function body. Improves code locality, saves cost of function call, but increases the code size. 

**Constant Propopagation**: replace a variable with the constant, if known. Also enables other algebraic simplification. 

**Algebraic simplification**: If an expression can be calculated or simplified at compile time, do it. May enable other optimization, e.g. dead code elimination. Note that "simpler" is hardware-dependent. 

e.g. `a * 4` -> `a << 2`
e.g. Multiplication by shift-and-add
e.g. Division by Multiplication

**Common subexpression elimination**: save the result of an expression if it's calculated and used multiple times. 

**Dead code elimination**: as its name suggests. Reduced run time, code size, register pressure and enable other optimization. 

#### Advanced optimization
**Loop invariant hoisting**: calculate repeated expressions used in the loop, like common subexpression elimination. 

**Loop unrolling**: replicate the loop body to decrease loop controp overhead, but increase the code size. 

**Loop interchange**: change nested loop order 

**Basic vectorization**: for loop vectorization. Even interleaved data can be vectorized.

**Masking/If conversion**: vector code cannot have branch, so some of the results are discared using a mask. 

## Compiler restriction
**Alias analysis**: compiler is conservative and have to know if argument pointers have any overlapping. A `restrict` keyword guarantees no aliasing. 

**Dataflow analysis**: 
**Whole program optimization**: defer mid-end and back-end optimization to link time for greater scope. A.k.a. link time code generation. 

**Profile-guided optimization**: compile with profile collecting information and do optimization.

e.g. knowing the branch probability, can put most useful branches together in code. 

#### Back-end
**Instruction selection**

**Instruction scheduling**

**Register Allocation**

**Software pipelining**

## Algo choice in sorting
The compiler choose different sorting algorithm at different cutoff, suboptimal for now. 

options: radix sort, merge sort, quick sort, insertion sort

## DRAM and Managed Language Performance
##### DRAM organization
36 bits are used for address, where the 21-35 (higher 15 bits) are used for physical page, and the 0-20 (last 21 bits) are used for virtual page offset. The last 6 pages refer to the same cache line (64 bytes). 

It's cheap to switch between read and write for different rows in the same memory bank (can change back to active instead of going through the full cycle)

For write, an entire cacheline is read, changed and written back. If the entire cacheline is overwritten, no read is necessary. 

**Tips**
Use struct of arrays instead of array of structs, for better spacial locality. 

#### Managed language
##### Python
Python interpreter compiles python source code to bytecode (just like compiler compiles C code to assembly), then the bytecode is executed. 

Save repeatedly called functions as local variables for faster calling (like a cache). 

Speed: Default functions (map, filter, reduce) > [func(x) for x in list] > for loop, due to C implementation used instead of bytecode execution. 

#### Java and JIT
**On Stack Replacement (OSR)**: instead of waiting for slow code to finish, compile an optimized stub and jump to it. 

**Monomorphic Interface Call**: Interface are usually not optimized, but when only one version of the virtual function is used, the virtual function can be inlined. If multiple are called, the most commonly used one is inlined. 

**Polymorphic Inline Caching (PIC)**: Devirtualize and inline multiple most common implementations seen at a particular call site. 

**TODO: understand it...**

## Caching and Cache-efficient Algorithm
L3 (LLC) cache is shared by all processors. (~30Mb) 
L2 cache is assigned for each processors. (~256 kB)
L1 cache is divided into data cache and instruction cache (each ~32kB), assigned for each processor. 

#### Cache models
Fully associative cache: each data is saved together with a tag. To find a block, a thorough search for the tag in the cache is used. 

Directed-Mapped cache: Every memory location has a unique line used in the cache (usually just ignore the lower bits and use high order bits as tag). To find a line, just check if it's in the corresponding line in cache.  

Set-associative cache: Every memory location has a range of lines to use in cache. A k-way associative cache means that each location has k lines possible to use. 
For cache size M=32kB, line (block) size B, address space w-bit 
address bit meaning: (start from higher bits)
tag length: w - lg(M/k)
set: lg(M/kB)
offset: lg(B)

#### Cache misses
**Cold miss**: the first time the cache is accessed
**Capacity miss**: the previous cache would have been evicted even if it's an associative cache
**Conflict miss**: Too many blocks from the same set in cache
**Sharing miss**: another process acquires exclusive access to the cache block

**LRU Lemma** for an algorithm that incurs Q cache misses on an ideal cache of size M, it incurs at most 2Q cache misses on a fully associative cache of size 2M that uses LRU replacement policy. 

**Cache-miss lemma**: if a program reads a set of r data segments, where the i-th segment consists of s_i bytes, and suppose that the sum of s_i (denoted by N) is smaller than M/3, and N/r >= B, then all segments fit into cache, and the number of misses to read them all is at most 3N/B. 

**Tall cache assumption**: B^2 < cM for some sufficiently small constant c<1. 
The problem with short cache is that for smaller number of cache lines with large cache line size, sometimes the data cannot fit into the cache even when the total number of bytes is smaller than the cache size (since a lot of space are wasted due to long cacheline. )

**Submatrix caching lemma**: Suppose that an n*n submatrix A is read into a tall cache satisfying B^2<=cM, where c<=1 is constant, and suppose that cM<=n^2<=M/3. Then A fits into the cache, and the number of misses to read all A's elements is at most 3n^2/B (follows from Cache-miss lemma). 

#### Matrix multiplication cache analysis
**For brute force multiplication using loops**: 

Tiling: for matrix multiplication for smaller submatrices so that each submatrix is smaller enough to fit into the cache and decrease cache misses. 

The total number of cache misses is given my O(n^3/BM^(1/2))
 (number of misses without tiling is O(n^3) for large matrix). 

For two-level cache, use two level of tiling. 

**For divide-and-conquer implementation**:

Tiling is already used in the D&C implementation. 

The algorithm does not require explicit knowledge of caches, and is therefore passively autotuned. This is called **Cache-oblivious algorithm**.  

## Cache-Oblivious Algorithms
Theorem: Let Q_p be the number of cache misses in a deterministic Cilk computation when run on P processors, each with a private cache, and let S_p be the number of successful steals during the computation, then in the ideal-cache model, we have Q_p = Q_1 + O(S_p M/B), where M is the cache size and B is the size of a cache block (usually the cache line).

Lack of memory bandwidth only happens in parallel computation. 

## Nondeterministic Parallel Programming
**Deterministic**: a program is deterministic on a given input if every memory location is updated with the same sequence of values in every execution. 

**Note**: Deterministic programming does not prohibit reversing memory updating order of different memory locations. 

#### Mutual Exclusion and Atomicity
**Atomicity**: a sequence of instructions is atomic if the rest of the system cannot ever view them as partially executed. At any moment, either NO or ALL instructions have executed. 

**Critical section**: a piece of code that accesses a shared data structure that must not be accessed by two or more threads at the same time. 

**Mutex**: an object with lock and unlock member functions. An attempt to lock an already locked mutex causes the thread to block (wait) until the mutex is unlocked. 

**Determinacy race**: two logically parallel instructions access the same memory location and at least one of them is a write. No determinacy race means the program is deterministic (on that input).

**Data race**: two logically parallel instructions *holding no locks in common* access the same memory location and at least one of them is a write. **Note** that data-race-free programs may be nondeterministic, since acquiring the lock is a determinacy race. 

##### Mutex properties
1. Yielding: returns control to the OS when blocked
   Spinning: consumes processor cycles while blocked
2. Reentrant: allows thread holding the lock to acquire it again
   Nonreentrant: may deadlock if the thread attempts to reacquire a mutex it already holds. 
3. Fair: puts blocked threads on a FIFO queue and unblocks thread that's been waiting the longest. 
   Unfair: any blocked thread may go

Competitive mutex: spin for a while, then yield. The time spinning should be equal to a context switch. 

##### Deadlock 
**Conditions**:
1. Mutual exclusion: each thread claims exclusive control over resources it holds
2. Nonpreemption: each thread does not release the resources it holds until it completes its use of them
3. Circular waiting: a cycle of threads, each waiting for the next one in the cycle

**Theorem**: locking mutexes in order can prevent deadlock. 

**Note**: holding mutexes across `cilk_sync` can cause deadlocking. 

##### Locking Anomaly
**convoying**: multiple threads of equal priority contend repeatedly for the same lock. This is not a deadlock, and usually happens during *steal*. The problem is that each unsuccessful lock acquiring requires giving up the remainder of the scheduling quantum and forcing a context switch, which is quite expensive. 

Solution: use nonblocking acquire, and randomly acquire again in case of failure. 

**Contention**: thread is blocking and cannot proceed, thus making the program essentially operating on only one thread. 

Greedy scheduler time: T_p <= T_1/p + T_infty + B, where B is the total time of all critical sections

#### Transactional Memory
**commit**: all memory updates in the critical region appear to take effect at once. 
**abort**: none of the memory update appear to take effect, and the transaction must be restarted. 

##### Definitions
**Conflict**: two or more transactions attemp to access the same location of transactional memory concurrently

**Contention resolution**: deciding how to resolve conflict

**Forward progress**: avoiding deadlock, livelock and starvation

**Livelock**: Two threads waiting for each other, but not blocked

**Starvation**: Thread cannot get the resource 

**Throughput**: Run as many transaction concurrently as possible

##### Main ideas
**Finite Ownership Array**: support `acquire`, `try_acquire` and `release` operations, and an owner function h that maps memory location to corresponding lock in the lock array. 

**Release-Sort-Reacquire**: *try to acquire* (nonblocking) the lock greedily before access the memory x. On conflict: 
1. Roll back the transaction without releasing locks already held
2. Release all locks with indexes larger than h[x]
3. Acquire lock[h[x]], blocking if already held
4. Reacquire released locks in sorted order, blocking if already held
5. Restart the transaction


## Synchronization without Locks
sequential consistency: the results of any execution is the same as if the operations of all processors were executed in some sequential order. 

##### Peterson's algorithm for mutual exclusion
Two processors each writes to a shared variable that determines the one that has the right to use the protected variable. The last one that writes to this shared variable spins (waits).

Theorem: Peterson's algorithm guarantees starvation freedom: while one thread wants to execute its critical section, the other thread cannot execute its critical section twice in a row. 

**Note**: this algorithm requires that the system supports sequential consistency. Modern hardware doesn't support it, for performance issues (instruction-level parallelism saves time)

##### Total store ordering
**Hardware reordering**: a LOAD can bypass a STORE to a *different* address, since STORE stores in a store buffer, and LOAD takes priority (since it's slow).

The only possible reordering here is a LOAD and a previous STORE, both of which must have different address. LOAD's and STORE's are not reordered with LOCK instructions. 

Total store ordering (TSO) is weaker than sequential consistency. 

**Memory fence**: a hardware action that enforces an ordering constraint between instructions before and after the fence. Call to memory fence is comparable to that of L2-cache access. j

To maintain memory fence, compiler fences are required to prevent compiler reordering between memory fence and critical section. 

**Theorem**: any n-thread deadlock-free mutual-exclusion algorithm using only LOAD and STORE memory operations requires Omega(n) space. 

**Theorem**: any n-thread deadlock-free mutual-exclusion algorithm on a modern machine must use an expensive operation such as a memory fence or an atomic compare-and-swap operation. 

##### Compare-and-swap (CAS)
The operation is executed atomically. A variable x is replaced by a new value x2 if it's equal to some value x1. 

**Theorem**: an n-thread deadlock-free mutual-exclusion algorithm using CAS can be implemented using Theta(1) space. (run a while loop with CAS to change the mutex)

This algorithm is non-blocking. 

##### ABA problem
For a lock-free stack, if one pop is stalled after reading current->next, another thread comes in, pops several nodes and reuse them, garbage can be put in the stack. 

Solution: add a version number for each node, or prevent reuse. 

## Graph Optimization
Representation: 
1. Adjacency matrix O(V^2)
2. Edge list O(E)
3. Adjacency list O(V+E)
   Faster lookup compared to edge list, more expensive to update graph
4. Compressed sparse row (CSR) O(V+E)
   Offset list + edge list, with the offset list providing fast lookup in edge list 

Use bitvector to represent visited nodes: no preformance improvement when edge-to-vertex ratio is small (the number of cache misses saved is smaller than the time to maintain the bitvectors)

parallelized BFS: in each iteration, visit all nodes in the frontier in parallel, add their neighbors to the next frontier, then filter out those visited before from the next frontier list. 
    span = O(DlogE), D is the diameter
    Work = O(V+E)

Direction-optimizing BFS: considering the amount of time used for filtering, the top-down parallelized BFS is slow when used on graphs with large frontier. In this case, we can instead consider bottom-up BFS, i.e. start with all bottom level nodes. 

#####Graph compression on CSR
use differential edge representation instead of node index. 

For each vertex v: 
    first edge: difference is Edges[Offsets[v]]-v
    i'th edge: difference is Edges[Offsets[v]+i]-Edges[Offsets[v]+i-1]

Variable-length codes for large difference: for number longer than 7 bits: cut into 7-bit blocks, add "continue" bit as the first bit, forming list of bytes

Run-length encoding: each group of numbers (numbers that need the same number of bits) have a header indicating number of bytes per integer and size of group

Optimization for high-degree vertex: split the edges into chunks, and set the first number of each chunk to be relative to source vertex. This way these chunks can be decoded in parallel.

##### Graph reordering
reassign IDs to vertices to improve locality. 





Misc
---------------------
 - `sqrt` is expensive
 - `mod` is expensive, unless working with constants (then compiler may come up with special tricks)
 - Use `restrict` to tell the compiler that the content of a pointer is only referred to using that pointer (thus enabling some parallization without concern that content of an array is changed by other operations concerning other pointers)
