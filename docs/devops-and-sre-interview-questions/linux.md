---
description: Commonly asked interview questions on networking for DevOps and Site Reliability Engineering (SRE) positions.
---

# Interview Questions on Linux

## Question 1: What is the load average in Linux `top` command, and how is it calculated?

Load average in Linux, as displayed in the htop command, represents the average system load over a specific time period. It is a measure of the CPU and I/O activity on the system. The load average is typically displayed in three values, corresponding to the last 1, 5, and 15 minutes.

Here's a breakdown of how it's calculated:

1. **Single-Core System:**

    If the load average is `1.0`, it means the system is fully utilized. For example, a load average of `2.0` on a single-core system suggests that the system is taking twice the time to process tasks than it would if it were idle.

2. **Multi-Core System:**

    On a system with multiple cores, a load average of `1.0` indicates that one core is fully utilized. A load average of `2.0` on a dual-core system means that both cores are fully utilized.

3. **Understanding Load Average Values:**

    The load average values represent the average number of processes that are either in a runnable state or waiting for I/O.

    A load average of `0.5` on a single-core system, for instance, means that the system was, on average, half utilized over the specified time period.

4. **Interpreting Load Average:**

    Values below the number of available cores generally indicate a healthy system.

    Values significantly higher than the number of cores may suggest a bottleneck, and the system might be struggling to keep up with the load.

5. **Time Intervals:**

    The three load average values correspond to different time intervals – 1 minute, 5 minutes, and 15 minutes. They provide a sense of the system's load over short, medium, and longer durations.

6. **Calculating Load Average:**

    The load average is calculated by tracking the number of processes in the system's run queue, both running and waiting for resources, over a specific time period.

In summary, load average is a valuable metric for understanding how busy a system is and whether it's handling the current workload effectively. High load averages can indicate potential performance issues, prompting further investigation into resource constraints or inefficient processes.

---


## Question 2: What do the columns `PRI`, `NI`, `VIRT`, `RES`, `SHR`, and `S` signify in the `htop` command?

In the `htop` command, which is an interactive process viewer for Unix systems, the `PRI`, `NI`, `VIRT`, `RES`, `SHR`, and `S` columns represent various attributes of the processes.

Here's an explanation of each:

1. **PRI (Priority):**

    This column shows the priority of the process. The priority is a value that determines the order in which processes are scheduled to run. Lower values indicate higher priority. In the context of htop, positive values represent user priority, and negative values represent system priority.

2. **NI (Nice value):**

    The nice value represents the "niceness" of a process. It is a user-space priority value that ranges from -20 to 19. A lower nice value means higher priority. Processes with a higher nice value are considered less important and receive fewer CPU resources.

3. **VIRT (Virtual Memory):**

    This column displays the total virtual memory used by a process. Virtual memory includes both physical RAM and swap space on disk. It represents the total address space that a process is able to access, including both used and unused portions.

4. **RES (Resident Set Size):**

    The RES column shows the resident set size, which is the portion of a process's memory that is held in RAM (non-swapped). It indicates the amount of physical memory that a process is actively using.

5. **SHR (Shared Memory):**

    This column displays the amount of shared memory used by a process. Shared memory is memory that can be simultaneously accessed by multiple processes. It allows processes to share data and communicate with each other more efficiently.

6. **S (Status):**

    The status column indicates the current status of the process. Common status codes include:

    - `R`: Running
    - `S`: Sleeping
    - `D`: Disk sleep (waiting for I/O)
    - `Z`: Zombie (terminated but not yet reaped by its parent)
    - `T`: Traced or stopped

Understanding these columns can provide valuable insights into the resource usage and behavior of processes on your system. Keep in mind that `htop` provides real-time updates, making it a powerful tool for monitoring and managing processes interactively.

---

