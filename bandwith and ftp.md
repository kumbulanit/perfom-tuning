
### **Exercise Objective**

Participants will learn to:
1. Set up an FTP server on a single Ubuntu instance.
2. Transfer files using FTP to measure bandwidth.
3. Use network tools to monitor FTP performance.
4. Interpret the results of bandwidth tests.
5. Optimize FTP configuration for better performance.

### **Prerequisites**
- **Ubuntu** single instance (bare metal or VM).
- Basic knowledge of Linux commands.
- Access to `sudo` privileges.
- A file or dataset (e.g., a 100MB file) to use for testing.

### **Setup (10 Minutes)**

#### 1. **Install vsftpd (FTP Server)**
   ```bash
   sudo apt update
   sudo apt install vsftpd -y
   ```

#### 2. **Configure the FTP Server**
   
   Edit the configuration file:
   ```bash
   sudo nano /etc/vsftpd.conf
   ```

   Update/add the following lines to the configuration to enable local user access:
   ```ini
   anonymous_enable=NO
   local_enable=YES
   write_enable=YES
   local_umask=022
   chroot_local_user=YES
   pasv_min_port=40000
   pasv_max_port=50000
   allow_writeable_chroot=YES
   ```
   
   **Explanation**:
   - `anonymous_enable=NO`: Disable anonymous login.
   - `local_enable=YES`: Allow local users to log in.
   - `write_enable=YES`: Allow file uploads.
   - `chroot_local_user=YES`: Restrict users to their home directory for security.
   - `pasv_min_port` and `pasv_max_port`: Limit the passive mode ports to a specific range.

   Restart the FTP service:
   ```bash
   sudo systemctl restart vsftpd
   ```

#### 3. **Create a User for FTP Testing**
   ```bash
   sudo adduser ftpuser
   sudo passwd ftpuser
   ```

   **Note**: Remember the password you set for `ftpuser` as you'll need it for the FTP client.

### **File Transfer and Bandwidth Measurement (20 Minutes)**

#### 1. **Create a Test File (100MB)**
   ```bash
   dd if=/dev/zero of=/home/ftpuser/testfile bs=1M count=100
   ```

   **Explanation**: This creates a 100MB file named `testfile` in the `ftpuser` home directory.

#### 2. **Install FTP Client and Tools for Measurement**
   ```bash
   sudo apt install ftp -y
   sudo apt install iperf3 -y
   ```

#### 3. **Use FTP to Transfer the File Locally**
   Open an FTP connection to the server using the local IP (`127.0.0.1`):
   ```bash
   ftp 127.0.0.1
   ```

   Log in with the credentials for `ftpuser`.

   **Commands to run** inside the FTP session:
   ```bash
   binary
   get testfile
   ```

   **Explanation**:
   - `binary`: Ensures the file is transferred in binary mode (no data conversion).
   - `get testfile`: Downloads the `testfile` from the FTP server.

#### 4. **Measure Bandwidth Usage with iperf3**
   To get an accurate measure of FTP bandwidth, you can simulate a file transfer scenario using iperf3.

   Start an iperf3 server:
   ```bash
   iperf3 -s
   ```

   Open another terminal session and run an iperf3 client to simulate a connection:
   ```bash
   iperf3 -c 127.0.0.1 -t 30
   ```

   **Explanation**: 
   - `-s`: Starts iperf3 in server mode.
   - `-c 127.0.0.1`: Connects to the local iperf3 server.
   - `-t 30`: The test will run for 30 seconds to measure bandwidth.

### **Results Interpretation (15 Minutes)**

1. **Check FTP Transfer Speed**:
   - During the FTP file download, look for the reported transfer speed (usually shown at the end of the download).
   - Note the transfer rate in MB/s or KB/s.

2. **Analyze iperf3 Output**:
   - `iperf3` will show bandwidth usage during the 30-second test.
   - Focus on metrics like `Bandwidth`, `Transfer`, and `Retransmits`.
   - Note any fluctuations in bandwidth and the average speed.

3. **Network Tools**:
   - `iftop` (optional installation): Visualize bandwidth usage in real-time.
     ```bash
     sudo apt install iftop
     sudo iftop -i lo
     ```

   **Explanation**: This will show bandwidth usage on the loopback interface (`lo`).

### **Optimization (15 Minutes)**

#### 1. **Optimize vsftpd Configuration**

- **Enable Compression**:
  Add the following line to `/etc/vsftpd.conf`:
  ```ini
  allow_writeable_chroot=YES
  ```
  This will enable passive mode to work correctly.

- **Increase Transfer Speed**:
  - Adjust `write_enable=YES` to allow uploads.
  - Enable passive mode for FTP using:
    ```ini
    pasv_enable=YES
    pasv_min_port=40000
    pasv_max_port=50000
    ```
  - Restart vsftpd:
    ```bash
    sudo systemctl restart vsftpd
    ```

#### 2. **Tune TCP/IP Settings for Better FTP Performance**
   Add these settings to `/etc/sysctl.conf`:
   ```ini
   net.core.rmem_max = 16777216
   net.core.wmem_max = 16777216
   net.ipv4.tcp_window_scaling = 1
   net.ipv4.tcp_rmem = 4096 87380 16777216
   net.ipv4.tcp_wmem = 4096 65536 16777216
   ```

   Apply the changes:
   ```bash
   sudo sysctl -p
   ```

   **Explanation**: These settings optimize TCP window sizes and buffer settings for better network throughput.

### **Discussion and Best Practices (10 Minutes)**

#### 1. **Discuss the Results**:
   - Compare the speeds seen in FTP vs. iperf3.
   - Discuss network bottlenecks (e.g., local machine limitations, TCP/IP configurations).

#### 2. **Best Practices**:
   - Use **binary mode** for FTP to prevent file corruption.
   - Keep FTP **anonymous login disabled** for security.
   - Implement **firewall rules** to restrict FTP access.
   - Use passive mode ports for better firewall compatibility.
   - Regularly **monitor bandwidth** usage with tools like `iperf3` and `iftop`.
   - Consider **encryption** (e.g., FTPS) for secure file transfers if used beyond local testing.
