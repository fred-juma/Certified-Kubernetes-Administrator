#### Task 1
Take a backup of the etcd cluster and save it to /opt/etcd-backup.db.

Retrieve etcd configuration

```bash
controlplane ~ ➜  ps -aux | grep etcd
root        3370  0.0  0.1 1125224 334076 ?      Ssl  10:17   0:59 kube-apiserver --advertise-address=192.26.233.3 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root        3388  0.0  0.0 11220028 55748 ?      Ssl  10:17   0:31 etcd --advertise-client-urls=https://192.26.233.3:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --experimental-initial-corrupt-check=true --experimental-watch-progress-notify-interval=5s --initial-advertise-peer-urls=https://192.26.233.3:2380 --initial-cluster=controlplane=https://192.26.233.3:2380 --key-file=/etc/kubernetes/pki/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://192.26.233.3:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://192.26.233.3:2380 --name=controlplane --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
root        8698  0.0  0.0   6744   656 pts/0    S+   10:34   0:00 grep --color=auto etcd
```

Set etcdctl to version 3 and check version

```bash
controlplane ~ ➜  ETCDCTL_API=3 etcdctl version
etcdctl version: 3.3.1
API version: 3.3

controlplane ~ ➜  
```

Take etcd snapshot

```bash
controlplane ~ ➜  ETCDCTL_API=3 etcdctl \
> --endpoints=https://127.0.0.1:2379 \
> --cacert=/etc/kubernetes/pki/etcd/ca.crt \
> --cert=/etc/kubernetes/pki/etcd/server.crt  \
> --key=/etc/kubernetes/pki/etcd/server.key  \
> snapshot save /opt/etcd-backup.db
Snapshot saved at /opt/etcd-backup.db

controlplane ~ ➜  
```

#### Task 2

Create a Pod called redis-storage with image: redis:alpine with a Volume of type emptyDir that lasts for the life of the Pod.

Specs on the below.

Pod named 'redis-storage' created

Pod 'redis-storage' uses Volume type of emptyDir

Pod 'redis-storage' uses volumeMount with mountPath = /data/redis

Create pod manifest

```bash
controlplane ~ ➜  kubectl run redis-storage --image=redis:alpine -o yaml --dry-run=client > redis-storage.yaml
```

Update the pod manifest with volume and volumeMount parameters


```bash
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
```

Apply pod manifest to create the pod

```bash
controlplane ~ ➜  kubectl apply -f redis-storage.yaml
pod/redis-storage created

controlplane ~ ➜  
```

Confir the pod is ready and running

```bash
controlplane ~ ➜  kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
redis-storage   1/1     Running   0          16s

controlplane ~ ➜  
```

#### Task 3

Create a new pod called super-user-pod with image busybox:1.28. Allow the pod to be able to set system_time.

The container should sleep for 4800 seconds.

Pod: super-user-pod

Container Image: busybox:1.28

SYS_TIME capabilities for the conatiner?

Create th epod manifest

```bash
controlplane ~ ➜ controlplane ~ ➜  kubectl run super-user-pod --image=busybox:1.28 -o yaml --dry-run=client -- sleep 4800 > super-user-pod.yaml
```

Update the pod manifest with securityContext capabilities

```bash
controlplane ~ ➜  vi super-user-pod.yaml


apiVersion: v1
kind: Pod
metadata:
  labels:
    run: super-user-pod
  name: super-user-pod
spec:
  containers:
  - args:
    - sleep
    - "4800"
    image: busybox:1.28
    name: super-user-pod
    securityContext:
      capabilities:
        ["SYS_TIME"]
```

Apply the pod manifest to create the pod

```bash
controlplane ~ ➜  kubectl apply -f super-user-pod.yaml
pod/super-user-pod created

controlplane ~ ➜  
```

Get pod status to confirm pod is running

```bash
controlplane ~ ➜  kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
redis-storage    1/1     Running   0          8m48s
super-user-pod   1/1     Running   0          5s

controlplane ~ ➜  
```

#### Task 4

A pod definition file is created at /root/CKA/use-pv.yaml. Make use of this manifest file and mount the persistent volume called pv-1. Ensure the pod is running and the PV is bound.

mountPath: /data

persistentVolumeClaim Name: my-pvc

