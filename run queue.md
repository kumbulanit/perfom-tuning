### **What is the Run Queue?**
The **Run Queue** in Linux indicates the number of processes that are ready and waiting to run. A high run queue number suggests that there might be CPU contention, and tasks are waiting for available CPU time.

### **Tools to Monitor the Run Queue**
Below are several tools to get the run queue length and monitor system load:

#### **1. Using `vmstat`**

The `vmstat` command provides an overview of system performance, including the run queue length.

- **Command**:
  ```bash
  vmstat 2 10
  ```
  - **Explanation**: 
    - The above command will display system performance every 2 seconds for a total of 10 iterations.
    - Look at the **`r`** column, which shows the number of processes in the run queue.
    
- **Output Breakdown**:
  ```
  procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
   r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
   2  0      0 5174924 164244 236548    0    0    21    18  179  239  1  0 98  0  0
  ```
  - `r`: Number of processes waiting for CPU.
  - `b`: Number of processes in uninterruptible sleep.

#### **Exercise**:
1. Open a terminal and run:
   ```bash
   vmstat 1 5
   ```
2. Observe the `r` column for five intervals. If the number in `r` is consistently higher than the number of CPU cores available, it indicates potential CPU contention.

#### **2. Using `top`**

The `top` command is a powerful monitoring tool that provides a dynamic, real-time view of system performance, including the load average (indicating run queue).

- **Command**:
  ```bash
  top
  ```
  - **Explanation**: 
    - Observe the **`load average`** values displayed at the top. They represent the system's load over the last 1, 5, and 15 minutes.
    - A higher load average compared to the number of CPU cores suggests a long run queue.

- **Output Breakdown**:
  ```
  top - 10:32:01 up 1 day,  4:34,  2 users,  load average: 1.24, 0.95, 0.88
  Tasks: 207 total,   1 running, 206 sleeping,   0 stopped,   0 zombie
  %Cpu(s):  4.0 us,  1.5 sy,  0.0 ni, 94.2 id,  0.2 wa,  0.0 hi,  0.1 si,  0.0 st
  ```
  - `load average: 1.24, 0.95, 0.88`: These numbers represent the average number of processes in the run queue over the last 1, 5, and 15 minutes.

#### **Exercise**:
1. Open `top` by running the command:
   ```bash
   top
   ```
2. Observe the **load average** values. If they consistently exceed the number of available CPU cores, the system might be overloaded.

#### **3. Using `uptime`**

The `uptime` command quickly provides the system load average, which indicates the length of the run queue.

- **Command**:
  ```bash
  uptime
  ```
  - **Explanation**:
    - The load averages show the average number of processes waiting for CPU time in the last 1, 5, and 15 minutes.

- **Output Example**:
  ```
  10:32:01 up 1 day,  4:34,  2 users,  load average: 1.24, 0.95, 0.88
  ```
  - If the 1-minute load average exceeds the number of available cores, it's a sign of immediate CPU contention.

#### **Exercise**:
1. Run the command:
   ```bash
   uptime
   ```
2. Compare the load averages with the number of CPU cores (`lscpu` command shows the core count). If the load average is higher than the CPU cores, there's CPU contention.

#### **4. Using `sar` from `sysstat`**

The `sar` command, part of the `sysstat` package, provides historical system activity reports, including run queue length.

- **Command**:
  ```bash
  sar -q 1 5
  ```
  - **Explanation**:
    - The `-q` flag shows queue length statistics.
    - The command outputs every second for 5 intervals.
  
- **Output Example**:
  ```
  10:32:01 AM  runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
  10:32:02 AM         1       211      0.20      0.12      0.10         0
  ```
  - `runq-sz`: Number of processes in the run queue.
  - `ldavg-1`, `ldavg-5`, `ldavg-15`: Load averages over 1, 5, and 15 minutes.

#### **Exercise**:
1. Run the command:
   ```bash
   sar -q 1 5
   ```
2. Observe the `runq-sz` and load averages. If `runq-sz` consistently exceeds available CPU cores, the system might need optimization.

#### **5. Monitoring CPU Load with `nmon`**

`nmon` is an interactive command-line tool that displays system performance, including run queue details.

- **Command**:
  ```bash
  nmon
  ```
  - Press `q` to display the run queue.
  - Press `c` to monitor CPU utilization.
  
- **Exercise**:
1. Install `nmon` if not already installed:
   ```bash
   sudo apt install nmon
   ```
2. Start `nmon`:
   ```bash
   nmon
   ```
3. Press `q` to view run queue statistics.

#### **6. Checking Load with `dstat`**

`dstat` is another versatile tool to monitor the run queue and load.

- **Command**:
  ```bash
  dstat -q -c
  ```
  - **Explanation**: The `-q` flag shows the run queue length, and `-c` shows CPU usage.

#### **Exercise**:
1. Run the command:
   ```bash
   dstat -q -c
   ```
2. Monitor the run queue in real-time. A high run queue might indicate that the CPU is a bottleneck.

### **Load Analysis Tips**
- The general guideline is to compare the load average or run queue length to the number of CPU cores:
  - If you have 4 CPU cores and the load average is consistently above 4, the system might be overloaded.
  - Occasional spikes above the number of cores are normal, but sustained high numbers require investigation.

### **Summary**
| Tool         | Command                        | Focus                          | Key Metric                 |
|--------------|--------------------------------|--------------------------------|----------------------------|
| `vmstat`     | `vmstat 2 10`                   | Run queue and CPU load         | `r` column (run queue)     |
| `top`        | `top`                           | Real-time system monitoring    | Load average (1, 5, 15 min)|
| `uptime`     | `uptime`                        | Quick load average             | Load average               |
| `sar`        | `sar -q 1 5`                    | Historical load analysis       | `runq-sz`, `ldavg`         |
| `nmon`       | `nmon`                          | Interactive monitoring         | Press `q` for run queue    |
| `dstat`      | `dstat -q -c`                   | Run queue and CPU              | `runq` column              |

