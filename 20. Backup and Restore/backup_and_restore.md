#### Backup and Restore

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

***The End***