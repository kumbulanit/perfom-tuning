





### **Java Application Lab Exercise: Troubleshooting GC, Memory Leaks, and Java Heap Issues**



### **Overview:**
Participants will:
1. Set up and run a Java web application that is deliberately misconfigured to trigger memory issues.
2. Use monitoring tools (like VisualVM, jConsole, or JMC) to diagnose the problem.
3. Analyze GC logs, memory usage, and heap dumps.
4. Apply fixes and optimizations to resolve the issues.

### **Objectives:**
- Understand how GC works in Java and identify different types of collectors.
- Identify memory leaks and inefficient memory usage.
- Learn how to read GC logs and interpret heap usage.
- Diagnose and fix memory issues related to Java heap.
- Explore how different GC algorithms affect performance.

---
create a file and add the below code with the name MemoryTroubleshootingApp.java

```java
import java.util.*;
import java.lang.management.*;

public class MemoryTroubleshootingApp {
    // Static collection to simulate memory leak
    private static final List<byte[]> memoryLeakList = new ArrayList<>();
    
    public static void main(String[] args) throws InterruptedException {
        // Start memory monitoring thread
        startMemoryMonitoring();
        
        // Demonstrate different memory scenarios
        memoryLeakSimulation();
        inefficientCollectionUsage();
        largeObjectAllocation();
        
        // Keep application running
        Thread.sleep(Long.MAX_VALUE);
    }
    
    // Memory Leak Simulation
    private static void memoryLeakSimulation() {
        System.out.println("Starting Memory Leak Simulation...");
        
        new Thread(() -> {
            while (true) {
                // Continuously add large byte arrays without releasing
                byte[] leakyBytes = new byte[1024 * 1024]; // 1MB allocation
                memoryLeakList.add(leakyBytes);
                
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
    
    // Inefficient Collection Usage
    private static void inefficientCollectionUsage() {
        System.out.println("Demonstrating Inefficient Collection Usage...");
        
        new Thread(() -> {
            List<String> largeList = new ArrayList<>();
            
            for (int i = 0; i < 1_000_000; i++) {
                largeList.add("Item " + i);
                
                // Simulate long-running process preventing GC
                if (i % 100_000 == 0) {
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
    }
    
    // Large Object Allocation
    private static void largeObjectAllocation() {
        System.out.println("Large Object Allocation Simulation...");
        
        new Thread(() -> {
            while (true) {
                // Allocate large objects to stress memory
                byte[][] largeObjectArray = new byte[1000][1024 * 1024]; // 1GB total
                
                try {
                    Thread.sleep(5000); // Wait before next allocation
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
    
    // Continuous Memory Monitoring
    private static void startMemoryMonitoring() {
        new Thread(() -> {
            MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
            
            while (true) {
                MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
                
                System.out.println("--- Memory Status ---");
                System.out.println("Used Memory: " + 
                    bytesToMegabytes(heapUsage.getUsed()) + " MB");
                System.out.println("Committed Memory: " + 
                    bytesToMegabytes(heapUsage.getCommitted()) + " MB");
                System.out.println("Max Memory: " + 
                    bytesToMegabytes(heapUsage.getMax()) + " MB");
                System.out.println("--------------------");
                
                try {
                    Thread.sleep(5000); // Monitor every 5 seconds
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
    
    // Helper method to convert bytes to megabytes
    private static long bytesToMegabytes(long bytes) {
        return bytes / (1024 * 1024);
    }
}
```

Save file as MemoryTroubleshootingApp.java
Compile: ``` javac MemoryTroubleshootingApp.java ```
Run: ``` java MemoryTroubleshootingApp ```




**Run the Application:**
- Start the Java application with specific JVM options that enable GC logging:

  ```bash
  java -Xms512m -Xmx512m -XX:+UseG1GC -XX:+PrintGCDetails  -Xloggc:gc.log MemoryTroubleshootingApp
  ```

**Explanation of JVM Options:**
- `-Xms512m`: Sets the initial heap size to 512 MB.
- `-Xmx512m`: Sets the maximum heap size to 512 MB.
- `-XX:+UseG1GC`: Use the G1 Garbage Collector.
- `-XX:+PrintGCDetails`: Print GC details.
- `-XX:+PrintGCDateStamps`: Add timestamps to GC logs.
- `-Xloggc:gc.log`: Save GC logs to `gc.log`.

#### **2. Monitoring and Troubleshooting:**

##### **A. Analyzing GC Logs:**
- Open the `gc.log` file and look for indications of frequent garbage collections.
- Check for Full GC events — if they are occurring too often, this is an indicator of potential memory pressure.

