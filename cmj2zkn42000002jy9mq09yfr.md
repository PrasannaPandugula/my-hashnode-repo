---
title: "Day 15: Networking in Kubernetes"
seoTitle: "Pod and Service Networking, Kubernetes CNI"
datePublished: Fri Dec 12 2025 14:52:36 GMT+0000 (Coordinated Universal Time)
cuid: cmj2zkn42000002jy9mq09yfr
slug: day-15-networking-in-kubernetes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1765266551521/1a848a96-7030-4d67-b436-70987a03894d.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1765551088890/4e28f47a-891a-4d9a-9ba2-74eacb8cf741.png
tags: k8s-series

---

## Pod Networking

To implement Pod Networking, K8s had a standard networking model approach.

**Networking Model**

1. Every POD should have an IP address.
    
2. Every POD should be able to communicate with every other POD in the same node.
    
3. Every POD should be able to communicate with every other POD on other nodes without NAT.
    

Kubernetes does not provide Pod networking, so it has to be implemented manually or use 3rd party CNI (Container Network Interface) plugins.

**CNI Plugins**: flannel, cilium, NSX(vmware).

## Configure CNI in K8s

CNI defines the responsibilities of container runtime; as per CNI Kubernetes is responsible for creating container network namespaces.

Kubernetes does below steps:

1. Container Runtime must create a network namespace.
    
2. Identify the network the container must attach to.
    
3. Container Runtime to invoke Network Plugin (bridge) when container is **added**.
    
4. Container Runtime to invoke Network Plugin (bridge) when container is **deleted**.
    
5. JSON format of the Network Configuration.
    

**How Container Runtime Accesses CNI Plugins:**

1. The CNI plugin must be invoked by the component within K8s that is responsible for creating containers.
    
2. Container Runtime (container**d**, cri-o) is responsible for creating containers.
    

**View CNI Configuration**

1. Network plugins are installed under **/opt/cni/bin** directory, container looks into this directory.
    
2. Which plugin and how to use configured in the below directories.
    
3. ```bash
       $ ls /etc/cni/net.d         #If multiple files are there, it will choose one in alphabetical order.
       10-bridge.conflist         
       10-flannel.conflist             #plugin to use
    ```
    

**View CNI Options**

```bash
$ cat /etc/cni/net.d/net-script.confg             # cni, network plugin cong file
{
  "cniVersion": "0.4.0",
  "name": "calico-network",
  "type": "calico",
  "etcd_endpoints": "http://127.0.0.1:2379",
  "log_level": "info",
  "ipMasq": true,
  "ipam": {                                 
    "type": "local-host",                         #CNI plugin to manage diff IP's
    "subnet": "10.244.0.0/16"
    "routes" [
     {"dst": "0.0.0.0/0"}
   ]
  },
  "policy": {
    "type": "k8s"
  },
  "kubernetes": {
    "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
  }
}
```

### IPAM (IP Address Management)

* Who assigns IPs to Pods
    
* Who are the virtual bridge networks in the nodes assigned IPs?.
    
* How to avoid duplicate IPs.
    
* Where is this info stored?
    

All these are taken care of by CNI, it has different plugins, such as host-local, DHCP, and we should add them to our script.

### Cleanup CNI plugin Flannel

To clean up Flannel CNI from the cluster, delete the Flannel DaemonSet, configuration file, as well as the ConfigMap hosting the Flannel configuration:

```bash
kubectl delete daemonset -n kube-flannel kube-flannel-ds
kubectl delete cm kube-flannel-cfg -n kube-flannel
rm /etc/cni/net.d/10-flannel.conflist
```

The Flannel CNI plugin does not support network policies.

### Install Calico CNI

Calico plugin that supports Network Policies, let’s see a sample example, we stop frontend app from communicating with the backend app.

1. First, install the Calico operator:
    
    ```bash
    #install CRD's not required for latest version.
    $ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.0/manifests/operator-crds.yaml # not required for latest version
    
    # Directly run this cmd
    $ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.2/manifests/tigera-operator.yaml
    ```
    
2. Download the custom resource definition yaml
    
    ```bash
    $ curl https://raw.githubusercontent.com/projectcalico/calico/v3.31.2/manifests/custom-resources.yaml -O
    ```
    
