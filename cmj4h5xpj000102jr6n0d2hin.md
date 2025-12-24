---
title: "Day 16:  Design and Install a Kubernetes Cluster"
seoTitle: "Kubernetes Cluster Design, Storage, Hosting Production Applications"
datePublished: Sat Dec 13 2025 15:52:49 GMT+0000 (Coordinated Universal Time)
cuid: cmj4h5xpj000102jr6n0d2hin
slug: day-16-design-and-install-a-kubernetes-cluster
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1765612117097/6e4e9ff5-82ba-45cc-b7bd-5c80f4535a59.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1765641095088/49117744-e26a-424b-8139-943171ac65a8.png
tags: k8s-series

---

## Design a Kubernetes Cluster

### Purpose

**Learning:** Want to create a cluster for learning purposes? Use the following options.

* Minikube.
    
* Single-node cluster with kubeadm/GCP/AWS/Azure.
    

**Development and Testing**

* Multi-node cluster with a single master node and multiple workers.
    
* Set up using the Kubeadm tool or quick provisioning using GCP or AWS or AKS.
    

### Hosting Production Applications

* High-availability multi-node cluster with multiple master nodes.
    
* Kubeadm, GCP or Kops on AWS or other supported platforms.
    
* Up to 50,000 nodes.
    
* Up to 150,000 PODs in the cluster.
    
* Up to 300.000 total containers.
    
* Up to 100 Pods per node.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1765613518053/d357ff21-eecc-4f43-8e71-37053d691c49.png align="left")
    

### Storage

1. High Performance- SSD backend storage.
    
2. Multiple Concurrent connections- Network-based storage.
    
3. Persistent shared volumes for shared access across multiple storage.
    
4. Label nodes with specific Disk types.
    
5. Use node selector to assign applications to node with specific disk.
    

## Choosing Kubernetes Infrastructure

### Turnkey Solutions

* We Provision VMs
    
* We Configure VMs
    
* We use scripts to deploy the cluster
    
* We maintain VMs by ourselves
    
* Ex: Kubernetes on AWS using kops.
    

### Hosted Solutions (Managed Solutions)

* Kubernetes-as-a-Service
    
* Provider provisions VMs
    
* Provider installs Kubernetes
    
* Provider maintains VMS
    
* E.g: Google Kubernetes Engine (GKE)
    

## High Availability Kubernetes Cluster Setup

1. As part of the High Availability Cluster setup, we should maintain multiple Master Nodes in an active-active status.
    
2. We use a load balancer to split traffic between multiple master nodes Ex: Nginx or HA Proxy.
    
3. The Scheduler and Controller Managers continuously monitor the cluster state and take actions. When multiple instances are running, duplicate actions could occur, so they operate in an **active–standby** mode. The active instance is selected using a **leader election** mechanism, while the others remain on standby.
    
    ```bash
    $ kube-controller-manager --leader-elect true  # which ever node process request they become active and other become standby
                              --leader-elect-lease-durition 15s    # default time
                              --leader-elect-renew-deadline 10s    # default time
                              --leader-elect-retry-period  2s   # every 2s these nodes try to become leader inscase any of the node fails other can pickup request.
    # same process goes with Scheduler
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1765639654875/73b76570-ab38-4a1b-b89e-684e4554e1ed.png align="left")