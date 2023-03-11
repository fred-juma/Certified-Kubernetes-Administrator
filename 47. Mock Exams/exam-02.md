Take a backup of the etcd cluster and save it to /opt/etcd-backup.db.


controlplane ~ ➜  ps -aux | grep etcd
root        3370  0.0  0.1 1125224 334076 ?      Ssl  10:17   0:59 kube-apiserver --advertise-address=192.26.233.3 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root        3388  0.0  0.0 11220028 55748 ?      Ssl  10:17   0:31 etcd --advertise-client-urls=https://192.26.233.3:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --experimental-initial-corrupt-check=true --experimental-watch-progress-notify-interval=5s --initial-advertise-peer-urls=https://192.26.233.3:2380 --initial-cluster=controlplane=https://192.26.233.3:2380 --key-file=/etc/kubernetes/pki/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://192.26.233.3:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://192.26.233.3:2380 --name=controlplane --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
root        8698  0.0  0.0   6744   656 pts/0    S+   10:34   0:00 grep --color=auto etcd



controlplane ~ ➜  ETCDCTL_API=3 etcdctl version
etcdctl version: 3.3.1
API version: 3.3

controlplane ~ ➜  

controlplane ~ ➜  ETCDCTL_API=3 etcdctl \
> --endpoints=https://127.0.0.1:2379 \
> --cacert=/etc/kubernetes/pki/etcd/ca.crt \
> --cert=/etc/kubernetes/pki/etcd/server.crt  \
> --key=/etc/kubernetes/pki/etcd/server.key  \
> snapshot save /opt/etcd-backup.db
Snapshot saved at /opt/etcd-backup.db

controlplane ~ ➜  



Create a Pod called redis-storage with image: redis:alpine with a Volume of type emptyDir that lasts for the life of the Pod.

Specs on the below.

Pod named 'redis-storage' created

Pod 'redis-storage' uses Volume type of emptyDir

Pod 'redis-storage' uses volumeMount with mountPath = /data/redis


controlplane ~ ➜  kubectl run redis-storage --image=redis:alpine -o yaml --dry-run=client > redis-storage.yaml

controlplane ~ ➜  

apiVersion: v1
kind: Pod
metadata:
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-storage
    volumeMounts:
    - mountPath: /data/redis
      name: redis-storage
  volumes:
  - name:  redis-storage
    emptyDir:

controlplane ~ ➜  kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
redis-storage   1/1     Running   0          16s

controlplane ~ ➜  



Create a new pod called super-user-pod with image busybox:1.28. Allow the pod to be able to set system_time.

The container should sleep for 4800 seconds.

Pod: super-user-pod

Container Image: busybox:1.28

SYS_TIME capabilities for the conatiner?


controlplane ~ ➜ kubectl run super-user-pod --image=busybox:1.28 -o yaml --dry-run=client > super-user-pod.yaml

controlplane ~ ➜  


apiVersion: v1
kind: Pod
metadata:
  name: super-user-pod
spec:
  containers:
  - image: busybox:1.28
    name: super-user-pod
    command: ["sleep"]
    args: ["4800"]
    securityContext:
      capabilities:
        add: ["SYS_TIME"]


controlplane ~ ➜  kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
redis-storage    1/1     Running   0          8m48s
super-user-pod   1/1     Running   0          5s

controlplane ~ ➜  


A pod definition file is created at /root/CKA/use-pv.yaml. Make use of this manifest file and mount the persistent volume called pv-1. Ensure the pod is running and the PV is bound.

mountPath: /data

persistentVolumeClaim Name: my-pvc

persistentVolume Claim configured correctly

pod using the correct mountPath

pod using the persistent volume claim?



controlplane ~ ➜  vi /root/CKA/use-pv.yaml


apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


ontrolplane ~ ✖ kubectl get persistentvolume
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-1   10Mi       RWO            Retain           Available                                   99s

controlplane ~ ➜  



controlplane ~ ➜  vi my-pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi


