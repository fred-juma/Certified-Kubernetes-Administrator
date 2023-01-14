#### Backup and Restore

#### Method 1

We have a working kubernetes cluster with a set of applications running. Let us first explore the setup.


```bash

controlplane ~ ➜  kubectl get all --all-namespaces
NAMESPACE      NAME                                       READY   STATUS    RESTARTS   AGE
default        pod/blue-987f68cb5-2h645                   1/1     Running   0          75s
default        pod/blue-987f68cb5-8rl4b                   1/1     Running   0          75s
default        pod/blue-987f68cb5-nttkg                   1/1     Running   0          75s
default        pod/red-d7f6f6b9b-s29vp                    1/1     Running   0          75s
default        pod/red-d7f6f6b9b-z4sxx                    1/1     Running   0          75s
kube-flannel   pod/kube-flannel-ds-j6hjv                  1/1     Running   0          5m35s
kube-system    pod/coredns-787d4945fb-49qd2               1/1     Running   0          5m36s
kube-system    pod/coredns-787d4945fb-4p742               1/1     Running   0          5m36s
kube-system    pod/etcd-controlplane                      1/1     Running   0          6m4s
kube-system    pod/kube-apiserver-controlplane            1/1     Running   0          6m4s
kube-system    pod/kube-controller-manager-controlplane   1/1     Running   0          6m4s
kube-system    pod/kube-proxy-bp9dg                       1/1     Running   0          5m36s
kube-system    pod/kube-scheduler-controlplane            1/1     Running   0          6m6s

NAMESPACE     NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       service/blue-service   NodePort    10.96.250.147   <none>        80:30082/TCP             75s
default       service/kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP                  6m7s
default       service/red-service    NodePort    10.110.189.67   <none>        80:30080/TCP             75s
kube-system   service/kube-dns       ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   6m5s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   5m35s
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   6m5s

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/blue      3/3     3            3           75s
default       deployment.apps/red       2/2     2            2           75s
kube-system   deployment.apps/coredns   2/2     2            2           6m5s

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
default       replicaset.apps/blue-987f68cb5       3         3         3       75s
default       replicaset.apps/red-d7f6f6b9b        2         2         2       75s
kube-system   replicaset.apps/coredns-787d4945fb   2         2         2       5m36s

controlplane ~ ➜ 
```

What is the version of ETCD running on the cluster? Image: registry.k8s.io/etcd:3.5.6-0

```bash
controlplane ~ ➜  kubectl describe pod etcd-controlplane --namespace kube-system
Name:                 etcd-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 controlplane/10.17.241.3
Start Time:           Thu, 12 Jan 2023 02:49:01 -0500
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.17.241.3:2379
                      kubernetes.io/config.hash: 890721b7f25d2c1012db223babae3f6a
                      kubernetes.io/config.mirror: 890721b7f25d2c1012db223babae3f6a
                      kubernetes.io/config.seen: 2023-01-12T02:49:01.180876522-05:00
                      kubernetes.io/config.source: file
Status:               Running
IP:                   10.17.241.3
IPs:
  IP:           10.17.241.3
Controlled By:  Node/controlplane
Containers:
  etcd:
    Container ID:  containerd://88cfed016c79d5d7dad10a8d2b4cddaa4e9f18479b59a04d933d93599345500e
    Image:         registry.k8s.io/etcd:3.5.6-0
    Image ID:      registry.k8s.io/etcd@sha256:dd75ec974b0a2a6f6bb47001ba09207976e625db898d1b16735528c009cb171c
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://10.17.241.3:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --experimental-initial-corrupt-check=true
      --experimental-watch-progress-notify-interval=5s
      --initial-advertise-peer-urls=https://10.17.241.3:2380
      --initial-cluster=controlplane=https://10.17.241.3:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://10.17.241.3:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://10.17.241.3:2380
      --name=controlplane
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    State:          Running
      Started:      Thu, 12 Jan 2023 02:48:50 -0500
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
      memory:     100Mi
    Liveness:     http-get http://127.0.0.1:2381/health%3Fexclude=NOSPACE&serializable=true delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get http://127.0.0.1:2381/health%3Fserializable=false delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
      /var/lib/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:            <none>

controlplane ~ ➜ 
```

