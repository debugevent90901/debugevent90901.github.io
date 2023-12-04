---
layout: post
title:  "Efficient Parallel Oblivious Sorting for Hardware Enclaves"
date:   2023-11-14 20:42:34 -0500
categories: project proposal
---
Team members: Tian Xie, Tianyao Gu

### SUMMARY

We are going to implement a parallel oblivious sorting algorithm in C++ using Intel SGX, which provably resists side-channel attacks. We will leverage multithreading, SIMD, as well as techniques such as prefetching and write back buffer to overlap computation and communication.



### BACKGROUND

Our motivation is to securely outsource data and computation to an untrusted worker equipped with secure processors such as Intel SGX. Although secure processors ensure data privacy through encryption, existing literature reveals that attackers can exploit memory access and page swap patterns during computations to glean information about the data.

Oblivious algorithms provide a verifiable shield to counter such side-channel attacks. This is because "Obliviousness" essentially demands that memory access and page swap patterns remain independent of secret data. In addition, sorting stands out as a crucial primitive in oblivious computation.

Our project is based on a prior [research paper](https://eprint.iacr.org/2023/1258) by Gu et.al. This paper proposed and implemented an efficient oblivious sorting in hardware enclaves, which achieves asymptotically optimal runtime. However, the algorithm, as presented and implemented, is single-threaded, posing limitations on its performance.

In our continuation of this work, we aim to enhance performance by parallelizing the oblivious sorting algorithm. As a byproduct, we will also obtain a parallel oblivious random shuffling algorithm.


### THE CHALLENGE

A worth-noting property of Intel SGX is its limited secure memory (especially in its earlier versions), which gives rise to most major challenges of our project listed below:

Several noteworthy challenges stem from the limited secure memory of Intel SGX, particularly in its earlier versions. These challenges, integral to our project, are outlined below:

1. Insufficient parallelism：For small data input, the algorithm may not have enough independent tasks to complete the sorting. We need to adjust the parameters of the algorithm (e.g., the bucket size and the number of mergesplit ways on each layer) to ensure enough speed up at high CPU count. We also need to parallelize some building blocks of the algorithm, such as the pseudo-random generator.

2. Task scheduling: The original implementation of the algorithm uses recursion to run both the batch-wise and in-batch tasks. While the impelmentation improves locality for single threaded programs, for multi-threaded programs running with openmp, recursion typically does not provide as much speedup as parallel loops.

3. Synchronization: Although most of the tasks are independent and just need to be synchronized at the end of each stage, we still need to perform some more advanced synchronization for reading the input and combining the output of the algorithm.

4. Secure Memory Limitations: The constrained size of Intel SGX's secure memory, especially in earlier versions, necessitates a page swap mechanism between secure and insecure memory when sorting data surpasses this limit. This involves resource-intensive cryptographic operations. Thus, our challenge is not only to parallelize the algorithm but also to optimize this page swap mechanism.

5. Handling Large Datasets: When the data size exceeds both secure and physical memory, additional page swaps with the disk become necessary. This presents both opportunities and challenges in effectively overlapping computation and I/O operations.

6. Memory Overhead with OpenMP: The use of tools like OpenMP may introduce extra memory overhead. To ensure optimal performance, we may need to implement our scheduling mechanisms rather than relying on existing tools.

7. Thread Management: We need to pre-declaring the number of threads before launching the Intel SGX enclave. And since each thread has a memory overhead, and each task requires some temporary memory, we need to choose the number of threads wisely.

8. Memory Fragmentation: Multi-threading introduces the potential for memory fragmentation, necessitating careful consideration during memory allocation.

9.  Syscall Limitations: Intel SGX prohibits syscalls in the secure environment, requiring a context switch before accessing the disk. As a result, batch operations are necessary to amortize the cost of context switches, complicating the parallel page swap mechanism.

10. Compiler Optimization Challenges: Compiler optimizations may inadvertently compromise the obliviousness of the algorithm. To mitigate this, we must implement SIMD optimizations directly using C++ intrinsics. This ensures that the algorithm remains oblivious while achieving the desired performance enhancements.


### RESOURCES

The algorithm by Gu et.al. is already made [open-source](https://github.com/odslib/oblsort), and we will use this repository as out codebase in this project.

We already have access to a server with support to Intel SGX, the specific hardware enclave we choose to use in this project. The server features 36 physical cores and have 1TB RAM and multiple TBs of disk space. For now, we don't need any other resources.



### GOALS AND DELIVERABLES

In general, our deliverables will be a combination of a software deliverable, a written report and a poster.

#### Software Deliverable

Our software implementation aims to parallelize the existing oblivious sorting algorithm on Intel SGX, enhancing its performance while preserving the oblivious property. We anticipate achieving at least a 10x improvement with 32 threads.

We have outlined four goals for these deliverables, with the first two considered minimal objectives, the first three expected to be completed, and the last goal viewed as additional work contingent on project progress:

1. Parallelization of the Algorithm in Intel SGX:
    Parallelize the multi-way butterfly network as outlined in the paper. The fixed memory access pattern of the butterfly network and the independence of each multi-way "mergesplit" operation make this task achievable.

2. Enhancement of Parallelism Using SIMD:
    Improve parallelism by leveraging the AVX2 instruction set for oblivious compare-and-swap operations. Assuming each element contains a payload occupying multiple memory words, SIMD will be employed to accelerate both oblivious compare-and-swap and histogram counting operations.

3. Optimization of Memory Efficiency During Parallel Computing:
    Address challenges arising from SGX enclave's limited secure memory size by optimizing memory efficiency during parallelization. This step is crucial for mitigating potential issues associated with multi-threading in SGX enclave.

4.  Implementation of a General Framework for Multithreading I/O:
    Develop a versatile framework for multithreading I/O between the SGX enclave and the operating system. The intention is to extend the applicability of our work beyond sorting and shuffling algorithms. The goal is to create a broader page-swapping framework, enabling the parallel execution of various algorithms within the SGX enclave.

#### Poster Deliverable

The poster session is designed to highlight the performance improvements achieved through our implementation. Our presentation will feature speedup graphs and output comparisons, considering diverse loads (such as the number of elements and the size of each element) and system configurations (enclave size, memory size, with or without SIMD). Additionally, we will conduct a thorough analysis to identify bottlenecks influencing the speedup of our implementation. Due to the project's nature, an interactive demonstration won't be feasible during the poster session.


### PLATFORM CHOICE

Due to the absence of secure hardware enclave support on the PSC bridge-2 machines, we have opted to utilize another Linux server located on the 3rd floor of Gates, which is equipped with Intel SGX support. It's worth noting that while our implementation is tailored for Intel SGX, the algorithm design is intended to be compatible with common hardware enclave architectures.

In terms of programming language, we have selected C++ for its ability to deliver high performance and provide low-level control over the operating system, with finer granularities. Additionally, we are exploring the potential use of OpenMP in Intel SGX, although its feasibility is yet to be confirmed based on the specific hardware characteristics.

This platform choice aligns well with our project's focus on parallelizing oblivious sorting within hardware enclaves, with Intel SGX being a prevalent and rational selection.


### Milestone Progress
We have successfully parallelized the oblivious shuffling algorithm in Intel SGX using OpenMP and SIMD.
Specifically, we did the following:

1. Configured OpenMP in the Makefile and set up the multi-threaded environment in the SGX config files.

2. Changed the butterfly routing from recursion to iterative, which reduces the overhead of parallel execution. ALso, we combined multiple batches into one batch when there's enough memory to increase parallelism.

3. Improved the parameter solver to ensure that there are enough parallel tasks on each butterfly network level (by reducing bucket size and balancing the mergesplit ways on each level.)

4. Used #omp parallel for to parallelize the execution of the butterfly network.

5. Used #omp parallel for to parallelize I/O with external memory.

6. Developed a reader/writer pool manager to fetch input and combine the final output, reduces contention between the threads.

7. Separate a central pseudo-random number generator (which uses AES counter mode) into multiple ones to reduce contention.

8. Used C++ intrinsics to accelerate element-wise oblivious exchange using SIMD. Applied different instruction sets such as AVX512, AVX2, and SSE2 to make the code compatible on a wide range of processors.

9. Set up unit tests on the algorithm outside the enclave for debugging and tuning. Installed Intel vtune software.

The current speedup results are listed below

![result1](/assets/simd_shuffle.png)

![result2](/assets/speedup_shuffle.png)

#### Compare to previous goals
We have completed most of the goals 1, 2, 3, and prepare to work on task 4. The list of the goal is the same as in the proposal.

For task 1, we still need to parallelize the non-oblivious external Mergesort, which comes after the data have been obliviously shuffled. This part occupies about 1/4 of the running time in the single threaded implementation.

The task 2 is fully completed. We don't think we can utilize SIMD in other parts of the program.

Task 3 is almost completed. We have been trying to utilizing the enclave memory to the maximum extent to increase parallelism and reduce I/O. Nevertheless, there still seems to be some memory fragmentation issue, which results in imperfect utilization.

Task 4 was meant to be optional, and we haven't started yet.

### SCHEDULE

|           Week           |                            Tasks                             |
| :----------------------: | :----------------------------------------------------------: |
| 11/12/2023 -- 11/18/2023 | Write the project proposal, set up the environment and get familiar with the code base. |
| 11/19/2023 -- 11/25/2023 | Implement the parallelism of MergeSplit butterfly network in specific hardware, Intel SGX. |
| 11/26/2023 -- 12/02/2023 | Enhance the parallelism using SIMD, optimize the memory efficiency. |
| 12/03/2023 -- 12/09/2023 | Implement the multithreading I/O and data prefetching between the Enclave and OS. |
| 12/10/2023 -- 12/16/2023 |         Write the final report and make the poster.          |

