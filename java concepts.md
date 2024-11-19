Here’s a practical exercise to measure and apply the performance tuning concepts discussed above. This exercise involves setting up a Java-based sample application on Ubuntu 22.04, running performance measurements, and optimizing based on the analysis. 

### **Objective**:
- Deploy a Java application on Ubuntu 22.04.
- Measure CPU, memory, disk, and network performance.
- Analyze the results to identify bottlenecks.
- Apply optimizations to improve the application's performance.

---

### **Step 1: Set Up Environment**

1. **Install Java Development Kit (JDK) on Ubuntu**:
   - Open a terminal and run the following commands to install Java:
     ```bash
     sudo apt update
     sudo apt install openjdk-17-jdk -y
     ```
   - Verify the installation:
     ```bash
     java -version
     ```

2. **Download a Sample Java Application**:
   - Download the sample application `Spring PetClinic`, a Java-based web application used for testing:
     ```bash
     wget https://github.com/spring-projects/spring-petclinic/archive/refs/heads/main.zip -O spring-petclinic.zip
     ```
   - Unzip the downloaded application:
     ```bash
     sudo apt install unzip -y
     unzip spring-petclinic.zip
     cd spring-petclinic-main
     ```

3. **Build and Run the Java Application**:
   - Install **Maven** (build tool) if not already installed:
     ```bash
     sudo apt install maven -y
     ```
   - Build the Java application:
     ```bash
     mvn clean install
     ```
   - Start the application:
     ```bash
     mvn spring-boot:run
     ```
   - The application will run on port 8080. Access it by navigating to `http://localhost:8080` in your browser.

---

### **Step 2: Performance Measurement and Analysis**

#### **CPU Profiling**

1. **Measure CPU Usage**:
   - Use `top` or `htop` to get an overview of CPU utilization while the application is running:
     ```bash
     top
     ```
   - Use `jconsole` or `VisualVM` (a Java profiler) to monitor the CPU usage of the application:
     ```bash
     sudo apt install jvisualvm -y
     jvisualvm
     ```
   - Attach to the running process and monitor CPU consumption.

2. **Analyze the Results**:
   - Identify if any particular method or thread is consuming excessive CPU.
   - Look for `hot spots` in the CPU profile.

3. **Optimization**:
   - Reduce the number of loops or complex computations.
   - Use thread pools for better thread management.

#### **Memory Utilization**

1. **Measure Memory Usage**:
   - Use `jstat` to get a snapshot of JVM memory usage:
     ```bash
     jstat -gc <PID>
     ```
   - Use `VisualVM` to monitor memory allocation and Garbage Collection (GC) activity.
   - Enable GC logs:
     ```bash
     mvn spring-boot:run -Djava.util.logging.config.file=logging.properties -Xms512M -Xmx1024M -XX:+PrintGCDetails
     ```

2. **Analyze the Results**:
   - Check heap usage and the frequency of GC events.
   - Observe if memory is being consumed quickly, leading to `OutOfMemoryError`.

3. **Optimization**:
   - Tune the heap size using `-Xms` and `-Xmx` parameters.
   - Reduce the frequency of object creation.

#### **Disk I/O Performance**

1. **Measure Disk I/O**:
   - Use `iostat` to monitor disk activity:
     ```bash
     sudo apt install sysstat -y
     iostat -x 2
     ```
   - Use `strace` to monitor file I/O operations in the application:
     ```bash
     strace -p <PID> -e trace=file
     ```

2. **Analyze the Results**:
   - Check if there’s a high `I/O wait` percentage, indicating disk bottlenecks.
   - Look for excessive read/write operations.

3. **Optimization**:
   - Enable file buffering.
   - Reduce disk writes by caching data in memory.

#### **Network I/O Monitoring**

1. **Measure Network Throughput**:
   - Use `iftop` to monitor network bandwidth usage:
     ```bash
     sudo apt install iftop -y
     sudo iftop -i <network-interface>
     ```
   - Use `tcpdump` to analyze network traffic:
     ```bash
     sudo tcpdump -i <network-interface> -n
     ```

2. **Analyze the Results**:
   - Check if there are any network delays or high latency.
   - Monitor for large or slow queries if your application involves database calls.

3. **Optimization**:
   - Use a CDN to offload static assets.
   - Optimize database queries to reduce network traffic.

#### **Database Optimization (If Database is Connected)**

1. **Install PostgreSQL (optional if using a database)**:
   ```bash
   sudo apt install postgresql postgresql-contrib -y
   sudo systemctl start postgresql
   sudo systemctl enable postgresql
   ```

