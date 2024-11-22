HTTP monitoring tools like **HTTPWatch** and **Fiddler** are typically used to analyze HTTP traffic and performance. Since these tools are primarily Windows-based and not natively available for Ubuntu, we'll adapt this exercise to use **alternative Linux-compatible tools** that provide similar functionality, like **cURL**, **wget**, and **Wireshark**. The goal is to understand HTTP performance and interpret results using tools accessible on Ubuntu.



### **Exercise Objective**

Participants will learn to:
1. Use `curl` and `wget` to perform HTTP requests.
2. Capture and analyze HTTP traffic with `Wireshark`.
3. Measure and interpret HTTP performance metrics.
4. Optimize HTTP configurations for better response times.

### **Prerequisites**
- **Ubuntu** single instance (bare metal or VM).
- Basic knowledge of Linux commands.
- Access to `sudo` privileges.
- A web server installed (e.g., Apache or Nginx).

### **Setup (10 Minutes)**

#### 1. **Install Apache Web Server**
   ```bash
   sudo apt update
   sudo apt install apache2 -y
   ```

   Verify the installation by visiting `http://localhost` in a web browser. You should see the default Apache welcome page.

#### 2. **Install Required Tools**
   ```bash
   sudo apt install curl wget wireshark -y
   ```

   **Note**: `Wireshark` requires sudo permissions for capturing network traffic.

### **HTTP Request Simulation (20 Minutes)**

#### 1. **Make Basic HTTP Requests with `curl`**
   
   Use `curl` to make a simple HTTP GET request to the local web server:
   ```bash
   curl -v http://localhost
   ```

   **Explanation**:
   - `-v`: Verbose mode, shows request/response headers and connection details.

   **Discussion Points**:
   - Observe HTTP status codes (e.g., `200 OK`).
   - Look at the response time, content length, and server response headers.

#### 2. **Download a File Using `wget`**

   Create a test HTML file:
   ```bash
   sudo bash -c 'echo "<h1>Hello World</h1>" > /var/www/html/test.html'
   ```

   Download the file using `wget`:
   ```bash
   wget http://localhost/test.html
   ```

   **Explanation**:
   - `wget` is a command-line utility to download files from the web.
   - Observe download time and the HTTP headers sent during the transaction.

#### 3. **Measure HTTP Response Time Using `curl`**
   
   Use `curl` with timing parameters:
   ```bash
   curl -o /dev/null -s -w 'Time to connect: %{time_connect}s\nTime to start transfer: %{time_starttransfer}s\nTotal time: %{time_total}s\n' http://localhost
   ```

   **Explanation**:
   - `-o /dev/null`: Discards output (only timing info matters here).
   - `-s`: Silent mode (suppresses progress).
   - `-w`: Displays custom output with timing metrics:
     - **`%{time_connect}`**: Time to establish a connection.
     - **`%{time_starttransfer}`**: Time until the server starts transferring data.
     - **`%{time_total}`**: Total time of the request.

### **Capturing HTTP Traffic with Wireshark (20 Minutes)**

#### 1. **Start Wireshark**
   Open Wireshark:
   ```bash
   sudo wireshark
   ```

   - Choose the **network interface** (likely `lo` for local traffic) to capture packets.
   - Apply a capture filter to only show HTTP traffic:
     ```
     http
     ```

#### 2. **Analyze HTTP Requests**
   
   Go back to your terminal and perform another `curl` request:
   ```bash
   curl -v http://localhost/test.html
   ```

   In Wireshark:
   - Look at the **GET request** details.
   - Examine HTTP headers, response times, and TCP handshake.
   - Analyze the **response payload** and status codes.

   **Discussion Points**:
   - Look for any **HTTP errors** (4xx or 5xx).
   - Check TCP handshake (SYN, SYN-ACK, ACK) to ensure a stable connection.
   - Examine how HTTP response time varies with multiple requests.

### **Results Interpretation (10 Minutes)**

1. **Analyze the `curl` and `wget` Output**:
   - **HTTP Status Codes**: Look for codes like `200 OK`, `404 Not Found`, etc.
   - **Response Headers**: Identify server type, content type, and cache settings.
   - **Timing Metrics**: Review connection time, data transfer start time, and total time.

2. **Wireshark Insights**:
   - Examine the **round-trip time** (RTT) for HTTP requests.
   - Look at the **payload size** in the response.
   - Monitor **TCP retransmissions**, if any (indicative of network issues).

### **Optimization (10 Minutes)**

#### 1. **Tune Apache Configuration for Better HTTP Performance**

   Open Apache configuration file:
   ```bash
   sudo nano /etc/apache2/apache2.conf
   ```

   **Optimize These Parameters**:
   - **KeepAlive**: Enable persistent connections for faster load times.
     ```ini
     KeepAlive On
     MaxKeepAliveRequests 100
     KeepAliveTimeout 5
     ```
   - **Enable Compression**:
     Add the following to enable compression of text files:
     ```bash
     sudo a2enmod deflate
     sudo systemctl restart apache2
     ```
   - **Limit HTTP Headers**:
     Ensure that only necessary headers are exposed to clients. Use:
     ```ini
     ServerTokens Prod
     ServerSignature Off
     ```

   Restart Apache to apply changes:
   ```bash
   sudo systemctl restart apache2
   ```

#### 2. **Measure the Impact of Optimization**
   
   Repeat the `curl` and `wget` tests to see if response times improved. Use Wireshark to verify any changes in HTTP traffic behavior.

### **Discussion and Best Practices (10 Minutes)**

1. **Best Practices for HTTP Monitoring**:
   - Use tools like `curl` and `wget` to **quickly validate HTTP performance**.
   - Monitor HTTP traffic with **Wireshark** to detect packet-level issues.
   - Enable **Keep-Alive connections** for faster multiple request handling.
   - Utilize **compression** to reduce bandwidth usage for static content.
   - Monitor for **HTTP status codes** regularly to detect potential issues.
   - Use **timing metrics** (`time_connect`, `time_total`) to optimize server-side configurations.

2. **Security Considerations**:
   - Ensure sensitive data is transmitted over **HTTPS**.
   - Restrict unnecessary headers to prevent information leaks.
