---
layout: post
title:  "Efficient Parallel Oblivious Sorting for Hardware Enclaves"
date:   2023-11-14 20:42:34 -0500
categories: project proposal
---
Team members: Tian Xie, Tianyao Gu

### SUMMARY

We are going to implement an optimized efficient oblivious sorting algorithm on Intel SGX with parallelism, where we will leverage the multithreading, SIMD and possibly OpenMP.



### BACKGROUND

Our motivation is to securely outsource data and computation to an untrusted worker equipped with a secure processor such as Intel SGX. Our goal is to safeguard the privacy of the data from both software attacks and physical attacks 



### THE CHALLENGE

The most worth-noting property of Intel SGX is that its memory is very limited, which gives rise to most major challanges of our project listed below:

1. The memory of Intel SGX is very limited so the entire butterfly network cannot fit inside the memory, so we will have to handle the memory page swaps between the Intel SGX and OS, which is expensive.
2. The Intel SGX iteself does not support syscalls, so it is very expensive when handling the file operations, such as open, read and write. Due to the large amount of data we are sorting, the file operations are common and need optimizing.
3. ...



### RESOURCES

Our project is based on a prior [research paper](https://eprint.iacr.org/2023/1258) of one of our teammate, Tanya Gu. This paper proposed an efficient oblivious sorting and shuffling algorithm in hardware enclaves. However, it is not parallelized and we decide to continue working on it to parallelize this algorithm. The algorithm is already implemented made [open-source](https://github.com/odslib/oblsort), and we will use this reposity as out codebase in this project.

We have already have access to a server with support to Intel SGX, the specific hardware enclave we choose to use in this project, so for now we don't need any other resources.



### GOALS AND DELIVERABLES

In general, our deliverables will be a combination of a software deliverable, a written report and a poster.

#### Software Deliverable

Our software implementation should ba capable of parallelize the exisiting implementation of the oblivious sorting algorithm on Intel SGX, which acclerates the execution time and improves the performance. For now, we cannot state a precise speedup goal, but we anticipate it to be obvious. 

To be more specific, we have 4 goals in this deliverables, the first 2 goals are the minimal goals when our work goes slow, the first 3 goals are expected to be done, and the last goal is considered as extra work when our project goes well:

1. Implement the parallelism of MergeSplit butterfly network in specific hardware, Intel SGX. 
2. Enhance the parallelism using SIMD
3. Optimize the memory efficiency
4. Implement the multithreading I/O and data prefetching between the Enclave and OS.

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

