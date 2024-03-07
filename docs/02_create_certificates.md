# Set environment variables
Create .env file and put:
```bash
MASTER_1=
MASTER_2=
MASTER_3=
WORKER_1=
WORKER_2=
WORKER_3=
LOADBALANCER=
POD_CIDR=10.244.0.0/16
SERVICE_CIDR=10.96.0.0/16
API_SERVICE=$(echo $SERVICE_CIDR | awk 'BEGIN {FS="."} ; { printf("%s.%s.%s.1", $1, $2, $3) }')
CLUSTER_DNS=$(echo $SERVICE_CIDR | awk 'BEGIN {FS="."} ; { printf("%s.%s.%s.10", $1, $2, $3) }')
KUBE_VERSION=$(curl -L -s https://dl.k8s.io/release/stable.txt)
INTERNAL_IP=$(ip addr show eth0 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```
Copy this file to all nodes and run this commands:
```bash
cat .env  >> ~/.bashrc 
source ~/.bashrc
```
# Update hosts
Run this commands on master nodes
```bash
echo $MASTER_1 "master-1" >> /etc/hosts
echo $MASTER_2 "master-2" >> /etc/hosts
echo $MASTER_3 "master-3" >> /etc/hosts
echo $WORKER_1 "worker-1" >> /etc/hosts
echo $WORKER_2 "worker-2" >> /etc/hosts
echo $WORKER_3 "worker-3" >> /etc/hosts
echo $LOADBALANCER "loadbalancer" >> /etc/hosts

```
If you have /etc/cloud/templates/hosts.debian.tmpl file run this commands also on master nodes:
```bash
echo $MASTER_1 "master-1" >> /etc/cloud/templates/hosts.debian.tmpl
echo $MASTER_2 "master-2" >> /etc/cloud/templates/hosts.debian.tmpl
echo $MASTER_3 "master-3" >> /etc/cloud/templates/hosts.debian.tmpl
echo $WORKER_1 "worker-1" >> /etc/hosts
echo $WORKER_2 "worker-2" >> /etc/hosts
echo $WORKER_3 "worker-3" >> /etc/hosts
echo $LOADBALANCER "loadbalancer" >> /etc/cloud/templates/hosts.debian.tmpl
```
Copy .env file to other nodes.
```bash
{
  for instance in master-2 master-3 loadbalancer  ; do
      scp  -o StrictHostKeyChecking=no .env ${instance}:~/.env
      ssh -o StrictHostKeyChecking=no ${instance}  'cat ~/.env  >> ~/.bashrc && source ~/.bashrc '
      
  done
}

```
# Provisioning a CA and Generating TLS Certificates

In this lab you will provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) using the popular openssl tool, then use it to bootstrap a Certificate Authority, and generate TLS certificates for the following components: etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, and kube-proxy.

# Where to do these?

You can do these on any machine with `openssl` on it. But you should be able to copy the generated files to the provisioned VMs. Or just do these from one of the master nodes.

In our case we do the following steps on the `master-1` node, as we have set it up to be the administrative client.

