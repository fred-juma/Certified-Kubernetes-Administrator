#### Monitoring and Logging

 **cAdvisor - Container Advisor**

- Component of kubelet
- It is responsible for collecting perfomance metrics from the pods and exposing them through the kubelet API to make the metrics available for the Metrics Server

**Deploying Metrics server**

**Option 1**:  in minicube: 

*minicube addons enable metrics-server*

**Option 2**: For other platforms - clone the metrics server from github repository then apply

*git clone https://github.com/kubernetes-incubator/metrics-server.git*

*kubectl apply -f /deploy/1.9+/*

This deploys a set of pods, services and roles to enable ,metric server poll perfomance metrics from the nodes

To view memory and cpu metrics of nodes

kubectl top node

To view perfomance metrics of pods

kubectl pod

Let us deploy metrics-server to monitor the PODs and Nodes. Pull the git repository for the deployment files.

```bash

controlplane ~ ➜  git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
Cloning into 'kubernetes-metrics-server'...
remote: Enumerating objects: 24, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 24 (delta 4), reused 0 (delta 0), pack-reused 12
Unpacking objects: 100% (24/24), done.

controlplane ~ ➜  
```

Observe the cloned repo
```bash
controlplane ~ ➜  ls -l
total 4
drwxr-xr-x 3 root root 4096 Nov 28 13:14 kubernetes-metrics-server
-rw-rw-rw- 1 root root    0 Nov 21 06:48 sample.yaml

controlplane ~ ➜  
```

Change into the repo and list contents
```bash
controlplane ~ ➜  cd kubernetes-metrics-server/

controlplane kubernetes-metrics-server on  master ➜  ls -l
total 32
-rw-r--r-- 1 root root 384 Nov 28 13:14 aggregated-metrics-reader.yaml
-rw-r--r-- 1 root root 303 Nov 28 13:14 auth-delegator.yaml
-rw-r--r-- 1 root root 324 Nov 28 13:14 auth-reader.yaml
-rw-r--r-- 1 root root 293 Nov 28 13:14 metrics-apiservice.yaml
-rw-r--r-- 1 root root 976 Nov 28 13:14 metrics-server-deployment.yaml
-rw-r--r-- 1 root root 249 Nov 28 13:14 metrics-server-service.yaml
-rw-r--r-- 1 root root 219 Nov 28 13:14 README.md
-rw-r--r-- 1 root root 612 Nov 28 13:14 resource-reader.yaml

controlplane kubernetes-metrics-server on  master ➜  
```

Apply the manifest files to create the resources

```bash
controlplane ~ ➜  kubectl apply -f kubernetes-metrics-server/
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created

controlplane ~ ➜  
```

List all resources in the cluster to view created components

```bash
controlplane ~ ➜  kubectl get all --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
default       pod/elephant                               1/1     Running   0          4m22s
default       pod/lion                                   1/1     Running   0          4m22s
default       pod/rabbit                                 1/1     Running   0          4m22s
kube-system   pod/coredns-6d4b75cb6d-2gw82               1/1     Running   0          6m36s
kube-system   pod/coredns-6d4b75cb6d-plm62               1/1     Running   0          6m36s
kube-system   pod/etcd-controlplane                      1/1     Running   0          6m52s
kube-system   pod/kube-apiserver-controlplane            1/1     Running   0          6m47s
kube-system   pod/kube-controller-manager-controlplane   1/1     Running   0          6m47s
kube-system   pod/kube-flannel-ds-tk724                  1/1     Running   0          6m23s
kube-system   pod/kube-flannel-ds-w2ssm                  1/1     Running   0          6m36s
kube-system   pod/kube-proxy-m2m5r                       1/1     Running   0          6m23s
kube-system   pod/kube-proxy-wr4rg                       1/1     Running   0          6m36s
kube-system   pod/kube-scheduler-controlplane            1/1     Running   0          6m50s
kube-system   pod/metrics-server-79b54f797c-xmrjd        1/1     Running   0          75s

NAMESPACE     NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP                  6m51s
kube-system   service/kube-dns         ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   6m49s
kube-system   service/metrics-server   ClusterIP   10.101.212.114   <none>        443/TCP                  75s

NAMESPACE     NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-flannel-ds   2         2         2       2            2           <none>                   6m45s
kube-system   daemonset.apps/kube-proxy        2         2         2       2            2           kubernetes.io/os=linux   6m49s

NAMESPACE     NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns          2/2     2            2           6m49s
kube-system   deployment.apps/metrics-server   1/1     1            1           75s

NAMESPACE     NAME                                        DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-6d4b75cb6d          2         2         2       6m37s
kube-system   replicaset.apps/metrics-server-79b54f797c   1         1         1       75s

controlplane ~ ➜  
```

To view perfomance metrics of nodes

```bash
controlplane ~ ➜  kubectl top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   299m         0%     966Mi           0%        
node01         23m          0%     266Mi           0%        

controlplane ~ ➜  
```

To view perfomance metrics of pods

```bash
controlplane ~ ➜  kubectl top pod
NAME       CPU(cores)   MEMORY(bytes)   
elephant   20m          32Mi            
lion       1m           18Mi            
rabbit     139m         253Mi           

controlplane ~ ➜  
```

#### Logging in docker

**View docker logs**

docker logs -f ecf # -f see live stream

**View running pods**

```bash
controlplane ~ ➜  kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          8s
```

**View kubernetes logs of the above pod**

```bash
controlplane ~ ➜  kubectl logs -f webapp-1
[2022-11-30 06:23:05,443] INFO in event-simulator: USER1 is viewing page2
[2022-11-30 06:23:06,444] INFO in event-simulator: USER2 is viewing page2
[2022-11-30 06:23:07,446] INFO in event-simulator: USER3 is viewing page2
[2022-11-30 06:23:08,447] INFO in event-simulator: USER2 is viewing page3
[2022-11-30 06:23:09,448] INFO in event-simulator: USER4 logged out
[2022-11-30 06:23:10,449] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
````

// -f option for live streaming of logs


If there are multiple containers in a pod, you must specify the name of the container explicitly in the command

```bash
controlplane ~ ➜  kubectl describe pods webapp-2
Name:         webapp-2
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.59
Start Time:   Wed, 30 Nov 2022 06:25:08 +0000
Labels:       name=webapp-2
Annotations:  <none>
Status:       Running
IP:           10.42.0.10
IPs:
  IP:  10.42.0.10
Containers:
  simple-webapp:
    Container ID:   containerd://212850f0f4e5cf051b6dfecf09011253ad4afa955b904e1dfdef9c4d99b917f7
    Image:          kodekloud/event-simulator
    Image ID:       docker.io/kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 30 Nov 2022 06:25:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      OVERRIDE_USER:  USER30
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dw697 (ro)
  db:
    Container ID:  containerd://ae4a05d563ed7cac81738d1a51d63c74860d00a0d6641f2072498ad554942c01
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:fcd85228d7a25feb59f101ac3a955d27c80df4ad824d65f5757a954831450185
    Port:          <none>
    Host Port:     <none>
```


The above pod has 2 containers running in it, *simple-webapp* and *db*. To view logs for a pod with multiple containers, you must specify the container name. In this example we view logs of the webapp container

```bash
controlplane ~ ➜  kubectl logs -f webapp-2 simple-webapp
[2022-11-30 06:25:10,239] INFO in event-simulator: USER2 logged out
[2022-11-30 06:25:11,241] INFO in event-simulator: USER4 logged out
[2022-11-30 06:25:12,242] INFO in event-simulator: USER4 is viewing page2
[2022-11-30 06:25:13,243] INFO in event-simulator: USER1 logged out
[2022-11-30 06:25:14,244] INFO in event-simulator: USER3 is viewing page1
[2022-11-30 06:25:15,245] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-11-30 06:25:15,245] INFO in event-simulator: USER2 logged in
[2022-11-30 06:25:16,246] INFO in event-simulator: USER4 logged out
[2022-11-30 06:25:17,279] INFO in event-simulator: USER4 logged in
[2022-11-30 06:25:18,280] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
```