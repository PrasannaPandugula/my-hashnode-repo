---
title: "Day 14: Networking in Kubernetes"
seoTitle: "Network Basics, Docker Networking, Explore Existing K8s Cluster"
datePublished: Sat Dec 06 2025 12:01:39 GMT+0000 (Coordinated Universal Time)
cuid: cmiu8toe2000202jj97bg6ss9
slug: day-14-networking-in-kubernetes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1765000996466/caa06e2b-41ab-431a-bbb5-9f54969f4e77.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1765022427615/529d98f3-e500-4fe1-a191-901d42c08c32.png
tags: k8s-series

---

## Network Basics

**Switching**: Through the switching, we can only enable communication between systems within the network; it can receive packets on the host network and deliver them to other systems within the same network.

```bash
$ ip link                      # check system interface
```

**Routing**: A System present in one network can reach a system in another network. To connect a system to the internet, add a new routing table to the network.

```bash
$ route                 # To check existing route details.
```

**Gateway**: Routing traffic from one network to another network.

```bash
$ ip route add 192.168.9.0/24 via 192.168.1.1    #Routing sys traffic via gateway (192.168.1.1)
```

**Commands**:

```bash
$ ip link                                    #list and modify interfaces on host
$ ip addr                                    # see IP address assigned to interfaces
$ ip addr add 192.168.1.10/24 dev eth0       #add the ip address to interface; these changes persist until system restarts
$ ip route or route                          # To view routing table.
$ ip route add 192.168.1.1                   # add entries to the routing table.
$ cat /proc/sys/net/ipv4/ip_forward          # To view ip_forward enabled or not(if host configured as a router)
```

## Docker Networking

Let’s understand different types of networking in Docker before diving into Kubernetes.

### None Network

With none network, a Docker container can not reach the outside world, and can’t access anyone from outside.

```bash
$ docker run --network none nginx
```

### Host Network

A container attached to the host network; there is no isolation between network and the container.

If a container uses the host network, a web app running on port 80 inside it is directly available on port 80 of the host. But you can't run another instance on the same port, because they all share the host's network stack.

```bash
$ docker run --network host ngnix
```

### Bridge Network (default)

When we install Docker, it creates a bridge network between Docker and the host.

```bash
$ docker netwrork ls                # see network type
$ ip link                           # on host bridge network name is docker0
```

When we create a container, Docker creates a network interface

```bash
$ docker run nginx
$ ip netns
```

Container gets attached to the bridge by creating virtual cable with two an interfaces.

```bash
$ ip link                      #see interface name
$ ip -n <namespace-name> link  #see other interface name   
$ curl http://172.17.0.3:80    #other containers in same host can access the negix app, cause we deployed in private instance
$ docker run nginx -p 8080:80 nginx   #external users can use app via port 8080.
```

How does traffic forward from one port to another port? Using iptables, we create an entry into NAT table.

```bash
$ iptables \
        -t nat \
        -A PREROUTING \
        -j DNAT \
        --dport 8080 \
        --to-destination 172.17.0.3:80
#list rules
$ iptables -nvl -t nat
```

## Explore Existing K8s Cluster

Check how many nodes are part of the cluster.

```bash
$ kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   6m20s   v1.34.0
node01         Ready    <none>          5m38s   v1.34.0
```

Get IP addresses of the nodes.

```bash
$ kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE     VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
controlplane   Ready    control-plane   8m23s   v1.34.0   192.168.56.177    <none>        Ubuntu 22.04.5 LTS   5.15.0-1083-gcp   containerd://1.6.26
node01         Ready    <none>          7m41s   v1.34.0   192.168.129.196   <none>        Ubuntu 22.04.5 LTS   5.15.0-1083-gcp   containerd://1.6.26
```

Identify the network interface of the control plane.

```bash
$ ip link
4: eth0@if11956: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default 
link/ether 6a:f6:a0:a4:33:d5 brd ff:ff:ff:ff:ff:ff link-netnsid 0
5: flannel.1: <BROADCAST,MULTICAST,UP,
# etho0
# MAC address 6a:f6:a0:a4:33:d5
```

Check MAC address of the node.

```bash
$ SSH node01    # ssh into worker node
$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: eth0@if410: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default 
    link/ether ba:dd:4c:4a:e9:af brd ff:ff:ff:ff:ff:ff link-netnsid 0
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UNKNOWN mode DEFAULT group default 
    link/ether a2:eb:86:fb:38:57 brd ff:f
# MAC add: ba:dd:4c:4a:e9:af
```

Check the interface/bridge created by `Containerd` on the `controlplane` node

```bash
$ ip link     #network interface that is a bridge created by Containerd, it starts with cni0
6: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UP mode DEFAULT group default qlen 1000
$ ip link show cni0      # check status of cni0
# interface: cni0
```

Suppose, ping google from the `controlplane` node, which route does it take? What is the IP address of the Default Gateway?

```bash
$ ip route show default
default via 169.254.1.1 dev eth0 
```

Check port numbers listening on the controlplane node.

```bash
$ netstat -nplt
$ netstat -nplt | grep scheduler 
tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      2907/kube-scheduler 
```

Check, Etcd is listening to multiple ports, which of these have more client connections?

```bash
$ netstat -anp | grep etcd | grep 2379 | wc -l
45
$ netstat -anp | grep etcd | grep 2380 | wc -l
1
```