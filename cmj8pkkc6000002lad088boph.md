---
title: "Day 17: Kubernetes Troubleshooting"
seoTitle: "Application Failure, Control Plane Failure"
datePublished: Tue Dec 16 2025 14:59:13 GMT+0000 (Coordinated Universal Time)
cuid: cmj8pkkc6000002lad088boph
slug: day-17-kubernetes-troubleshooting
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1765641602358/2bed0cea-9ab4-404e-a5d0-cd816300857e.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1765897133614/f7dc013c-5248-479c-add4-39913a21229b.png
tags: k8s-series

---

## Application Failure

User is unable to access the 2-tier application.

User → web (web-service) → DB (DB-service).

**Debugging**

1. Try to access the application via URL Ex: Curl http://web-service-ip:node-port, if its not working check the services.
    
2. Check the service.
    
    ```bash
    $ Kubectl describe svc web-service # look at selector or Endpoint details.
    ```
    
3. Look at Pod details
    
    ```bash
    $ Kubectl describe po <pod-name>
    # Look at the pod object file for selector names, status, and restart count. 
    $ kubectl logs web -f      #check application logs.
    # we can't see previous pod logs, use --previous
    $ kubectl logs web -f --previous
    ```
    
4. Check DB Pod logs and Status.
    

## Control Plane Failure

```bash
$ kubectl get nodes
$ kubectl get pods
$ kubectl get
$ kubectl get pods -n kube-system      # incase cluster deployed with Kubeadm tool 
$ service kube-apiserver status         # if we deploy control plane components deployed as service.
$ service controller-manager status 
$ service kube-scheduler status  
$ kubectl logs kube-apiserver-master -n <namespace-name>
$ sudo journalctl -u kube-apiserver    # service configured natively on master node.
```