At what address can you reach the ETCD cluster from the controlplane node? --listen-client-urls=https://127.0.0.1:2379

or

Check the ETCD Service configuration in the ETCD POD


```bash
controlplane ~ ➜  ps aux | grep etcd
root        2733  0.0  0.0 11220796 56420 ?      Ssl  02:48   0:28 etcd --advertise-client-urls=https://10.17.241.3:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --experimental-initial-corrupt-check=true --experimental-watch-progress-notify-interval=5s --initial-advertise-peer-urls=https://10.17.241.3:2380 --initial-cluster=controlplane=https://10.17.241.3:2380 --key-file=/etc/kubernetes/pki/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://10.17.241.3:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://10.17.241.3:2380 --name=controlplane --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
root        2734  0.0  0.1 1124520 336876 ?      Ssl  02:48   0:56 kube-apiserver --advertise-address=10.17.241.3 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root       11638  0.0  0.0   6612   656 pts/0    S+   03:07   0:00 grep etcd

controlplane ~ ➜  
```

Where is the ETCD server certificate file located? --cert-file=/etc/kubernetes/pki/etcd/server.crt


Where is the ETCD CA Certificate file located?  --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt 

Key file? --key-file=/etc/kubernetes/pki/etcd/server.key




Backup candidates

1. Resource Configuration
- Resources created declaratively, the yaml files can be stored in code repo e.g. github
- For resource configuration created imperatively (on the CLI) query the kube-apiserver

kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml

Tools
Vellero - can help taking backups of the clusre using kube-apiserver


2. ETCD 

- Backing up the etcd cluster hosted on master nodes. 
- The etcd service is configured to store data as a directory: 

--data-dir=/var/lib/etcd

- Configure the backup tool to backup the above diretory
- etcd also comes with a built-in snapshot service


The master node in our cluster is planned for a regular maintenance reboot tonight. While we do not anticipate anything to go wrong, we are required to take the necessary backups. Take a snapshot of the ETCD database using the built-in snapshot functionality.

Store the backup file at location /opt/snapshot-pre-boot.db


First check the etcdctl api version
```bash
controlplane ~ ➜  ETCDCTL_API=3 etcdctl version
etcdctl version: 3.3.13
API version: 3.3

controlplane ~ ➜  
```

take the etcdl snapshot

```bash
controlplane ~ ➜  ETCDCTL_API=3 etcdctl \
> snapshot save /opt/snapshot-pre-boot.db \
> --endpoints=https://127.0.0.1:2379 \
> --cacert=/etc/kubernetes/pki/etcd/ca.crt \
> --cert=/etc/kubernetes/pki/etcd/server.crt \
> --key=/etc/kubernetes/pki/etcd/server.key
Snapshot saved at /opt/snapshot-pre-boot.db

controlplane ~ ➜  
```



To view the backup status

```bash
controlplane ~ ➜  ETCDCTL_API=3 etcdctl \
> snapshot status /opt/snapshot-pre-boot.db
aef10e43, 4499, 893, 2.2 MB

controlplane ~ ➜  
```




Restore the original state of the cluster using the backup file.

- stop the kube-apiserver

service kube-apiserver stop

- Then restore the snapshot

```bash
controlplane ~ ➜  ETCDCTL_API=3 etcdctl \
> snapshot restore /opt/snapshot-pre-boot.db \
> --data-dir /var/lib/etcd-from-backup
2023-01-12 03:45:45.233120 I | mvcc: restore compact to 3996
2023-01-12 03:45:45.238319 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32

controlplane ~ ➜  
```

We have now restored the etcd snapshot to a new path on the controlplane - /var/lib/etcd-from-backup, so, the only change to be made in the YAML file, is to change the hostPath for the volume called etcd-data from old directory (/var/lib/etcd) to the new directory (/var/lib/etcd-from-backup).


