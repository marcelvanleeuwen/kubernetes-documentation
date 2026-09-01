# Configure Kubernetes Access on macOS

### This guide explains how to copy a Kubernetes configuration from a remote server to macOS and verify access to the cluster:

Prerequisites
Before continuing, make sure:
- kubectl is installed on the Mac.
- SSH access to 192.168.8.201 is available.
- The remote user has a Kubernetes configuration file at ~/.kube/config.
<br>

### Create the hidden .kube directory in the current user’s home directory. This directory is used to store the Kubernetes configuration file and related settings:

```sh
mkdir -p ~/.kube
```
<br>

### Copy the Kubernetes configuration file from the remote server at 192.168.8.201 to the local .kube directory on the MacBook using SCP:

```sh
scp sysopmarcel@192.168.8.201:~/.kube/config ~/.kube/config
```
<br>

### Secure the configuration file so that only the current user can read or modify it:

```sh
chmod 600 ~/.kube/config
```
<br>

### Verifies that kubectl can connect to the Kubernetes cluster and displays all cluster nodes along with their current status:

```sh
kubectl get nodes
```
<i>If the connection is successful, the cluster nodes are displayed. Healthy nodes normally show the status Ready.</i>