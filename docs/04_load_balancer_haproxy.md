
## The Kubernetes Frontend Load Balancer

In this section you will provision an external load balancer to front the Kubernetes API Servers. The `kubernetes-the-hard-way` static IP address will be attached to the resulting load balancer.


### Provision a Network Load Balancer

A NLB operates at [layer 4](https://en.wikipedia.org/wiki/OSI_model#Layer_4:_Transport_layer) (TCP) meaning it passes the traffic straight through to the back end servers unfettered and does not interfere with the TLS process, leaving this to the Kube API servers.

Login to `loadbalancer` instance using SSH Terminal.

[//]: # (host:loadbalancer)


```bash
sudo apt-get update && sudo apt-get install -y haproxy
```


Create HAProxy configuration to listen on API server port on this host and distribute requests evently to the two master nodes.

```bash
cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg
frontend kubernetes
    bind ${LOADBALANCER}:6443
    option tcplog
    mode tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server master-1 ${MASTER_1}:6443 check fall 3 rise 2
    server master-2 ${MASTER_2}:6443 check fall 3 rise 2
    server master-2 ${MASTER_3}:6443 check fall 3 rise 2
EOF
```

```bash
sudo systemctl restart haproxy
```

### Verification

[//]: # (sleep:2)

Make a HTTP request for the Kubernetes version info:

```bash
curl  https://${LOADBALANCER}:6443/version -k
```

> output

```
{
  "major": "1",
  "minor": "24",
  "gitVersion": "${KUBE_VERSION}",
  "gitCommit": "aef86a93758dc3cb2c658dd9657ab4ad4afc21cb",
  "gitTreeState": "clean",
  "buildDate": "2022-07-13T14:23:26Z",
  "goVersion": "go1.18.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

Prev: [Common tools](03_common_tools.md)<br>
Next: [Installing CRI on the Kubernetes Worker Nodes](09-install-cri-workers.md)