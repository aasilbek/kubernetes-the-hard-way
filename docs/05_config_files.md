# Generating Kubernetes Configuration Files for Authentication

In this lab you will generate [Kubernetes configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), also known as "kubeconfigs", which enable Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

Note: It is good practice to use file paths to certificates in kubeconfigs that will be used by the services. When certificates are updated, it is not necessary to regenerate the config files, as you would have to if the certificate data was embedded. Note also that the cert files don't exist in these paths yet - we will place them in later labs.

User configs, like `admin.kubeconfig` will have the certificate info embedded within them.

## Client Authentication Configs

In this section you will generate kubeconfig files for the `controller manager`, `kube-proxy`, `scheduler` clients and the `admin` user.

### Kubernetes Public IP Address

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the load balancer will be used, so let's first get the address of the loadbalancer into a shell variable such that we can use it in the kubeconfigs for services that run on worker nodes. The controller manager and scheduler need to talk to the local API server, hence they use the localhost address.



### The kube-proxy Kubernetes Configuration File

Generate a kubeconfig file for the `kube-proxy` service:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=/var/lib/kubernetes/pki/ca.crt \
    --server=https://${LOADBALANCER}:6443 \
    --kubeconfig=/var/lib/kubernetes/kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=/var/lib/kubernetes/pki/kube-proxy.crt \
    --client-key=/var/lib/kubernetes/pki/kube-proxy.key \
    --kubeconfig=/var/lib/kubernetes/kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=/var/lib/kubernetes/kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=/var/lib/kubernetes/kube-proxy.kubeconfig
}
```

Results:

```
kube-proxy.kubeconfig
```

Reference docs for kube-proxy [here](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)

### The kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `kube-controller-manager` service:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=/var/lib/kubernetes/pki/ca.crt \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=/var/lib/kubernetes/pki/kube-controller-manager.crt \
    --client-key=/var/lib/kubernetes/pki/kube-controller-manager.key \
    --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig
}
```

Results:

```
kube-controller-manager.kubeconfig
```

Reference docs for kube-controller-manager [here](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)

### The kube-scheduler Kubernetes Configuration File

Generate a kubeconfig file for the `kube-scheduler` service:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=/var/lib/kubernetes/pki/ca.crt \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=/var/lib/kubernetes/kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=/var/lib/kubernetes/pki/kube-scheduler.crt \
    --client-key=/var/lib/kubernetes/pki/kube-scheduler.key \
    --kubeconfig=/var/lib/kubernetes/kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=/var/lib/kubernetes/kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=/var/lib/kubernetes/kube-scheduler.kubeconfig
}
```

Results:

```
kube-scheduler.kubeconfig
```

Reference docs for kube-scheduler [here](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)

### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=/var/lib/kubernetes/admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key \
    --embed-certs=true \
    --kubeconfig=/var/lib/kubernetes/admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=/var/lib/kubernetes/admin.kubeconfig

  kubectl config use-context default --kubeconfig=/var/lib/kubernetes/admin.kubeconfig
}
```

Results:

```
admin.kubeconfig
```

Reference docs for kubeconfig [here](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)


# Generating the Data Encryption Config and Key

Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to [encrypt](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data) cluster data at rest, that is, the data stored within `etcd`.

In this lab you will generate an encryption key and an [encryption config](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration) suitable for encrypting Kubernetes Secrets.

## The Encryption Key


Generate an encryption key:

```bash
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File

Create the `encryption-config.yaml` encryption config file:

```bash
cat > /var/lib/kubernetes/encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```




Reference: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#encrypting-your-data

##

## Distribute the Kubernetes Configuration Files

Copy the appropriate `kube-proxy` kubeconfig files to each worker instance:

```bash
for instance in worker-1 worker-2 worker-3; do
  scp  -o  StrictHostKeyChecking=no /var/lib/kubernetes/kube-proxy.kubeconfig ${instance}:/var/lib/kubernetes/
done
```

Copy kubeconfig files to each controller instance:

```bash
for instance in  master-2 master-3 ; do
  scp  -o  StrictHostKeyChecking=no /var/lib/kubernetes/*config* ${instance}:/var/lib/kubernetes/
done
```



Prev: [Load balancer configuration](04_load_balancer_haproxy.md)
Next: [Generating the Data Encryption Config and Key](06-data-encryption-keys.md)