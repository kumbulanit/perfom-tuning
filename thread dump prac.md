### Practical Example: Thread Dump Analysis on the Spring Pet Clinic Application



### **Step 1: Install and Set Up the Spring Pet Clinic Application**

#### **Prerequisites**
- Java Development Kit (JDK) 11 or later.
- Maven (for building the application).
- A web browser to access the application.

#### **Download the Spring Pet Clinic Application**
1. Clone the Spring Pet Clinic repository from GitHub:
   ```bash
   git clone https://github.com/spring-projects/spring-petclinic.git
   ```

2. Navigate to the project directory:
   ```bash
   cd spring-petclinic
   ```

3. Build the application using Maven:
   ```bash
   mvn clean package
   ```

4. Run the application:
   ```bash
   mvn spring-boot:run
   ```

   The application will start on `http://localhost:8080`.

5. Open a browser and navigate to `http://localhost:8080` to verify that the application is running.

---

### **Step 2: Simulate a Problem in the Application**

To simulate a problem, we will introduce artificial delays in the code. For example, let's add a sleep operation in one of the service methods.

#### **Modify the Code**
1. Open the file `src/main/java/org/springframework/samples/petclinic/visit/VisitService.java`.

2. Add a sleep operation in the `findVisitsByPetId` method:
   ```java
   @Override
   public List<Visit> findVisitsByPetId(Integer petId) {
       try {
           Thread.sleep(5000); // Simulate a 5-second delay
       } catch (InterruptedException e) {
           Thread.currentThread().interrupt();
       }
       return visitRepository.findByPetId(petId);
   }
   ```

3. Save the file and restart the application:
   ```bash
   mvn spring-boot:run
   ```

---

### **Step 3: Generate a Thread Dump Using `jstack`**

#### **Find the Process ID (PID)**
1. Use the `jps` command to find the PID of the Spring Pet Clinic application:
   ```bash
   jps -v
   ```
   Output:
   ```
   12345 org.springframework.samples.petclinic.PetClinicApplication ...
   ```

   Note the PID (e.g., `12345`).

#### **Generate the Thread Dump**
1. Use `jstack` to generate a thread dump:
   ```bash
   jstack 12345 > threaddump.txt
   ```

   This saves the thread dump to a file named `threaddump.txt`.

---

### **Step 4: Analyze the Thread Dump Using Tools**

#### **Option 1: Analyze with `jstack` Output**
Open the `threaddump.txt` file and search for threads in the `RUNNABLE` state. Look for stack traces involving `Thread.sleep`.

Example excerpt from the thread dump:
```plaintext
"pool-1-thread-1" #10 prio=5 os_prio=0 tid=0x00007f8c0c0d6000 nid=0x1b9a runnable [0x00007f8bf8dfc000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at org.springframework.samples.petclinic.visit.VisitService.findVisitsByPetId(VisitService.java:25)
        at org.springframework.samples.petclinic.visit.VisitController.showVisits(VisitController.java:45)
```

Interpretation:
- The thread `pool-1-thread-1` is sleeping for 5 seconds due to the artificial delay introduced in the `findVisitsByPetId` method.

---

#### **Option 2: Analyze with VisualVM**
1. **Install VisualVM:**
   - Download VisualVM from [https://visualvm.github.io/](https://visualvm.github.io/).
   - Extract the archive and run the `visualvm` executable.

2. **Connect to the Running Application:**
   - Launch VisualVM.
   - In the "Applications" section, you should see the Spring Pet Clinic application listed with its PID.

3. **Generate a Thread Dump:**
   - Right-click on the application and select **Threads > Thread Dump**.
   - VisualVM will display the thread dump in a readable format.

4. **Analyze the Results:**
   - Look for threads in the `TIMED_WAITING` state.
   - Identify the method causing the delay (`Thread.sleep` in this case).

---

#### **Option 3: Analyze with FastThread**
1. **Upload the Thread Dump:**
   - Go to [https://fastthread.io/](https://fastthread.io/).
   - Upload the `threaddump.txt` file.

2. **View the Analysis:**
   - FastThread provides a visual representation of the thread states.
   - It highlights threads in `RUNNABLE`, `BLOCKED`, and `WAITING` states.

3. **Interpret the Results:**
   - Look for threads with high CPU usage or long-running operations.
   - Identify the problematic method (`findVisitsByPetId` in this case).

---

### **Step 5: Resolve the Issue**

Based on the analysis, we can resolve the issue by removing the artificial delay:

1. Open the `VisitService.java` file.
2. Remove the `Thread.sleep` call:
   ```java
   @Override
   public List<Visit> findVisitsByPetId(Integer petId) {
       return visitRepository.findByPetId(petId);
   }
   ```

3. Save the file and restart the application:
   ```bash
   mvn spring-boot:run
   ```

4. Generate another thread dump to confirm that the issue is resolved.

---
