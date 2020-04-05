---
title: Check Cluster
tags: [formatting]
keywords: notes,
summary: "play with some definition on k8s like pod, deployment, services, networking and other"
sidebar: mydoc_sidebar
permalink: mydoc_k8s-cluster.html
folder: mydoc
toc: false
---

## Check Cluster

Ssebelum mulai belajar, ada baiknya check cluster yang akan di gunakan

### Check status komponen

run this command `kubectl get cs` to check:
```
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

### Check cluster info
run this command `kubectl cluster-info` to get:

```
Kubernetes master is running at https://192.168.0.10:6443
KubeDNS is running at https://192.168.0.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

```

### Check node
run this command `kubectl get nodes -o wide` to get:

```
kubectl get nodes -o wide
NAME                 STATUS   ROLES    AGE    VERSION   INTERNAL-IP      OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master.ridwan.com    Ready    master   4d7h   v1.17.3   192.168.0.10     Ubuntu 18.04.4 LTS   4.15.0-88-generic   docker://19.3.7
worker1.ridwan.com   Ready    <none>   4d6h   v1.17.3   192.168.0.11     Ubuntu 18.04.4 LTS   4.15.0-88-generic   docker://19.3.7
worker2.ridwan.com   Ready    <none>   4d6h   v1.17.3   192.168.0.12     Ubuntu 18.04.4 LTS   4.15.0-88-generic   docker://19.3.7
worker3.ridwan.com   Ready    <none>   4d6h   v1.17.3   192.168.0.13     Ubuntu 18.04.4 LTS   4.15.0-88-generic   docker://19.3.7

```

### PoC Web server NGINX
run this command `kubectl run first-nginx --image=nginx --replicas=2 --port=80`
```
deployment.apps/first-nginx created
```

will create pod
```
kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
first-nginx-79fb65f4f6-jnhsx   1/1     Running   0          2m11s
first-nginx-79fb65f4f6-xhrkw   1/1     Running   0          2m11s
```

and deployment like:
```
kubectl get deployment
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
first-nginx   2/2     2            2           3m54s
```

Exposing port for external Access
```
kubectl expose deployment first-nginx --port=80 --type=LoadBalancer
service/first-nginx exposed
```

check all services
```
kubectl get service
NAME          TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
first-nginx   LoadBalancer   10.111.213.204   192.168.0.102   80:30106/TCP   70s
```

Validate
now we validate if the services run with right
```
curl -I 192.168.0.102
HTTP/1.1 200 OK
Server: nginx/1.17.9

```

How is works
run with this command `kubectl describe service first-nginx`
```
Name:                     first-nginx
Namespace:                default
Labels:                   run=first-nginx
Annotations:              <none>
Selector:                 run=first-nginx
Type:                     LoadBalancer
IP:                       10.111.213.204
LoadBalancer Ingress:     192.168.0.102
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30106/TCP
Endpoints:                10.36.0.1:80,10.44.0.1:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