```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.24.234.6:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://10.24.234.6:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --experimental-initial-corrupt-check=true
    - --experimental-watch-progress-notify-interval=5s
    - --initial-advertise-peer-urls=https://10.24.234.6:2380
    - --initial-cluster=controlplane=https://10.24.234.6:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://10.24.234.6:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://10.24.234.6:2380
    - --name=controlplane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: registry.k8s.io/etcd:3.5.6-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health?exclude=NOSPACE&serializable=true
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health?serializable=false
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
status: {}
``` 

With this change, /var/lib/etcd on the container points to /var/lib/etcd-from-backup on the controlplane (which is what we want).

When this file is updated, the ETCD pod is automatically re-created as this is a static pod placed under the /etc/kubernetes/manifests directory.

As the ETCD pod has changed it will automatically restart, and also kube-controller-manager and kube-scheduler. Wait 1-2 to mins for this pods to restart. You can run the command: watch "crictl ps | grep etcd" to see when the ETCD pod is restarted.

If the etcd pod is not getting Ready 1/1, then restart it by kubectl delete pod -n kube-system etcd-controlplane and wait 1 minute.

Note 3: This is the simplest way to make sure that ETCD uses the restored data after the ETCD pod is recreated. You don't have to change anything else.

```bash

controlplane ~ ➜  watch "crictl ps | grep etcd"



Every 2.0s: crictl ps | grep etcd                                                           controlplane: Thu Jan 12 04:49:00 2023

Every 2.0s: crictl ps | grep etcd                                                                                                         controlplane: Thu Jan 12 04:49:28 2023

28e37368c7d76       fce326961ae2d       2 minutes ago       Running             etcd                      0                   95eb37d941428       etcd-controlplane
```

View all resources

```bash
controlplane ~ ➜  kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/blue-987f68cb5-4sdnf   1/1     Running   0          13m
pod/blue-987f68cb5-bh5x7   1/1     Running   0          13m
pod/blue-987f68cb5-njvtb   1/1     Running   0          13m
pod/red-d7f6f6b9b-44j5g    1/1     Running   0          13m
pod/red-d7f6f6b9b-rmwp8    1/1     Running   0          13m

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/blue-service   NodePort    10.97.233.141    <none>        80:30082/TCP   13m
service/kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP        16m
service/red-service    NodePort    10.109.209.195   <none>        80:30080/TCP   13m

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/blue   3/3     3            3           13m
deployment.apps/red    2/2     2            2           13m

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/blue-987f68cb5   3         3         3       13m
replicaset.apps/red-d7f6f6b9b    2         2         2       13m

controlplane ~ ➜  
```



Other Options:


systemctl daemon-reload

service etcd restart

Start the kube-apiserver service

service kube-apiserver start

Note:

With all the etcd commands, specify the following:
- certificate files for authentication
- endpoint to the etcd cluster and CA certificate
- etcd server certificate
- the key

etcdctl is a command line client for etcd

To make use of etcdctl for tasks such as back up and restore, make sure that you set the ETCDCTL_API to 3

You can do this by exporting the variable ETCDCTL_API prior to using the etcdctl client. This can be done as follows:

export ETCDCTL_API=3

and then check the version

etcdl version


#### Method 2

In this lab environment, you will get to work with multiple kubernetes clusters where we will practice backing up and restoring the ETCD database.

The student-node has the kubectl client and has access to all the Kubernetes clusters that are configured in thie lab environment.

```bash
student-node ~ ➜  kubectl get all --all-namespaces
NAMESPACE     NAME                                                READY   STATUS    RESTARTS      AGE
kube-system   pod/coredns-6d4b75cb6d-d9hbb                        1/1     Running   0             67m
kube-system   pod/coredns-6d4b75cb6d-vtwvd                        1/1     Running   0             67m
kube-system   pod/etcd-cluster1-controlplane                      1/1     Running   0             67m
kube-system   pod/kube-apiserver-cluster1-controlplane            1/1     Running   0             67m
kube-system   pod/kube-controller-manager-cluster1-controlplane   1/1     Running   0             67m
kube-system   pod/kube-proxy-cd9z9                                1/1     Running   0             66m
kube-system   pod/kube-proxy-zsj62                                1/1     Running   0             67m
kube-system   pod/kube-scheduler-cluster1-controlplane            1/1     Running   0             67m
kube-system   pod/weave-net-2jrrx                                 2/2     Running   0             66m
kube-system   pod/weave-net-fnlhr                                 2/2     Running   1 (67m ago)   67m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  67m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   67m

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   67m
kube-system   daemonset.apps/weave-net    2         2         2       2            2           <none>                   67m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           67m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-6d4b75cb6d   2         2         2       67m

student-node ~ ➜  
```
How many clusters are defined in the kubeconfig on the student-node? - 2

