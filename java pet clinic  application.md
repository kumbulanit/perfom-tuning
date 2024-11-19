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
### *** ADD SOME LOAD ON THE APPLICATION *** ####

To load test a Java-based **Spring PetClinic** application, you can use **Apache JMeter**, which is a popular open-source tool designed for load testing. Below are the instructions and a sample JMeter test plan setup to create a load test for the Java PetClinic application.

### Step-by-Step Guide to Load Test with Apache JMeter

#### **1. Install Apache JMeter**

First, install **Apache JMeter** on your system:

```bash
# Download JMeter (change version number if a newer version is available)
wget https://downloads.apache.org//jmeter/binaries/apache-jmeter-5.5.tgz

# Extract the downloaded file
tar -xvzf apache-jmeter-5.5.tgz

# Move JMeter to a directory (optional)
sudo mv apache-jmeter-5.5 /opt/jmeter

# Add JMeter bin folder to PATH for easier command-line access
echo 'export PATH=$PATH:/opt/jmeter/bin' >> ~/.bashrc
source ~/.bashrc
```

#### **2. Configure JMeter Test Plan**

You'll need to create a JMeter Test Plan to simulate user actions on the PetClinic application. Here’s a quick overview of the components required:

1. **Thread Group**: Represents the number of users and how the users interact with the app.
2. **HTTP Request**: Configures HTTP requests to specific endpoints in the PetClinic app.
3. **Listeners**: Used to visualize and record the test results.

##### **Example JMeter Test Plan Structure**

1. **Thread Group**:
   - **Number of Threads (users)**: Defines how many users will access the application concurrently.
   - **Ramp-Up Period**: The time it takes to start all threads.
   - **Loop Count**: How many times each user will perform the actions.

2. **HTTP Requests**:
   - Create HTTP requests to simulate browsing different pages of the PetClinic app (like home, find owners, and add a new owner).

3. **Listeners**:
   - Add a Listener to monitor the test results (e.g., "View Results Tree" or "Summary Report").

#### **3. Creating the Test Plan Using JMeter GUI**

1. Open JMeter:
   ```bash
   jmeter
   ```
2. In the GUI, follow these steps:

   - **Add Thread Group**:
     1. Right-click on **Test Plan** → **Add** → **Threads (Users)** → **Thread Group**.
     2. Set:
        - **Number of Threads**: `100` (example for 100 concurrent users).
        - **Ramp-Up Period**: `60` (users will start over 60 seconds).
        - **Loop Count**: `10` (each user repeats the actions 10 times).

   - **Add HTTP Request Defaults** (optional):
     1. Right-click on **Test Plan** → **Add** → **Config Element** → **HTTP Request Defaults**.
     2. Set **Server Name or IP**: (e.g., `localhost` if running locally).
     3. Set **Port Number**: `8080` (default for Spring PetClinic).

   - **Add HTTP Requests**:
     1. Right-click on **Thread Group** → **Add** → **Sampler** → **HTTP Request**.
     2. Create multiple requests to simulate user actions, for example:
        - Home Page:
          - **Name**: `Home Page`.
          - **Path**: `/`.
          - **Method**: `GET`.
        - Find Owners Page:
          - **Name**: `Find Owners`.
          - **Path**: `/owners/find`.
          - **Method**: `GET`.
        - Add Owner:
          - **Name**: `Add Owner`.
          - **Path**: `/owners/new`.
          - **Method**: `POST`.
          - **Parameters**: Add necessary parameters (e.g., `firstName`, `lastName`, `address`).

   - **Add Listeners**:
     1. Right-click on **Thread Group** → **Add** → **Listener**.
     2. Choose a listener like **View Results Tree** or **Summary Report** for analyzing the results.

#### **4. Running the Test from CLI (Non-GUI Mode)**

Once the test plan (`petclinic_test_plan.jmx`) is configured and saved, you can run it from the command line to avoid GUI overhead.

```bash
# Run JMeter in non-GUI mode
jmeter -n -t /path/to/your/petclinic_test_plan.jmx -l results.jtl -e -o /path/to/output/folder
```

- **`-n`**: Non-GUI mode.
- **`-t`**: Test plan file.
- **`-l`**: Result log file.
- **`-e`**: Generate dashboard report.
- **`-o`**: Output folder for dashboard report.

#### **5. Analyzing Results**

After running the test, you can analyze results using:
- **Summary Report**: Provides a quick overview.
- **Dashboard Report**: Generates a detailed HTML report for in-depth analysis.

### **Example JMeter CLI Test Plan (Without GUI)**