[//]: # (host:master-1)

## Certificate Authority

In this section you will provision a Certificate Authority that can be used to generate additional TLS certificates.

Create a CA certificate, then generate a Certificate Signing Request and use it to create a private key:
```bash
mkdir -p /var/lib/kubernetes/pki/
cd /var/lib/kubernetes/pki/
```

```bash
{
  # Create private key for CA
  openssl genrsa -out ca.key 2048

  # Create CSR using the private key
  openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA/O=Kubernetes" -out ca.csr

  # Self sign the csr using its own private key
  openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial -out ca.crt -days 1000
}
```

Results:

```
ca.crt
ca.key
```

Reference : https://kubernetes.io/docs/tasks/administer-cluster/certificates/#openssl

The `ca.crt` is the Kubernetes Certificate Authority certificate and `ca.key` is the Kubernetes Certificate Authority private key.
You will use the `ca.crt` file in many places, so it will be copied to many places.

The `ca.key` is used by the CA for signing certificates. And it should be securely stored. In this case our master node(s) is our CA server as well, so we will store it on master node(s). There is no need to copy this file elsewhere.

## Client and Server Certificates

In this section you will generate client and server certificates for each Kubernetes component and a client certificate for the Kubernetes `admin` user.

### The Admin Client Certificate

Generate the `admin` client certificate and private key:

```bash
{
  # Generate private key for admin user
  openssl genrsa -out admin.key 2048

  # Generate CSR for admin user. Note the OU.
  openssl req -new -key admin.key -subj "/CN=admin/O=system:masters" -out admin.csr

  # Sign certificate for admin user using CA servers private key
  openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out admin.crt -days 1000
}
```

Note that the admin user is part of the **system:masters** group. This is how we are able to perform any administrative operations on Kubernetes cluster using `kubectl` utility.

Results:

```
admin.key
admin.crt
```

The `admin.crt` and `admin.key` file gives you administrative access. We will configure these to be used with the `kubectl` tool to perform administrative functions on Kubernetes.

### The Kubelet Client Certificates

We are going to skip certificate configuration for Worker Nodes for now. We will deal with them when we configure the workers.
For now let's just focus on the control plane components.

### The Controller Manager Client Certificate

Generate the `kube-controller-manager` client certificate and private key:

```bash
{
  openssl genrsa -out kube-controller-manager.key 2048

  openssl req -new -key kube-controller-manager.key \
    -subj "/CN=system:kube-controller-manager/O=system:kube-controller-manager" -out kube-controller-manager.csr

  openssl x509 -req -in kube-controller-manager.csr \
    -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-controller-manager.crt -days 1000
}
```

Results:

```
kube-controller-manager.key
kube-controller-manager.crt
```


### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:


```bash
{
  openssl genrsa -out kube-proxy.key 2048

  openssl req -new -key kube-proxy.key \
    -subj "/CN=system:kube-proxy/O=system:node-proxier" -out kube-proxy.csr

  openssl x509 -req -in kube-proxy.csr \
    -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-proxy.crt -days 1000
}
```

Results:

```
kube-proxy.key
kube-proxy.crt
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:



```bash
{
  openssl genrsa -out kube-scheduler.key 2048

  openssl req -new -key kube-scheduler.key \
    -subj "/CN=system:kube-scheduler/O=system:kube-scheduler" -out kube-scheduler.csr

  openssl x509 -req -in kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-scheduler.crt -days 1000
}
```

Results:

```
kube-scheduler.key
kube-scheduler.crt
```

### The Kubernetes API Server Certificate

The kube-apiserver certificate requires all names that various components may reach it to be part of the alternate names. These include the different DNS names, and IP addresses such as the master servers IP address, the load balancers IP address, the kube-api service IP address etc.

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```bash
cat > openssl.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[v3_req]
basicConstraints = critical, CA:FALSE
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster
DNS.5 = kubernetes.default.svc.cluster.local
IP.1 = ${API_SERVICE}
IP.2 = ${MASTER_1}
IP.3 = ${MASTER_2}
IP.4 = ${MASTER_3}
IP.5 = ${LOADBALANCER}
IP.6 = 127.0.0.1
EOF
```

Generate certs for kube-apiserver

```bash
{
  openssl genrsa -out kube-apiserver.key 2048

  openssl req -new -key kube-apiserver.key \
    -subj "/CN=kube-apiserver/O=Kubernetes" -out kube-apiserver.csr -config openssl.cnf

  openssl x509 -req -in kube-apiserver.csr \
    -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-apiserver.crt -extensions v3_req -extfile openssl.cnf -days 1000
}
```

Results:

```
kube-apiserver.crt
kube-apiserver.key
```

# The Kubelet Client Certificate

This certificate is for the API server to authenticate with the kubelets when it requests information from them

```bash
cat > openssl-kubelet.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[v3_req]
basicConstraints = critical, CA:FALSE
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth
EOF
```

Generate certs for kubelet authentication

```bash
{
  openssl genrsa -out apiserver-kubelet-client.key 2048

  openssl req -new -key apiserver-kubelet-client.key \
    -subj "/CN=kube-apiserver-kubelet-client/O=system:masters" -out apiserver-kubelet-client.csr -config openssl-kubelet.cnf

  openssl x509 -req -in apiserver-kubelet-client.csr \
    -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver-kubelet-client.crt -extensions v3_req -extfile openssl-kubelet.cnf -days 1000
}
```

Results:

```
apiserver-kubelet-client.crt
apiserver-kubelet-client.key
```


### The ETCD Server Certificate

Similarly ETCD server certificate must have addresses of all the servers part of the ETCD cluster

The `openssl` command cannot take alternate names as command line parameter. So we must create a `conf` file for it:

```bash
cat > openssl-etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = ${MASTER_1}
IP.2 = ${MASTER_2}
IP.3 = ${MASTER_3}
IP.4 = 127.0.0.1
EOF
```

Generates certs for ETCD

```bash
{
  openssl genrsa -out etcd-server.key 2048

  openssl req -new -key etcd-server.key \
    -subj "/CN=etcd-server/O=Kubernetes" -out etcd-server.csr -config openssl-etcd.cnf

  openssl x509 -req -in etcd-server.csr \
    -CA ca.crt -CAkey ca.key -CAcreateserial -out etcd-server.crt -extensions v3_req -extfile openssl-etcd.cnf -days 1000
}
```

Results:

```
etcd-server.key
etcd-server.crt
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as describe in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

Generate the `service-account` certificate and private key:

```bash
{
  openssl genrsa -out service-account.key 2048

  openssl req -new -key service-account.key \
    -subj "/CN=service-accounts/O=Kubernetes" -out service-account.csr

  openssl x509 -req -in service-account.csr \
    -CA ca.crt -CAkey ca.key -CAcreateserial -out service-account.crt -days 1000
}
```

Results:

```
service-account.key
service-account.crt
```

## Distribute the Certificates

Copy the appropriate certificates and private keys to each instance:
```bash
{
for instance in  master-2 master-3  ; do
    ssh -o StrictHostKeyChecking=no ${instance}  sudo mkdir -p /var/lib/kubernetes/pki
    scp  -o StrictHostKeyChecking=no /var/lib/kubernetes/pki/* ${instance}:/var/lib/kubernetes/pki/
done

for instance in worker-1 worker-2 worker-3; do
    ssh -o StrictHostKeyChecking=no $instance sudo mkdir -p /var/lib/kubernetes/pki
    scp -o StrictHostKeyChecking=no /var/lib/kubernetes/pki/{ca.crt,kube-proxy.crt,kube-proxy.key} \
    $instance:/var/lib/kubernetes/pki/
done

}
```


Prev: [Requirements](01_requirements.md)<br>
Next: [Install common tools](03_common_tools.md)