```bash
student-node ~ ➜  kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://cluster1-controlplane:6443
  name: cluster1
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.162.94.9:6443
  name: cluster2
contexts:
- context:
    cluster: cluster1
    user: cluster1
  name: cluster1
- context:
    cluster: cluster2
    user: cluster2
  name: cluster2
current-context: cluster1
kind: Config
preferences: {}
users:
- name: cluster1
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: cluster2
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED

student-node ~ ➜  
```

How many nodes (both controlplane and worker) are part of cluster1? - 2

First switch contect to cluster1

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1"
```

Then get the nodes in cluster 1
```bash
student-node ~ ➜  kubectl get nodes -o wide
NAME                    STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
cluster1-controlplane   Ready    control-plane   74m   v1.24.0   10.162.94.6    <none>        Ubuntu 18.04.6 LTS   5.4.0-1093-gcp   containerd://1.6.6
cluster1-node01         Ready    <none>          73m   v1.24.0   10.162.94.12   <none>        Ubuntu 18.04.6 LTS   5.4.0-1093-gcp   containerd://1.6.6

student-node ~ ➜   
```

What is the name of the controlplane node in cluster2?

First switch contect to cluster2

```bash
student-node ~ ➜  kubectl config use-context cluster2 - cluster2-controlplane
Switched to context "cluster2".

student-node ~ ➜  
```

Then get the nodes in cluster 2

```bash
student-node ~ ➜  kubectl get nodes -o wide
NAME                    STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
cluster2-controlplane   Ready    control-plane   76m   v1.24.0   10.162.94.9    <none>        Ubuntu 18.04.6 LTS   5.4.0-1093-gcp   containerd://1.6.6
cluster2-node01         Ready    <none>          75m   v1.24.0   10.162.94.15   <none>        Ubuntu 18.04.6 LTS   5.4.0-1093-gcp   containerd://1.6.6

student-node ~ ➜  
```

How is ETCD configured for cluster1?

First switch contect to cluster1

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".
```
If you check out the pods running in the kube-system namespace in cluster1, you will notice that etcd is running as a pod:

```bash
student-node ~ ➜  kubectl get pods -n kube-system | grep etcd
etcd-cluster1-controlplane                      1/1     Running   0             79m
```

This means that ETCD is set up as a Stacked ETCD Topology where the distributed data storage cluster provided by etcd is stacked on top of the cluster formed by the nodes managed by kubeadm that run control plane components.

How is ETCD configured for cluster2? - External ETCD

First switch contect to cluster2

```bash
student-node ~ ➜  kubectl config use-context cluster2 
Switched to context "cluster2".

student-node ~ ➜  

If you check out the pods running in the kube-system namespace in cluster1, you will notice that there are NO etcd pods running in this cluster!

student-node ~ ➜  kubectl get all -A
NAMESPACE     NAME                                                READY   STATUS    RESTARTS      AGE
critical      pod/critical-deployment-240616-5bf45c9459-nb5cf     1/1     Running   0             15m
critical      pod/critical-deployment-240616-5bf45c9459-x49mv     1/1     Running   0             15m
kube-system   pod/coredns-6d4b75cb6d-h9bw8                        1/1     Running   0             81m
kube-system   pod/coredns-6d4b75cb6d-qcrs2                        1/1     Running   0             81m
kube-system   pod/kube-apiserver-cluster2-controlplane            1/1     Running   0             81m
kube-system   pod/kube-controller-manager-cluster2-controlplane   1/1     Running   0             81m
kube-system   pod/kube-proxy-6s6mj                                1/1     Running   0             81m
kube-system   pod/kube-proxy-rwbkz                                1/1     Running   0             81m
kube-system   pod/kube-scheduler-cluster2-controlplane            1/1     Running   0             81m
kube-system   pod/weave-net-k6gbg                                 2/2     Running   1 (81m ago)   81m
kube-system   pod/weave-net-xfhrl                                 2/2     Running   0             81m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  81m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   81m

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   81m
kube-system   daemonset.apps/weave-net    2         2         2       2            2           <none>                   81m

NAMESPACE     NAME                                         READY   UP-TO-DATE   AVAILABLE   AGE
critical      deployment.apps/critical-deployment-240616   2/2     2            2           15m
kube-system   deployment.apps/coredns                      2/2     2            2           81m

NAMESPACE     NAME                                                    DESIRED   CURRENT   READY   AGE
critical      replicaset.apps/critical-deployment-240616-5bf45c9459   2         2         2       15m
kube-system   replicaset.apps/coredns-6d4b75cb6d                      2         2         2       81m
```

