Here's a bash script to install and start **Minikube**, **Docker**, and **Helm** on Ubuntu 22.04, ensuring all components are up and running:

```bash
#!/bin/bash

# Function to print status messages
print_status() {
  echo "===================================================="
  echo "$1"
  echo "===================================================="
}

# Update the system
print_status "Updating the system"
sudo apt update && sudo apt upgrade -y

# Install dependencies for Minikube and Docker
print_status "Installing dependencies"
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Install Docker
print_status "Installing Docker"
sudo apt remove docker docker-engine docker.io containerd runc
sudo apt install -y docker.io

# Enable and start Docker service
print_status "Enabling and starting Docker service"
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

# Install Minikube
print_status "Installing Minikube"
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64

# Start Minikube with Docker driver
print_status "Starting Minikube with Docker driver"
minikube start --driver=docker --force

# Install Helm
print_status "Installing Helm"
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install -y apt-transport-https --no-install-recommends
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt update
sudo apt install -y helm

# Verify installations
print_status "Verifying Docker installation"
docker --version
print_status "Verifying Minikube installation"
minikube version
print_status "Verifying Helm installation"
helm version

# Check the status of Minikube
print_status "Checking Minikube status"
minikube status

# Test Minikube cluster
print_status "Deploying test application to Minikube"
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
kubectl expose deployment hello-minikube --type=NodePort --port=8080

# Display the Minikube IP and the test service URL
MINIKUBE_IP=$(minikube ip)
NODE_PORT=$(kubectl get service hello-minikube --output=jsonpath='{.spec.ports[0].nodePort}')
echo "Test application is accessible at: http://$MINIKUBE_IP:$NODE_PORT"

print_status "Setup complete. Docker, Minikube, and Helm are up and running!"
```

### Steps Taken by the Script
1. **Update System**: Ensures the system is up-to-date.
2. **Install Dependencies**: Installs the required dependencies.
3. **Install Docker**:
   - Removes any existing Docker installations.
   - Installs Docker and enables it to start at boot.
4. **Install Minikube**:
   - Downloads and installs Minikube.
   - Starts Minikube using Docker as the driver.
5. **Install Helm**: Installs Helm for Kubernetes package management.
6. **Verify Installations**: Checks the installed versions of Docker, Minikube, and Helm.
7. **Deploy Test Application**: Deploys a simple test application to Minikube to verify it is working correctly.

### Running the Script
Save the script to a file named `install_minikube_docker_helm.sh` and make it executable:

```bash
chmod +x install_minikube_docker_helm.sh
```

Run the script:

```bash
./install_minikube_docker_helm.sh
```

### Additional Notes
- You may need to **log out and log back in** after the Docker installation for group changes to take effect.
- If you encounter any permission issues with Docker, try `sudo chmod 666 /var/run/docker.sock`.
