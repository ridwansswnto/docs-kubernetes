---
title: Pod
tags: [formatting]
keywords: notes,
summary: "play with some definition on k8s like pod, deployment, services, networking and other"
sidebar: mydoc_sidebar
permalink: mydoc_k8s-pod.html
folder: k8s
toc: true
---

## POD
The pod is a group of one or more containers and the smallerst deployable unit in kubernetes:
Each Pod is isolated by the following Linux namespaces:
a. The process ID (PID)
b. The network namespace
c. The interprocess communication (IPC)
d. The unix time share (UTS)

### Pod manifest
untuk membuat manifest Pod, biasanya terdapat 4 key yaitu
```
apiVersion:
kind:
metadata:


spec:


```

### LAB 1
create a basic pod and explore pod with `kubectl get` and `kubectl describe`

- Create pod manifest with `nginx-pod.yaml`

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    tier: dev
spec:
  containers:
  - name: nginx-container
    image: nginx
```


- Create pod with `kubectl create -f nginx-pod.yaml`

```
pod/nginx-pod created
```


- Get get pod with `kubectl get pod` and `kubectl get pod -o wide`

```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-pod                1/1     Running   0          102s


NAME                     READY   STATUS    RESTARTS   AGE     IP          NODE                  NOMINATED NODE   READINESS GATES
nginx-pod                1/1     Running   0          5m28s   10.46.0.3   worker-2.ridwan.com   <none>           <none>
```


- Get detailed of pod with `kubectl describe pod nginx-pod`

```
Name:         nginx-pod
Namespace:    default
Priority:     0
Node:         worker-2.ridwan.com/192.168.0.102
Start Time:   Sun, 05 Apr 2020 10:53:57 +0000
Labels:       app=nginx
              tier=dev
Annotations:  <none>
Status:       Running
IP:           10.46.0.3
IPs:
  IP:  10.46.0.3
Containers:
  nginx-container:
    Container ID:   docker://6745d415fdb66e4c2e0310aeba1e7582b8770be98245691a787b914e94a7e3aa
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:282530fcb7cd19f3848c7b611043f82ae4be3781cb00105a1d593d7e6286b596
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 05 Apr 2020 10:54:42 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-gtk48 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-gtk48:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-gtk48
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                          Message
  ----    ------     ----  ----                          -------
  Normal  Scheduled  10m   default-scheduler             Successfully assigned default/nginx-pod to worker-2.ridwan.com
  Normal  Pulling    10m   kubelet, worker-2.ridwan.com  Pulling image "nginx"
  Normal  Pulled     10m   kubelet, worker-2.ridwan.com  Successfully pulled image "nginx"
  Normal  Created    10m   kubelet, worker-2.ridwan.com  Created container nginx-container
  Normal  Started    10m   kubelet, worker-2.ridwan.com  Started container nginx-container
```

- testing with ping pod and get inside to the pod

```
ridwan@master:~$ ping 10.46.0.3
PING 10.46.0.3 (10.46.0.3) 56(84) bytes of data.
64 bytes from 10.46.0.3: icmp_seq=1 ttl=64 time=8.62 ms
64 bytes from 10.46.0.3: icmp_seq=2 ttl=64 time=3.03 ms
64 bytes from 10.46.0.3: icmp_seq=3 ttl=64 time=1.11 ms

ridwan@master:~$ kubectl exec -it nginx-pod -- /bin/sh
# hostname
nginx-pod
```

- testing with create test html page, get inside to the pod and create html

```
cat <<EOF > /usr/share/nginx/html/test.html
<!DOCTYPE html>
<html>
<head>
<title>Testing..</title>
</head>
<body>
<h1 style="color:rgb(90,70,250);">Hello, Kubernetes...!</h1>
<h2>Congratulations, you passed :-) </h2>
</body>
</html>
EOF
```

- Expose Pod using NodePort service

```
kubectl expose pod nginx-pod --type=NodePort --port=80
```

- Get information about service that we created before with `kubectl get svc nginx-pod`

```
Name:                     nginx-pod
Namespace:                default
Labels:                   app=nginx
                          tier=dev
Annotations:              <none>
Selector:                 app=nginx,tier=dev
Type:                     NodePort
IP:                       10.98.95.94
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32618/TCP
Endpoints:                10.46.0.3:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

- Access the Pod using IP of worker that pod running and NodePort

{% include image.html file="image1.png" caption="halaman yang sudah dibuat" %}

{% include image.html file="testajadulu.jpg" caption="halaman yang sudah dibuat" %}


- Delete svc with `kubectl delete svc nginx-pod` and pod with `kubectl delete pod nginx-pod`

```
service "nginx-pod" deleted
pod "nginx-pod" deleted
```


### LAB 2
we will create pod with 2 different container

```yml
apiVersion: v1
kind: Pod
metadata:
  name: first-pod
spec:
  containers:
  - name: nginx
    image: nginx
  - name: centos
    image: centos
    command: ["/bin/sh", "-c", "while : ;do curl http://localhost:80/; sleep 10; done"]
```

create pod file and run with `kubectl create -f first-pod.yml`
check pod event with `kubectl describe pod first-pod | grep -A20 Events`
```
Events:
  Type    Reason     Age        From                         Message
  ----    ------     ----       ----                         -------
  Normal  Scheduled  <unknown>  default-scheduler            Successfully assigned default/first-pod to worker3.ridwan.com
  Normal  Pulling    2m29s      kubelet, worker3.ridwan.com  Pulling image "nginx"
  Normal  Pulled     2m24s      kubelet, worker3.ridwan.com  Successfully pulled image "nginx"
  Normal  Created    2m24s      kubelet, worker3.ridwan.com  Created container nginx
  Normal  Started    2m24s      kubelet, worker3.ridwan.com  Started container nginx
  Normal  Pulling    2m24s      kubelet, worker3.ridwan.com  Pulling image "centos"
  Normal  Pulled     2m20s      kubelet, worker3.ridwan.com  Successfully pulled image "centos"
  Normal  Created    2m20s      kubelet, worker3.ridwan.com  Created container centos
  Normal  Started    2m19s      kubelet, worker3.ridwan.com  Started container centos
```

now we access centos container with `kubectl exec first-pod -it -c centos -- /bin/bash` and curl:
```
 curl -I http://localhost:80
HTTP/1.1 200 OK
Server: nginx/1.17.9
```

kesimpulannya adalah pod menghubungkan 2 container yang berbeda dalam linux network namespace yang sama