Also, there is NO static pod configuration for etcd under the static pod path, to check that:

SSH to cluster2 controlnode

```bash
student-node ~ ➜  ssh cluster2-controlplane
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1093-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

cluster2-controlplane ~ ➜  
```

Then list the manifests for the control plane. There is no etcd

```bash
cluster2-controlplane ~ ➜  ls /etc/kubernetes/manifests/ | grep -i etcd



cluster2-controlplane ~ ➜  ls -l /etc/kubernetes/manifests/ 
total 12
-rw------- 1 root root 3832 Jan 14 04:59 kube-apiserver.yaml
-rw------- 1 root root 3365 Jan 14 04:59 kube-controller-manager.yaml
-rw------- 1 root root 1435 Jan 14 04:59 kube-scheduler.yaml

cluster2-controlplane ~ ➜  
```

However, if you inspect the process on the controlplane for cluster2, you will see that that the process for the kube-apiserver is referencing an external etcd datastore:

```bash
cluster2-controlplane ~ ➜  ps -ef | grep etcd
root        1780    1416  0 04:59 ?        00:05:41 kube-apiserver --advertise-address=10.162.94.9 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.pem --etcd-certfile=/etc/kubernetes/pki/etcd/etcd.pem --etcd-keyfile=/etc/kubernetes/pki/etcd/etcd-key.pem --etcd-servers=https://10.162.94.21:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root       10168    9709  0 06:26 pts/0    00:00:00 grep etcd

cluster2-controlplane ~ ➜  
```

You can see the same information by inspecting the kube-apiserver pod (which runs as a static pod in the kube-system namespace): etcd-servers=https://10.162.94.21:2379

