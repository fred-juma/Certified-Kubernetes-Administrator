#### Task

Take the backup of ETCD at the location /opt/etcd-backup.db on the controlplane node.

#### Solution

View the pods to check the etcd pod 

```bash
controlplane ~ ➜  kubectl get pod -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS      AGE
admin2406     deploy1-78d8d47657-p8c92               1/1     Running   0             3m4s
admin2406     deploy2-d6b48767c-n9k7s                1/1     Running   0             3m4s
admin2406     deploy3-5bdcc4b88c-w8dh5               1/1     Running   0             3m3s
admin2406     deploy4-6b96456775-xrh5n               1/1     Running   0             3m3s
admin2406     deploy5-5dc4d5d78b-hp4qb               1/1     Running   0             3m3s
alpha         alpha-mysql-6d945ffc78-6vnfl           0/1     Pending   0             3m6s
default       gold-nginx-69d8d76f55-sjfcq            1/1     Running   0             3m12s
kube-system   coredns-565d847f94-b5lqx               1/1     Running   0             69m
kube-system   coredns-565d847f94-m9rxb               1/1     Running   0             69m
kube-system   etcd-controlplane                      1/1     Running   0             69m
kube-system   kube-apiserver-controlplane            1/1     Running   0             69m
kube-system   kube-controller-manager-controlplane   1/1     Running   0             69m
kube-system   kube-proxy-4hqmg                       1/1     Running   0             69m
kube-system   kube-proxy-rsz5q                       1/1     Running   0             68m
kube-system   kube-scheduler-controlplane            1/1     Running   0             69m
kube-system   weave-net-5d69h                        2/2     Running   0             68m
kube-system   weave-net-fc2tv                        2/2     Running   1 (69m ago)   69m

controlplane ~ ➜  
```

Set the *ETCDCTL_API* to *version 3*  and confirm


```bash
controlplane ~ ➜  ETCDCTL_API=3 etcdctl version
etcdctl version: 3.3.13
API version: 3.3

controlplane ~ ➜  
```

View the etcd service to list the configuration of etcd

```bash
controlplane ~ ➜  ps aux | grep etcd
root        3324  0.0  0.0 11219900 66344 ?      Ssl  01:17   2:22 etcd --advertise-client-urls=https://192.9.128.3:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --experimental-initial-corrupt-check=true --experimental-watch-progress-notify-interval=5s --initial-advertise-peer-urls=https://192.9.128.3:2380 --initial-cluster=controlplane=https://192.9.128.3:2380 --key-file=/etc/kubernetes/pki/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://192.9.128.3:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://192.9.128.3:2380 --name=controlplane --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
root        3344  0.0  0.1 1119480 338592 ?      Ssl  01:17   4:07 kube-apiserver --advertise-address=192.9.128.3 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root       13645  0.0  0.0   6744   720 pts/0    S+   02:40   0:00 grep --color=auto etcd

controlplane ~ ➜  
```

Take etcd snapshot

```bash

controlplane ~ ➜  ETCDCTL_API=3 etcdctl \
> --endpoints=https://127.0.0.1:2379 \
> --cacert=/etc/kubernetes/pki/etcd/ca.crt \
> --cert=/etc/kubernetes/pki/etcd/server.crt  \
> --key=/etc/kubernetes/pki/etcd/server.key \
> snapshot save /opt/etcd-backup.db
Snapshot saved at /opt/etcd-backup.db

controlplane ~ ➜  
```
***The End***