**Questions:**
1. What type of GC events are frequently occurring?
2. Is the GC time (pause time) affecting the application's performance?

##### **B. Using JVisualVM / JConsole:**
- Launch JVisualVM to monitor the running Java application:

  ```bash
  jvisualvm
  ```

- Connect to the running process (`sample-memory-leak-app`) and observe:
  - **Heap Usage**: Look for sudden spikes or steady increases (which can indicate a memory leak).
  - **Threads**: Check if any threads are stuck or blocked.
  - **GC Activity**: Observe how often the GC is triggered and how much memory it frees.

**Questions:**
1. Is the heap usage constantly increasing?
2. Are there threads that appear to be blocked or consuming too much CPU?

##### **C. Taking a Heap Dump:**
- Capture a heap dump using JVisualVM:
  - Right-click on the Java process.
  - Choose "Heap Dump" and save it.
- Analyze the heap dump to look for memory leaks. Check for classes with a large number of instances or unexpected memory growth.

**Questions:**
1. Which objects are consuming the most memory?
2. Can you identify a specific class that appears to be causing a memory leak?

#### **3. Advanced Analysis: GC Algorithms and Tuning:**

##### **A. Experimenting with Different GC Algorithms:**
- Restart the application using different GC algorithms and observe the behavior.

**Example JVM Options for Different GCs**:
1. **Serial GC** (single-threaded, suitable for smaller apps):

   ```bash
   java -Xms512m -Xmx512m -XX:+UseSerialGC -XX:+PrintGCDetails -Xloggc:gc-serial.log -MemoryTroubleshootingApp
   ```

2. **Parallel GC** (multi-threaded for high throughput):

   ```bash
   java -Xms512m -Xmx512m -XX:+UseParallelGC -XX:+PrintGCDetails -Xloggc:gc-parallel.log -MemoryTroubleshootingApp
   ```

3. **CMS (Concurrent Mark-Sweep) GC** (low pause time):

   ```bash
   java -Xms512m -Xmx512m -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -Xloggc:gc-cms.log -MemoryTroubleshootingApp
   ```

- Compare the GC logs for each algorithm. Determine which one performs better for this application.

**Questions:**
1. Which GC algorithm minimized the pause times?
2. Which algorithm handled the memory leak better?

##### **B. Profiling the Application with JFR (Java Flight Recorder):**
- Start the application with JFR enabled:

  ```bash
  java -Xms512m -Xmx512m -XX:+UseG1GC -XX:+FlightRecorder -XX:StartFlightRecording=duration=10m,filename=app-recording.jfr -MemoryTroubleshootingApp
  ```

- Open the `.jfr` file in JDK Mission Control (JMC) for detailed profiling.
- Look at memory allocation, heap usage, thread activity, and hotspots.

**Questions:**
1. What are the hotspots (methods/classes consuming the most resources)?
2. Is there any significant CPU or memory contention?

#### **4. Fixing and Optimizing:**

- Fix known memory leaks in the code:
  - Check if objects are not properly released.
  - Look for collections (e.g., Lists, Maps) that keep growing.
  - Fix `try-with-resources` or close resource leaks (e.g., file handles, database connections).

**Code Sample**:

```java
// Before Optimization (Leaky Code)
List<String> dataCache = new ArrayList<>();

public void loadData() {
    // This method keeps adding data indefinitely, causing a memory leak.
    for (int i = 0; i < 1000; i++) {
        dataCache.add("Data item " + i);
    }
}

// After Optimization (Fixed Code)
public void loadData() {
    dataCache.clear(); // Clear old data before loading new data
    for (int i = 0; i < 1000; i++) {
        dataCache.add("Data item " + i);
    }
}
```

#### **5. Load Testing:**

- Use Apache JMeter to simulate a load on the application and trigger GC events:

  ```bash
  sudo apt-get install jmeter
  jmeter
  ```

- Create a test plan that simulates multiple users accessing the application.
- Monitor the behavior of the application under load using JVisualVM or JConsole.

**Questions:**
1. How does the application perform under load?
2. Are there any Full GC events during the load test?

#### **6. Best Practices Recap:**
- Use appropriate GC algorithms for your workload.
- Avoid memory leaks by properly managing resources.
- Monitor heap usage and GC activity regularly.
- Use profiling tools to identify and resolve memory issues early.
- Optimize code to minimize object creation and GC overhead.

---

### **Exercise Summary and Key Takeaways:**
Participants will:
- Diagnose memory issues using GC logs, heap dumps, and profiling tools.
- Analyze how different GC algorithms affect performance.
- Identify and fix memory leaks.
- Apply best practices to optimize Java applications for performance.

