
# 🖥️ xv6 Scheduler Enhancement – Round Robin vs MLFQ

## Introduction

Modern operating systems rely on **process scheduling algorithms** to efficiently allocate CPU time among competing processes. The performance of CPU-bound and I/O-bound applications is heavily influenced by the scheduling policy.

The default **Round Robin (RR)** scheduler in xv6 treats all processes equally by assigning fixed time slices in a cyclic order. While fair, it can lead to suboptimal performance when CPU-intensive and I/O-intensive processes run concurrently.

This project implements an **improved scheduler, Multi-Level Feedback Queue (MLFQ)**, in xv6 and compares its performance against the default Round Robin scheduler. The aim is to understand how scheduler design impacts **process behavior, response time, and overall system throughput**.

---

## Objectives

* Set up and configure the **xv6 environment** using QEMU and the RISC-V toolchain.
* Study the existing scheduling mechanism (`proc.c`) and understand Round Robin scheduling.
* Implement the **MLFQ scheduler**, dynamically adjusting process priorities based on CPU usage.
* Create user-level programs simulating **CPU-bound** and **I/O-bound** workloads.
* Analyze and compare **completion time** and **CPU ticks** under both schedulers.
* Demonstrate measurable advantages of MLFQ over Round Robin in **mixed workloads**.

---

## Background and Scheduler Selection

### 1. Round Robin Scheduling (Baseline)

* Assigns **equal CPU time** to each process in a cyclic order.
* **Drawback:** CPU-bound processes can monopolize CPU cycles, leading to **poor responsiveness** for I/O-bound tasks.

### 2. Multi-Level Feedback Queue (MLFQ)

* Uses **multiple queues** with different priority levels:

  * I/O-bound processes (frequently yield) stay in **high-priority queues**.
  * CPU-bound processes are gradually moved to **lower-priority queues**.
  * **Priority boosting** prevents starvation.

**Hypothesis:** MLFQ will reduce turnaround time for I/O-bound processes while maintaining acceptable performance for CPU-bound ones.

---

## Implementation Details

### Files Modified

* `kernel/proc.c` – scheduling logic updated for **MLFQ queues**, promotion/demotion.
* `kernel/proc.h` – added **priority** and **CPU tick tracking** in `struct proc`.
* `kernel/sysproc.c`, `kernel/syscall.c`, `kernel/syscall.h`, `kernel/usys.S` – added new system call `getprocinfo()`.
* `user/getprocinfo_test.c` – user program to test process info retrieval.
* `user/cpubound.c` and `user/iobound.c` – CPU-bound and I/O-bound test programs.

### New System Call: `getprocinfo`

```c
int getprocinfo(int pid, struct procinfo *info);
```

**Purpose:** Returns process statistics such as:

* Total CPU ticks used
* Number of times scheduled

Used to **measure scheduler performance**.

### MLFQ Scheduler Logic

1. Each process starts in the **highest-priority queue** (queue 0).
2. If a process consumes its full time slice without yielding, it is **demoted** to a lower-priority queue.
3. If a process sleeps or performs I/O, it is **promoted** back up.
4. **Priority boost** occurs periodically to prevent starvation.

---

## Application Design

### CPU-Bound Program

* Computes all prime numbers up to 50,000.
* Rarely yields CPU.

### I/O-Bound Program

* Performs repeated small I/O operations (writing to a file, printing).
* Spends more time on I/O than computation.

---

## Testing and Performance Analysis

### Methodology

1. Run both programs under **Round Robin** and **MLFQ** schedulers individually.
2. Repeat experiments multiple times for consistency.
3. Record performance metrics using `getprocinfo`.

### Observations

| Scheduler   | Process   | CPU Ticks Used | Times Scheduled |
| ----------- | --------- | -------------- | --------------- |
| Round Robin | CPU-bound | 7              | 22              |
| Round Robin | I/O-bound | 31             | 667             |
| MLFQ        | CPU-bound | 7              | 16              |
| MLFQ        | I/O-bound | 27             | 634             |

---

## Graphical Analysis

* **CPU-bound CPU Ticks:** Both RR and MLFQ used 7 ticks → same computational effort.
* **CPU-bound Times Scheduled:** RR = 22, MLFQ = 16 → fewer context switches under MLFQ.
* **I/O-bound CPU Ticks:** RR = 31, MLFQ = 27 → less waiting time in MLFQ.
* **I/O-bound Times Scheduled:** RR = 667, MLFQ = 634 → fewer context switches, improved efficiency.

---

## Discussion

* **MLFQ** improves responsiveness for I/O-bound processes by dynamically adjusting priorities.
* CPU-bound processes maintain same CPU tick usage but with **fewer context switches**, improving efficiency.
* Round Robin is fair but leads to **higher scheduling overhead** and less efficient CPU utilization in mixed workloads.

---

## Conclusion

* Scheduler design directly affects **system performance and responsiveness**.
* **MLFQ advantages:**

  * Faster response for I/O-bound/interactive processes.
  * Improved CPU utilization through fewer context switches.
  * Maintains fairness via priority boosting, preventing starvation.

---

## References

* **xv6 Public Repository:** [https://github.com/mit-pdos/xv6-riscv](https://github.com/mit-pdos/xv6-riscv)


