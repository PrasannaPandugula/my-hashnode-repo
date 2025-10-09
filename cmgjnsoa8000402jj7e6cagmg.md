---
title: "Day3 Deep Dive into Kubernetes Pods"
seoTitle: "Kubernetes pods, Containers, Trobleshhoting pods"
datePublished: Thu Oct 09 2025 16:55:53 GMT+0000 (Coordinated Universal Time)
cuid: cmgjnsoa8000402jj7e6cagmg
slug: day3-deep-dive-into-kubernetes-pods
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1760024484501/74af9289-9c05-4a64-90fe-0648d1738966.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1760028913164/b4cde345-0ebf-4783-8b49-2f297b41d3ec.png
tags: k8s-series

---

## What is a Pod?

A pod groups one or more containers and is a core building block block on Kubernetes architecture. Ex: K8s can automatically replace a pod when it goes down, add CPU and memory to it when it needed with in k8s cluster, or even replicate it to scale out. Pods are assigned IP addresses.

### Multi-container Pod

A pod that runs more then one container together on the same node, sharing the same network(IP and port space) and storage volumes, it is mainly used when containers need to work together, like part of same service that must share data or communicate via localhost.

### Pods Overview

![](https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg align="left")

### Pods Lifecycle and Troubleshooting Phases

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1760028732850/23a424b9-78a7-4a6b-8546-1a484480c9aa.png align="center")