controlplane ~ ➜  kubectl apply -f my-pvc.yaml 
persistentvolumeclaim/my-pvc created

controlplane ~ ➜  

controlplane ~ ➜  kubectl get persistentvolumeclaim
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pvc   Bound    pv-1     10Mi       RWO                           6s

controlplane ~ ➜  

apiVersion: v1
kind: Pod
metadata:
  name: use-pv
spec:
  volumes:
  - name: pv-1
    persistentVolumeClaim:
      claimName: my-pvc
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
    - mountPath: /data
      name: pv-1
    

controlplane ~ ➜  kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
redis-storage    1/1     Running   0          25m
super-user-pod   1/1     Running   0          16m
use-pv           1/1     Running   0          6m56s

controlplane ~ ➜  


Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.

Deployment : nginx-deploy. Image: nginx:1.16

Image: nginx:1.16

Task: Upgrade the version of the deployment to 1:17

Task: Record the changes for the image upgrade

controlplane ~ ➜  kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=1
deployment.apps/nginx-deploy created

controlplane ~ ➜  



controlplane ~ ➜  kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-7dc9dd795f-bnl46   1/1     Running   0          10s
pod/redis-storage                   1/1     Running   0          26m
pod/super-user-pod                  1/1     Running   0          18m
pod/use-pv                          1/1     Running   0          8m17s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   57m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   1/1     1            1           10s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-7dc9dd795f   1         1         1       10s

controlplane ~ ➜  


controlplane ~ ➜  kubectl set image deployment nginx-deploy nginx=1.17 --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/nginx-deploy image updated

controlplane ~ ➜  


controlplane ~ ➜  kubectl rollout history deployment nginx-deploy
deployment.apps/nginx-deploy 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deploy nginx=1.17 --record=true


controlplane ~ ➜  



Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace . The private key exists in the location: /root/CKA/john.key and csr at /root/CKA/john.csr.

Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.

Please refer the documentation to see an example. The documentation tab is available at the top right of terminal.

CSR: john-developer Status:Approved

Role Name: developer, namespace: development, Resource: Pods

Access: User 'john' has appropriate permissions








Create a static pod on node01 called nginx-critical with image nginx and make sure that it is recreated/restarted automatically in case of a failure.

Use /etc/kubernetes/manifests as the Static Pod path for example.

static pod configured under /etc/kubernetes/manifests ?

Pod nginx-critical-node01 is up and running




controlplane ~ ➜  kubectl get node node01 --show-labels
NAME     STATUS   ROLES    AGE   VERSION   LABELS
node01   Ready    <none>   63m   v1.26.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux

controlplane ~ ➜  


controlplane ~ ✖ kubectl run nginx-critical --image=nginx -o yaml --dry-run=client > /etc/kubernetes/manifests/nginx-critical.yaml

controlplane ~ ➜  vi /etc/kubernetes/manifests/nginx-critical.yaml

controlplane ~ ➜  kubectl apply -f /etc/kubernetes/manifests/nginx-critical.yaml
pod/nginx-critical created

controlplane ~ ➜  kubectl get pod
NAME                            READY   STATUS             RESTARTS   AGE
nginx-critical                  1/1     Running            0          6s
nginx-critical-controlplane     1/1     Running            0          15s
nginx-deploy-78bf874d5d-p4lwh   0/1     ImagePullBackOff   0          13m
nginx-deploy-7dc9dd795f-bnl46   1/1     Running            0          15m
redis-storage                   1/1     Running            0          42m
super-user-pod                  1/1     Running            0          33m
use-pv                          1/1     Running            0          23m

controlplane ~ ➜ 


apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-critical
  name: nginx-critical
spec:
  nodeName: node01
  containers:
  - image: nginx
    name: nginx-critical


Create a nginx pod called nginx-resolver using image nginx, expose it internally with a service called nginx-resolver-service. Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup. Record results in /root/CKA/nginx.svc and /root/CKA/nginx.pod

Pod: nginx-resolver created

Service DNS Resolution recorded correctly

