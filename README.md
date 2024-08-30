# Kubernetes Cluster Setup on EC2 Instances

This guide provides steps to set up a Kubernetes cluster with one control plane node and one worker node on EC2 instances. 

## Prerequisites

1. **EC2 Instances**:
   - **Control Node**: `t2.medium`
   - **Worker Node**: `t2.micro`

2. **Operating System**: Ubuntu

## Setup Instructions

### 1. Prepare Both Control and Worker Nodes

On both the control node and worker node:

1. **Disable Swap**:
   ```bash
   sudo swapoff -a
   sudo sed -i '/swap/s/^/#/' /etc/fstab
   ```
### Load Kernel Modules:
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

```
### Configure Sysctl Parameters:

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```

### Configure Sysctl Parameters:
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
```
### Install Containerd:

```
sudo apt-get update
sudo apt update -y
sudo apt-get install containerd -y
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i "s/SystemdCgroup = false/SystemdCgroup = true/g" /etc/containerd/config.toml
sudo systemctl restart containerd
```
### Install Kubernetes Components:

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```
## Initialize the Control Node

On the control node:

Initialize the Kubernetes Cluster:

`sudo kubeadm init --pod-network-cidr=192.168.0.0/16`

Set Up kubectl:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install Calico Network Add-on:
`kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml`

Create Join Command:
`kubeadm token create --print-join-command`

Copy the output of this command. It will look something like this:
`sudo kubeadm join <control-plane-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>`

## 3. Join the Worker Node
On the worker node, use the join command obtained from the control node:

Run the Join Command:
`sudo kubeadm join <control-plane-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>`

## 4. Verify the Cluster
On the control node:

Check Node Status:
`kubectl get nodes`

# Output:

![image](https://github.com/user-attachments/assets/3ea31872-76f8-42a0-9763-274e5b4854f0)
![image](https://github.com/user-attachments/assets/2cfd49e2-6684-41a3-8968-30188146b695)


control node
![image](https://github.com/user-attachments/assets/8012d008-86eb-4573-b164-ac80a5ac5a94)
![image](https://github.com/user-attachments/assets/a4636a9b-656c-4ea7-9685-d62dbfa206be)
![image](https://github.com/user-attachments/assets/de169e6e-532c-4a4a-ad85-90503071690f)



worker node
![image](https://github.com/user-attachments/assets/b13d42e7-15a4-4ba7-ae79-bce032b55874)
![image](https://github.com/user-attachments/assets/13aa2cec-da0d-432b-9446-6bfd61866626)


# TASK 2: COnfiguration of Grafana-server

# Installing Grafana on Ubuntu

This guide provides instructions to install Grafana on an Ubuntu machine using the APT repository.

## Prerequisites

Before installing Grafana, ensure that you have `apt-transport-https`, `software-properties-common`, and `wget` installed on your system.

## Installation Steps

### 1. Install Prerequisite Packages

Run the following command to install the required packages:

```bash
sudo apt-get install -y apt-transport-https software-properties-common wget
```
### 2. Import the GPG Key
Create a directory for the GPG key and import it:

```
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```

### 3. Add the Grafana Repository
For Stable Releases
Add the stable release repository:

```echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

For Beta Releases (Optional)
If you want to use beta releases, add the beta release repository:

`echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list`

## 4. Update the Package List
Update the list of available packages:

`sudo apt-get update`

## 5. Install Grafana
To Install Grafana OSS
Run the following command to install the latest Open Source Software (OSS) release:
`sudo apt-get install grafana-enterprise`

To Install Grafana Enterprise
Run the following command to install the latest Enterprise release:
`sudo apt-get install grafana-enterprise`

## 6. Start and Enable Grafana Service
After installation, start and enable the Grafana service:

```
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

## 7. Access Grafana
Grafana runs on port 3000 by default. You can access it in your web browser at:

`http://<your-server-ip>:3000`

The default login credentials are:
```
Username: admin
Password: admin
```
You will be prompted to change the password upon first login.

# Now, After this you will see the Grafana Dashboard

![image](https://github.com/user-attachments/assets/0c35ba05-6a64-4daa-97d4-d422848348b0)
![image](https://github.com/user-attachments/assets/14491e31-d151-4189-b40c-ffb8e2d086ed)
![image](https://github.com/user-attachments/assets/b33f5e90-2f14-4b0a-bb63-28e9c2561649)
![image](https://github.com/user-attachments/assets/641c21e5-7989-436b-8496-84035d757258)
![image](https://github.com/user-attachments/assets/03cc98fa-87e4-45fb-b0b8-258517bbc52f)






