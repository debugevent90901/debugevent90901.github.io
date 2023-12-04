---
layout: post
title:  "Milestone Report"
date:   2023-12-03 20:42:34 -0500
categories: project milestone

---

Team members: Tian Xie, Tianyao Gu



## Schedule

|           Week           |                            Tasks                             |
| :----------------------: | :----------------------------------------------------------: |
| 12/03/2023 -- 12/06/2023 | Parallelization of the non-oblivious external Mergesort (Tian Xie) <br />Continue optimizating memory efficiency (Tianyao Gu) |
| 12/06/2023 -- 12/09/2023 | Implement multithreading I/O (Tian Xie) <br />Continue optimizating memory efficiency (Tianyao Gu) |
| 12/10/2023 -- 12/13/2023 | Continue implementing multithreading I/O (Tian Xie) <br />Continue optimizating memory efficiency (Tianyao Gu) |
| 12/13/2023 -- 12/16/2023 | Writing final report (Tian Xie) <br />Preparing for poster session (Tianyao Gu) |



## Progress so far

We have successfully parallelized the oblivious shuffling algorithm in Intel SGX using OpenMP and SIMD.
Specifically, our key accomplishments include:

1. Configured `OpenMP` in the Makefile and set up the multi-threaded environment in the SGX config files.

2. Changed the butterfly routing from recursion to iterative, which reduces the overhead of parallel execution. Also, we combined multiple batches into one batch when there's enough memory to increase parallelism.

3. Improved the parameter solver to ensure that there are enough parallel tasks on each butterfly network level (by reducing bucket size and balancing the mergesplit ways on each level.)

4. Applied `#omp parallel for` to parallelize the butterfly network executions.

5. Applied `#omp parallel for` to parallelize I/O operations with external memory.

6. Developed a reader/writer pool manager to fetch input and combine the final output, thereby reducing thread contention.

7. Separated a central pseudo-random number generator (which uses AES counter mode) to multiple ones to reduce contention.

8. Utilized C++ intrinsics to accelerate element-wise oblivious exchange using SIMD. Incorporated different instruction sets such as AVX512, AVX2, and SSE2 for broader processor compatibility.

9. Established unit tests on the algorithm outside the enclave for debugging and tuning. Installed Intel vtune software.



## Goals and Deliverables

Our proposal outlined four main goals:

1. Parallelization of the Algorithm in Intel SGX: Parallelize the multi-way butterfly network as outlined in the paper. The fixed memory access pattern of the butterfly network and the independence of each multi-way “mergesplit” operation make this task achievable.

   For this task, we still need to parallelize the non-oblivious external Mergesort, which comes after the data have been obliviously shuffled. This part occupies about 1/4 of the running time in the single threaded implementation.

2. Enhancement of Parallelism Using SIMD: Improve parallelism by leveraging the AVX2 instruction set for oblivious compare-and-swap operations. Assuming each element contains a payload occupying multiple memory words, SIMD will be employed to accelerate both oblivious compare-and-swap and histogram counting operations.

   This task is fully completed. We don't think we can utilize SIMD in other parts of the program.

3. Optimization of Memory Efficiency During Parallel Computing: Address challenges arising from SGX enclave’s limited secure memory size by optimizing memory efficiency during parallelization. This step is crucial for mitigating potential issues associated with multi-threading in SGX enclave.

   This task is almost completed. We have been trying to utilizing the enclave memory to the maximum extent to increase parallelism and reduce I/O. Nevertheless, there still seems to be some fragmentation problem, which results in imperfect utilization.

4. Implementation of a General Framework for Multithreading I/O: Develop a versatile framework for multithreading I/O between the SGX enclave and the operating system. The intention is to extend the applicability of our work beyond sorting and shuffling algorithms. The goal is to create a broader page-swapping framework, enabling the parallel execution of various algorithms within the SGX enclave.

   This was meant to be optional, and we haven't started yet.

   

We're confident in achieving goals 1-3 by the poster session. For goal 4, we're committed to making progress and showcasing it in our best efforts.

To conclude, we think we are doing well by the milestone and are able to produce all the deliverables. 



## Poster Session Plan

During our poster session, we plan to focus on:

1. Introducing the project's background, since it might be unfamiliar to many people.
2. Detailing the parallelization techniques used, including OpenMP, SIMD, memory optimization, and Multithreading I/O, etc.
3. Presenting speedup graphs and output comparisons under various workloads (e.g., the number of elements and the size of each element) and system configurations (e.g., enclave size, memory size, SIMD). 
4. Analyzing bottlenecks impacting our implementation's speedup. 

Due to the project's nature, an interactive demonstration won't be feasible during the poster session.



## Preliminary Results

Currently, we have two speedup graphs representing our preliminary results, which are listed below:

![result1](/parallel-oblsort/assets/simd_shuffle.png)

![result2](/parallel-oblsort/assets/speedup_shuffle.png)

