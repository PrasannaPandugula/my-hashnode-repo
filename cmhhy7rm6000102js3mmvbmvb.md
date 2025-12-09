---
title: "Day 5: Services and Networking in K8s"
seoTitle: "Kubernetes, Ingress, DNS, Service Discovery"
datePublished: Sun Nov 02 2025 16:51:44 GMT+0000 (Coordinated Universal Time)
cuid: cmhhy7rm6000102js3mmvbmvb
slug: day-5-services-and-networking-in-k8s
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1762099473524/2147a4e7-54a8-4ede-a421-48dd4084fded.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1762102268726/4bcc812f-6d2a-4f21-87d5-b32beeb9389d.png
tags: k8s-series

---

## Introduction

* When we deploy an application in Kubernetes, our Pods are not directly accessible from the outside world. To make them reachable, Kubernetes provides two key networking components: **Services** and **Ingress**.
    

## Types of Services

We use different services to expose pods

### ClusterIP

* we can access deployed applications within the cluster.
    
* ```yaml
            apiVersion: v1
            kind: Service
            metadata:
              name: my-app-service
            spec:
              selector:
                app: my-app
              ports:
                - port: 80         # service port
                  targetPort: 8080 # container port
              type: ClusterIP    # we change based on requirement
    ```
    

```bash
kubectl get svc -n <namespace>
kubectl describe svc my-app-service  # Verify clusterIP, Ports, Endpoints
```

## NodePort

* Expose the app outside the cluster via NodeIp Ex: 30080
    
* Useful for testing or small setups Ex: Kind, Minikube
    
* Traffic path → User → NodePort → ClusterIP → Pod
    

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-nodeport
spec:
  selector:
    app: my-app
  ports:
    - port: 80         # service port (inside cluster)
      targetPort: 8080 # container port (in pod)
      nodePort: 30080  # external port on each node (range: 30000–32767)
  type: NodePort
```

* Test external access using node’s public IP (EC2, VM)
    
* curl http://&lt;NodePublicIP&gt;:40080
    

### LoadBalancer

* Create a cloud load balancer (AWS, Azure, GCP etc..)
    
* Route eternal traffic → LoadBalancer → NodePort → ClusteIP → Pod
    
* Automatically gets public external ip for internet access.
    

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-lb
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

Test: curl http://&lt;EXTERNAL-IP&gt;

## Ingress

* Ingress works like a **traffic controller** or **gateway** that manages **HTTP/HTTPS routing** to multiple Services.
    
* Instead of using separate NodePorts or LoadBalancers for each app, we can route traffic based on **hostnames** or **paths**.
    

### Service vs Ingress

* **Service**: it handles internal networking and ensures stable IPs and Pods.
    
* **Ingress**: it provides smart routing SSL termination, and single-entry access for multiple apps.
    

## Kubernetes DNS and Service Discovery

In a traditional world, apps need IP addresses to communicate. However, in Kubernetes, Pod IPs **change frequently** whenever a Pods restarts or reschedules.

### Service Discovery

* Applications automatically communicate with each other without hard-coding IPs.
    
* In Kubernetes, every **Service** automatically gets a **stable DNS name** and a **cluster-internal IP**.
    

### How it Works

* Kubernetes uses an internal DNS called coreDNS (a built-in add-on)
    
* Whenever we create a service, Kubernetes automatically updates DNS record for it.
    
* Ex: Service name: backend = backend.default.svc.cluster.local \[DNS name\]
    

Kubernetes DNS + Service Discovery = Reliable internal communication between microservices.