Pod DNS resolution recorded correctly




Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace . The private key exists in the location: /root/CKA/john.key and csr at /root/CKA/john.csr.

Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.

Please refer the documentation to see an example. The documentation tab is available at the top right of terminal.

CSR: john-developer Status:Approved

Role Name: developer, namespace: development, Resource: Pods

Access: User 'john' has appropriate permissions

##### Create CertificateSigningRequest


```bash
controlplane ~ ➜  cat /root/CKA/john.csr | base64 | tr -d "\n"
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQVByVHNwN1M4Mkp2MmVwM2phNWFoeEcwNytmTGdoK21QSnBVUGdMbzAvV0ZmOWE3CkpISEtBRDgvRlAyWXcrN2tqTWdzd0tiS0loOEFDRCtEWkhTL2hFK2xhakd2ZFBHM1dvR3h6dlVCV2ZMSUd2VFUKR3hLMlZRNGgzdjhNTE54WGphc2hVcGNkeDdncVhRMjhZUGFybFg3TkZzQ2lld3A0MmpJUjl2WmN6M2N3N0FjRQpnTWo0VmNZSFE0c3lzQUNkam9IKzFXQkNBanAvYVVuTEhIYUN0blV0Tis5SVovc0ZRQ1lVTW5nV2kvSlFIaS82CjZSbXlqZmh4eU5YUnBCRHJOa0pqNWxBZ2tlblExWFNQWVJjcmRsYWQyMkQ3eEE1bFF3dWUvVVJIVVdJNzFDVFkKbTByckYwMlpiL2FKcEg3aEoyUXNseFRJalpFeE8rcTRZWUtvTkdzQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQWlqbUJFejNuVnBlc3lpYS9OazVsU2IrVHRVMFhYTDlRbnpPTWtDVXlCOERtN3Y1L2VjR2ZuCmpjdVU2UFpnUmZyaGczU3dEL294S0l4N0pVZ1RDUU5xSUtKcy96QzAxNnhydDFQU3JBaXVwTVpkNGRzdVZvczIKUDNsa3BOUUk1WU13QW52SFBDQ3F5SGVnSGpnbW9zQUZuS0JRajE2T0hQd2FCcUZVcHJsOS9kTlBGMkwyVzNoZwp5K2N2Sm5xSEJJSGRZa3hJcmJndGxISm12WXp0L0tkWWRoUTlpL3BPUS9DQ2JjYy8xTkR1cHVrSmZUeUk5MklYCkV1ekU5Mmp2VGxTaXRsSWFCR0w2SFZ6Z0ZVbmxDS3p1bzllK2JLTUFDVmYxeHQrZXpZb3ljTStWWG41b3c1MU8KYkdZSVVwWjdXUDY4UWhwZ1RURjVqU0luMW5JdE9oNksKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
controlplane ~ ➜  
```

john.yaml
```bash
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQVByVHNwN1M4Mkp2MmVwM2phNWFoeEcwNytmTGdoK21QSnBVUGdMbzAvV0ZmOWE3CkpISEtBRDgvRlAyWXcrN2tqTWdzd0tiS0loOEFDRCtEWkhTL2hFK2xhakd2ZFBHM1dvR3h6dlVCV2ZMSUd2VFUKR3hLMlZRNGgzdjhNTE54WGphc2hVcGNkeDdncVhRMjhZUGFybFg3TkZzQ2lld3A0MmpJUjl2WmN6M2N3N0FjRQpnTWo0VmNZSFE0c3lzQUNkam9IKzFXQkNBanAvYVVuTEhIYUN0blV0Tis5SVovc0ZRQ1lVTW5nV2kvSlFIaS82CjZSbXlqZmh4eU5YUnBCRHJOa0pqNWxBZ2tlblExWFNQWVJjcmRsYWQyMkQ3eEE1bFF3dWUvVVJIVVdJNzFDVFkKbTByckYwMlpiL2FKcEg3aEoyUXNseFRJalpFeE8rcTRZWUtvTkdzQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQWlqbUJFejNuVnBlc3lpYS9OazVsU2IrVHRVMFhYTDlRbnpPTWtDVXlCOERtN3Y1L2VjR2ZuCmpjdVU2UFpnUmZyaGczU3dEL294S0l4N0pVZ1RDUU5xSUtKcy96QzAxNnhydDFQU3JBaXVwTVpkNGRzdVZvczIKUDNsa3BOUUk1WU13QW52SFBDQ3F5SGVnSGpnbW9zQUZuS0JRajE2T0hQd2FCcUZVcHJsOS9kTlBGMkwyVzNoZwp5K2N2Sm5xSEJJSGRZa3hJcmJndGxISm12WXp0L0tkWWRoUTlpL3BPUS9DQ2JjYy8xTkR1cHVrSmZUeUk5MklYCkV1ekU5Mmp2VGxTaXRsSWFCR0w2SFZ6Z0ZVbmxDS3p1bzllK2JLTUFDVmYxeHQrZXpZb3ljTStWWG41b3c1MU8KYkdZSVVwWjdXUDY4UWhwZ1RURjVqU0luMW5JdE9oNksKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```


