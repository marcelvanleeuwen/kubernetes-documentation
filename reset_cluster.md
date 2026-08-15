# Reset Kubernetes cluster:
<br>
Warning: These commands permanently remove Kubernetes cluster configuration and local workload data from the node.


### Run the following steps on every control-plane and worker node:

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

Run this command only on control-plane nodes, as etcd data is stored there.

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

### Remove CNI network interfaces:

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

### Clean iptables:<br>

Warning: The following commands remove all iptables rules on the node, including rules unrelated to Kubernetes.

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
<br>

### Reboot each node:

```sh
sudo reboot
```