```bash
cluster2-controlplane ~ ➜  kubectl -n kube-system describe pod kube-apiserver-cluster2-controlplane 
Name:                 kube-apiserver-cluster2-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 cluster2-controlplane/10.162.94.9
Start Time:           Sat, 14 Jan 2023 05:00:00 +0000
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.162.94.9:6443
                      kubernetes.io/config.hash: e0479bc6380ab82317beb83d33906a8a
                      kubernetes.io/config.mirror: e0479bc6380ab82317beb83d33906a8a
                      kubernetes.io/config.seen: 2023-01-14T04:59:59.390543335Z
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   10.162.94.9
IPs:
  IP:           10.162.94.9
Controlled By:  Node/cluster2-controlplane
Containers:
  kube-apiserver:
    Container ID:  containerd://bca23d411b2f7b5888caa24abac0f22e0d046d32147320e36c0b3c796a1d07ca
    Image:         k8s.gcr.io/kube-apiserver:v1.24.0
    Image ID:      k8s.gcr.io/kube-apiserver@sha256:a04522b882e919de6141b47d72393fb01226c78e7388400f966198222558c955
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=10.162.94.9
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --enable-admission-plugins=NodeRestriction
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/etc/kubernetes/pki/etcd/ca.pem
      --etcd-certfile=/etc/kubernetes/pki/etcd/etcd.pem
      --etcd-keyfile=/etc/kubernetes/pki/etcd/etcd-key.pem
      --etcd-servers=https://10.162.94.21:2379
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
      --requestheader-allowed-names=front-proxy-client
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
      --requestheader-extra-headers-prefix=X-Remote-Extra-
      --requestheader-group-headers=X-Remote-Group
      --requestheader-username-headers=X-Remote-User
      --secure-port=6443
      --service-account-issuer=https://kubernetes.default.svc.cluster.local
      --service-account-key-file=/etc/kubernetes/pki/sa.pub
      --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
      --service-cluster-ip-range=10.96.0.0/12
      --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
      --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    State:          Running
      Started:      Sat, 14 Jan 2023 04:59:46 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        250m
    Liveness:     http-get https://10.162.94.9:6443/livez delay=10s timeout=15s period=10s #success=1 #failure=8
    Readiness:    http-get https://10.162.94.9:6443/readyz delay=0s timeout=15s period=1s #success=1 #failure=3
    Startup:      http-get https://10.162.94.9:6443/livez delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/ca-certificates from etc-ca-certificates (ro)
      /etc/kubernetes/pki from k8s-certs (ro)
      /etc/ssl/certs from ca-certs (ro)
      /usr/local/share/ca-certificates from usr-local-share-ca-certificates (ro)
      /usr/share/ca-certificates from usr-share-ca-certificates (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  etc-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ca-certificates
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
  usr-local-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/local/share/ca-certificates
    HostPathType:  DirectoryOrCreate
  usr-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/share/ca-certificates
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:            <none>

cluster2-controlplane ~ ➜  
```

What is the default data directory used the for ETCD datastore used in cluster1?
Remember, this cluster uses a Stacked ETCD topology. - --data-dir=/var/lib/etcd


First, switch context to cluster1

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  
```

Describe the etcd pod

```bash
student-node ~ ➜  kubectl describe pods etcd-cluster1-controlplane -n kube-system
Name:                 etcd-cluster1-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 cluster1-controlplane/10.162.94.6
Start Time:           Sat, 14 Jan 2023 04:59:59 +0000
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.162.94.6:2379
                      kubernetes.io/config.hash: 9c1110318654070a1c27a94074443f3a
                      kubernetes.io/config.mirror: 9c1110318654070a1c27a94074443f3a
                      kubernetes.io/config.seen: 2023-01-14T04:59:40.191300934Z
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   10.162.94.6
IPs:
  IP:           10.162.94.6
Controlled By:  Node/cluster1-controlplane
Containers:
  etcd:
    Container ID:  containerd://d6cb7ece18ca7ee21115966773981cfe0f347dc67a9cb4d32eed113be63d272c
    Image:         k8s.gcr.io/etcd:3.5.3-0
    Image ID:      k8s.gcr.io/etcd@sha256:13f53ed1d91e2e11aac476ee9a0269fdda6cc4874eba903efd40daf50c55eee5
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://10.162.94.6:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --experimental-initial-corrupt-check=true
      --initial-advertise-peer-urls=https://10.162.94.6:2380
      --initial-cluster=cluster1-controlplane=https://10.162.94.6:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://10.162.94.6:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://10.162.94.6:2380
      --name=cluster1-controlplane
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    State:          Running
      Started:      Sat, 14 Jan 2023 04:59:43 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
      memory:     100Mi
    Liveness:     http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
      /var/lib/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:            <none>

student-node ~ ➜  
```

For the subsequent questions, you would need to login to the External ETCD server.

```bash
student-node ~ ➜  ssh 10.162.94.21
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1093-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

etcd-server ~ ➜  

etcd-server ~ ➜  hostname
etcd-server