Here’s a simple script to create a basic JMeter test plan using the command line:

```bash
#!/bin/bash

# Paths
JMETER_HOME="/opt/jmeter"
TEST_PLAN="petclinic_test_plan.jmx"
RESULTS_FILE="results.jtl"
OUTPUT_DIR="output"

# Create Test Plan file
cat <<EOL > $TEST_PLAN
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2" properties="5.0" jmeter="5.5">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="PetClinic Load Test" enabled="true">
      <stringProp name="TestPlan.comments"></stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
      <elementProp name="TestPlan.user_defined_variables" elementType="Arguments">
        <collectionProp name="Arguments.arguments"/>
      </elementProp>
      <stringProp name="TestPlan.user_define_classpath"></stringProp>
    </TestPlan>
    <hashTree>
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="Thread Group" enabled="true">
        <stringProp name="ThreadGroup.on_sample_error">continue</stringProp>
        <elementProp name="ThreadGroup.main_controller" elementType="LoopController">
          <boolProp name="LoopController.continue_forever">false</boolProp>
          <stringProp name="LoopController.loops">10</stringProp>
        </elementProp>
        <stringProp name="ThreadGroup.num_threads">100</stringProp>
        <stringProp name="ThreadGroup.ramp_time">60</stringProp>
        <longProp name="ThreadGroup.start_time">1629253760000</longProp>
        <longProp name="ThreadGroup.end_time">1629253760000</longProp>
        <boolProp name="ThreadGroup.scheduler">false</boolProp>
        <stringProp name="ThreadGroup.duration"></stringProp>
        <stringProp name="ThreadGroup.delay"></stringProp>
      </ThreadGroup>
      <hashTree>
        <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Home Page" enabled="true">
          <stringProp name="HTTPSampler.domain">localhost</stringProp>
          <stringProp name="HTTPSampler.port">8080</stringProp>
          <stringProp name="HTTPSampler.protocol">http</stringProp>
          <stringProp name="HTTPSampler.path">/</stringProp>
          <stringProp name="HTTPSampler.method">GET</stringProp>
        </HTTPSamplerProxy>
        <hashTree/>
      </hashTree>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
EOL

# Run JMeter
$JMETER_HOME/bin/jmeter -n -t $TEST_PLAN -l $RESULTS_FILE -e -o $OUTPUT_DIR
```




### **** ADD DYANMIC TRAFFIC *** ####


To create a more dynamic and realistic load test where the number of users spikes randomly at different intervals, you can use Apache JMeter's **"Ultimate Thread Group"** plugin, which allows you to define custom ramp-up periods, hold times, and spikes. Below, I'll guide you through setting up a script that simulates random spikes in user traffic.

### **Step-by-Step Guide for Random Spikes Using JMeter**

#### **1. Install JMeter Plugins Manager**

The **Ultimate Thread Group** is not included by default in JMeter, so you need to install the **Plugins Manager** to get it:

1. Download the **Plugins Manager JAR** file:
   
   ```bash
   wget https://jmeter-plugins.org/files/packages/jpgc-cmd-2.3.jar
   ```

2. Copy the JAR to JMeter's `lib/ext` folder:

   ```bash
   sudo mv jpgc-cmd-2.3.jar /opt/jmeter/lib/ext/
   ```

3. Start JMeter and go to **Options > Plugins Manager**, then install the **Custom Thread Groups** plugin.

4. Restart JMeter after installing the plugin.

#### **2. Configuring the Ultimate Thread Group**

Here’s how to create a test plan with random spikes:

1. **Open JMeter GUI**:

   ```bash
   jmeter
   ```

2. In the JMeter GUI:
   
   - **Add Ultimate Thread Group**:
     1. Right-click on **Test Plan** → **Add** → **Threads (Users)** → **Ultimate Thread Group**.
   
   - In the **Ultimate Thread Group**, configure the schedule with rows for each spike:
     - Each row will define a traffic pattern for a certain time period.

3. **Setup Example for Random Spikes**:

   Here’s an example configuration with a series of random spikes:

   | Row | Start Users | Ramp-Up Time (sec) | Hold Time (sec) | Shutdown Time (sec) |
   |-----|-------------|--------------------|------------------|---------------------|
   | 1   | 50          | 60                 | 120              | 30                  |
   | 2   | 200         | 30                 | 60               | 10                  |
   | 3   | 100         | 10                 | 90               | 15                  |
   | 4   | 300         | 20                 | 60               | 20                  |
   | 5   | 20          | 5                  | 180              | 10                  |
   
   - **Start Users**: Number of users to start.
   - **Ramp-Up Time**: Time to gradually start users (in seconds).
   - **Hold Time**: Duration to maintain the user count.
   - **Shutdown Time**: Time to gradually stop users.

