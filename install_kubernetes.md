# Install Kubernetes on bare metal:
<br>
### Install kubectl on macOS (management machine):
<br>
<br>

```sh
brew install kubectl
```
```sh
kubectl version --client
```
<br>
Check which shell you are using

```sh
echo $SHELL
```
<br>

If you use zsh:

```sh
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
source ~/.zshrc
```
<br>

Creates a k alias for kubectl.
<br>

```sh
echo 'alias k=kubectl' >> ~/.zshrc
source ~/.zshrc
```
<br>

### Before you begin:
<ul>
  <li>A compatible Linux host. This guide uses Ubuntu Server 26.04 LTS.</li>
  <li>2GB or more RAM</li>
  <li>2 CPUs or more for control plane machines</li>
  <li>Full network connectivity</li>
  <li>Unique hostname, MAC address</li>
  <li>Required Kubernetes ports must be open between the control-plane and worker nodes</li>
</ul>
<br>

### Preferred Operating system: 

Ubuntu Server LTS 26.04
<br>

### Install Ubuntu Server:

Perform the following steps on every control-plane and worker node before initializing or joining the cluster
<ul>
  <li>Install Ubuntu Server</li>
  <li>Configure a static ip address during installation</li>
  <li>Set the timezone</li>
  
  ```sh
  sudo timedatectl set-timezone Europe/Amsterdam
  ```

  <li>Reboot</li>
  <li>Login</li>
  <li>Update Ubuntu Server</li>
</ul>
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

Check hostname
<br>

```sh
hostname
```
<br>

### disable linux swap:

```sh
sudo swapoff -a
```
```sh
sudo sed -i '/\sswap\s/ s/^\(.*\)$/#\1/g' /etc/fstab
```
```sh
swapon --show
```
If the command returns no output, swap is disabled successfully
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

<li>Check whether SystemdCgroup is set to true</li>
<br>

```sh
sudo containerd config dump | grep SystemdCgroup
```
<br>

<li>If the output is not SystemdCgroup = true, run the following commands</li>
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

<li>Check / if true proceed futher / if not true do the steps above</li>
<br>

```sh
sudo systemctl enable --now containerd
sudo systemctl restart containerd
```
<br>
```sh
sudo containerd config dump | grep SystemdCgroup
```
<br>

### Add kubernetes repository:

This guide uses Kubernetes v1.36. If you use a different Kubernetes minor version, update the repository URL accordingly.
<br>

Add repo, adjust version.
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
if you set by accident "export KUBECONFIG=/etc/kubernetes/admin.conf" and are not root user you get an error.<br>
<br>
<i>example: error: error loading config file "/etc/kubernetes/admin.conf": open /etc/kubernetes/admin.conf: permission denied</i><br>
<br>
you can fix this by executing the following command:
<br>

```sh
unset KUBECONFIG
```
<br>

### Install the Flannel CNI plugin:

Flannel is a simple CNI plugin suitable for homelabs, learning environments, and small Kubernetes clusters. For       production environments, consider alternatives such as Cilium or Calico, depending on your networking and security requirements.
<br>

```sh
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
<br>
Verify that Flannel is running.
<br>

```sh
kubectl get nodes
```
All nodes should show the status Ready.

```sh
kubectl get pods -n kube-flannel
```
All Flannel pods should show the status Running.
<br>

### Uninstall Flannel:

```sh
kubectl delete -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
<br>

### Install MetalLB:

MetalLB is a commonly used load balancer implementation for bare-metal Kubernetes clusters. Kubernetes does not include a load balancer for bare-metal environments by default. MetalLB provides external IP addresses for Services of type LoadBalancer.
<br>

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.16.1/config/manifests/metallb-native.yaml
```
<br>
Verify the installed MetalLB version.
```sh
kubectl get deployment -n metallb-system controller -o jsonpath='{.spec.template.spec.containers[0].image}'
```
<br>

Configure a MetalLB IP address pool.

Choose a free IP range from your local network that is outside your DHCP pool. Replace <YOUR_FREE_IP_RANGE> in the manifest below before applying it.

Create the file on the management machine where kubectl is configured. In this guide, the file is stored at ~/kubernetes-manifests/metallb-config.yaml.

```sh
mkdir -p ~/kubernetes-manifests
```
```sh
nano ~/kubernetes-manifests/metallb-config.yaml
```
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadatßa:
  name: lan-pool
  namespace: metallb-system
spec:
  addresses:
    - <YOUR_FREE_IP_RANGE>
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: lan-advertisement
  namespace: metallb-system
```

```sh
kubectl apply -f ~/kubernetes-manifests/metallb-config.yaml
```

```sh
kubectl get ipaddresspools -n metallb-system
```

The output should show the lan-pool IP address pool. MetalLB can now assign addresses from the configured range to Services of type LoadBalancer.
<br>

### Test MetalLB with an example service:

Deploy a test application and expose it using a Service of type LoadBalancer:
```sh
kubectl create deployment nginx --image=nginx
```
```sh
kubectl expose deployment nginx --type=LoadBalancer --port=80
```

Verify that MetalLB assigned an external IP address:
```sh
kubectl get services
```
The nginx service should show an EXTERNAL-IP from the IP address range configured in lan-pool.

To remove the test application after verification:
```sh
kubectl delete service nginx
```
```sh
kubectl delete deployment nginx
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

```sh
kubectl get nodes
```

After the control-plane node is initialized successfully, kubeadm displays a kubeadm join command. Save this command, as it is required to add worker nodes to the cluster. The join token expires after 24 hours; if it has expired, generate a new join command on the control-plane node.
