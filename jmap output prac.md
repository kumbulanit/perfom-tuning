### Practical Example: Heap Dump Analysis on the Spring Pet Clinic Application

In this practical example, we will use the **Spring Pet Clinic** application to demonstrate how to generate and analyze heap dumps. We'll simulate a memory leak scenario, generate a heap dump using `jmap`, and analyze it with Eclipse MAT.

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

4. Start the application:
   ```bash
   mvn spring-boot:run
   ```

   The application will start on `http://localhost:8080`.

5. Open a browser and navigate to `http://localhost:8080` to verify that the application is running.

---

## **Step 2: Simulate a Memory Leak**

To simulate a memory leak, we will introduce a static cache in the `VisitService` class that holds onto `Visit` objects indefinitely.

### **Modify the Code**
1. Open the file `src/main/java/org/springframework/samples/petclinic/visit/VisitService.java`.

2. Add a static cache to store all visits:
   ```java
   import java.util.ArrayList;
   import java.util.List;

   public class VisitService {
       // Static cache to simulate memory leak
       private static final List<Visit> visitCache = new ArrayList<>();

       public void saveVisit(Visit visit) {
           visitCache.add(visit); // Add visit to the cache
           System.out.println("Visit added to cache: " + visit.getId());
       }

       public List<Visit> getAllVisits() {
           return visitCache; // Return all cached visits
       }
   }
   ```

3. Modify the `VisitController` to use the cache:
   Open `src/main/java/org/springframework/samples/petclinic/visit/VisitController.java` and update the `saveVisit` method:
   ```java
   @PostMapping("/owners/{ownerId}/pets/{petId}/visits/new")
   public String processVisitForm(@Valid Visit visit, BindingResult result, @PathVariable("petId") int petId, Model model) {
       if (result.hasErrors()) {
           model.addAttribute("visit", visit);
           return "pets/createOrUpdateVisitForm";
       } else {
           vetService.saveVisit(visit); // Save visit to the database
           visitService.saveVisit(visit); // Add visit to the static cache
           return "redirect:/owners/{ownerId}";
       }
   }
   ```

4. Save the changes and restart the application:
   ```bash
   mvn spring-boot:run
   ```

---

## **Step 3: Generate Load to Trigger the Memory Leak**

To simulate high memory usage, we will repeatedly add visits to the system.

1. Open a browser and navigate to `http://localhost:8080/owners/1`.
2. Click on a pet (e.g., "Leo") and add multiple visits by filling out the form and submitting it repeatedly.
3. Monitor the logs to see the visits being added to the cache:
   ```
   Visit added to cache: 1
   Visit added to cache: 2
   Visit added to cache: 3
   ...
   ```

---

## **Step 4: Generate a Heap Dump Using `jmap`**

### **Find the Process ID (PID)**
1. Use the `jps` command to find the PID of the Spring Pet Clinic application:
   ```bash
   jps -v
   ```
   Output:
   ```
   12345 org.springframework.samples.petclinic.PetClinicApplication ...
   ```

   Note the PID (e.g., `12345`).

### **Generate the Heap Dump**
1. Use `jmap` to generate a heap dump:
   ```bash
   jmap -dump:live,format=b,file=petclinic_heapdump.hprof 12345
   ```

   This saves the heap dump to a file named `petclinic_heapdump.hprof`.

---

## **Step 5: Analyze the Heap Dump Using Eclipse MAT**

### **Install Eclipse MAT**
1. Download Eclipse MAT from [https://www.eclipse.org/mat/](https://www.eclipse.org/mat/).
2. Extract the archive and run the `MemoryAnalyzer` executable.

### **Load the Heap Dump**
1. Open Eclipse MAT and load the `petclinic_heapdump.hprof` file.

### **Run the "Leak Suspects" Report**
1. After loading the heap dump, click on the **"Leak Suspects"** report.
2. The report highlights potential memory leaks:
   - A large number of `Visit` objects are consuming significant memory.
   - These objects are referenced by a static `ArrayList` (`visitCache`) in the `VisitService` class.

### **Examine the Histogram**
1. Go to the **"Histogram"** view.
2. Sort objects by retained heap size.
3. Notice that `org.springframework.samples.petclinic.visit.Visit` objects are consuming a large amount of memory.

### **Investigate Object References**
1. Select the `Visit` class in the histogram.
2. Use the **"Path to GC Roots"** feature:
   - The `Visit` objects are held by the static `ArrayList` (`visitCache`) in the `VisitService` class.
   - This prevents the `Visit` objects from being garbage collected, causing a memory leak.

---

## **Step 6: Interpretation of Results**

### **Key Findings**
1. **Static Cache Issue:**
   - The `VisitService` class maintains a static `ArrayList` (`visitCache`) that holds all `Visit` objects.
   - As new visits are added, the list grows without bounds, leading to excessive memory usage.

2. **Root Cause:**
   - The static `visitCache` is never cleared or managed properly, resulting in a memory leak.

### **Recommendations**
1. **Implement Cache Eviction:**
   - Replace the unbounded `ArrayList` with a bounded cache (e.g., `LinkedHashMap` with LRU eviction).
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

## **Step 7: Resolve the Issue**

1. Update the `VisitService` class to use a bounded cache:
   ```java
   import java.util.LinkedHashMap;
   import java.util.Map;

   public class VisitService {
       // Bounded cache with LRU eviction
       private static Map<Integer, Visit> visitCache = new LinkedHashMap<>(100, 0.75f, true) {
           protected boolean removeEldestEntry(Map.Entry<Integer, Visit> eldest) {
               return size() > 100; // Limit cache size to 100 entries
           }
       };

       public void saveVisit(Visit visit) {
           visitCache.put(visit.getId(), visit); // Add visit to the cache
           System.out.println("Visit added to cache: " + visit.getId());
       }

       public Map<Integer, Visit> getAllVisits() {
           return visitCache; // Return all cached visits
       }
   }
   ```

2. Save the changes and restart the application:
   ```bash
   mvn spring-boot:run
   ```

3. Generate another heap dump and analyze it to confirm that the issue is resolved.

---

## **Conclusion**

In this example, we demonstrated how to:
1. Set up and run the Spring Pet Clinic application.
2. Simulate a memory leak by introducing a static cache.
3. Generate and analyze heap dumps using `jmap` and Eclipse MAT.
4. Identify and resolve the root cause of the memory leak.
