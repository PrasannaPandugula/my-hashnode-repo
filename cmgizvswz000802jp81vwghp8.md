---
title: "Day2: Kubernetes Architecture"
seoTitle: "Kubernetes architecture, master node, cluster node"
datePublished: Thu Oct 09 2025 05:46:29 GMT+0000 (Coordinated Universal Time)
cuid: cmgizvswz000802jp81vwghp8
slug: day2-kubernetes-architecture
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1759985891024/2aee6ac8-9726-4845-b6a1-a4d9f16db108.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1759988752879/31ce5f2b-861e-42c7-bf76-2fdae6e2cda1.png
tags: k8s-series

---

## Kubernetes Architecture

It follows master-worker (control- plane- node) structure that manages containerized applications across a cluster. The control plane oversees and schedules workloads, while worker nodes run the containers. This design ensures the scalability, high availability, and automated management of applications.

![](https://miro.medium.com/v2/resize:fit:700/1*NAiucG1RxJ-GMh3CBbfOxQ.png align="left")

## Kubernetes Master Node (Control Plane)

It receives inputs from CLI (command line interface), or UI (user interface) via an API. We define pods, replica sets, and services via YAML file to Kubernetes to maintain; it consists of multiple components.

### API Server

* It transmits data through HTTP using JSON.
    
* It processes requests, validates them, and instructs the relevant service or controller to update the state object in etcd.
    
* Allows user to configure workloads across the cluster.
    
* ```bash
          kubectl cluster-info  #(shows url of K8s server Ex: https://<api-server-ip>:6443)
          kubectl api-versions
          kubectl api-resources
    ```
    

### Scheduler

* Watch for new requests coming from the API Server and assign them to healthy nodes.
    
* Assesses nodes to select an unscheduled pod that should be placed on CPU and memory requests, policies, data locality, labels, and affinities of the workload.
    
* If there are no suitable nodes, the pods are put in a pending state until such a node appears.
    
    ```bash
    kubectl get pods -n kube-system | grep kube-schedular    #(check scheduler status)
    kubectl -n kube-system logs kube-scheduler-<node-name>   #(view scheduler logs)
    kubectl get pods -o wide                                 #(list scheduled nodes)
    ```
    

### Controller Manager

* It’s a single process that encompasses all of the controllers within Kubernetes.
    
* Controllers run as a single process in a DaemonSet to reduce complexity.
    
* It checks the current state of nodes it is tasked to control, and determines if there are any differences and resolves them, if any.
    
* ```bash
          kubectl -n kube-system describe pod kube-controller-manager-<node-name>  (CM-event, health check)
          kubectl -n kube-system logs kube-controller-manager-<node-name>          (view loogs)
          kubectl get pods -n kube-system | grep kube-controller-manager           (CM status check)
    ```
    

### **Etcd (Key-Value Store)**

* It is a database, Kubernetes uses to back up all cluster data.
    
* It stores the entire configuration and state of the cluster.
    
* The master node queries etcd to retrieve parameters for the state of the nodes, pods, and containers.
    
* ```bash
          kubectl get pods -n kube-system | grep ectd           (etcd pod status check)
          kubectl -n kube-system logs etcd-<node-name>          (view logs)
          kubectl -n kube-system describe pod etcd-<node-name>  (describe etcd pod)
    ```
    

## Kubernetes Worker Node (Data Plane)

Worker node listens to api server for new work assignments; they execute the work assignments and then report the result back to the Kubernetes master node.

### Kubelet

* Kubelet runs on every node in the cluster, also watches tasks sent from the API server, executes tasks, and communicates information about the state and health of containers to Kubernetes.
    
* It also monitors the pods and reports back to the control plane if pod is not fully functional.
    
* By installing kubelet, the nodes’ CPU, RAM, and storage become part of broader cluster.
    
* ```bash
          kubectl get nodes
          kubectl describe node <node-name>
          sudo journalctl -u kubelet -f       (Run on node, kubelet isnot managed as pod)
    ```
    

### Kube-proxy

* It ensures that each node gets its IP address, implements local iptables and rules to handle routing, traffic, and load-balancing.
    
* It allows network communication between servers and pods, and is responsible for routing network traffic.
    
* ```bash
          kubectl get pods -n kube-system | grep kube-proxy             (check proxy status)
          kubectl -n kube-system logs kube-proxy-<node-name>            (view logs)
          kubectl -n kube-system describe pod kube-proxy-<node-name>    (describe kube-proxy pod)
    ```
    

### **Container Runtime**

* It pulls images from container registry and starts and stops containers; a 3rd party plugin, such as docker performs this action.
    
* Popular container runtime examples include containerd, Docker, and CRI-O.
    
* ```bash
          #Docker                         #containerd                        #CRI-O
          docker ps                       crictl ps                          crictl ps
          docker --version                containerd --version               crio --version
          sudo journalctl -u docker -f    sudo journalctl -u containerd -f   sudo journalctl -u crio -f  (debug containers)
    ```
    

## Conclusion

In today’s blog, we learn about the architecture of Kubernetes and its components with useful kubectl commands, in the coming days, we will focus on pods, set of Kubernetes in different servers, and deployment scripts.