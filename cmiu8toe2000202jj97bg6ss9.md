---
title: "Day 14: Networking in Kubernetes"
seoTitle: "Network Basics, Docker Networking"
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