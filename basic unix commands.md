Here's a practical exercise that involves using basic UNIX commands like `grep`, `find`, and `vi` for performance tuning. This exercise is designed to take around 30 minutes, focusing on analyzing log files and optimizing system performance using basic command-line tools.

### **Objective**
Learn how to use basic UNIX commands to:
1. Identify performance-related issues in system logs.
2. Locate specific files consuming resources.
3. Edit configuration files for tuning.

### **Prerequisites**
- A Linux system (like Ubuntu 22.04).
- Basic familiarity with the command line.
- Access to a system with sample logs (if not, generate or simulate them).

### **Instructions**

#### **Step 1: Analyzing Log Files with `grep`**
1. **Purpose**: Use `grep` to search system logs for performance-related issues.
   
2. **Preparation**:
   - Open a terminal window.
   - If you don't have sample logs, generate a sample log file using:
     ```bash
     sudo dmesg > /var/log/sample_system.log
     ```

3. **Task**:
   - Use `grep` to filter log entries that are related to errors or warnings:
     ```bash
     grep -i "error" /var/log/sample_system.log
     grep -i "warning" /var/log/sample_system.log
     ```
   - Find specific CPU or memory-related messages:
     ```bash
     grep -i "cpu" /var/log/sample_system.log
     grep -i "memory" /var/log/sample_system.log
     ```

4. **Exercise**:
   - Identify any suspicious lines in the log file and note the timestamp.
   - Create a file with only the `error` and `warning` lines using `grep` and redirect it to a file called `error_report.log`:
     ```bash
     grep -i "error\|warning" /var/log/sample_system.log > ~/error_report.log
     ```

#### **Step 2: Finding Large Files and Performance Hogs with `find`**
1. **Purpose**: Use `find` to locate large files or configuration files that may affect performance.

2. **Preparation**:
   - Ensure you have some sample directories and files to work with. If needed, create dummy files:
     ```bash
     sudo mkdir -p /var/log/performance_logs
     sudo dd if=/dev/zero of=/var/log/performance_logs/largefile.log bs=1M count=100
     ```

3. **Task**:
   - Locate files larger than 50MB in the `/var/log` directory:
     ```bash
     sudo find /var/log -type f -size +50M
     ```
   - Find all configuration files in `/etc` modified in the last 7 days:
     ```bash
     sudo find /etc -type f -mtime -7
     ```
   - Search for `.log` files that contain "Out of Memory" errors:
     ```bash
     sudo find /var/log -type f -name "*.log" -exec grep -l "Out of Memory" {} \;
     ```

4. **Exercise**:
   - Identify large files and analyze whether they are needed.
   - Delete or move old large log files that are no longer required:
     ```bash
     sudo find /var/log -type f -name "*.log" -mtime +30 -exec rm {} \;
     ```

#### **Step 3: Editing Configuration Files with `vi`**
1. **Purpose**: Use `vi` to modify configuration files for performance tuning.

2. **Preparation**:
   - Locate a configuration file to edit, e.g., `/etc/sysctl.conf`.
   - Create a backup before editing:
     ```bash
     sudo cp /etc/sysctl.conf /etc/sysctl.conf.bak
     ```

3. **Task**:
   - Open the configuration file in `vi`:
     ```bash
     sudo vi /etc/sysctl.conf
     ```
   - In the file, navigate to the end and add the following lines to optimize network performance:
     ```
     # Network performance tuning
     net.core.rmem_max=16777216
     net.core.wmem_max=16777216
     net.ipv4.tcp_window_scaling=1
     ```
   - Save the file in `vi` by pressing `Esc`, typing `:wq`, and hitting `Enter`.

4. **Exercise**:
   - Apply the changes using:
     ```bash
     sudo sysctl -p
     ```
   - Verify if the changes are applied:
     ```bash
     sudo sysctl net.core.rmem_max
     sudo sysctl net.core.wmem_max
     sudo sysctl net.ipv4.tcp_window_scaling
     ```

### **Summary Questions**
1. **`grep`**:
   - What command would you use to search for a specific error message in all `.log` files within the `/var/log` directory?
   - How can you create a filtered log file showing only CPU-related entries?

2. **`find`**:
   - What is the command to find all files in `/etc` that have been modified in the last 3 days?
   - How can you identify and delete `.log` files larger than 100MB?

3. **`vi`**:
   - What steps do you follow to edit a configuration file using `vi`?
   - After making network performance adjustments, how do you apply and verify the changes?

### **Expected Outcome**
- By the end of this exercise, you should understand how to:
  - Use `grep` to analyze logs.
  - Use `find` to identify problematic files.
  - Edit configuration files using `vi` for performance tuning.


### **1. `htop` - Interactive Process Viewer**
**Description**: An enhanced version of `top`, `htop` is a real-time system-monitoring tool that displays running processes, memory usage, CPU usage, and more in a user-friendly, interactive interface.

**Installation**:
```bash
sudo apt update
sudo apt install htop
```

**Usage**:
- Launch `htop`:
  ```bash
  htop
  ```
- Use the arrow keys to navigate.
- Press `F6` to sort by different columns (e.g., CPU, memory).
- Press `F9` to kill a process interactively.

**Best Practices**:
- Use `htop` to identify resource-heavy processes.
- Look for processes with consistently high CPU or memory consumption for potential tuning.
- Save configurations by pressing `F2` and editing the options.

### **2. `iostat` - I/O Statistics**
**Description**: Part of the `sysstat` package, `iostat` provides detailed statistics on CPU, device utilization, and I/O performance.

**Installation**:
```bash
sudo apt update
sudo apt install sysstat
```

**Usage**:
- Display CPU and I/O statistics:
  ```bash
  iostat
  ```
- Monitor disk I/O every 5 seconds, 3 times:
  ```bash
  iostat -x 5 3
  ```

