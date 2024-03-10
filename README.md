
# Kubernetes The Hard Way 

This guide is not for people looking for a fully automated command to bring up a Kubernetes cluster.
Kubernetes The Hard Way is optimized for learning, which means taking the long route to ensure you understand each task required to bootstrap a Kubernetes cluster.

This tutorial is a modified version of the original developed by [Kelsey Hightower](https://github.com/kelseyhightower/kubernetes-the-hard-way).
While the original one uses GCP as the platform to deploy kubernetes,  we use ubuntu servers  to deploy a cluster . If you prefer the cloud version, refer to the original one [here](https://github.com/kelseyhightower/kubernetes-the-hard-way)


Please note that with this particular challenge, it is all about the minute detail. If you miss one tiny step anywhere along the way, it's going to break!

## Target Audience

The target audience for this tutorial is someone planning to support a production Kubernetes cluster and wants to understand how everything fits together.

## Cluster Details

Kubernetes The Hard Way guides you through bootstrapping a highly available Kubernetes cluster with end-to-end encryption between components and RBAC authentication.

* [Kubernetes](https://github.com/kubernetes/kubernetes) Latest version
* [Container Runtime](https://github.com/containerd/containerd) Latest version
* [Weave Networking](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/)
* [etcd](https://github.com/coreos/etcd) v3.5.9
* [CoreDNS](https://github.com/coredns/coredns) v1.9.4

### Node configuration

We will be building the following:

* Two control plane nodes (`master-1`, `master-2` and `master-3`) running the control plane components as operating system services.
* Two worker nodes (`worker-1`, `worker-2` and `worker-3`)
* One loadbalancer VM running [HAProxy](https://www.haproxy.org/) to balance requests between the two API servers.

## Content

* [Requirements](01_requirements.md)<br>
* [Create Certificates](02_create_certificates.md)</br>
* [Install common tools](03_common_tools.md)
* [Load balancer configuration](04_load_balancer_haproxy.md)
* [Generating Kubernetes Configuration Files for Authentication](05_config_files.md)
* [Bootstrapping the etcd Cluster](06_etcd_configuration.md)
* [Bootstrapping the Kubernetes Control Plane](07_controllers_configuration.md)
* [Bootstrapping the Kubernetes Workers ](08_worker_configuration.md)
* [Provisioning Pod Network](09_pod_networking.md)
* [Common Configs ](10_common_configs.md)</br>