### **Advanced Java Application Lab: Challenging Heap Issues, Memory Leaks, and GC Pauses**

#### **Overview:**
Participants will work on a Java-based web application with several hidden performance problems related to memory management. These problems will be challenging to detect, requiring in-depth analysis using monitoring tools, code inspection, and performance tuning techniques.

### **Lab Setup:**

#### **1. Requirements**
- Ubuntu 22.04 environment
- JDK 11 or higher
- Maven for building the Java project
- Monitoring tools: VisualVM, JConsole, Eclipse Memory Analyzer (MAT), GCViewer
- Apache JMeter for load testing
- Docker (optional) to run a containerized version of the database for added complexity
- A Java-based web application (e.g., a Spring Boot application) with pre-built issues. This lab will use an example `BookStore` application, which will have embedded problems.

#### **2. Download and Install Dependencies**
```bash
# Update and install necessary packages
sudo apt update
sudo apt install openjdk-11-jdk maven jconsole visualvm docker.io -y

# Install Apache JMeter
wget https://downloads.apache.org/jmeter/binaries/apache-jmeter-5.5.tgz
tar -xvzf apache-jmeter-5.5.tgz
cd apache-jmeter-5.5/bin

# Download Eclipse Memory Analyzer (MAT)
wget https://www.eclipse.org/downloads/download.php?file=/mat/1.12.0/rcp/MemoryAnalyzer-1.12.0.20200115-linux.gtk.x86_64.zip
unzip MemoryAnalyzer-1.12.0.20200115-linux.gtk.x86_64.zip -d ~/eclipse-mat
```

### **3. Set Up the Application:**
1. Download the **BookStore** application source code.
   ```bash
   # Clone the application code (pre-configured with hidden issues)
   git clone https://github.com/example/bookstore-challenge.git
   cd bookstore-challenge

   # Build the application
   mvn clean install
   ```

2. Start the application:
   ```bash
   # Run the application
   java -Xmx512m -jar target/bookstore-application.jar
   ```

### **Lab Description:**

#### **Part 1: Load Testing to Identify Issues**

1. **Load the Application with JMeter**:
   - Use Apache JMeter to simulate high load on the application.
   - Start with a low number of virtual users, then gradually increase to stress the system.
   - Focus on specific operations like searching for books, checking out, or updating inventory.

   **JMeter Load Test Setup:**
   - Open Apache JMeter (`apache-jmeter-5.5/bin/jmeter.sh`).
   - Create a new **Thread Group**.
   - Set up HTTP requests that mimic typical user behavior (e.g., login, search for books, add to cart).
   - Run the test with 50, 100, and 200 users and record response times.
   - Look for **long response times** or **timeouts**.

#### **Part 2: Detect Subtle Memory Leaks**

1. **Analyze the Java Heap Using VisualVM**:
   - Connect VisualVM to the running `BookStore` application.
   - Monitor the heap usage over time. Look for any **gradual increase in memory usage**.
   - Take **heap dumps** periodically under different load conditions.
   - Use the "Heap Dump" feature to inspect the heap and search for **retained memory** that should have been garbage collected.

2. **Trigger Full GC and Inspect GC Logs**:
   - Enable detailed GC logging:
     ```bash
     java -Xmx512m -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintHeapAtGC -jar target/bookstore-application.jar
     ```
   - Run the load test again and observe GC behavior in the logs.
   - Look for **long GC pauses** or frequent **Full GC events**.

#### **Part 3: Troubleshoot GC Pauses and Memory Issues**

1. **Use Eclipse Memory Analyzer (MAT)**:
   - Load the heap dumps into Eclipse MAT and analyze the results.
   - Use the **Leak Suspects Report** to identify potential memory leaks.
   - Check for objects that are not being freed due to lingering references (e.g., static fields or caches).
   - Investigate the **Dominator Tree** to see what is consuming the most memory.

2. **Examine GC Algorithm Behavior**:
   - Switch between different GC algorithms (e.g., `G1`, `CMS`, `Parallel`) and test their impact:
     ```bash
     # Use G1GC
     java -Xmx512m -XX:+UseG1GC -jar target/bookstore-application.jar
     
     # Use ParallelGC
     java -Xmx512m -XX:+UseParallelGC -jar target/bookstore-application.jar
     
     # Use CMS
     java -Xmx512m -XX:+UseConcMarkSweepGC -jar target/bookstore-application.jar
     ```
   - Observe differences in GC behavior using VisualVM and the GC logs.
   - Note how different GC algorithms handle memory allocation and collection.

