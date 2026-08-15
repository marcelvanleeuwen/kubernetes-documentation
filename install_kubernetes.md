# Install Kubernetes on bare metal:

### Before you begin:

- A compatible Linux host. This guide uses Ubuntu Server 26.04 LTS.
- 2GB or more RAM
- 2 CPUs or more for control plane machines
- Full network connectivity
- Unique hostname, MAC address
- Required Kubernetes ports must be open between the control-plane and worker nodes
<br>

### Preferred Operating System: 

Ubuntu Server 26.04 LTS
<br>

### Install Ubuntu Server:

Perform the following steps on every control-plane and worker node before initializing or joining the cluster.

- Install Ubuntu Server
- Configure a static IP address during installation
- Set the timezone
  
  ```sh
  sudo timedatectl set-timezone Europe/Amsterdam
  ```

- Reboot
- Login
- Update Ubuntu Server
<br>

### Update Ubuntu Server:

```sh
sudo apt-get update && sudo apt-get upgrade -y 
```
Reboot the node if a kernel update was installed.
<br>

### Set hostname:

```sh
sudo hostnamectl set-hostname "new-hostname"
```
<i>example: k8s-cp-1</i>
<br>

Check the hostname
<br>

```sh
hostname
```
<br>

### Disable Linux Swap:

```sh
sudo swapoff -a
```
```sh
sudo sed -i '/\sswap\s/ s/^\(.*\)$/#\1/g' /etc/fstab
```
```sh
swapon --show
```
If the command returns no output, swap is disabled successfully.
<br>

### Set up container runtime prerequisites:

Load the required kernel modules and configure them to load automatically after a reboot.
<br>

```sh
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf > /dev/null
```

```sh
sudo modprobe overlay
```

```sh
sudo modprobe br_netfilter
```
<br>

### Configure required sysctl parameters. These persist across reboots:

```sh
echo -e "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1" | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf > /dev/null
```
<br>

### Apply the sysctl parameters without rebooting:

```sh
sudo sysctl --system
```
<br>

### Install containerd:

```sh
sudo apt-get update
sudo apt-get install -y containerd
```
<br>

### Configure containerd:

Check whether SystemdCgroup is set to true.
<br>

```sh
sudo containerd config dump | grep SystemdCgroup
```
<br>

If the output is not SystemdCgroup = true, run the following commands.
<br>

```sh
sudo mkdir -p /etc/containerd
```
```sh
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
```
```sh
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```
<br>

Enable and start containerd, then verify the active configuration.
<br>

```sh
sudo systemctl enable containerd
sudo systemctl restart containerd
```
<br>
```sh
sudo containerd config dump | grep SystemdCgroup
```
<br>

### Add Kubernetes repository:

This guide uses Kubernetes v1.36. If you use a different Kubernetes minor version, update the repository URL accordingly.
<br>

Add the repository. Adjust the version if necessary.
<br>

```sh
sudo apt-get install -y ca-certificates curl gpg
```
```sh
sudo mkdir -p -m 755 /etc/apt/keyrings
```
```sh
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```sh
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
<br>

Verify that the Kubernetes repository was added successfully.
<br>

```sh
cat /etc/apt/sources.list.d/kubernetes.list
```

<i>example: deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /</i>
<br>

Update the package index.
<br>

```sh
sudo apt update
```
<br>

### Install Kubernetes packages:

```sh
sudo apt install -y kubelet kubeadm kubectl
```
```sh
sudo apt-mark hold kubelet kubeadm kubectl
```
```sh
sudo systemctl enable --now kubelet
```
Run the following command only on the first control-plane node.
```sh
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
The pod network CIDR 10.244.0.0/16 is used by Flannel. Do not use a range that overlaps with your local network.

After the control-plane node is initialized successfully, kubeadm displays a kubeadm join command. Save this command, as it is required to add worker nodes to the cluster. The join token expires after twenty-four hours; if it has expired, generate a new join command on the control-plane node.

On the control-plane node, configure kubectl for your regular user.
<br>
```sh
mkdir -p $HOME/.kube
```
```sh
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
```sh
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
<br>

If you are logged in as the root user, set the following environment variable instead.
<br>
```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
```

<br>
<i>example: error: error loading config file "/etc/kubernetes/admin.conf": open /etc/kubernetes/admin.conf: permission denied</i><br>
<br>
If you accidentally set KUBECONFIG as a regular user, you may get a permission error. Remove it with:
<br>

```sh
unset KUBECONFIG
```
<br>

### Add a worker node to the cluster:

Complete all node preparation steps on the worker node before joining it to the cluster.

If you saved the kubeadm join command displayed during kubeadm init, use that command on the worker node.

If you no longer have the command, or if its token has expired, generate a new join command on the control-plane node:

```sh
sudo kubeadm token create --print-join-command
```
Copy the generated kubeadm join command and run it on the worker node to add it to the cluster.

On the control-plane node or management machine, verify that the worker node joined the cluster:

```sh
kubectl get nodes
```