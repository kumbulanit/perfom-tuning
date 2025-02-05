### **Heap Dump Analysis: Understanding and Identifying Memory Leaks Using `jmap`**

## **1. What is a Heap Dump?**
A heap dump is a snapshot of the Java application's memory at a specific point in time. It contains detailed information about all objects in the heap, including:
- Object types
- Object sizes
- References between objects
- Garbage collection (GC) roots

Heap dumps are crucial for diagnosing memory leaks, high memory usage, and other memory-related issues.

---

## **2. Why Analyze Heap Dumps?**
Memory leaks occur when objects are no longer needed but are still referenced by the application, preventing them from being garbage collected. Analyzing heap dumps helps:
- Identify which objects are consuming excessive memory.
- Determine why these objects are not being garbage collected.
- Locate the root cause of memory leaks.

---

## **3. Generating a Heap Dump Using `jmap`**

The `jmap` tool, included in the JDK, is used to generate heap dumps.

### **Syntax**
```bash
jmap -dump:live,format=b,file=<heap_dump_file> <PID>
```
- `-dump`: Generates a heap dump.
- `live`: Includes only live objects (those reachable from GC roots).
- `format=b`: Specifies binary format.
- `file=<heap_dump_file>`: Saves the heap dump to a file.
- `<PID>`: Process ID of the Java application.

### **Steps**
1. **Find the Process ID (PID):**
   Use `jps` to find the PID of the running Java application:
   ```bash
   jps -v
   ```
   Example output:
   ```
   12345 PetClinicApplication ...
   ```

2. **Generate the Heap Dump:**
   ```bash
   jmap -dump:live,format=b,file=heapdump.hprof 12345
   ```

   This saves the heap dump to a file named `heapdump.hprof`.

---

## **4. Tools for Analyzing Heap Dumps**

Several tools can analyze heap dumps effectively:
1. **Eclipse MAT (Memory Analyzer Tool):** A powerful, open-source tool for analyzing heap dumps.
2. **VisualVM:** Provides built-in heap analysis capabilities.
3. **JProfiler:** A commercial tool with advanced memory profiling features.
4. **YourKit:** Another commercial tool for heap analysis.

For this training, we'll focus on **Eclipse MAT**, as it is free and widely used.

---

## **5. Simple to Advanced Ways of Analyzing Heap Dumps**

### **Step 1: Load the Heap Dump in Eclipse MAT**
1. Download and install Eclipse MAT from [https://www.eclipse.org/mat/](https://www.eclipse.org/mat/).
2. Open Eclipse MAT and load the `heapdump.hprof` file.

### **Step 2: Use the "Leak Suspects" Report**
1. After loading the heap dump, click on the **"Leak Suspects"** report.
2. This report identifies potential memory leaks based on:
   - Dominator trees (objects holding large amounts of memory).
   - Unreachable objects.
   - Large object graphs.

### **Step 3: Examine the Histogram**
1. Go to the **"Histogram"** view.
2. Sort objects by retained heap size.
3. Look for:
   - Unexpectedly large numbers of objects.
   - Objects that should have been garbage collected.

### **Step 4: Investigate Object References**
1. Select an object from the histogram.
2. Use the **"Path to GC Roots"** feature to find out why the object is still reachable.
3. Look for:
   - Static references.
   - Thread-local variables.
   - Caches or collections holding onto objects unnecessarily.

### **Step 5: Advanced Analysis**
1. Use the **OQL (Object Query Language)** to run custom queries on the heap dump.
   Example query to find all instances of a specific class:
   ```sql
   SELECT * FROM com.example.MyClass
   ```
2. Analyze complex object graphs using the **Dominator Tree** view.

---

## **6. Example: Analyzing a Heap Dump for a Memory Leak**

### **Scenario**
We suspect a memory leak in the Spring Pet Clinic application. Let's analyze a heap dump to confirm and locate the issue.

### **Steps**
1. **Generate the Heap Dump:**
   ```bash
   jmap -dump:live,format=b,file=petclinic_heapdump.hprof 12345
   ```

2. **Load the Heap Dump in Eclipse MAT:**
   - Open Eclipse MAT and load `petclinic_heapdump.hprof`.

3. **Run the "Leak Suspects" Report:**
   - The report highlights several potential memory leaks.
   - One major issue is a large number of `Visit` objects being held in memory.

4. **Examine the Histogram:**
   - In the histogram, sort objects by retained heap size.
   - Notice that `java.util.ArrayList` is consuming a significant amount of memory.

5. **Investigate Object References:**
   - Select `java.util.ArrayList` in the histogram.
   - Use the **"Path to GC Roots"** feature.
   - Discover that the `ArrayList` is part of a static cache (`VisitCache`) in the `VisitService` class.

6. **Analyze the Code:**
   - Open the `VisitService.java` file.
   - Find the static cache:
     ```java
     public class VisitService {
         private static List<Visit> visitCache = new ArrayList<>();

         public void addVisit(Visit visit) {
             visitCache.add(visit);
         }
     }
     ```
   - The `visitCache` is never cleared, causing memory to grow indefinitely.

---

## **7. Interpretation of Results**

### **Key Findings**
1. **Large Number of `Visit` Objects:**
   - The `VisitService` class maintains a static `ArrayList` (`visitCache`) that holds all `Visit` objects.
   - As new visits are added, the list grows without bounds, leading to excessive memory usage.

2. **Static Reference:**
   - The static `visitCache` prevents `Visit` objects from being garbage collected, even if they are no longer needed.

3. **Root Cause:**
   - The `visitCache` is not being cleared or managed properly, resulting in a memory leak.

### **Recommendations**
1. **Implement Cache Eviction:**
   - Use a bounded cache (e.g., `LinkedHashMap` with LRU eviction) instead of an unbounded `ArrayList`.
   - Example:
     ```java
     private static Map<Integer, Visit> visitCache = new LinkedHashMap<>(100, 0.75f, true) {
         protected boolean removeEldestEntry(Map.Entry<Integer, Visit> eldest) {
             return size() > 100; // Limit cache size to 100 entries
         }
     };
     ```

2. **Monitor Memory Usage:**
   - Use tools like VisualVM or JConsole to monitor memory usage and detect issues early.

---

## **8. Conclusion**

Heap dump analysis is a critical skill for diagnosing memory leaks and optimizing Java applications. By following these steps:
1. Generating heap dumps using `jmap`.
2. Analyzing heap dumps with tools like Eclipse MAT.
3. Identifying problematic objects and their references.
4. Resolving memory leaks by modifying the code.

