
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
    server master-3 ${MASTER_3}:6443 check fall 3 rise 2

frontend proxy-http
    mode tcp
    option tcplog
    bind *:80
    use_backend proxy-http

backend proxy-http
    mode tcp
    balance roundrobin
    server worker-1 ${WORKER_1}:80 check
    server worker-2 ${WORKER_2}:80 check
    server worker-3 ${WORKER_3}:80 check

frontend proxy-https
    mode tcp
    option tcplog
    bind *:443
    use_backend proxy-https

backend proxy-https
    mode tcp
    balance roundrobin
    server worker-1 ${WORKER_1}:443 check
    server worker-2 ${WORKER_2}:443 check
    server worker-3 ${WORKER_3}:443 check
EOF
```

```bash
sudo systemctl restart haproxy
sudo systemctl status haproxy
```

Prev: [Common tools](03_common_tools.md)<br>
Next: [Generating Kubernetes Configuration Files for Authentication](05_config_files.md)