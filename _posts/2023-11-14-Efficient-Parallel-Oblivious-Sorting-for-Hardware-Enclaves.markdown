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

1. Secure Memory Limitations: The constrained size of Intel SGX's secure memory, especially in earlier versions, necessitates a page swap mechanism between secure and insecure memory when sorting data surpasses this limit. This involves resource-intensive cryptographic operations. Thus, our challenge is not only to parallelize the algorithm but also to optimize this page swap mechanism.

2. Handling Large Datasets: When the data size exceeds both secure and physical memory, additional page swaps with the disk become necessary. This presents both opportunities and challenges in effectively overlapping computation and I/O operations.

3. Memory Overhead with OpenMP: The use of tools like OpenMP may introduce extra memory overhead. To ensure optimal performance, we may need to implement our scheduling mechanisms rather than relying on existing tools.

4. Thread Management: Pre-declaring the number of threads before launching the Intel SGX enclave, coupled with each thread consuming non-negligible stack space, requires careful maintenance of a thread pool.

5. Memory Fragmentation: Multi-threading introduces the potential for memory fragmentation, necessitating careful consideration during memory allocation.

6. Syscall Limitations: Intel SGX prohibits syscalls in the secure environment, requiring a context switch before accessing the disk. As a result, batch operations are necessary to amortize the cost of context switches, complicating the parallel page swap mechanism.

7. Compiler Optimization Challenges: Compiler optimizations may inadvertently compromise the obliviousness of the algorithm. To mitigate this, we must implement SIMD optimizations directly using C++ intrinsics. This ensures that the algorithm remains oblivious while achieving the desired performance enhancements.


### RESOURCES

The algorithm by Gu et.al. is already made [open-source](https://github.com/odslib/oblsort), and we will use this repository as out codebase in this project.

We already have access to a server with support to Intel SGX, the specific hardware enclave we choose to use in this project. The server features 36 physical cores and have 1TB RAM and multiple TBs of disk space. For now, we don't need any other resources.



### GOALS AND DELIVERABLES

In general, our deliverables will be a combination of a software deliverable, a written report and a poster.

#### Software Deliverable

Our software implementation aims to parallelize the existing oblivious sorting algorithm on Intel SGX, enhancing its performance while preserving the oblivious property. Although we do not currently specify a precise speedup goal, we anticipate achieving at least a 10x improvement on our 36-core machine.

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


### SCHEDULE

|           Week           |                            Tasks                             |
| :----------------------: | :----------------------------------------------------------: |
| 11/12/2023 -- 11/18/2023 | Write the project proposal, set up the environment and get familiar with the code base. |
| 11/19/2023 -- 11/25/2023 | Implement the parallelism of MergeSplit butterfly network in specific hardware, Intel SGX. |
| 11/26/2023 -- 12/02/2023 | Enhance the parallelism using SIMD, optimize the memory efficiency. |
| 12/03/2023 -- 12/09/2023 | Implement the multithreading I/O and data prefetching between the Enclave and OS. |
| 12/10/2023 -- 12/16/2023 |         Write the final report and make the poster.          |