Apply the CSR

```bash
controlplane ~ ➜  kubectl apply -f john.yaml 
certificatesigningrequest.certificates.k8s.io/john created

controlplane ~ ➜  
```


View the CSR, status is pending

```bash
controlplane ~ ➜  kubectl get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
csr-6qzc2   35m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:kv5mjk    <none>              Approved,Issued
csr-zn44f   36m   kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued
john        34s   kubernetes.io/kube-apiserver-client           kubernetes-admin           24h                 Pending

controlplane ~ ➜  
```

##### Approve certificate signing request 

```bash
controlplane ~ ➜  kubectl certificate approve john
certificatesigningrequest.certificates.k8s.io/john approved

controlplane ~ ➜  
```

View csr status. Approved and Issued.

```bash
controlplane ~ ➜  kubectl get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
csr-6qzc2   37m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:kv5mjk    <none>              Approved,Issued
csr-zn44f   37m   kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued
john        2m    kubernetes.io/kube-apiserver-client           kubernetes-admin           24h                 Approved,Issued

controlplane ~ ➜  
```

##### Get the certificate

```bash
controlplane ~ ➜  kubectl get csr/john -o yaml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"certificates.k8s.io/v1","kind":"CertificateSigningRequest","metadata":{"annotations":{},"name":"john"},"spec":{"expirationSeconds":86400,"request":"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQVByVHNwN1M4Mkp2MmVwM2phNWFoeEcwNytmTGdoK21QSnBVUGdMbzAvV0ZmOWE3CkpISEtBRDgvRlAyWXcrN2tqTWdzd0tiS0loOEFDRCtEWkhTL2hFK2xhakd2ZFBHM1dvR3h6dlVCV2ZMSUd2VFUKR3hLMlZRNGgzdjhNTE54WGphc2hVcGNkeDdncVhRMjhZUGFybFg3TkZzQ2lld3A0MmpJUjl2WmN6M2N3N0FjRQpnTWo0VmNZSFE0c3lzQUNkam9IKzFXQkNBanAvYVVuTEhIYUN0blV0Tis5SVovc0ZRQ1lVTW5nV2kvSlFIaS82CjZSbXlqZmh4eU5YUnBCRHJOa0pqNWxBZ2tlblExWFNQWVJjcmRsYWQyMkQ3eEE1bFF3dWUvVVJIVVdJNzFDVFkKbTByckYwMlpiL2FKcEg3aEoyUXNseFRJalpFeE8rcTRZWUtvTkdzQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQWlqbUJFejNuVnBlc3lpYS9OazVsU2IrVHRVMFhYTDlRbnpPTWtDVXlCOERtN3Y1L2VjR2ZuCmpjdVU2UFpnUmZyaGczU3dEL294S0l4N0pVZ1RDUU5xSUtKcy96QzAxNnhydDFQU3JBaXVwTVpkNGRzdVZvczIKUDNsa3BOUUk1WU13QW52SFBDQ3F5SGVnSGpnbW9zQUZuS0JRajE2T0hQd2FCcUZVcHJsOS9kTlBGMkwyVzNoZwp5K2N2Sm5xSEJJSGRZa3hJcmJndGxISm12WXp0L0tkWWRoUTlpL3BPUS9DQ2JjYy8xTkR1cHVrSmZUeUk5MklYCkV1ekU5Mmp2VGxTaXRsSWFCR0w2SFZ6Z0ZVbmxDS3p1bzllK2JLTUFDVmYxeHQrZXpZb3ljTStWWG41b3c1MU8KYkdZSVVwWjdXUDY4UWhwZ1RURjVqU0luMW5JdE9oNksKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==","signerName":"kubernetes.io/kube-apiserver-client","usages":["client auth"]}}
  creationTimestamp: "2023-03-11T11:51:38Z"
  name: john
  resourceVersion: "3617"
  uid: 0e38b26b-2af5-439f-9a09-7aea1d1ac50d
spec:
  expirationSeconds: 86400
  groups:
  - system:masters
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQVByVHNwN1M4Mkp2MmVwM2phNWFoeEcwNytmTGdoK21QSnBVUGdMbzAvV0ZmOWE3CkpISEtBRDgvRlAyWXcrN2tqTWdzd0tiS0loOEFDRCtEWkhTL2hFK2xhakd2ZFBHM1dvR3h6dlVCV2ZMSUd2VFUKR3hLMlZRNGgzdjhNTE54WGphc2hVcGNkeDdncVhRMjhZUGFybFg3TkZzQ2lld3A0MmpJUjl2WmN6M2N3N0FjRQpnTWo0VmNZSFE0c3lzQUNkam9IKzFXQkNBanAvYVVuTEhIYUN0blV0Tis5SVovc0ZRQ1lVTW5nV2kvSlFIaS82CjZSbXlqZmh4eU5YUnBCRHJOa0pqNWxBZ2tlblExWFNQWVJjcmRsYWQyMkQ3eEE1bFF3dWUvVVJIVVdJNzFDVFkKbTByckYwMlpiL2FKcEg3aEoyUXNseFRJalpFeE8rcTRZWUtvTkdzQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQWlqbUJFejNuVnBlc3lpYS9OazVsU2IrVHRVMFhYTDlRbnpPTWtDVXlCOERtN3Y1L2VjR2ZuCmpjdVU2UFpnUmZyaGczU3dEL294S0l4N0pVZ1RDUU5xSUtKcy96QzAxNnhydDFQU3JBaXVwTVpkNGRzdVZvczIKUDNsa3BOUUk1WU13QW52SFBDQ3F5SGVnSGpnbW9zQUZuS0JRajE2T0hQd2FCcUZVcHJsOS9kTlBGMkwyVzNoZwp5K2N2Sm5xSEJJSGRZa3hJcmJndGxISm12WXp0L0tkWWRoUTlpL3BPUS9DQ2JjYy8xTkR1cHVrSmZUeUk5MklYCkV1ekU5Mmp2VGxTaXRsSWFCR0w2SFZ6Z0ZVbmxDS3p1bzllK2JLTUFDVmYxeHQrZXpZb3ljTStWWG41b3c1MU8KYkdZSVVwWjdXUDY4UWhwZ1RURjVqU0luMW5JdE9oNksKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
  username: kubernetes-admin
status:
  certificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM5RENDQWR5Z0F3SUJBZ0lRTVAxZEg5U21XM1puUjBjazRCRkpXVEFOQmdrcWhraUc5dzBCQVFzRkFEQVYKTVJNd0VRWURWUVFERXdwcmRXSmxjbTVsZEdWek1CNFhEVEl6TURNeE1URXhORGd4T1ZvWERUSXpNRE14TWpFeApORGd4T1Zvd0R6RU5NQXNHQTFVRUF4TUVhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDCkFRb0NnZ0VCQVByVHNwN1M4Mkp2MmVwM2phNWFoeEcwNytmTGdoK21QSnBVUGdMbzAvV0ZmOWE3SkhIS0FEOC8KRlAyWXcrN2tqTWdzd0tiS0loOEFDRCtEWkhTL2hFK2xhakd2ZFBHM1dvR3h6dlVCV2ZMSUd2VFVHeEsyVlE0aAozdjhNTE54WGphc2hVcGNkeDdncVhRMjhZUGFybFg3TkZzQ2lld3A0MmpJUjl2WmN6M2N3N0FjRWdNajRWY1lIClE0c3lzQUNkam9IKzFXQkNBanAvYVVuTEhIYUN0blV0Tis5SVovc0ZRQ1lVTW5nV2kvSlFIaS82NlJteWpmaHgKeU5YUnBCRHJOa0pqNWxBZ2tlblExWFNQWVJjcmRsYWQyMkQ3eEE1bFF3dWUvVVJIVVdJNzFDVFltMHJyRjAyWgpiL2FKcEg3aEoyUXNseFRJalpFeE8rcTRZWUtvTkdzQ0F3RUFBYU5HTUVRd0V3WURWUjBsQkF3d0NnWUlLd1lCCkJRVUhBd0l3REFZRFZSMFRBUUgvQkFJd0FEQWZCZ05WSFNNRUdEQVdnQlFMUlpkeEN3cnkxZURZVWtaVWhWMW4KTEtBSGp6QU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFvZ1E4N3pwcW16U2RTN1creUlVU09XV3cxY0xBK1pldwoxdERqK08veUxUanE5SVBkdHlzcHR3a0lRV0JqSVowWmIvU1Fhd3FIbFl6MEdqcDNKdVFzVVdERTRZVC9WOElMCkxxN1psWUdtRFh1L0MxeDFMbVhqck1FZDBJTlNBVGV6SWV6ODFuVHJRVFhQeFozRG9GdW1JVyt5V1VFQVZYUUcKbm5iNlgwb2czY1RhU1dxdjh2QTAxUVFDNHlZbFZaTUZ0QmtNcmkzRXRFRTd4cXFYRVhRT0hxNzhKNUF1RlBjKwpUTHpKRjBWcnQ5QVdJSlNzRURPR0ZiVkNPS010bnlLZ2RENGlpYjd1MXZiSmxMcUFZb3RyanEvb05uWDJGVXJPCmRXVUkyV3ZBbHdtK08zN3Q5TS85MUFab1ZpbmI5Ym8xV0p4Uk1MOWRuWitpa2xrclJaRDdSdz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  conditions:
  - lastTransitionTime: "2023-03-11T11:53:19Z"
    lastUpdateTime: "2023-03-11T11:53:19Z"
    message: This CSR was approved by kubectl certificate approve.
    reason: KubectlApprove
    status: "True"
    type: Approved

controlplane ~ ➜ 
```

