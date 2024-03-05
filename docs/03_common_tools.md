# Installing common Tools

```bash
sudo apt update
sudo apt-get install dnsutils # We need this tool for netstat -tulpn command .(To check ports which our services will use)

```

# Installing Container Runtime on the Kubernetes Nodes

 CRI is a standard interface for the management of containers. Since v1.24 the use of dockershim has been fully deprecated and removed from the code base. [containerd replaces docker](https://kodekloud.com/blog/kubernetes-removed-docker-what-happens-now/) as the container runtime for Kubernetes, and it requires support from [CNI Plugins](https://github.com/containernetworking/plugins) to configure container networks, and [runc](https://github.com/opencontainers/runc) to actually do the job of running containers.

Reference: https://github.com/containerd/containerd/blob/main/docs/getting-started.md

### Download and Install Container Networking

The commands in this lab must be run on each instance. Login to each controller instance using SSH Terminal.

Here we will install the container runtime `containerd` from the Ubuntu distribution and  CNI tools from the Kubernetes distribution. 

Set up the Kubernetes `apt` repository

```bash
{
  sudo mkdir -p /etc/apt/keyrings
  curl -fsSL https://pkgs.k8s.io/core:/stable:/${KUBE_LATEST}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

  echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/${KUBE_LATEST}/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
}
```

Install `containerd` and CNI tools, first refreshing `apt` repos to get up to date versions.

```bash
{
  sudo apt update
  sudo apt install -y containerd kubernetes-cni ipvsadm ipset
}
```

Set up `containerd` configuration to enable systemd Cgroups

```bash
{
  sudo mkdir -p /etc/containerd

  containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sudo tee /etc/containerd/config.toml
}
```

Now restart `containerd` to read the new configuration

```bash
sudo systemctl restart containerd
```

## Install kubectl

The [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl) command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

Reference: [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

We will be using `kubectl` early on to generate `kubeconfig` files for the controlplane components.

### Linux

```bash

curl -LO "https://dl.k8s.io/release/$KUBE_VERSION/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### Verification

Verify `kubectl` is installed:

```
kubectl version -o yaml
```

output will be similar to this, although versions may be newer:

```
kubectl version -o yaml
clientVersion:
  buildDate: "2023-11-15T16:58:22Z"
  compiler: gc
  gitCommit: bae2c62678db2b5053817bc97181fcc2e8388103
  gitTreeState: clean
  gitVersion: v1.28.4
  goVersion: go1.20.11
  major: "1"
  minor: "28"
  platform: linux/amd64
kustomizeVersion: v5.0.4-0.20230601165947-6ce0bf390ce3

The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

Don't worry about the error at the end as it is expected. We have not set anything up yet!

Prev: [Requirements of Nodes](01_requirements.md)</br>
Next: [Bootstrapping the Kubernetes Worker Nodes](10-bootstrapping-kubernetes-workers.md)
