Below is a comprehensive practical exercise for hardware monitoring on Ubuntu using a range of tools. This will include monitoring CPU, memory, disk performance, temperature, network I/O, and more.

### **1. Installing Monitoring Tools**
To get started, let's install the necessary tools for the exercise. These tools will help you monitor various aspects of hardware performance.

```bash
sudo apt update
sudo apt install -y nmon vmstat sysstat htop iotop dstat lm-sensors smartmontools hdparm lshw nvme-cli inxi iperf3
```

### **2. Monitoring Exercises**

#### **A. CPU Monitoring**

1. **Using `vmstat`**:
   - Command: `vmstat 2 10`
     - **Explanation**: This command will display system statistics every 2 seconds for 10 iterations. Pay attention to columns like:
       - `r`: Run queue length (number of processes waiting for CPU).
       - `us`: CPU time spent in user mode.
       - `sy`: CPU time spent in system mode.
       - `id`: CPU idle time.

2. **Using `nmon`**:
   - Command: `nmon`
     - Press `c` for CPU information.
     - Press `m` for memory information.
     - Press `d` for disk I/O.
     - Press `n` for network.
   - **Explanation**: Use `nmon` to interactively monitor CPU utilization. It shows CPU usage per core and average CPU load.

3. **Using `htop`**:
   - Command: `htop`
     - **Explanation**: `htop` is a real-time process monitoring tool. It provides a detailed overview of CPU usage per core, load average, and active processes.

#### **B. Memory Monitoring**

1. **Using `vmstat`**:
   - Command: `vmstat -s`
     - **Explanation**: This will show memory statistics such as free, used, and swap memory.

2. **Using `free`**:
   - Command: `free -h`
     - **Explanation**: Displays memory usage in a human-readable format, showing total, used, and available memory.

#### **C. Disk Monitoring**

1. **Using `iotop`** (Requires root privileges):
   - Command: `sudo iotop`
     - **Explanation**: This shows real-time disk I/O usage by processes.

2. **Using `dstat`**:
   - Command: `dstat -cdngy`
     - **Explanation**: This shows CPU, disk I/O, network, and process statistics in real-time.

3. **Using `nmon`**:
   - Command: `nmon`
     - Press `d` for disk I/O.
   - **Explanation**: Displays detailed information about disk read/write speeds.

4. **Using `lsblk`**:
   - Command: `lsblk`
     - **Explanation**: Lists all block devices with information about disk type, partitions, and mount points.

5. **Checking Disk Speed using `hdparm`**:
   - Command: `sudo hdparm -Tt /dev/sda`
     - **Explanation**: Tests the read speed of the disk `/dev/sda`. It will show buffered and cached reads.

6. **Disk Type and Health using `smartctl`**:
   - Command: `sudo smartctl -a /dev/sda`
     - **Explanation**: Shows detailed disk health status, temperature, error counts, and more.

#### **D. Network I/O Monitoring**

1. **Using `iftop`** (Requires root privileges):
   - Command: `sudo iftop`
     - **Explanation**: Provides a real-time view of network traffic by connection.

2. **Using `iperf3` for Network Speed Testing**:
   - **Server**: `iperf3 -s` (on one machine to act as a server).
   - **Client**: `iperf3 -c <server-ip>` (from another machine to test network speed).
     - **Explanation**: Measures bandwidth between two machines.

3. **Using `nload`**:
   - Command: `sudo apt install nload && sudo nload`
     - **Explanation**: Monitors network bandwidth usage on a network interface.

#### **E. Temperature Monitoring**

1. **Using `lm-sensors`**:
   - Command to detect sensors: `sudo sensors-detect`
   - View sensor readings: `sensors`
     - **Explanation**: Displays CPU temperature, fan speeds, and other sensor information.

2. **Checking Temperature for NVMe Drives**:
   - Command: `sudo nvme smart-log /dev/nvme0`
     - **Explanation**: Shows detailed information about NVMe SSD health and temperature.

#### **F. Hardware Information**

1. **Using `lshw`**:
   - Command: `sudo lshw -short`
     - **Explanation**: Provides detailed information about the hardware components in the system.
   - Command to check CPU info: `lscpu`
   - Command to check GPU info: `lshw -C display`

2. **Using `inxi`**:
   - Command: `inxi -Fxz`
     - **Explanation**: Displays a summary of hardware, including CPU, memory, storage, network interfaces, and more.

#### **G. Disk I/O Performance Testing**

1. **Using `fio`** (Flexible I/O Tester):
   - Install `fio`: `sudo apt install -y fio`
   - Run a basic I/O test:
     ```bash
     sudo fio --name=randwrite --ioengine=libaio --rw=randwrite --bs=4k --direct=1 --size=1G --numjobs=4 --time_based --runtime=60 --group_reporting
     ```
     - **Explanation**: This command performs a random write I/O test on the disk. Adjust parameters for different types of I/O tests.

#### **H. Web Server / App Server Monitoring**

1. **Using Apache Benchmark (`ab`)**:
   - Install: `sudo apt install apache2-utils`
   - Command: `ab -n 100 -c 10 http://localhost:8080/`
     - **Explanation**: This tests the performance of a web server by sending 100 requests with a concurrency of 10.

#### **I. Storage Device Information**

1. **Using `nvme-cli` for NVMe SSDs**:
   - Command: `sudo nvme list`
     - **Explanation**: Lists all NVMe drives connected to the system.

2. **Using `smartctl` for S.M.A.R.T. Data**:
   - Command: `sudo smartctl -i /dev/sda`
     - **Explanation**: Shows general information and capabilities of the disk `/dev/sda`.

#### **J. Run Queue Monitoring**

1. **Using `top`**:
   - Command: `top`
     - **Explanation**: Displays an overview of the system, including the load average which represents the run queue length.

2. **Using `vmstat` for Run Queue Length**:
   - Command: `vmstat 1 5`
     - **Explanation**: The first column, `r`, shows the run queue length.

### **Summary of Key Monitoring Tools and Commands**

| Tool        | Use Case                              | Command Example                          |
|-------------|--------------------------------------|------------------------------------------|
| `vmstat`    | CPU, memory, I/O                      | `vmstat 2 10`                            |
| `nmon`      | Interactive system monitoring         | `nmon`                                   |
| `htop`      | Process monitoring                    | `htop`                                   |
| `iotop`     | Disk I/O                              | `sudo iotop`                             |
| `dstat`     | Comprehensive system stats            | `dstat -cdngy`                           |
| `lm-sensors`| Temperature monitoring                | `sensors`                                |
| `smartctl`  | Disk health                           | `sudo smartctl -a /dev/sda`              |
| `hdparm`    | Disk speed test                       | `sudo hdparm -Tt /dev/sda`               |
| `iftop`     | Network bandwidth                     | `sudo iftop`                             |
| `iperf3`    | Network speed testing                 | `iperf3 -s` / `iperf3 -c <server-ip>`    |
| `fio`       | Disk I/O performance                  | `sudo fio --name=randwrite --ioengine=...` |
| `ab`        | Web server performance                | `ab -n 100 -c 10 http://localhost:8080/` |
| `lshw`      | Hardware info                         | `sudo lshw -short`                       |
| `inxi`      | System information                    | `inxi -Fxz`                              |