**Best Practices**:
- Look for devices with high utilization percentages.
- High `await` times indicate possible disk I/O bottlenecks.
- Use `-x` for extended statistics to get more insights.

### **3. `vmstat` - System Performance**
**Description**: Provides reports on virtual memory, processes, I/O, CPU activity, and more, allowing you to understand system performance at a glance.

**Installation**:
`vmstat` is typically pre-installed on most Linux distributions.

**Usage**:
- Basic usage:
  ```bash
  vmstat 5 3
  ```
  This shows performance metrics every 5 seconds, three times.
- Detailed output:
  ```bash
  vmstat -s
  ```

**Best Practices**:
- Look for high swap activity (`si` and `so` columns) indicating memory pressure.
- Check the `r` column (run queue) for processes waiting for CPU time.
- Use `vmstat` for a quick summary of memory, swap, and CPU usage.

### **4. `netstat`/`ss` - Network Statistics**
**Description**: Tools for network monitoring; `netstat` provides statistics on network connections, routing tables, and interface statistics. `ss` is a faster alternative.

**Usage**:
- Check active network connections:
  ```bash
  netstat -tunapl
  ss -tunapl
  ```
- Display network interface statistics:
  ```bash
  netstat -i
  ss -i
  ```

**Best Practices**:
- Use `ss` instead of `netstat` for faster results.
- Monitor for unusually high numbers of connections, which might indicate a potential network or security issue.
- Investigate the source and destination of heavy traffic using `netstat` or `ss`.

### **5. `sar` - System Activity Report**
**Description**: Also part of the `sysstat` package, `sar` collects and reports system activity statistics like CPU, memory, network, and I/O over time.

**Installation**:
```bash
sudo apt update
sudo apt install sysstat
```

**Usage**:
- Report CPU utilization every 2 seconds, 5 times:
  ```bash
  sar 2 5
  ```
- Check memory usage:
  ```bash
  sar -r
  ```
- Report network statistics:
  ```bash
  sar -n DEV 1 3
  ```

**Best Practices**:
- Schedule `sar` to run regularly using cron to collect long-term performance data.
- Use `-f` to read data from saved logs for historical analysis.
- Look at `sar` results to identify trends or anomalies over time.

### **6. `iftop` - Network Bandwidth Monitoring**
**Description**: A command-line tool for monitoring bandwidth usage by host and IP.

**Installation**:
```bash
sudo apt update
sudo apt install iftop
```

**Usage**:
- Run `iftop`:
  ```bash
  sudo iftop
  ```
- Use `h` for help, `n` to show IPs, and `P` to show port numbers.

**Best Practices**:
- Use `iftop` to identify which hosts are consuming excessive bandwidth.
- Monitor network interfaces individually using:
  ```bash
  sudo iftop -i eth0
  ```
- Investigate any unexpected outgoing traffic for potential issues.

### **7. `dstat` - Versatile Resource Statistics**
**Description**: Combines the functionality of tools like `vmstat`, `iostat`, `netstat`, and more into a single tool for viewing performance metrics.

**Installation**:
```bash
sudo apt update
sudo apt install dstat
```

**Usage**:
- Show CPU, disk, and network statistics:
  ```bash
  dstat
  ```
- Display stats for a specific process:
  ```bash
  dstat -p <PID>
  ```

**Best Practices**:
- Use `dstat` as a real-time monitoring tool for a comprehensive overview.
- Customize the output with specific flags to narrow down the focus.
- Use `-n` to focus on network, `-d` for disks, and `-c` for CPU.

### **8. `top` - Real-time Process Monitoring**
**Description**: The `top` command provides a dynamic real-time view of a running system, displaying the most CPU-intensive processes.

**Usage**:
- Start `top`:
  ```bash
  top
  ```
- Press `Shift + p` to sort by CPU usage.
- Press `Shift + m` to sort by memory usage.

**Best Practices**:
- Use `top` to identify processes that are consuming the most resources.
- Consider adjusting `nice` values to modify process priority if certain processes consistently consume too much CPU.
- Use `htop` for a more user-friendly alternative with better visuals.

### **9. `lsof` - List Open Files**
**Description**: Lists open files and the processes that opened them, helping to identify which files are being accessed, modified, or locked.

**Installation**:
`lsof` is typically pre-installed on most Linux distributions.

**Usage**:
- List all open files:
  ```bash
  lsof
  ```
- Check which processes are using a specific file:
  ```bash
  lsof /var/log/syslog
  ```
- Find processes using a particular port:
  ```bash
  sudo lsof -i :80
  ```

**Best Practices**:
- Use `lsof` to identify locked files if you encounter permission or access issues.
- Useful for detecting unexpected network connections or file accesses.

### **10. `nmon` - Performance Monitor**
**Description**: An interactive performance monitoring tool that provides a detailed view of CPU, memory, network, disk, and process utilization.

**Installation**:
```bash
sudo apt update
sudo apt install nmon
```

**Usage**:
- Start `nmon`:
  ```bash
  nmon
  ```
- Use keys like `c` for CPU, `m` for memory, `n` for network, `d` for disk stats, and `q` to quit.

**Best Practices**:
- Use `nmon` for a comprehensive real-time view of all system metrics.
- Export `nmon` data for offline analysis using:
  ```bash
  nmon -f -s 10 -c 30
  ```
  This collects data every 10 seconds for 30 iterations.

### **Conclusion**
Each of these tools offers a unique perspective on system performance. Use them in combination to get a holistic view:
- Use `htop` for a quick overview of process activity.
- Employ `iostat` and `vmstat` for deeper insights into I/O and memory usage.
- Utilize `iftop` and `nmon` to track network usage.
- Analyze open files with `lsof` to understand file and port interactions.
