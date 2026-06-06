# Tips & Trick
<br>

#### Install kubectl and control your cluster from MacOS.

<li>Install Kubectl</li>
<br>

```sh
brew install kubectl
```
<br>

```sh
mkdir -p /Users/{username}/.kube
```
<br>

<li>Copy config file to MacOS</li>
<br>

```sh
scp sysopmarcel@192.168.x.xxx:~/.kube/config ~/.kube/config
```
<br>

<li>Test</li>
<br>

```sh
kubectl get nodes
```
<br>

#### Change roles label
<br>

```sh
kubectl label node k8s-worker node-role.kubernetes.io/worker=
```
```sh
kubectl get nodes
```
<br>

#### Connect old k8s-worker to new control plane
<br>
Remove old configuration data

```sh
sudo kubeadm reset
```

```sh
sudo rm -rf /etc/cni/net.d
```

```sh
sudo rm -rf ~/.kube
```

```sh
sudo rm -rf /var/lib/kubelet/*
```
<br>
Done with wipping old configuration data
<br>
<br>

Command to add node. Execute this on the control plane.
<br>

```sh
kubeadm token create --print-join-command --ttl 0
```
<br>
Use output to add node. So execute this command on the node to add.
<br>
<br>


#### Install Bash 5.x, Bash Completion, KubeCtl on MacOS
<br>

##### Install HomeBrew
<br>

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
<br>

add the following to
<br>
```sh
nano ~/.bash_profile
```

```sh
#Homebrew pad toevoegen (voor ARM Macs = Apple Silicon)
eval "$(/opt/homebrew/bin/brew shellenv)"
```
<br>

#### Add an alias to shorten the kubectl command

add the folowing to ~/.bashrc

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