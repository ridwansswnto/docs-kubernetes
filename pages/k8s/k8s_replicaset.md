---
title: Replicaset
tags: [formatting]
keywords: notes,
summary: "play with some definition on k8s like pod, deployment, services, networking and other"
sidebar: mydoc_sidebar
permalink: mydoc_k8s-replicaset.html
folder: k8s
toc: false
---

## Pod and ReplicaSet
RS is a term for API objects in Kubernetes that refer to Pod replicas. bisa di bilang untuk control dari set sebuah pod.
RS memastikan bahwa Pod, berjalan secara baik setiap saat dan dapat spesifik jumlahnya berdasarkan user.

Untuk lebih jelasnya di bawah


a. kube-controller-manager daemon helps to maintain the resource running in its desired state. example. have 3 pod replica
b. kube-scheduler daemon on master, takes charge of assigning tasks to healthy noes
c. The selector is used for deciding which pods it covers.

### Crete RS
run command with `kubectl create -f first-replicaset.yml`
```
kubectl describe rs first-replicaset
Name:         first-replicaset
Namespace:    default
Selector:     project=first-web,role=frontend
Labels:       version=0.0.1
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  0 Running / 3 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  env=dev
           project=first-web
           role=frontend
  Containers:
   first-web:
    Image:        nginx:latest
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  5s    replicaset-controller  Created pod: first-replicaset-t8n8k
  Normal  SuccessfulCreate  5s    replicaset-controller  Created pod: first-replicaset-xtqsf
  Normal  SuccessfulCreate  5s    replicaset-controller  Created pod: first-replicaset-dlrzr
```

get detailed to pods `kubectl describe pods first-replicaset-dlrzr`
```
Labels:       env=dev
              project=first-web
              role=frontend
Annotations:  <none>
Status:       Running
IP:           10.44.0.1
IPs:
  IP:           10.44.0.1
Controlled By:  ReplicaSet/first-replicaset
```

kubernetes put rs on every nodes
```
NAME                     READY   STATUS    RESTARTS   AGE     IP          NODE                 NOMINATED NODE   READINESS GATES
first-replicaset-dlrzr   1/1     Running   0          3m58s   10.42.0.1   worker3.ridwan.com   <none>           <none>
first-replicaset-t8n8k   1/1     Running   0          3m58s   10.44.0.1   worker1.ridwan.com   <none>           <none>
first-replicaset-xtqsf   1/1     Running   0          3m58s   10.36.0.1   worker2.ridwan.com   <none>           <none>
```

get label
```
kubectl get pods -L project -L role -L env
NAME                     READY   STATUS    RESTARTS   AGE     PROJECT     ROLE       ENV
first-replicaset-62zw5   1/1     Running   0          90s     first-web   frontend   dev
first-replicaset-t8n8k   1/1     Running   0          5m47s   first-web   frontend   dev
first-replicaset-xtqsf   1/1     Running   0          5m47s   first-web   frontend   dev
```