etcd-server ~ ➜ 
```

What is the default data directory used the for ETCD datastore used in cluster2? --data-dir=/var/lib/etcd-data


```bash
etcd-server ~ ➜  ps -ef | grep etcd
etcd         832       1  0 04:59 ?        00:01:44 /usr/local/bin/etcd --name etcd-server --data-dir=/var/lib/etcd-data --cert-file=/etc/etcd/pki/etcd.pem --key-file=/etc/etcd/pki/etcd-key.pem --peer-cert-file=/etc/etcd/pki/etcd.pem --peer-key-file=/etc/etcd/pki/etcd-key.pem --trusted-ca-file=/etc/etcd/pki/ca.pem --peer-trusted-ca-file=/etc/etcd/pki/ca.pem --peer-client-cert-auth --client-cert-auth --initial-advertise-peer-urls https://10.162.94.21:2380 --listen-peer-urls https://10.162.94.21:2380 --advertise-client-urls https://10.162.94.21:2379 --listen-client-urls https://10.162.94.21:2379,https://127.0.0.1:2379 --initial-cluster-token etcd-cluster-1 --initial-cluster etcd-server=https://10.162.94.21:2380 --initial-cluster-state new
root        1193    1032  0 06:37 pts/0    00:00:00 grep etcd

etcd-server ~ ➜  
```

How many nodes are part of the ETCD cluster that etcd-server is a part of? - 1 member

```bash
etcd-server ~ ➜  ETCDCTL_API=3 etcdctl \
>  --endpoints=https://127.0.0.1:2379 \
>  --cacert=/etc/etcd/pki/ca.pem \
>  --cert=/etc/etcd/pki/etcd.pem \
>  --key=/etc/etcd/pki/etcd-key.pem \
>   member list
27b92c90b94b8ef8, started, etcd-server, https://10.162.94.21:2380, https://10.162.94.21:2379, false

etcd-server ~ ➜  
```

Take a backup of etcd on cluster1 and save it on the student-node at the path /opt/cluster1.db

First change context to cluster 1

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  
```

Next, inspect the endpoints and certificates used by the etcd pod. We will make use of these to take the backup.

```bash
student-node ~ ➜  kubectl describe  pods -n kube-system etcd-cluster1-controlplane  | grep advertise-client-urls
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.162.94.6:2379
      --advertise-client-urls=https://10.162.94.6:2379

student-node ~ ➜  
```

```bash
student-node ~ ➜  kubectl describe  pods -n kube-system etcd-cluster1-controlplane  | grep pki
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
    Path:          /etc/kubernetes/pki/etcd

student-node ~ ➜  
```

SSH to the controlplane node of cluster1 and then take the backup using the endpoints and certificates we identified above:

```bash
student-node ~ ➜  kubectl get nodes -A
NAME                    STATUS   ROLES           AGE    VERSION
cluster1-controlplane   Ready    control-plane   105m   v1.24.0
cluster1-node01         Ready    <none>          104m   v1.24.0
```

```bash

student-node ~ ➜  ssh cluster1-controlplane
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1093-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

cluster1-controlplane ~ ➜ 
```

save the snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot save /opt/cluster1.db \
--endpoints=https://10.168.38.9:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key 

Snapshot saved at /opt/cluster1.db

cluster1-controlplane ~ ➜ 
```

Finally, copy the backup to the student-node. To do this, go back to the student-node and use scp as shown below:

```bash
student-node ~ ➜  scp cluster1-controlplane:/opt/cluster1.db /opt
cluster1.db                                                                                                        100% 2088KB 112.3MB/s   00:00    

student-node ~ ➜ 
```

An ETCD backup for cluster2 is stored at /opt/cluster2.db. Use this snapshot file to carryout a restore on cluster2 to a new path /var/lib/etcd-data-new.

Once the restore is complete, ensure that the controlplane components on cluster2 are running.

The snapshot was taken when there were objects created in the critical namespace on cluster2. These objects should be available post restore.


Step 1. Copy the snapshot file from the student-node to the etcd-server. In the example below, we are copying it to the /root directory:

```bash
student-node ~ ➜  scp /opt/cluster2.db etcd-server:/root
cluster2.db                                                                                                                                   100% 2064KB  70.5MB/s   00:00    

student-node ~ ➜  
```

Step 2: Restore the snapshot on the cluster2. Since we are restoring directly on the etcd-server, we can use the endpoint https:/127.0.0.1. Use the same certificates that were identified earlier. Make sure to use the data-dir as /var/lib/etcd-data-new:

SSH to the etcd server

```bash
student-node ~ ➜  ssh etcd-server
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1093-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

etcd-server ~ ➜ 
```

Carry out the snapshot restore

```bash

