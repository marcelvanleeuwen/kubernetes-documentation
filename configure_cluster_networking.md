# Configure cluster networking

### Install the Flannel CNI plugin:

Flannel is a simple CNI plugin suitable for homelabs, learning environments, and small Kubernetes clusters. For production environments, consider alternatives such as Cilium or Calico, depending on your networking and security requirements.
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
<br>

All nodes should show the status Ready.

```sh
kubectl get pods -n kube-flannel
```
<br>

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
metadata:
  name: lan-pool
  namespace: metallb-system
spec:
  addresses:
    - "Free ip range" 
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: lan-advertisement
  namespace: metallb-system
```
<br>

<i>example: 192.168.5.0/24</i>
<br>

Save the file and exit the editor before applying the configuration.

```sh
kubectl apply -f ~/kubernetes-manifests/metallb-config.yaml
```
<br>

```sh
kubectl get ipaddresspools -n metallb-system
```
<br>

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

The EXTERNAL-IP may briefly show pending; wait until it shows an address from lan-pool.

The nginx service should show an EXTERNAL-IP from the IP address range configured in lan-pool.

To remove the test application after verification:
```sh
kubectl delete service nginx
```
```sh
kubectl delete deployment nginx
```
<br>