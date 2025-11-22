---
title: "Day7: Kubernetes Backup and Restore"
seoTitle: "Kubernetes backup strategies, etcd snapshot"
datePublished: Sat Nov 22 2025 18:30:40 GMT+0000 (Coordinated Universal Time)
cuid: cmiamk1p5000102jshdfc08gi
slug: day7-kubernetes-backup-and-restore
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1763807272576/5fcc902e-a72c-4d20-a30e-7bc078c2d296.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1763826748686/771a48e1-fd5c-4ac1-85fc-353387935343.png
tags: k8s-series

---

## Backup Candidates

1\. Resource Configuration.

2\. etcd Cluster.

3\. Persistent volumes.

### Resource Configuration

We deploy number of application in K8s using Pods, Deployments, Service definition file.

* **Approach1**: create manifest file using Declarative approach its a preferred approach and can store it in source code repository even we lost entire cluster we can reapply manifest file from git .
    
* ```bash
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```
    
    **Approach2:** Clearing Kube-apiserver
    
    \- use kubectl or access kube-apiserver directly save all objects created on cluster as copy.
    
* ```bash
    kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml # add commands in backup scripts
    ```
    

## **Backup Tool:**

* Use ARK(VELERO) tool take backup of Kubernetes cluster by kube-apiserver.
    

## etcd Cluster

* it stores state of kubernetes cluster, nodes information etc.
    
* **Approach1**: while configuring etcd we specify the location where data can store in particular directory.
    

```bash
etcd.service
```

### **Using** `etcdctl` (Snapshot-based Backup)

1. backup of etcd database using snapshot save command.
    

```bash
etcdctl --endpoints=https://[127.0.0.1]:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot.db  #file location
```

2. check snapshot status command.
    

```bash
$ etcdctl \ 
     snapshot status snapshot.db 
```

3. Restore the cluster from this backup later point of time follow below steps.
    
    * stop kube-apiserver and run restore command
        
    * ```bash
        $ systemctl stop kube-apiserver
        $ etcdctl \ 
          snapshot restore snapshot.db
          --data-dir /var/lib/etcd-from-backup  # it initializes new cluster configuration with data directory
          
        $ etc.service
        $ systemctl deamon-reload  
        $ systemctl restart etcd
        $ systemctl start kube-apiserver.
        ```
        
        ```bash
        #Restore snapshot example
        etcdutl snapshot restore /opt/snapshot-pre-boot.db --data-dir /var/lib/etcd-from-backup
        #example output
        etcdutl snapshot restore /opt/snapshot-pre-boot.db --data-dir /var/lib/etcd-from-backup
        2025-04-24T09:38:07Z    info    snapshot/v3_snapshot.go:265     restoring snapshot      {"path": "/opt/snapshot-pre-boot.db", "wal-dir": "/var/lib/etcd-from-backup/member/wal", "data-dir": "/var/lib/etcd-from-backup", "snap-dir": "/var/lib/etcd-from-backup/member/snap", "initial-memory-map-size": 10737418240}
        2025-04-24T09:38:07Z    info    membership/store.go:141 Trimming membership information from the backend...
        2025-04-24T09:38:07Z    info    membership/cluster.go:421       added member    {"cluster-id": "cdf818194e3a8c32", "local-member-id": "0", "added-peer-id": "8e9e05c52164694d", "added-peer-peer-urls": ["http://localhost:2380"]}
        2025-04-24T09:38:07Z    info    snapshot/v3_snapshot.go:293     restored snapshot       {"path": "/opt/snapshot-pre-boot.db", "wal-dir": "/var/lib/etcd-from-backup/member/wal", "data-dir": "/var/lib/etcd-from-backup", "snap-dir": "/var/lib/etcd-from-backup/member/snap", "initial-memory-map-size": 10737418240}
        ```
        
        4. update newly restored directory path in file `/etc/kubernetes/manifests/etcd.yaml` as host path
            
            ```bash
             ...
              volumes:
              - hostPath:
                  path: /var/lib/etcd-from-backup # Newly restored backup directory
                  type: DirectoryOrCreate
                name: etcd-data
            ```
            
        
    
    5\. `ETCD` pod is automatically re-created as this is a static pod placed under the `/etc/kubernetes/manifests` directory watch logs.
    
    ```bash
    watch crictl ps
    ```
    
    7. once the updated etcd is up check pods, deployments.
        
    
    ```bash
    kubectl get deployments, services
    ```
    
    **Note**: so far we have seen backup using etcd and clearing kubeapi-server.
    
    * in managed kubernets cluster like AKS, EKS we won’t be having access to etcd. In this case backup using clearing kube-apiserver is better approach.