#### **Part 4: In-Depth Analysis of Memory Leaks and GC Performance**

1. **Hidden Memory Leak - Soft Leak**:
   - In the `BookStore` application, there is a "soft leak" caused by a **LinkedHashMap** that caches book search results without ever removing old entries.
   - Participants need to find this soft leak using heap dump analysis tools.
   - Look for objects that **grow slowly** over time.

2. **Unpredictable GC Pauses**:
   - The `BookStore` application has a background job (implemented with `ScheduledExecutorService`) that occasionally spikes CPU usage and causes GC pauses.
   - Use JConsole to monitor **CPU usage**, **thread activity**, and **memory consumption** over time.
   - Detect how and when these spikes occur.

#### **Part 5: Optimization and Best Practices**

1. **Tune the GC**:
   - Experiment with GC tuning flags like:
     ```bash
     # Increase heap size to reduce GC frequency
     java -Xms256m -Xmx1024m -XX:+UseG1GC -jar target/bookstore-application.jar
     
     # Reduce Full GC impact
     java -Xms512m -Xmx1024m -XX:MaxGCPauseMillis=200 -jar target/bookstore-application.jar
     
     # Configure memory pools
     java -Xms512m -Xmx1024m -XX:NewRatio=3 -XX:+UseConcMarkSweepGC -jar target/bookstore-application.jar
     ```
   - Observe the impact of these changes on GC behavior and application performance.

2. **Fix the Soft Leak**:
   - Modify the `LinkedHashMap` to use a **size-limited cache** or a proper **eviction policy** (like `LRU`).
   - Validate the fix by running load tests and ensuring no memory growth over time.

3. **Optimize Background Job**:
   - Adjust the `ScheduledExecutorService` to use **lower-priority threads**.
   - Consider **reducing the frequency** of background tasks to avoid GC pauses.

---

### **Submission Requirements:**
Participants must:
1. Provide a report detailing the analysis and findings (including heap dumps, GC logs).
2. Document any changes made to resolve memory leaks or GC issues.
3. Provide before-and-after performance metrics from JMeter tests.
4. Offer recommendations for optimal GC settings based on their experiments.




To make the lab more challenging and realistic, let's include a dynamic load test that progressively stresses the application and causes different behaviors under varying loads. This will help identify issues that only become apparent under specific conditions, like high concurrency or fluctuating user traffic.

### **Adding a Dynamic Load Test to the Java Application Lab**

#### **Dynamic Load Testing Overview**
Participants will use **Apache JMeter** to create a load test that dynamically changes the load conditions over time. This will simulate real-world scenarios where traffic can fluctuate, causing intermittent performance problems that might be challenging to detect. The aim is to identify how the application responds to spikes and drops in load, uncovering hidden memory leaks, GC pauses, and Java heap issues.

### **Step-by-Step Instructions**

#### **1. Setup for Dynamic Load Test**

1. **Create the JMeter Test Plan**:
   - Start Apache JMeter: 
     ```bash
     apache-jmeter-5.5/bin/jmeter.sh
     ```
   - Create a new **Test Plan**.
   - Add a **Thread Group** to the Test Plan:
     - Right-click on the Test Plan → Add → Threads (Users) → Thread Group.
   - Set the **Thread Group** parameters to simulate dynamic load:
     - **Number of Threads (users):** Set initially to `10`.
     - **Ramp-Up Period:** Set to `10` seconds.
     - **Loop Count:** Set to `Forever`.

2. **Add a **Loop Controller**: 
   - This will help simulate fluctuating load.
   - Right-click on the Thread Group → Add → Logic Controller → Loop Controller.
   - Set **Loop Count** to `5` (or more to extend the testing period).

3. **Dynamic Load Increase Using Timer**:
   - Add a **Constant Throughput Timer** under the Thread Group to control the request rate.
     - Right-click on the Thread Group → Add → Timer → Constant Throughput Timer.
     - Set the throughput rate to dynamically increase over time (e.g., `30` requests/min at the start).
   - Use **Uniform Random Timer** to add randomness to the requests:
     - Right-click on the Thread Group → Add → Timer → Uniform Random Timer.
     - Set the delay to a random value (e.g., between `200ms` and `1 second`).

4. **HTTP Requests**:
   - Add an HTTP Request Sampler to simulate typical user actions.
   - Right-click on the Thread Group → Add → Sampler → HTTP Request.
   - Configure it to:
     - **Server Name or IP**: `localhost`
     - **Port Number**: The port your application runs on (e.g., `8080`).
     - **Path**: Choose different paths (e.g., `/search`, `/checkout`, `/inventory`) for variety.

