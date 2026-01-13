---
title: "Setting Up the Headlamp Dashboard for Kubernetes"
seoTitle: "Headlamp Kubernetes dashboard, Kubernetes dashboard alternative"
datePublished: Tue Jan 13 2026 07:07:04 GMT+0000 (Coordinated Universal Time)
cuid: cmkc917tn000e02l1bfq7hp2n
slug: setting-up-the-headlamp-dashboard-for-kubernetes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1768287564962/05f5297e-26a5-42e4-af95-215384005b9e.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1768287946638/aa88acf4-45e1-4af5-886d-8f2a65932b74.png
tags: techblogwithprasanna

---

### Introduction

With the official Kubernetes Dashboard now retired, the need for a modern, user-friendly, and secure Kubernetes UI has become more important than ever. Headlamp, an open-source Kubernetes web UI, emerges as a powerful alternative that offers real-time cluster visibility, extensibility, and a developer-friendly experience.

[https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)

## Headlamp Integration with Cluster

### Install Headlamp using Helm

1. Add the helm repositories.
    

```plaintext
helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
```

2. Install headlamp
    

```plaintext
helm install my-headlamp headlamp/headlamp --namespace kube-system
```

**Note:** kube-system namespace is managed by Kubernetes only admins have access to it and we usually won’t install regular pods in this namespace.

Once the installation done you would see below details.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1768285517493/86be0bb9-0834-43af-aeb0-15cb066254cd.png align="center")

3. Check Pod Status.
    
    ```bash
    $ kubectl get pods -n kube-system
    ```
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1768285657337/b1b0954b-e135-43b3-944a-da0c520eedf3.png align="center")

### Service Account Creation

Once Headlamp is up and running, be sure to enable access to it by creating service account.

1. Create a Service Account.
    

```bash
kubectl -n kube-system create serviceaccount headlamp-admin
```

2. Grant permissions to service account.
    

```bash
kubectl create clusterrolebinding headlamp-admin --serviceaccount=kube-system:headlamp-admin --clusterrole=cluster-admin
```

3. Create a token, to access Headlamp dashboard.
    

```bash
kubectl create token headlamp-admin -n kube-system
```

### Access Headlamp Dashboard

1. Port-forward the Headlamp pod to view the UI in your browser.
    

```bash
kubectl port-forward -n kube-system service/my-headlamp 9090:80 --address 0.0.0.0
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1768286424150/e9b2f943-f176-404c-b681-44fd4af3ef87.png align="center")

2. Go to browser run below URL.
    

```bash
http://localhost:9090
```

3. Headlamp was running, asking us to enter the token. Generate token by running create token command mentioned above.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1768286523143/a01287c2-e3be-4cea-b938-ee21382e28c1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1768286693024/ab00cd04-64dc-4f06-8394-80bb8dde3457.png align="center")

4. After entering the token, we gained access to the Headlamp dashboard and its cluster components.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1768286816061/54b7f9c1-87c6-4e45-8483-841b647c809f.png align="center")

Explore the workload details.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1768286951637/346fed0d-c82f-4d64-ad2b-53e9153fa9e6.png align="center")

## Conclusion

Headlamp offers a modern, user-friendly way to manage Kubernetes clusters. With easy installation and browser access, it provides a lightweight, real-time UI for all your workloads.