persistentVolume Claim configured correctly

pod using the correct mountPath

pod using the persistent volume claim?



Get the persistent volume, confirm name and storage


```bash
controlplane ~ ➜ kubectl get persistentvolume
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-1   10Mi       RWO            Retain           Available                                   99s

controlplane ~ ➜  
```

Create persistentVolumeClaim manifest

```bash

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
```

Apply the pvc manifest to create the pvc object

```bash

controlplane ~ ➜  kubectl apply -f my-pvc.yaml 
persistentvolumeclaim/my-pvc created

controlplane ~ ➜  
```

Confirm the pvc is created and bound

```bash
controlplane ~ ➜  kubectl get persistentvolumeclaim
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pvc   Bound    pv-1     10Mi       RWO                           6s

controlplane ~ ➜  
```

Update the pod definition provided with the volume and volumeMount

```bash

controlplane ~ ➜  vi /root/CKA/use-pv.yaml
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
```    

Get pods and confirm the new pod is ready and running

```bash
controlplane ~ ➜  kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
redis-storage    1/1     Running   0          25m
super-user-pod   1/1     Running   0          16m
use-pv           1/1     Running   0          6m56s

controlplane ~ ➜  
```

##### Task 5


Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.

Deployment : nginx-deploy. Image: nginx:1.16

Image: nginx:1.16

Task: Upgrade the version of the deployment to 1:17

Task: Record the changes for the image upgrade

Create deployment

```bash
controlplane ~ ➜  kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=1
deployment.apps/nginx-deploy created

controlplane ~ ➜  
```

Confirm the deployment object and resources are created and running

```bash

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
```

Update the deployment image and record the rollout

```bash
controlplane ~ ➜  kubectl set image deployment nginx-deploy nginx=1.17 --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/nginx-deploy image updated

controlplane ~ ➜  
```

Check the rollout history

```bash
controlplane ~ ➜  kubectl rollout history deployment nginx-deploy
deployment.apps/nginx-deploy 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deploy nginx=1.17 --record=true


controlplane ~ ➜  
```

#### Task 6

Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace . The private key exists in the location: /root/CKA/john.key and csr at /root/CKA/john.csr.

Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.

Please refer the documentation to see an example. The documentation tab is available at the top right of terminal.

CSR: john-developer Status:Approved

Role Name: developer, namespace: development, Resource: Pods

Access: User 'john' has appropriate permissions



Create CertificateSigningRequest by first converting the provided *csr* into base64


```bash
controlplane ~ ➜  cat /root/CKA/john.csr | base64 | tr -d "\n"
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQVByVHNwN1M4Mkp2MmVwM2phNWFoeEcwNytmTGdoK21QSnBVUGdMbzAvV0ZmOWE3CkpISEtBRDgvRlAyWXcrN2tqTWdzd0tiS0loOEFDRCtEWkhTL2hFK2xhakd2ZFBHM1dvR3h6dlVCV2ZMSUd2VFUKR3hLMlZRNGgzdjhNTE54WGphc2hVcGNkeDdncVhRMjhZUGFybFg3TkZzQ2lld3A0MmpJUjl2WmN6M2N3N0FjRQpnTWo0VmNZSFE0c3lzQUNkam9IKzFXQkNBanAvYVVuTEhIYUN0blV0Tis5SVovc0ZRQ1lVTW5nV2kvSlFIaS82CjZSbXlqZmh4eU5YUnBCRHJOa0pqNWxBZ2tlblExWFNQWVJjcmRsYWQyMkQ3eEE1bFF3dWUvVVJIVVdJNzFDVFkKbTByckYwMlpiL2FKcEg3aEoyUXNseFRJalpFeE8rcTRZWUtvTkdzQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQWlqbUJFejNuVnBlc3lpYS9OazVsU2IrVHRVMFhYTDlRbnpPTWtDVXlCOERtN3Y1L2VjR2ZuCmpjdVU2UFpnUmZyaGczU3dEL294S0l4N0pVZ1RDUU5xSUtKcy96QzAxNnhydDFQU3JBaXVwTVpkNGRzdVZvczIKUDNsa3BOUUk1WU13QW52SFBDQ3F5SGVnSGpnbW9zQUZuS0JRajE2T0hQd2FCcUZVcHJsOS9kTlBGMkwyVzNoZwp5K2N2Sm5xSEJJSGRZa3hJcmJndGxISm12WXp0L0tkWWRoUTlpL3BPUS9DQ2JjYy8xTkR1cHVrSmZUeUk5MklYCkV1ekU5Mmp2VGxTaXRsSWFCR0w2SFZ6Z0ZVbmxDS3p1bzllK2JLTUFDVmYxeHQrZXpZb3ljTStWWG41b3c1MU8KYkdZSVVwWjdXUDY4UWhwZ1RURjVqU0luMW5JdE9oNksKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
controlplane ~ ➜  
```

