---
title: ConfigMap
tags: [formatting]
keywords: notes,
summary: "play with some definition on k8s like pod, deployment, services, networking and other"
sidebar: mydoc_sidebar
permalink: mydoc_k8s-configmap.html
folder: k8s
toc: true
---

## ConfigMap
Configmap store configuration maps/dictionaries of key-value data


### LAB 1
creating Configmap from "multiple file" & consuming it inside Pod from "volumes"

1.  Create Configmap

    * Create directory `nginx-configmap-vol/`
```bash
echo -n 'Non-sensitive data inside file-1' > nginx-configmap-vol/file-1.txt
echo -n 'Non-sensitive data inside file-2' > nginx-configmap-vol/file-2.txt
```

    * create configmap
```bash
kubectl create configmap nginx-configmap-vol --from-file=nginx-configmap-vol/file-1.txt --from-file=nginx-configmap-vol/file-2.txt
configmap/nginx-configmap-vol created
```

    * get info with `kubectl get configmaps`
```bash
NAME                  DATA   AGE
nginx-configmap-vol   2      26s
```

    * get detail info `kubectl describe configmaps nginx-configmap-vol`
    ```bash
    Name:         nginx-configmap-vol
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>
    ```
    ```bash
    Data
    ====
    file-1.txt:
    ----
    Non-sensitive data inside file-1
    file-2.txt:
    ----
    Non-sensitive data inside file-2
    Events:  <none>
    ```

1. Create manifest Pod for consume that configmap `nginx-pod-configmap-vol.yaml`

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod-configmap-vol
    spec:
      containers:
      - name: nginx-container
        image: nginx
        volumeMounts:
        - name: test-vol
          mountPath: "/etc/non-sensitive-data"
          readOnly: true
      volumes:
        - name: test-vol
          configMap:
            name: nginx-configmap-vol
            items:
            - key: file-1.txt
              path: file-a.txt
            - key: file-2.txt
              path: file-b.txt
    ```

3. create pod and validate
    * Create `kubectl create -f nginx-pod-configmap-vol.yaml`

    * display `kubectl describe pod nginx-pod-configmap-vol`
    ```
    Name:         nginx-pod-configmap-vol
    Namespace:    default
    Priority:     0
    Node:         worker-2.ridwan.com/192.168.0.102
    Start Time:   Wed, 08 Apr 2020 22:10:14 +0000
    Labels:       <none>
    Annotations:  <none>
    Status:       Running
    IP:           10.46.0.3
    IPs:
      IP:  10.46.0.3
    Containers:
      nginx-container:
        Container ID:   docker://57b7ed957cc890615daf45081d4ea74447244655c89956e5afd4d750cd86f9a0
        Image:          nginx
        Image ID:       docker-pullable://nginx@sha256:282530fcb7cd19f3848c7b611043f82ae4be3781cb00105a1d593d7e6286b596
        Port:           <none>
        Host Port:      <none>
        State:          Running
          Started:      Wed, 08 Apr 2020 22:10:19 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
          /etc/non-sensitive-data from test-vol (ro)
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-gtk48 (ro)
    Conditions:
      Type              Status
      Initialized       True
      Ready             True
      ContainersReady   True
      PodScheduled      True
    Volumes:
      test-vol:
        Type:      ConfigMap (a volume populated by a ConfigMap)
        Name:      nginx-configmap-vol
        Optional:  false
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
      Normal  Scheduled  59s   default-scheduler             Successfully assigned default/nginx-pod-configmap-vol to worker-2.ridwan.com
      Normal  Pulling    58s   kubelet, worker-2.ridwan.com  Pulling image "nginx"
      Normal  Pulled     54s   kubelet, worker-2.ridwan.com  Successfully pulled image "nginx"
      Normal  Created    54s   kubelet, worker-2.ridwan.com  Created container nginx-container
      Normal  Started    54s   kubelet, worker-2.ridwan.com  Started container nginx-container
    ```

    * Validate
    ```bash
  kubectl exec nginx-pod-configmap-vol ls /etc/non-sensitive-data
  file-a.txt
    file-b.txt
    ```
    ```bash
    kubectl exec nginx-pod-configmap-vol cat /etc/non-sensitive-data/file-a.txt
    Non-sensitive data inside file-1
    ```

### LAB 2
create configmap from literal value

1.  Create configmap

    ```
    kubectl create configmap redis-configmap-env --from-literal=file.1=file.a --from-literal=file.2=file.b
    configmap/redis-configmap-env created
    ```

1.  Get info

    ```
    kubectl describe configmaps redis-configmap-env
    Name:         redis-configmap-env
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Data
    ====
    file.1:
    ----
    file.a
    file.2:
    ----
    file.b
    Events:  <none>
    ```

3.  Create manifest pod

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: redis-pod-configmap-env
    spec:
      containers:
      - name: redis-container
        image: redis
        env:
          - name: FILE_1
            valueFrom:
              configMapKeyRef:
                name: redis-configmap-env
                key: file.1
          - name: FILE_2
            valueFrom:
              configMapKeyRef:
                name: redis-configmap-env
                key: file.2
                restartPolicy: Never
    ```

4. Create pod, get info and validate
    * create pod
    ```
    kubectl create -f  redis-pod-configmap-env.yaml
    ```

    * validate pod
    ```
    Name:         redis-pod-configmap-env
Namespace:    default
Priority:     0
Node:         worker-1.ridwan.com/192.168.0.101
Start Time:   Thu, 09 Apr 2020 00:19:27 +0000
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.40.0.3
IPs:
  IP:  10.40.0.3
Containers:
  redis-container:
    Container ID:   docker://88a9ee67d2e340a5915ef77772b433f1fd3438d43d8734db5dd46a4dc04dcfa6
    Image:          redis
    Image ID:       docker-pullable://redis@sha256:a732b1359e338a539c25346a50bf0a501120c41dc248d868e546b33e32bf4fe4
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 09 Apr 2020 00:19:33 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      FILE_1:  <set to the key 'file.1' of config map 'redis-configmap-env'>  Optional: false
      FILE_2:  <set to the key 'file.2' of config map 'redis-configmap-env'>  Optional: false
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
  Normal  Scheduled  7s    default-scheduler             Successfully assigned default/redis-pod-configmap-env to worker-1.ridwan.com
  Normal  Pulling    6s    kubelet, worker-1.ridwan.com  Pulling image "redis"
  Normal  Pulled     2s    kubelet, worker-1.ridwan.com  Successfully pulled image "redis"
  Normal  Created    1s    kubelet, worker-1.ridwan.com  Created container redis-container
  Normal  Started    1s    kubelet, worker-1.ridwan.com  Started container redis-container
    ```

    * Validate
    ```
    kubectl exec redis-pod-configmap-env env | grep FILE
    FILE_1=file.a
    FILE_2=file.b    
    ```

untuk lebih detail ada beberapa link yang dapat membantu menjelaskan secara detail
1. [ultimate configMap Guide](https://matthewpalmer.net/kubernetes-app-developer/articles/ultimate-configmap-guide-kubernetes.html)
2. [Introduction secret and configmap](https://opensource.com/article/19/6/introduction-kubernetes-secrets-and-configmaps)


<script src="https://gist.github.com/ridwansswnto/2b1b414f5151c2213257f4aebbba60cf.js"></script>



<script src="https://gist.github.com/ridwansswnto/ec82e7dd949438f1794394a1eb4f85dd.js"></script>
