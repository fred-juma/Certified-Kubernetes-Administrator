#### Daemon Sets

DaemonSet ensures a copy of a pod is present in each node on a cluster. Whenever a node is added to the cluster, a replica of the pod is automatically added to the node. Likewise, when a node is removed, the pod is automatically removed.

The DaemonSet ensures that one copy of the pod is always present in all the nodes in a cluster

#### Use cases:

+ Monitoring agents
+ Log viewers
+ kube-proxy
+ Networking soliutions e.g. weave-net



#### From kubernetes v1.12 daemonSet uses default scheduler and NodeAffinity to schedule pods on nodes

How many DaemonSets are created in the cluster in all namespaces?

```bash
controlplane ~ ➜  kubectl get DaemonSets --all-namespaces
NAMESPACE     NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kube-flannel-ds   1         1         1       1            1           <none>                   4m14s
kube-system   kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   4m16s

controlplane ~ ➜  
```

#### Deploy a DaemonSet for FluentD Logging.

Use the given specifications.

+ Name: elasticsearch
+ Namespace: kube-system
+ Image: k8s.gcr.io/fluentd-elasticsearch:1.20

```bash
controlplane ~ ➜  vi elasticsearch-daemonset.yaml

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec: 
      containers:
      - name: elasticsearch
        image: k8s.gcr.io/fluentd-elasticsearch:1.20
```

#### Apply the manifest definition file to create the resources
```bash
controlplane /etc/kubernetes/manifests ➜  kubectl apply -f static-busybox.yaml pod/static-busybox created

controlplane /etc/kubernetes/manifests ➜  
```

#### Check the created resources
```
 controlplane ~ ➜  kubectl get all --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-6d4b75cb6d-9b58f               1/1     Running   0          14m
kube-system   pod/coredns-6d4b75cb6d-c8t7n               1/1     Running   0          14m
kube-system   pod/elasticsearch-x7q4x                    1/1     Running   0          22s
kube-system   pod/etcd-controlplane                      1/1     Running   0          14m
kube-system   pod/kube-apiserver-controlplane            1/1     Running   0          14m
kube-system   pod/kube-controller-manager-controlplane   1/1     Running   0          14m
kube-system   pod/kube-flannel-ds-4t674                  1/1     Running   0          14m
kube-system   pod/kube-proxy-cjwjt                       1/1     Running   0          14m
kube-system   pod/kube-scheduler-controlplane            1/1     Running   0          14m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  14m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   14m

NAMESPACE     NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/elasticsearch     1         1         1       1            1           <none>                   22s
kube-system   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   14m
kube-system   daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   14m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           14m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-6d4b75cb6d   2         2         2       14m

controlplane ~ ➜  
```

***The End***