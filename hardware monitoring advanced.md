### **Advanced Monitoring and Troubleshooting Exercise**

This exercise focuses on advanced monitoring of **Run Queue**, **Network I/O**, **Disk I/O**, **Memory**, and **CPU** under load conditions. ---

### **Objective**
1. Deploy a sample application on an Ubuntu server.
2. Generate load on the application and system resources.
3. Use advanced monitoring tools to detect and troubleshoot issues.
4. Analyze metrics and provide optimization recommendations.

---

### **Pre-requisites**
1. Ubuntu Server 22.04 or later.
2. SSH access to the server.
3. Install required tools:
   ```bash
   sudo apt update
   sudo apt install sysstat htop dstat iostat nmon iperf3 apache2 stress-ng -y
   ```

---

### **Part 1: Deploy the Sample Application**

#### **1. Install and Configure Apache**
Apache is used as the sample application:
```bash
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
```

#### **2. Add a Test Page**
Create a basic HTML file to serve traffic:
```bash
echo "<h1>Performance Testing</h1><p>Simulating Load...</p>" | sudo tee /var/www/html/index.html
```

---

### **Part 2: Generate Load on the Application**

#### **1. Use `stress-ng` to Create Load on CPU, Memory, and Disk**
Run the following command to simulate a load for 10 minutes:
```bash
sudo stress-ng --cpu 4 --vm 2 --vm-bytes 1G --hdd 2 --timeout 600s
```

#### **2. Use `ab` (Apache Benchmark) to Simulate HTTP Requests**
```bash
sudo apt install apache2-utils -y
ab -n 5000 -c 100 http://localhost/
```

#### **3. Simulate Network I/O Load Using `iperf3`**
Run `iperf3` server on the test server:
```bash
iperf3 -s
```

From another machine, run the client to generate traffic:
```bash
iperf3 -c <server-ip> -t 600
```

---

### **Part 3: Advanced Monitoring**

#### **1. Monitor Run Queue**
Use `sar` to monitor the run queue:
```bash
sar -q 1 10
```
- Focus on the `runq-sz` column for average queue size.
- Check if the queue exceeds the number of CPU cores.

#### **2. Monitor Network I/O**
Use `sar` and `iftop`:
```bash
sar -n DEV 1 10
sudo apt install iftop -y
sudo iftop -i <interface>
```
- Look for abnormal RX/TX rates or packet drops.

#### **3. Monitor Disk I/O**
Use `iotop` and `sar`:
```bash
sudo iotop -o
sar -d 1 10
```
- Analyze `iowait` in `sar` and identify top disk-consuming processes in `iotop`.

#### **4. Monitor Memory**
Use `vmstat`, `free`, and `htop`:
```bash
vmstat 1 10
free -m
htop
```
- Check for excessive paging (`si`, `so`) and memory pressure.

#### **5. Monitor CPU**
Use `mpstat` and `htop`:
```bash
mpstat -P ALL 1
htop
```
- Analyze individual core utilization and overall CPU performance.

---

### **Part 4: Troubleshooting and Optimization**

#### **Tasks**
1. **Run Queue**:
   - Identify high queue lengths and correlate them with CPU usage.
   - Recommendation: Increase CPU cores or reduce CPU-intensive workloads.

2. **Network I/O**:
   - Check for high network traffic or dropped packets.
   - Recommendation: Optimize Apache configuration for concurrent connections.

3. **Disk I/O**:
   - Look for high `iowait` or disk-intensive processes.
   - Recommendation: Use faster storage (e.g., SSD) or optimize I/O-heavy processes.

4. **Memory**:
   - Identify memory leaks or excessive paging.
   - Recommendation: Increase RAM or optimize memory-hungry processes.

5. **CPU**:
   - Identify CPU bottlenecks and correlate with run queue.
   - Recommendation: Adjust application thread pools or CPU scheduling.

---

### **Deliverables**

1. **Report:**
   - Document observations for each metric.
   - Provide recommendations for resolving identified issues.

2. **Presentation:**
   - Summarize findings and present optimizations.

---