etcd-server ~ ➜  ETCDCTL_API=3 etcdctl snapshot restore /root/cluster2.db --data-dir /var/lib/etcd-data-new \
> --endpoints=https://127.0.0.1:2379 \
> --cacert=/etc/etcd/pki/ca.pem \
> --cert=/etc/etcd/pki/etcd.pem \
> --key=/etc/etcd/pki/etcd-key.pem
{"level":"info","ts":1673685491.8856657,"caller":"snapshot/v3_snapshot.go:296","msg":"restoring snapshot","path":"/root/cluster2.db","wal-dir":"/var/lib/etcd-data-new/member/wal","data-dir":"/var/lib/etcd-data-new","snap-dir":"/var/lib/etcd-data-new/member/snap"}
{"level":"info","ts":1673685491.9045105,"caller":"mvcc/kvstore.go:388","msg":"restored last compact revision","meta-bucket-name":"meta","meta-bucket-name-key":"finishedCompactRev","restored-compact-revision":3624}
{"level":"info","ts":1673685491.9129672,"caller":"membership/cluster.go:392","msg":"added member","cluster-id":"cdf818194e3a8c32","local-member-id":"0","added-peer-id":"8e9e05c52164694d","added-peer-peer-urls":["http://localhost:2380"]}
{"level":"info","ts":1673685491.9200473,"caller":"snapshot/v3_snapshot.go:309","msg":"restored snapshot","path":"/root/cluster2.db","wal-dir":"/var/lib/etcd-data-new/member/wal","data-dir":"/var/lib/etcd-data-new","snap-dir":"/var/lib/etcd-data-new/member/snap"}

etcd-server ~ ➜  
```

Step 3: Update the systemd service unit file for etcdby running vi /etc/systemd/system/etcd.service and add the new value for data-dir:

```bash
etcd-server ~ ➜  vi /etc/systemd/system/etcd.service

etcd-server ~ ➜  



[Unit]
Description=etcd key-value store
Documentation=https://github.com/etcd-io/etcd
After=network.target

[Service]
User=etcd
Type=notify
ExecStart=/usr/local/bin/etcd \
  --name etcd-server \
  --data-dir=/var/lib/etcd-data-new \
  --cert-file=/etc/etcd/pki/etcd.pem \
  --key-file=/etc/etcd/pki/etcd-key.pem \
  --peer-cert-file=/etc/etcd/pki/etcd.pem \
  --peer-key-file=/etc/etcd/pki/etcd-key.pem \
  --trusted-ca-file=/etc/etcd/pki/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/pki/ca.pem \
  --peer-client-cert-auth \
  --client-cert-auth \
  --initial-advertise-peer-urls https://10.168.38.9:2380 \
  --listen-peer-urls https://10.168.38.9:2380 \
  --advertise-client-urls https://10.168.38.9:2379 \
  --listen-client-urls https://10.168.38.9:2379,https://127.0.0.1:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster etcd-server=https://10.168.38.9:2380 \
  --initial-cluster-state new
Restart=on-failure
RestartSec=5
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```

Step 4: make sure the permissions on the new directory is correct (should be owned by etcd user):

```bash
etcd-server ~ ➜  chown -R etcd:etcd /var/lib/etcd-data-new

etcd-server ~ ➜  


etcd-server ~ ➜  ls -ld /var/lib/etcd-data-new/
drwx------ 3 etcd etcd 4096 Jan 14 08:38 /var/lib/etcd-data-new/

etcd-server ~ ➜  


```

Step 5: Finally, reload and restart the etcd service.

```bash
etcd-server ~ ➜  systemctl daemon-reload

etcd-server ~ ➜  systemctl restart etcd

etcd-server ~ ➜  systemctl status etcd
● etcd.service - etcd key-value store
   Loaded: loaded (/etc/systemd/system/etcd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2023-01-14 08:42:21 UTC; 5s ago
     Docs: https://github.com/etcd-io/etcd
 Main PID: 1668 (etcd)
    Tasks: 44 (limit: 541680)
   CGroup: /system.slice/etcd.service
   ````

***The End***