# Tips & Trick
<br>

### Install kubectl and control your cluster from MacOS:
<br>

Install Kubectl:<br>
<br>

```sh
brew install kubectl
```
<br>

```sh
mkdir -p /Users/{username}/.kube
```
<br>

Copy config file to MacOS:<br>
<br>

```sh
scp sysopmarcel@192.168.x.xxx:~/.kube/config ~/.kube/config
```
<br>

Test:<br>
<br>

```sh
kubectl get nodes
```
<br>

### Change roles label:
<br>

```sh
kubectl label node k8s-worker node-role.kubernetes.io/worker=worker
```
<br>

### Command to add node. Execute this on the control plane.
<br>

```sh
kubeadm token create --print-join-command --ttl 0
```
<br>
Use output to add node. So execute this command on the node to add.
<br>
<br>

### Install HomeBrew
<br>

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
<br>

```sh
#Homebrew pad toevoegen (voor ARM Macs = Apple Silicon)
eval "$(/opt/homebrew/bin/brew shellenv)"
```
<br>

### Add an alias to shorten the kubectl command

add the folowing to ~/.zhsrc

```sh
alias k='kubectl'
```
<br>

#### Show ETCD options

```sh
sudo cat /etc/kubernetes/manifests/etcd.yaml
```
<br>

#### Show Kube-scheduler options

```sh
sudo cat /etc/kubernetes/manifests/kube-scheduler.yaml
```
<br>

#### Show Kube-controller options

```sh
sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml 
```
<br>

#### Check if Kubelet is runing and show options

```sh
ps -aux | grep kubelet
```