Update the CSR manifest with the base64 encoded csr and the namespace
controlplane ~ ➜  vi john-developer.yaml

```bash
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
  namespace: development
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQVByVHNwN1M4Mkp2MmVwM2phNWFoeEcwNytmTGdoK21QSnBVUGdMbzAvV0ZmOWE3CkpISEtBRDgvRlAyWXcrN2tqTWdzd0tiS0loOEFDRCtEWkhTL2hFK2xhakd2ZFBHM1dvR3h6dlVCV2ZMSUd2VFUKR3hLMlZRNGgzdjhNTE54WGphc2hVcGNkeDdncVhRMjhZUGFybFg3TkZzQ2lld3A0MmpJUjl2WmN6M2N3N0FjRQpnTWo0VmNZSFE0c3lzQUNkam9IKzFXQkNBanAvYVVuTEhIYUN0blV0Tis5SVovc0ZRQ1lVTW5nV2kvSlFIaS82CjZSbXlqZmh4eU5YUnBCRHJOa0pqNWxBZ2tlblExWFNQWVJjcmRsYWQyMkQ3eEE1bFF3dWUvVVJIVVdJNzFDVFkKbTByckYwMlpiL2FKcEg3aEoyUXNseFRJalpFeE8rcTRZWUtvTkdzQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQWlqbUJFejNuVnBlc3lpYS9OazVsU2IrVHRVMFhYTDlRbnpPTWtDVXlCOERtN3Y1L2VjR2ZuCmpjdVU2UFpnUmZyaGczU3dEL294S0l4N0pVZ1RDUU5xSUtKcy96QzAxNnhydDFQU3JBaXVwTVpkNGRzdVZvczIKUDNsa3BOUUk1WU13QW52SFBDQ3F5SGVnSGpnbW9zQUZuS0JRajE2T0hQd2FCcUZVcHJsOS9kTlBGMkwyVzNoZwp5K2N2Sm5xSEJJSGRZa3hJcmJndGxISm12WXp0L0tkWWRoUTlpL3BPUS9DQ2JjYy8xTkR1cHVrSmZUeUk5MklYCkV1ekU5Mmp2VGxTaXRsSWFCR0w2SFZ6Z0ZVbmxDS3p1bzllK2JLTUFDVmYxeHQrZXpZb3ljTStWWG41b3c1MU8KYkdZSVVwWjdXUDY4UWhwZ1RURjVqU0luMW5JdE9oNksKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```


Apply the CSR

```bash
controlplane ~ ➜  kubectl apply -f john-developer.yaml 
certificatesigningrequest.certificates.k8s.io/john-developer created

controlplane ~ ➜  
```


View the CSR, status is pending

```bash
controlplane ~ ➜  kubectl get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
csr-6qzc2   35m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:kv5mjk    <none>              Approved,Issued
csr-zn44f   36m   kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued
john-developer        34s   kubernetes.io/kube-apiserver-client           kubernetes-admin           24h                 Pending

controlplane ~ ➜  
```

Approve certificate signing request 

```bash
controlplane ~ ➜  kubectl certificate approve john-developer
certificatesigningrequest.certificates.k8s.io/john-developer approved

controlplane ~ ➜  
```

View csr status. Approved and Issued.

```bash
controlplane ~ ➜  kubectl get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
csr-6qzc2   37m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:kv5mjk    <none>              Approved,Issued
csr-zn44f   37m   kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued
john-developer        2m    kubernetes.io/kube-apiserver-client           kubernetes-admin           24h                 Approved,Issued

controlplane ~ ➜  
```


