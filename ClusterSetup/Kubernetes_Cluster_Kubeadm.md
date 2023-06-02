---
# kubernetes Cluster-1.27 Setup using Kubeadm
---
  we will setup K8s Cluster on AWS using Ubuntu Instances
 - Refer https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
 - here version 
    - Ubuntu: 
    - kubernetes Cluster: 1.27
    - 
 
To install Kubernetes using kubeadm on Ubuntu, we need to follow a series of steps. Here's a step-by-step guide for Kubernetes installation on Ubuntu using kubeadm

---
### Setup-1: Instance requirements
 ```
  Instances with 2 CPU & 2 GB RAM for MASTER NODE
  Instances with 1 CPU & 2 GB RAM for Worker NODE-1
  Instances with 1 CPU & 2 GB RAM for Worker NODE-2
  ```
#### Master NODE :
 - The t2.medium (AWS) instance type is suitable for running the master node in a Kubernetes cluster.The t2.medium instance offers a good balance of resources for a master node in most scenarios
   - Here are the specifications of the t2.medium instance type:
      - CPU: 2 vCPUs
      - Memory: 4 GB
      - Network Performance: Moderate
#### Worker NODE-1 and Worker NODE-2:
- For worker nodes in a Kubernetes cluster, a minimum configuration of 1 vCPU and 2 GB RAM is recommended.These resources are typically sufficient for running the workload containers on worker nodes.
- The t2.small instance type can be used as a worker node in a Kubernetes cluster, but it's important to consider the resource limitations of this instance type. Here are the specifications of the t2.small instance:
     - CPU: 1 vCPU
     - Memory: 2 GB
    - Network Performance: Low to Moderate
---
### Setup-2: Network connectivity(all machines are same VPC)
- To achieve full network connectivity between all machines in a Kubernetes cluster. Ensure that all machines in your Kubernetes cluster are in the same Virtual Private Cloud (VPC)

### Setup-3: Kubeadm Port Requirements
- Certain ports are open on your machines(https://kubernetes.io/docs/reference/ports-and-protocols/)
- On Master Node
    ```
    6443/tcp for Kubernetes API Server
    2379-2380 for etcd server client API
    6783/tcp,6784/udp for Weavenet CNI
    10248-10260 for Kubelet API, Kube-scheduler, Kube-controller-manager, Read-Only Kubelet API, Kubelet health
    30000-32767 for NodePort Services
    80,8080,443 Generic Ports
    ```
 - On Slave Nodes
    ```
    6783/tcp,6784/udp for Weavenet CNI
    10248-10260 for Kubelet API etc
    30000-32767 for NodePort Services
    ```
  ### Setup-4: Run on all nodes of the cluster as root user
  ### Setup-5: Disable SWAP(All the 3 nodes)
   - Disable swap in order for the kubelet to work properly
   ```
   swapoff -a
   sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
   ```
 ### Setup-6: Install Docker(All the 3 nodes)
 -  Docker as the container engine in your Kubernetes cluster
 -  Install Docker on all the machines (nodes) in your Kubernetes cluster. You can use the following commands to install Docker on Ubuntu
 - To install Docker on Ubuntu using the official Docker installation script
 - Update System Packages: Before installing Docker, update the system packages to ensure you have the latest versions. Open a terminal and run the following commands:
    ```
    sudo apt-get update
     sudo apt-get upgrade
      ```
      
 ### Setup-7: Download and Run the Docker Installation Script:(All the 3 nodes)
 - Download the Docker installation script using the following command:
   
  ```
    curl -fsSL https://get.docker.com -o get-docker.sh
  ```
  
  -  Verify the Integrity of the Script (Optional): It's a good practice to verify the integrity of the installation script. You can do this by downloading the PGP key and comparing its fingerprint with the expected fingerprint. Here's how you can do it:
  -  
   ```
       curl -fsSL https://get.docker.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```
    - Install Docker: Run the Docker installation script using the following command:
   ```
       sudo sh get-docker.sh
   ```
   
   - Verify Docker Installation: Verify that Docker is installed and running correctly by running the following command:
   
     ```
       docker version
       ```
### Setup-8: Install CNI : cri-dockerd(All the 3 nodes)    
- To install, on a Linux system that uses systemd, and already has Docker Engine installed 


```
# Run these commands as root
###Install GO###
wget https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x ./installer_linux
./installer_linux
source ~/.bash_profile

git clone https://github.com/Mirantis/cri-dockerd.git


cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket

```


