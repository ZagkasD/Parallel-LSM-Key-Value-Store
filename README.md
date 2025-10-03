# Parallel-LSM-Key-Value-Store

A **Key-Value Store** engine developed as a university project for the Operating Systems course. This implementation leverages the **Log-Structured Merge-Tree (LSM-Tree)** data structure and features **multithreading and parallelism** to significantly boost read performance.

***

## Overview and Core Architecture

The project implements a structured storage layer designed for fast writes and reads by maintaining sorted data both in memory and on disk.

### The LSM-Tree Components

1.  **Memtable (In-Memory)**
    * New key-value pairs are first inserted here using the `db_add()` operation.
    * It is backed by a **log file** on disk to ensure data persistence and crash recovery.
    * Once full, the Memtable contents are merged to disk and stored in a new SST file.

2.  **SST (Sorted String Table - On-Disk)**
    * This is the persistent storage layer, organized into multiple levels (Level0 to LevelN).
    * A **compaction** process is triggered when levels become too large, merging and moving files to the next level to maintain efficiency.

### Read Operation Flow

A key retrieval (`db_get()`) always checks the **Memtable** first. If the key is not found, the search then proceeds to traverse the **SST** levels.

***

## Parallelism and Concurrency

The core innovation of this project is the introduction of parallelism to optimize resource utilization, particularly for concurrent requests.

### 1. Concurrent Search (`db_get()`)

The search mechanism is parallelized using **separate mutexes** for the primary structures:

* A lock for accessing the **Memtable**.
* A separate lock for accessing the **SST**.

This allows the system to handle concurrent searches efficiently: while one thread is searching the Memtable, another thread (which has already finished its Memtable check) can simultaneously be searching the SST.

### 2. Dynamic Thread Allocation

* Multithreading is used to parallelize testing functions (`_read_test` and `_write_test`) to simulate a concurrent environment.
* Threads are dynamically allocated to the read and write pools based on the ratio of requests. For example, if there are 90% writes and 10% reads, more threads are assigned to the write pool to optimize execution time.

***

## Performance Benchmarks

Performance was measured using 600,000 read operations and 600,000 write operations, focusing on **Total Response Time** and **Throughput ($R$ in bits/sec)**.

The results demonstrated a significant performance gain up to 64 threads:

| Threads | Total Time (sec) | Throughput (bit/sec) |
| :---: | :---: | :---: |
| 2 | 19.000 | 2,021,052 |
| 8 | 12.000 | 3,200,000 |
| **64** | **10.000** | **3,840,000** |
| 128 | 10.000 | 3,840,000 |

> **Conclusion:** The optimal performance was achieved at **64 threads**, yielding a peak throughput of **3,840,000 bit/sec** and the minimum total cost of **10 seconds**.