##### Export the issued certificate from the CertificateSigningRequest

```bash
controlplane ~ ➜  kubectl get csr john -o jsonpath='{.status.certificate}'| base64 -d > john.crt

controlplane ~ ➜  cat john.crt 
-----BEGIN CERTIFICATE-----
MIIC9DCCAdygAwIBAgIQMP1dH9SmW3ZnR0ck4BFJWTANBgkqhkiG9w0BAQsFADAV
MRMwEQYDVQQDEwprdWJlcm5ldGVzMB4XDTIzMDMxMTExNDgxOVoXDTIzMDMxMjEx
NDgxOVowDzENMAsGA1UEAxMEam9objCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC
AQoCggEBAPrTsp7S82Jv2ep3ja5ahxG07+fLgh+mPJpUPgLo0/WFf9a7JHHKAD8/
FP2Yw+7kjMgswKbKIh8ACD+DZHS/hE+lajGvdPG3WoGxzvUBWfLIGvTUGxK2VQ4h
3v8MLNxXjashUpcdx7gqXQ28YParlX7NFsCiewp42jIR9vZcz3cw7AcEgMj4VcYH
Q4sysACdjoH+1WBCAjp/aUnLHHaCtnUtN+9IZ/sFQCYUMngWi/JQHi/66Rmyjfhx
yNXRpBDrNkJj5lAgkenQ1XSPYRcrdlad22D7xA5lQwue/URHUWI71CTYm0rrF02Z
b/aJpH7hJ2QslxTIjZExO+q4YYKoNGsCAwEAAaNGMEQwEwYDVR0lBAwwCgYIKwYB
BQUHAwIwDAYDVR0TAQH/BAIwADAfBgNVHSMEGDAWgBQLRZdxCwry1eDYUkZUhV1n
LKAHjzANBgkqhkiG9w0BAQsFAAOCAQEAogQ87zpqmzSdS7W+yIUSOWWw1cLA+Zew
1tDj+O/yLTjq9IPdtysptwkIQWBjIZ0Zb/SQawqHlYz0Gjp3JuQsUWDE4YT/V8IL
Lq7ZlYGmDXu/C1x1LmXjrMEd0INSATezIez81nTrQTXPxZ3DoFumIW+yWUEAVXQG
nnb6X0og3cTaSWqv8vA01QQC4yYlVZMFtBkMri3EtEE7xqqXEXQOHq78J5AuFPc+
TLzJF0Vrt9AWIJSsEDOGFbVCOKMtnyKgdD4iib7u1vbJlLqAYotrjq/oNnX2FUrO
dWUI2WvAlwm+O37t9M/91AZoVinb9bo1WJxRML9dnZ+iklkrRZD7Rw==
-----END CERTIFICATE-----

controlplane ~ ➜  
```

