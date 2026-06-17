# Reset Kubernetes cluster
<br>

On each node do:<br>
<br>

```sh
sudo kubeadm reset -f
```

```sh
sudo systemctl stop kubelet
```

```sh
sudo rm -rf /etc/kubernetes
```

```sh
sudo rm -rf /var/lib/etcd
```

```sh
sudo rm -rf /var/lib/kubelet
```

```sh
sudo rm -rf /var/lib/cni
```
```sh
sudo rm -rf /etc/cni
```

```sh
sudo rm -rf $HOME/.kube
```
<br>
<br>

Clean network:<br>
<br>

```sh
sudo ip link delete cni0 2>/dev/null
```

```sh
sudo ip link delete flannel.1 2>/dev/null
```

```sh
sudo ip link delete kube-ipvs0 2>/dev/null
```
<br>
<br>

Clean iptables:<br>
<br>

```sh
sudo iptables -F
```

```sh
sudo iptables -t nat -F
```

```sh
sudo iptables -t mangle -F
```

```sh
sudo iptables -X
```

```sh
sudo systemctl start containerd
```

```sh
sudo systemctl start kubelet
```
<br>
<br>

Reboot all nodes:<br>
<br>

```sh
sudo reboot
```