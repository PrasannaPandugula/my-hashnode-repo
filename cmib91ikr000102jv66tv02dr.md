---
title: "Day8: Kubernetes Security Overview"
seoTitle: "Kubernetes Security, TLS certificate, Network Policies "
datePublished: Sun Nov 23 2025 05:00:07 GMT+0000 (Coordinated Universal Time)
cuid: cmib91ikr000102jv66tv02dr
slug: day8-kubernetes-security-overview
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1763870508249/d12e2bf7-41ff-4305-8dd4-ae1ceb629f05.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1763873797198/df986d63-da80-43ec-9eec-d440073bd157.png
tags: k8s-series

---

## Security Primitives

Kubernetes is the go-to platform for hosting production-grade applications, and security is of prime concern.

* All access to the host must be secured, so Root access and password-based authentication are disabled; only SSH key-based authentication is made available.
    

### Types to Secure kube-apiserver

* it is the center of all operations within Kubernetes; it interacts with kubectl by accessing the api directory to perform any operation in cluster, so we need to have controlling access to the kube-apiserver.
    

Let’s focus on these aspects.

**Authentication:** Who can access the cluster?

1\. Static Token File.

2\. Certificates.

3\. LDAP- External Authentication Provider.

4\. Service Accounts - for machines

**Authorization**: What can they do?

1\. RBAC Authorization (Role-Based Access Control).

2\. ABAC Authorization (Attributed-Based Access Control).

3\. Node Authorization.

4\. Webhook Mode.

### TLS Certificates

* Communication between cluster and worker node components, like etcd Cluster, Kubectl, Kube Controller Manager, Kube-Scheduler, and Kube-Proxy, is secured using TLS encryption.
    

### Network Policies

* Communication between applications within clusters.
    
* By default, all pods can access all other pods in the cluster. We can restrict access using network policies.
    

## Summary

So far, we have covered the key Kubernetes components and the essential ways to secure them, and we’ll explore an in-depth explanation of each component in upcoming blogs.