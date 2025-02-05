### Practical Example: Showcasing Different GC Algorithms Using the Spring Pet Clinic Application

In this practical example, we will use the **Spring Pet Clinic** application to demonstrate the impact of different garbage collection (GC) algorithms. We'll configure and run the application with various GC algorithms, monitor their performance, and analyze the results step by step.

---

## **Step 1: Set Up the Spring Pet Clinic Application**

### **Prerequisites**
- Java Development Kit (JDK) 11 or later.
- Maven (for building the application).

### **Download and Run the Application**
1. Clone the Spring Pet Clinic repository:
   ```bash
   git clone https://github.com/spring-projects/spring-petclinic.git
   ```

2. Navigate to the project directory:
   ```bash
   cd spring-petclinic
   ```

3. Build the application:
   ```bash
   mvn clean package
   ```

4. Start the application with default settings:
   ```bash
   mvn spring-boot:run
   ```

5. Open a browser and navigate to `http://localhost:8080` to verify that the application is running.

---

## **Step 2: Simulate High Memory Usage**

To simulate high memory usage, we will modify the application to create a large number of objects during runtime.

### **Modify the Code**
1. Open the file `src/main/java/org/springframework/samples/petclinic/visit/VisitService.java`.

2. Add a method to generate a large number of visits:
   ```java
   import java.util.ArrayList;
   import java.util.List;

   public class VisitService {
       private static final List<Visit> visitCache = new ArrayList<>();

       public void generateVisits(int count) {
           for (int i = 0; i < count; i++) {
               Visit visit = new Visit();
               visit.setDescription("Generated Visit " + i);
               visitCache.add(visit);
               System.out.println("Generated visit: " + i);
           }
       }

       public List<Visit> getAllVisits() {
           return visitCache;
       }
   }
   ```

3. Modify the `VisitController` to trigger visit generation:
   Open `src/main/java/org/springframework/samples/petclinic/visit/VisitController.java` and add an endpoint:
   ```java
   @GetMapping("/generate-visits")
   public String generateVisits(@RequestParam(defaultValue = "1000") int count) {
       visitService.generateVisits(count);
       return "redirect:/owners";
   }
   ```

4. Save the changes and restart the application:
   ```bash
   mvn spring-boot:run
   ```

---

## **Step 3: Configure and Test Different GC Algorithms**

We will test the following GC algorithms:
1. **Parallel GC**
2. **G1 GC**
3. **ZGC**

For each algorithm, we will:
- Configure the JVM options.
- Trigger high memory usage.
- Monitor GC behavior using VisualVM or GC logs.

---

### **Test 1: Parallel GC**

#### **Configuration**
Run the application with Parallel GC:
```bash
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xmx512m -XX:+UseParallelGC"
```
- `-Xmx512m`: Limits the heap size to 512 MB.
- `-XX:+UseParallelGC`: Enables Parallel GC.

#### **Trigger High Memory Usage**
Open a browser and navigate to:
```
http://localhost:8080/generate-visits?count=10000
```
This generates 10,000 visits, simulating high memory usage.

#### **Monitor Performance**
1. Use **VisualVM** or **JConsole** to monitor:
   - Heap usage.
   - GC pause times.
   - Throughput.

2. Observe:
   - Parallel GC may cause long pause times due to stop-the-world collections.
   - Throughput is high, but latency may suffer.

---

### **Test 2: G1 GC**

#### **Configuration**
Run the application with G1 GC:
```bash
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xmx512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200"
```
- `-Xmx512m`: Limits the heap size to 512 MB.
- `-XX:+UseG1GC`: Enables G1 GC.
- `-XX:MaxGCPauseMillis=200`: Targets pause times of 200ms or less.

#### **Trigger High Memory Usage**
Navigate to:
```
http://localhost:8080/generate-visits?count=10000
```

#### **Monitor Performance**
1. Use **VisualVM** or **JConsole** to monitor:
   - Heap usage.
   - GC pause times.
   - Throughput.

2. Observe:
   - G1 GC provides shorter, more predictable pause times compared to Parallel GC.
   - Throughput may be slightly lower due to concurrent GC activity.

---

### **Test 3: ZGC**

#### **Configuration**
Run the application with ZGC (requires JDK 11+):
```bash
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xmx512m -XX:+UseZGC"
```
- `-Xmx512m`: Limits the heap size to 512 MB.
- `-XX:+UseZGC`: Enables ZGC.

#### **Trigger High Memory Usage**
Navigate to:
```
http://localhost:8080/generate-visits?count=10000
```

#### **Monitor Performance**
1. Use **VisualVM** or **JConsole** to monitor:
   - Heap usage.
   - GC pause times.
   - Throughput.

2. Observe:
   - ZGC achieves near-zero pause times (<10ms).
   - Slightly higher memory overhead due to color mapping.

---

## **Step 4: Analyze Results**

### **Comparison Table**

| **Metric**          | **Parallel GC**      | **G1 GC**            | **ZGC**              |
|---------------------|----------------------|----------------------|----------------------|
| **Pause Times**     | Long (seconds)      | Short (~200ms)       | Near-zero (<10ms)    |
| **Throughput**      | High                | Moderate             | Moderate             |
| **Memory Overhead** | Low                 | Low                  | High                 |
| **Use Case**        | Batch processing    | Interactive apps     | Large heaps, low-latency |

---

## **Step 5: Conclusion**

By testing the Spring Pet Clinic application with different GC algorithms, we observed the following:
1. **Parallel GC** is ideal for batch processing tasks where throughput is more important than latency.
2. **G1 GC** strikes a balance between throughput and latency, making it suitable for most modern applications.
3. **ZGC** provides near-zero pause times, making it ideal for applications requiring ultra-low latency, especially on large heaps.

### **Recommendations**
- For **batch processing**, use Parallel GC.
- For **interactive applications**, use G1 GC.
- For **large heaps with strict latency requirements**, use ZGC.

### **Next Steps**
- Experiment with additional configurations (e.g., heap sizes, pause time targets).
- Use tools like **GC logs** or **Eclipse MAT** to analyze memory usage in detail.