##### Create Role 

```bash
controlplane ~ ➜  kubectl create role pod-developer --verb=create,get,list,update,delete --resource=pods --namespace=development
role.rbac.authorization.k8s.io/pod-developer created

controlplane ~ ➜   
```

```bash
controlplane ~ ➜  kubectl get role -n development
NAME            CREATED AT
pod-developer   2023-03-11T12:00:37Z

controlplane ~ ➜  
```

```bash

controlplane ~ ➜  kubectl describe role pod-developer -n development
Name:         pod-developer
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [create get list update delete]

controlplane ~ ➜  
```

#####  create RoleBinding 

```bash
controlplane ~ ➜  kubectl create rolebinding pod-developer-binding --role=jpod-developer --user=john --namespace=development
rolebinding.rbac.authorization.k8s.io/pod-developer-binding created

controlplane ~ ➜  kubectl get rolebinding -n development
NAME                    ROLE                  AGE
pod-developer-binding   Role/jpod-developer   12s

controlplane ~ ➜  kubectl describe rolebinding pod-developer-binding -n development
Name:         pod-developer-binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  jpod-developer
Subjects:
  Kind  Name  Namespace
  ----  ----  ---------
  User  john  

controlplane ~ ➜  
```

apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: john-pod-reader-binding
  namespace: development
subjects:
- kind: User
  name: john 
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role 
  name: pod-reader 
  apiGroup: rbac.authorization.k8s.io


##### Add user to kubeconfig

- Step 1: add new credentials

```bash
controlplane ~ ➜  kubectl config set-credentials john --client-key=/root/CKA/john.key --client-certificate=john.crt --embed-certs=true
User "john" set.

controlplane ~ ➜  
```

- step 2: add the context:

```bash
controlplane ~ ➜  kubectl config set-context john --cluster=kubernetes --user=john
Context "john" created.

controlplane ~ ➜  
```

- step 3: change the context to john

```bash
controlplane ~ ➜  kubectl config use-context john
Switched to context "john".

controlplane ~ ➜  

```

```bash
controlplane ~ ➜  kubectl auth can-i create pods --as john --namespace development
yes

controlplane ~ ➜  
```

Create a test pod
```bash
controlplane ~ ➜  kubectl run nginx --image=nginx
pod/nginx created
```

View created pod

```bash

controlplane ~ ➜  kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          8s

```