4. **Add HTTP Requests**:
   - Similar to the previous steps, add HTTP requests under the **Ultimate Thread Group** to simulate interactions with the Java PetClinic application.

5. **Add a Listener**:
   - Right-click on **Ultimate Thread Group** → **Add** → **Listener** → **Summary Report** or **View Results in Table**.

#### **3. Running the Test from CLI with Ultimate Thread Group**

You can run the test using a JMeter script that integrates the **Ultimate Thread Group** to simulate random spikes. Below is an example JMeter test plan file (`petclinic_spike_test_plan.jmx`) configured with spikes.

#### **CLI JMeter Test Plan Example with Random Spikes**

Here's a shell script to generate a JMeter test plan with random spikes using the Ultimate Thread Group:

```bash
#!/bin/bash

# Paths
JMETER_HOME="/opt/jmeter"
TEST_PLAN="petclinic_spike_test_plan.jmx"
RESULTS_FILE="spike_test_results.jtl"
OUTPUT_DIR="spike_test_output"

# Create JMeter Test Plan with Ultimate Thread Group
cat <<EOL > $TEST_PLAN
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2" properties="5.0" jmeter="5.5">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="PetClinic Load Test with Spikes" enabled="true">
      <stringProp name="TestPlan.comments"></stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
      <elementProp name="TestPlan.user_defined_variables" elementType="Arguments">
        <collectionProp name="Arguments.arguments"/>
      </elementProp>
      <stringProp name="TestPlan.user_define_classpath"></stringProp>
    </TestPlan>
    <hashTree>
      <kg.apc.jmeter.threads.UltimateThreadGroup guiclass="kg.apc.jmeter.threads.UltimateThreadGroupGui" testclass="kg.apc.jmeter.threads.UltimateThreadGroup" testname="Ultimate Thread Group" enabled="true">
        <collectionProp name="ultimatethreadgroupdata">
          <!-- Spike 1 -->
          <collectionProp>
            <stringProp name="0">50</stringProp>          <!-- Start Users -->
            <stringProp name="1">60</stringProp>          <!-- Ramp-Up Time (sec) -->
            <stringProp name="2">120</stringProp>         <!-- Hold Time (sec) -->
            <stringProp name="3">30</stringProp>          <!-- Shutdown Time (sec) -->
          </collectionProp>
          <!-- Spike 2 -->
          <collectionProp>
            <stringProp name="0">200</stringProp>
            <stringProp name="1">30</stringProp>
            <stringProp name="2">60</stringProp>
            <stringProp name="3">10</stringProp>
          </collectionProp>
          <!-- Spike 3 -->
          <collectionProp>
            <stringProp name="0">100</stringProp>
            <stringProp name="1">10</stringProp>
            <stringProp name="2">90</stringProp>
            <stringProp name="3">15</stringProp>
          </collectionProp>
          <!-- Spike 4 -->
          <collectionProp>
            <stringProp name="0">300</stringProp>
            <stringProp name="1">20</stringProp>
            <stringProp name="2">60</stringProp>
            <stringProp name="3">20</stringProp>
          </collectionProp>
          <!-- Spike 5 -->
          <collectionProp>
            <stringProp name="0">20</stringProp>
            <stringProp name="1">5</stringProp>
            <stringProp name="2">180</stringProp>
            <stringProp name="3">10</stringProp>
          </collectionProp>
        </collectionProp>
      </kg.apc.jmeter.threads.UltimateThreadGroup>
      <hashTree>
        <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Home Page" enabled="true">
          <stringProp name="HTTPSampler.domain">localhost</stringProp>
          <stringProp name="HTTPSampler.port">8080</stringProp>
          <stringProp name="HTTPSampler.protocol">http</stringProp>
          <stringProp name="HTTPSampler.path">/</stringProp>
          <stringProp name="HTTPSampler.method">GET</stringProp>
        </HTTPSamplerProxy>
        <hashTree/>
      </hashTree>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
EOL

# Run JMeter
$JMETER_HOME/bin/jmeter -n -t $TEST_PLAN -l $RESULTS_FILE -e -o $OUTPUT_DIR
```

### **Analyzing Results with Spikes**
1. **Summary Report**: View the response times, throughput, and failures.
2. **Dashboard Report**: A detailed HTML report with graphs to visualize the load pattern and resource behavior.

This approach will help identify how the application scales during unexpected load spikes and whether it handles stress effectively.