3. Edit the custom-resources.yaml and update the cidr field
    
    ```yaml
    $ ls      # do ls on same directory, you see the file, edit cidr
    custom-resources.yaml
    $ vim custom-resources.yaml 
    apiVersion: operator.tigera.io/v1
    kind: Installation
    metadata:
      name: default
    spec:
      calicoNetwork:
        ipPools:
        - name: default-ipv4-ippool
          blockSize: 26
          cidr: 172.17.0.0/16                  # update cidr
          encapsulation: VXLANCrossSubnet
          natOutgoing: Enabled
          nodeSelector: all()
    ---
    apiVersion: operator.tigera.io/v1
    kind: APIServer
    metadata:
      name: default
    spec: {}
    ```
    
    ```bash
    # after file edit, run below commands. 
    $ kubectl apply -f custom-resources.yaml
    $ watch kubectl get pods -A
    NAMESPACE         NAME                                       READY   STATUS    RESTARTS
     AGE
    calico-system     calico-apiserver-7d7d4656fb-4nqpk          1/1     Running   0
     57s
    calico-system     calico-apiserver-7d7d4656fb-khknp          1/1     Running   0
     57s
    default           backend                                    1/1     Running   0
     35m
    default           frontend                                   1/1     Running   0
    ```
    
    4. Redeploy your applications.
        
        ```bash
        controlplane ~ ✖ kubectl get pods -o wide
        NAME       READY   STATUS    RESTARTS   AGE     IP             NODE           NOMINATED NODE   READINESS GATES
        backend    1/1     Running   0          2m19s   172.17.49.72   controlplane   <none>           <none>
        frontend   1/1     Running   0          2m20s   172.17.49.71   controlplane   <none>           <none>
        ```
        
    5. ```bash
           $ kubectl exec -it frontend -- curl -m 5 172.17.49.72  # front end app not able to access backend app since we applied network policies via calcio.
           curl: (28) Connection timed out after 5002 milliseconds
           command terminated with exit code 28
           # backend app ip: 172.17.49.72
        ```
        
    
    Refer to the official doc: [https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)
    

## Service Networking

We allow Pods to access services on another Pod using Service Network; we rarely configure direct communication between Pods.

How IP addresses get assigned to Nodes and services made available to external users.

* On every node, the kubelet monitors the kube-apiserver for workload changes. When the scheduler assigns a Pod to that node, the kubelet launches the Pod’s containers and invokes the CNI plugin to set up the Pod’s networking.
    
* Every node runs **kube-proxy**, which listens for changes from the **kube-apiserver**. When a Service is created, kube-proxy updates the networking rules on all nodes so that traffic can reach the right Pods across the cluster.
    

### **Components**

**ClusterIP:** Internal Pod-Pod communication is a default.

**NodePort**: Exposes service on each node’s IP for external access.

LoadBalancer:

Check IP address and subnet mask assigned to the controlplane node's primary network interface.

```bash
$ ip addr show eth0
4: eth0@if38947: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 3a:eb:aa:28:a9:30 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.138.195/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::38eb:aaff:fe28:a930/64 scope link 
       valid_lft forever preferred_lft forever
# this is IP 192.168.138.195/32
```

Check the range of IP addresses configured for PODs on this cluster.

```bash
cat /etc/kubernetes/manifests/kube-controller-manager.yaml   | grep <cluster-name>
```

Check IP Range configured for the services within the cluster.

```bash
cat /etc/kubernetes/manifests/kube-controller-manager.yaml   | grep cluster-ip-range
```

What type of proxy is the `kube-proxy` configured to use

```bash
kubectl logs -n kube-system <kube-proxy-pod-name>
```

## DNS in Kubernetes

When we set up a cluster, Kubernetes deploys a built-in DNS server, In the cluster all POD’s and Services gets there own IP address.

1. We create a service to access application. Whenever a service is created, K8s DNS creates a record for the service, it maps the service name to IP address.
    

DNS Name for Service:

```bash
$ curl http://web-service.apps.svc.custer.local  #
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1765451856199/9f3cc8a7-4224-4d43-ad15-de7ebabcd05c.png align="left")

### Core DNS in Kubernetes

1. Two pods in the cluster communicate with each other, we add their IP address to host file.
    
    ```bash
    $ cat >> /etc/hosts
      web 10.244.2.5       # Pod2 address
    $ cat >> /etc/hosts
      web 10.244.1.5       # Pod1 address
    ```
    
2. When we create thousands of Pod each minute, this solution does not solve our problem; we will move all records to DNS server file (10.96.0.10).
    
3. ```bash
     $ cat >> /etc/resolv.conf
       name   10.96.0.10
    ```
    

Identify the DNS solution implemented in the cluster.

```bash
kubectl get pods -n kube-system
```

Where is the configuration file located for configuring the CoreDNS service?

```bash
$ kubectl -n kube-system describe deployments.apps coredns | grep -A2 Args | grep Corefile
```

How is the Corefile passed into the CoreDNS POD?

```bash
$ kubectl describe configmap coredns -n kube-system  # Passed as kubernetes
```