2. **Check Database Performance**:
   - Use `pg_stat_activity` to check active queries:
     ```sql
     SELECT * FROM pg_stat_activity;
     ```
   - Use `EXPLAIN ANALYZE` to analyze SQL queries.

3. **Optimization**:
   - Add indexes to frequently queried columns.
   - Use connection pooling.

---

### **Step 3: Java-Specific Performance Tuning**

1. **Enable JMX Monitoring**:
   - Modify JVM parameters in the application's `pom.xml` to enable JMX monitoring:
     ```xml
     <properties>
       <maven.compiler.source>11</maven.compiler.source>
       <maven.compiler.target>11</maven.compiler.target>
       <jvm.args>
         -Dcom.sun.management.jmxremote
         -Dcom.sun.management.jmxremote.port=9090
         -Dcom.sun.management.jmxremote.authenticate=false
         -Dcom.sun.management.jmxremote.ssl=false
       </jvm.args>
     </properties>
     ```
   - Restart the application.

2. **Heap Dump Analysis**:
   - Trigger a heap dump:
     ```bash
     jmap -dump:live,format=b,file=heapdump.hprof <PID>
     ```
   - Analyze the heap dump using **MAT (Memory Analyzer Tool)**:
     ```bash
     sudo apt install mat -y
     mat heapdump.hprof
     ```

3. **Garbage Collection Tuning**:
   - Adjust the GC algorithm:
     ```bash
     -XX:+UseG1GC -XX:MaxGCPauseMillis=200
     ```
   - Monitor GC logs for efficiency.

### **Step 4: Conclusion and Report**

1. **Document Findings**:
   - Record measurements before and after optimizations.
   - Summarize the impact of each change.
   - Note any remaining bottlenecks for further tuning.

2. **Submit the Report**:
   - Write a report detailing the observations, changes, and performance improvements.
   - Share insights on which optimizations were most effective.

---
### *** OR USE VISUALVM *** ###

### **Step-by-Step Installation for VisualVM on Ubuntu 22.04**

1. **Download VisualVM**:
   - Visit the [official VisualVM download page](https://visualvm.github.io/download.html) and download the latest **VisualVM** release.
   - Alternatively, you can use `wget` to download it directly:
     ```bash
     wget https://github.com/oracle/visualvm/releases/download/2.1.6/visualvm_216.zip
     ```
   - Adjust the version number in the URL to the latest release if needed.

2. **Extract the Downloaded Zip File**:
   - Unzip the downloaded file:
     ```bash
     sudo apt install unzip -y
     unzip visualvm_216.zip
     ```
   - The files will be extracted to a folder called `visualvm`.

3. **Run VisualVM**:
   - Navigate to the extracted folder:
     ```bash
     cd visualvm/bin
     ```
   - Run VisualVM using the following command:
     ```bash
     ./visualvm
     ```
   - VisualVM should now launch, and you can start monitoring your Java application by attaching it to the running process.

### **Optional Setup - Creating a Desktop Shortcut for VisualVM**

1. **Create a Desktop Shortcut**:
   - Use a text editor like `nano` to create a `.desktop` file:
     ```bash
     nano ~/.local/share/applications/visualvm.desktop
     ```
   - Add the following content to the file:
     ```ini
     [Desktop Entry]
     Version=1.0
     Type=Application
     Name=VisualVM
     Exec=/path/to/visualvm/bin/visualvm
     Icon=/path/to/visualvm/etc/visualvm.ico
     Terminal=false
     Categories=Development;Java;
     ```
   - Replace `/path/to/visualvm` with the actual path to your `visualvm` folder.

2. **Make the Shortcut Executable**:
   ```bash
   chmod +x ~/.local/share/applications/visualvm.desktop
   ```

### **Using VisualVM**

- Once VisualVM is launched, you can:
  - Attach it to a running Java process.
  - Monitor CPU and memory usage.
  - Profile the application to find bottlenecks.
  - Analyze heap dumps.

### **Alternative Tools for Java Monitoring**

If you have trouble with VisualVM, here are some alternatives you can use:

1. **JConsole**: This is bundled with the JDK and provides basic monitoring features.
   ```bash
   jconsole
   ```
   - JConsole is a simpler GUI-based tool for monitoring Java applications.

2. **Java Mission Control (JMC)**: Another advanced tool for performance analysis.
   - Download it from [Oracle's website](https://www.oracle.com/java/technologies/javase/products-jmc7-downloads.html).

3. **Command-Line Tools**:
   - `jstat` for Java Virtual Machine statistics.
   - `jmap` for memory maps and heap dumps.
   - `jstack` for thread stack traces.
