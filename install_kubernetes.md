# Install Kubernetes bare metal
<br>

#### Before you begin
<br>
<ul>
  <li>A compatible Linux host (I use the prefered OS Ubuntu LTS 26.04 because most</li>
  <li>Kubernetes development is done on Ubuntu)</li>
  <li>2GB or more RAM</li>
  <li>2 CPUs or more for control plane machines</li>
  <li>Full network connectivity</li>
  <li>Unique hostname, MAC address</li>
  <li>Certain ports open</li>
</ul>
<br>

#### Prefered OS 
<br>
Ubuntu Server LTS 26.04 (<i>because most Kubernetes development is done on Ubuntu</i>)
<br>
<br>

#### Install Ubuntu Server
<br>
<ul>
  <li>Install Ubuntu Server</li>
  <li>Set static ip during installation</li>
  <li>Set time</li>
  
  ```sh
  sudo timedatectl set-timezone Europe/Amsterdam
  ```

  <li>Reboot</li>
</ul>
<ul>
<li>Login</li>
<li>Update Ubuntu Server</li>
</ul>
<br>

#### Update Ubuntu Server
<br>

```sh
sudo apt-get update && sudo apt-get upgrade -y 
```
<br>

#### Set hostname
<br>

```sh
sudo hostnamectl set-hostname "new-hostname"
```
<i>example: k8s-cp-1</i>
<br>
<br>

<li>Check hostname</li>
<br>

```sh
hostname
```
<br>

#### disable linux swap and remove any existing swap partitions
<br>

```sh
sudo swapoff -a
```
```sh
sudo sed -i '/\sswap\s/ s/^\(.*\)$/#\1/g' /etc/fstab
```
<br>

#### Setting up container runtime prereq
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

#### Setup required sysctl params, these persist across reboots.
<br>

```sh
echo -e "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1" | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf > /dev/null
```
<br>

#### Apply sysctl params without reboot
<br>

```sh
sudo sysctl --system
```
<br>

#### Install containerd
<br>

```sh
sudo apt-get install docker.io containerd
```
<br>


#### Set containerd config
<br>

<li>Check if SystemdCgroup = true</li>
<br>

```sh
sudo containerd config dump | grep SystemdCgroup
```
<br>

<li>if false do</li>
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
```sh
sudo containerd config dump | grep SystemdCgroup
```
<br>

<li>if true proceed futher</li>
<br>

```sh
sudo systemctl start containerd
```
<br>

#### Install kubernetes packages
<br>

<li>check latest release</li>
<br>

```sh
curl -L -s https://dl.k8s.io/release/stable.txt
```
<br>

<li>add repo, adjust version</li>
<br>

```sh
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```sh
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
<br>

<li>check if repo is added</li>
<br>

```sh
cat /etc/apt/sources.list.d/kubernetes.list
```

<i>example: deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /</i>
<br>
<br>

<li>Update repositories</li>
<br>

```sh
sudo apt update
```
<br>

#### Install Kubernetes
<br>

```sh
sudo apt install -y kubelet kubeadm kubectl
```
```sh
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
<br>
<br>

To start using your cluster, you need to run the following as a regular user:
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

Alternatively, if you are the root user, you can run:
<br>
```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
```
<br>

if you set by accident "export KUBECONFIG=/etc/kubernetes/admin.conf" and are not root user you get an error.
<br>
<i>
example: error: error loading config file "/etc/kubernetes/admin.conf": open /etc/kubernetes/admin.conf: permission denied
</i>
<br>
you can fix this by executing the following command:
<br>
```sh
unset KUBECONFIG
```

#### Install CNI (Flannel) via manifest
<br>

```sh
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
<br>
<li>Check if Flannel is running</li>
<br>

```sh
kubectl get pods -n kube-flannel
```
<br>

#### Install MetalLB via manifest
<br>

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.16.1/config/manifests/metallb-native.yaml
```
<br>

<li>Check version installed</li>
<br>

```sh
kubectl get deployment -n metallb-system controller -o jsonpath='{.spec.template.spec.containers[0].image}'
```
<br>

#### Add worker to control plane
<br>

```sh
kubeadm join (command showed during kubadm init)
```
<i>example: kubeadm join 192.168.1.3:6443 --token s2qk11.mjmkdghtjklicwqrquz9l \
	--discovery-token-ca-cert-hash sha256:c8183f87425a00639c345cv7fc247fe5g3h7324cbe80e02e3601ca5135dc66</i>
<br>

#### Change roles label
<br>

```sh
kubectl label node k8s-worker-1 node.role.kubernetes.io/worker=worker
```
