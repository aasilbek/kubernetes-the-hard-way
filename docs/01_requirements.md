
## Hardware Requirements

- 4 GB of RAM
- 2 CPU
- 40 GB disk space

In this example we get 7 instances (3 master nodes, 3 worker nodes and one loadbalancer)


## Software Requirements
- Ubuntu 20.04.6 LTS


## Access all VMs
Firstly we should choose one system from where we will perform administrative tasks. (creating certificates , config files. and distributing them to the other nodes. ) I chose master-1 node for this tasks. So my master-1 node must have  access for other nodes without password. 
I created SSH key pair on my master-1 node and put public key to ~/.ssh/authorized_keys  in all nodes. 
Also you can just put private key to your master-1 node if your cloud provider gives you private key. 


Generate SSH key pair on `master-1` node:

[//]: # (host:master-1)

```bash
ssh-keygen
```

Leave all settings to default by pressing `ENTER` at any prompt.


```bash
cat ~/.ssh/id_rsa.pub
```
Put this key to all nodes in this file  ~/.ssh/authorized_keys



Next: [Create Certificates](02_create_certificates.md)