Export the issued certificate from the CertificateSigningRequest

```bash
controlplane ~ ➜  kubectl get csr john -o jsonpath='{.status.certificate}'| base64 -d > john-developer.crt
```


Create Role 

```bash
controlplane ~ ➜  kubectl create role developer --verb=create,get,list,update,delete --resource=pods --namespace=development
role.rbac.authorization.k8s.io/developer created  
```

View role
```bash
controlplane ~ ➜  kubectl get roles -n development
NAME        CREATED AT
developer   2023-03-12T05:49:15Z
```

Describe role

```bash

controlplane ~ ➜  kubectl describe role developer -n development
Name:         developer
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
controlplane ~ ➜  kubectl create rolebinding developer-binding --role=developer --user=john --namespace=development
role.rbac.authorization.k8s.io/developer created  
```

Get the created rolebinding

```bash
controlplane ~ ➜  controlplane ~ ➜  kubectl get rolebinding -n development
NAME                ROLE             AGE
developer-binding   Role/developer   8m31s

controlplane ~ ➜  
```

Describe the rolebinding


```bash

controlplane ~ ➜  kubectl describe rolebinding developer-binding -n development
Name:         developer-binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  developer
Subjects:
  Kind  Name  Namespace
  ----  ----  ---------
  User  john  

controlplane ~ ➜   
```

Confirm if user *john* has permissions to create pods in *development* namespace


```bash
controlplane ~ ➜  kubectl auth can-i create pods --as john --namespace development
yes

controlplane ~ ➜  
```

Create a test pod
```bash
controlplane ~ ➜  kubectl run nginx --image=nginx -n development
pod/nginx created

controlplane ~ ➜  
```

View created pod

```bash

controlplane ~ ➜  kubectl get pod -n development
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          30s

controlplane ~ ➜  

```



#### Task 7

Create a static pod on node01 called nginx-critical with image nginx and make sure that it is recreated/restarted automatically in case of a failure.

Use /etc/kubernetes/manifests as the Static Pod path for example.

static pod configured under /etc/kubernetes/manifests ?

Pod nginx-critical-node01 is up and running


Create the static pod inside the */etc/kubernetes/manifests* directory


```bash


controlplane ~  ➜ kubectl run nginx-critical --image=nginx -o yaml --dry-run=client > /etc/kubernetes/manifests/nginx-critical.yaml
```

Update the pod spec to include nodeName

```bash

controlplane ~ ➜  vi /etc/kubernetes/manifests/nginx-critical.yaml

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
    ```

Apply the pod manifest to create the pod object

```bash

controlplane ~ ➜  kubectl apply -f /etc/kubernetes/manifests/nginx-critical.yaml
pod/nginx-critical created
```

Confirm the pod is created, and running on node01

```bash
controlplane ~ ➜  kubectl get pod -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP             NODE           NOMINATED NODE   READINESS GATES
nginx-critical                  1/1     Running   0          30s     10.244.192.6   node01         <none>           <none>
nginx-critical-controlplane     1/1     Running   0          79s     10.244.0.4     controlplane   <none>           <none>
nginx-deploy-7dc9dd795f-7cf7c   1/1     Running   0          25m     10.244.192.4   node01         <none>           <none>
nginx-resolver                  1/1     Running   0          5m18s   10.244.192.5   node01         <none>           <none>
redis-storage                   1/1     Running   0          38m     10.244.192.1   node01         <none>           <none>
super-user-pod                  1/1     Running   0          32m     10.244.192.2   node01         <none>           <none>
use-pv                          1/1     Running   0          26m     10.244.192.3   node01         <none>           <none>

controlplane ~ ➜  
```

#### Task 8

Create a nginx pod called nginx-resolver using image nginx, expose it internally with a service called nginx-resolver-service. Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup. Record results in /root/CKA/nginx.svc and /root/CKA/nginx.pod

Pod: nginx-resolver created

Service DNS Resolution recorded correctly

Pod DNS resolution recorded correctly


Create pod

```bash
controlplane ~ ➜  kubectl run nginx-resolver --image=nginx
pod/nginx-resolver created
```

Expose pod as type *clusterIP*

```bash

controlplane ~ ➜ kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80
service/nginx-resolver-service exposed

controlplane ~ ➜  
```




