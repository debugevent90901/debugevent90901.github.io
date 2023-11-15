---
layout: post
title:  "Efficient Parallel Oblivious Sorting for Hardware Enclaves"
date:   2023-11-14 20:42:34 -0500
categories: project proposal
---
Team members: Tian Xie, Tianyao Gu

### SUMMARY

We are going to implement a parallel oblivious sorting algorithm in C++ with Intel SGX, where we will leverage multithreading and SIMD.



### BACKGROUND

Our motivation is to securely outsource data and computation to an untrusted worker equipped with secure processors such as Intel SGX. While secure processors preserve data privacy using encryption, various literatures have demonstrated that attackers can exploit the memory access and page swap patterns during the computation to learn useful information about the data.

Oblivious algorithms offer a provable protection against such side-channel attacks, since ``obliviousness'' essentially requires that the memory access and page swap patterns are independent of the secret data. Sorting is one of the most commonly needed primitives in oblivious computation. 

Our project is based on a prior [research paper](https://eprint.iacr.org/2023/1258) by Gu et.al. This paper proposed and implemented an efficient oblivious sorting in hardware enclaves that achieves asymptotically optimal runtime. However, the algorithm is presented and implemented with a single-thread, which significantly limits its performance.

We decide to continue the work. by parallelizing the oblivious shuffling/sorting algorithm.


### THE CHALLENGE

A worth-noting property of Intel SGX is its limited secure memory (especially in its earlier versions), which gives rise to most major challenges of our project listed below:

1. When the amount of data to sort exceeds the size of the secure memory, we need a page swap mechanism between the secure and insecure memory, which involves expensive cryptographic operations. Therefore, we need to not only parallelize the algorithm itself but also this page swap mechanism. 
2. When the amount of data to sort also exceeds the size of the physical memory, we need to further perform page swaps with the disk. This offers us both opportunities and challenges to overlap computation and I/O.
3. Using tools such as openmp may introduce extra memory overhead. Therefore, we may need to implement the scheduling by ourselves for best performance.
4. The number of threads needs to be declared before the launch of the Intel SGX enclave, and each thread consumes some non-negligible amount of stack space. Therefore, we need to maintain some sort of thread pool.
5.  The Intel SGX does not allow making syscalls in the secure environment, so we need to perform a context switch before we can access the disk. Therefore, we need to utilize some batch operations to amortize the cost of context switches, which further increases the complexity of the aforementioned parallel page swap mechanism.
6. Optimization by the compiler might break the obliviousness of the algorithm. Therefore, we need to implement SIMD optimizations using C++ intrinsics directly.


### RESOURCES

 The algorithm by Gu et.al. is already made [open-source](https://github.com/odslib/oblsort), and we will use this repository as out codebase in this project.

We already have access to a server with support to Intel SGX, the specific hardware enclave we choose to use in this project. The server features 36 physical cores and have 1TB RAM and several terabytes of disk space. For now, we don't need any other resources.



### GOALS AND DELIVERABLES

In general, our deliverables will be a combination of a software deliverable, a written report and a poster.

#### Software Deliverable

Our software implementation should be capable of parallelize the existing implementation of the oblivious sorting algorithm on Intel SGX, which accelerates the execution time and improves the performance. For now, we cannot state a precise speedup goal, but we anticipate it to be obvious. 

To be more specific, we have 4 goals in this deliverables, the first 2 goals are the minimal goals when our work goes slow, the first 3 goals are expected to be done, and the last goal is considered as extra work when our project goes well:

1. Parallelize the algorithm in Intel SGX. Specifically, the multi-way butterfly network presented in the paper.
2. Enhance the parallelism using SIMD
3. Optimize memory efficiency during parallel computing
4. Implement a general framework for multithreading I/O between the SGX Enclave and OS.

#### Poster Deliverable

The poster session will showcase the performance enhancements achieved by our implementation. We will present speedup graphs and output comparisons under varying loads. However, the project's nature precludes an interactive demonstration.



### PLATFORM CHOICE

Since the PSC bridge-2 machines do not support secure hardware enclaves, we choose to work on another Linux server that supports Intel SGX. Although our implementation uses Intel SGX, the algorithm design should work for any common hardware enclave architecture.

For the programming language, we choose to use C++ with SIMD and multithreading. We will also explore the possibility of using OpenMP in Intel SGX, which we are not sure for now due to the hardware characteristics.

We think this platform choice makes sense as our project is targeted to parallelize the oblivious sorting in hardware enclaves, among which Intel SGX is a popular and reasonable choice. Also, C++ offers great performance and low-level control of the OS with smaller granularities.



### SCHEDULE

|           Week           |                            Tasks                             |
| :----------------------: | :----------------------------------------------------------: |
| 11/12/2023 -- 11/18/2023 | Write the project proposal, set up the environment and get familiar with the code base. |
| 11/19/2023 -- 11/25/2023 | Implement the parallelism of MergeSplit butterfly network in specific hardware, Intel SGX. |
| 11/26/2023 -- 12/02/2023 | Enhance the parallelism using SIMD, optimize the memory efficiency. |
| 12/03/2023 -- 12/09/2023 | Implement the multithreading I/O and data prefetching between the Enclave and OS. |
| 12/10/2023 -- 12/16/2023 |         Write the final report and make the poster.          |