5. **Add More Thread Groups**:
   - Create additional Thread Groups to simulate various types of users (e.g., browsing users, checkout users).
   - Configure each Thread Group with different load parameters to create a **mixed load environment**.

#### **2. Gradual Load Increase with JMeter**

1. **Add a Scheduled Load Increase**:
   - In each Thread Group, use the **Scheduler** option:
     - Check "Scheduler" at the bottom of the Thread Group settings.
     - Set the test duration to gradually increase over 10-15 minutes.
     - Change the Number of Threads dynamically every 2-3 minutes using a Timer.

2. **Add Listeners to Track Performance**:
   - Use the following listeners to track performance metrics:
     - **View Results in Table**
     - **Summary Report**
     - **Aggregate Report**
     - **Response Time Graph**
     - **Active Threads Over Time**
   - These listeners will help in analyzing spikes in response time, failed requests, and throughput.

#### **3. Run the Dynamic Load Test**

1. Start the **JMeter test** with the initial load.
2. Observe the **real-time changes** in JMeter as the load increases.
3. Note the application’s behavior when:
   - The load is low.
   - There is a gradual increase in load.
   - There are sudden spikes in the number of users.
   - The load decreases back to a low level.

#### **4. Monitor Application Behavior Under Load**

1. **JVM Monitoring**:
   - Use `VisualVM` or `JConsole` to monitor memory usage, thread activity, and CPU load.
   - Check for **memory leaks**, **GC pauses**, and heap behavior.
   - Look for unexpected spikes in heap size or GC activity during load increases.

2. **GC Logs Analysis**:
   - Ensure the application is started with detailed GC logs enabled:
     ```bash
     java -Xms512m -Xmx1024m -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCApplicationStoppedTime -XX:+UseG1GC -jar target/bookstore-application.jar
     ```
   - Use `GCViewer` to visualize the logs:
     - Download [GCViewer](https://github.com/chewiebug/GCViewer).
     - Load the GC log file into GCViewer to analyze GC pause times, frequency, and heap behavior.

3. **Heap Dump Analysis**:
   - Take **heap dumps** when the load is high to analyze potential memory leaks.
   - Use **MAT (Eclipse Memory Analyzer)** to analyze heap dumps.

#### **5. Dynamic Load Observations**

1. **GC Pauses**:
   - Look for patterns like **increased Full GC events** or long pauses during peak load.
   - Note how the **GC algorithm** responds to sudden load changes (e.g., does it handle the load without increasing pause times?).

2. **Memory Leak Symptoms**:
   - Monitor the memory allocation in `VisualVM` or `JConsole`. Look for a **slow but steady increase** in heap usage without subsequent decreases after load drops.
   - Analyze heap dumps if memory usage continues to grow.

3. **Thread Behavior**:
   - Monitor the thread count during the load test. Check if certain threads are stuck or not releasing resources properly.
   - Use VisualVM’s **Threads** tab to detect deadlocks or high CPU-consuming threads.

#### **6. Challenges:**

1. The `BookStore` application has hidden **memory leaks** that are tied to specific user behaviors (e.g., searching repeatedly for books with certain keywords causes retained memory in a custom cache).
2. Under high load, there are intermittent **GC pauses** due to improperly tuned GC settings. Participants must identify which GC algorithm is most suitable for this application.
3. There are obscure **Java heap issues** linked to data structures that retain objects longer than needed.
4. **Hidden problems** with connection and thread pools may only become apparent under a sustained high load.

### **Best Practices for Participants**:

1. **Focus on Consistency**: Identify patterns across multiple test runs to distinguish between random noise and actual issues.
2. **Use GC Tools**: Don't rely solely on `VisualVM`; also use **MAT** and **GCViewer** for detailed analysis.
3. **Tune Iteratively**: Change one JVM or application setting at a time and rerun tests to measure the impact.
4. **Resource Contention**: Be wary of thread contention during high load—watch for high CPU threads that could indicate locking issues.
5. **Dynamic Testing**: Include load spikes, steady increases, and drops in your JMeter test to see how the application behaves under varying conditions.

### **Deliverables**:

1. A report detailing the following:
   - Analysis of JVM metrics and GC behavior.
   - Identification of memory leaks and their root cause.
   - Observations of heap usage and optimization suggestions.
   - Summary of JMeter test results, including throughput, response times, and errors.
2. **Heap Dumps** and GC Logs with analysis.
3. Suggestions for **optimal GC settings** based on observations.
4. Application code changes (if necessary) to fix memory issues.
