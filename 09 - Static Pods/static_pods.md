### Static Pods

These nodes are managed independently by kubelet on a node, with the presence of container runtime.

They are not dependent on kubernetes control plane therefore it can be used to deploy the control plane components such as kubeapi-server, kube-scheduler etc.
Kube-scheduler has no effect on these pods

#### Use Cases
+ Deploy kubernetes control plane components as pods on nodes

#### How can one do that?

+ Install kubelet on all the master nodes
+ create pod definition files that uses docker images of the various control plane components e.g. controller-manager.yaml, kubeapi-server.yaml, etcd.yaml etc.
+ Place the definition files in the designated manifests directory in the node
+ Configure the kubelet to read the pod definition files from this manifest directory 
+ kubelet takes control of deploying these resources as pods on the nodes by checking this directory periodically and creates pods defined
+ If any of these static services crush, kubelet will attempt to restart them

#### How to configure the designated folder
+ Can be any directory on the host / node
+ That directory path is passed to the kubelet as an option while running the service. 
+ The option is named --pod-manifest-path and then the directory is pointed e.g. 

```{r echo=FALSE, eval=FALSE}
kubelet.service
ExecStart=/usr/local/bin/kubelet \\
.
.
.
  --pod-manifest-path=/etc/kubernetes/manifests
```

+ Instead of providing the path directly on the kubelet service file, another way to configure the path would be by providing a path to another config file using the config option and then defining the directory path as static pod path to in that file

e.g.

```{r echo=FALSE, eval=FALSE}
kubelet.service
ExecStart=/usr/local/bin/kubelet \\
.
.
.
  --config=kubeconfig.yaml
```

Then add the the path to the manifest file kubeconfig.yaml

```{r echo=FALSE, eval=FALSE}
staticPodPath: /etc/kubernetes/manifests
```

#### View static pods by the *docker ps* command



 kubeapi-server server is aware of static pods created by kubelet and lists a read only mirror of their details in kubectl get pods results. However you cannot edit or delete them as usual pods. They can only be edited or deleted by modifying the definition files in the manifest folder.

The static pods have node name appended to ther name

#### Viewing components created as static pods

```bash
controlplane ~ ➜  kubectl get all --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-6d4b75cb6d-spgsc               1/1     Running   0          6m23s
kube-system   pod/coredns-6d4b75cb6d-x2ns2               1/1     Running   0          6m23s
kube-system   pod/etcd-controlplane                      1/1     Running   0          6m35s
kube-system   pod/kube-apiserver-controlplane            1/1     Running   0          6m37s
kube-system   pod/kube-controller-manager-controlplane   1/1     Running   0          6m35s
kube-system   pod/kube-flannel-ds-q8vgw                  1/1     Running   0          6m12s
kube-system   pod/kube-flannel-ds-t8vqv                  1/1     Running   0          6m24s
kube-system   pod/kube-proxy-chjvh                       1/1     Running   0          6m12s
kube-system   pod/kube-proxy-ljlml                       1/1     Running   0          6m24s
kube-system   pod/kube-scheduler-controlplane            1/1     Running   0          6m35s

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  6m38s
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   6m36s

NAMESPACE     NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-flannel-ds   2         2         2       2            2           <none>                   6m33s
kube-system   daemonset.apps/kube-proxy        2         2         2       2            2           kubernetes.io/os=linux   6m36s

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           6m36s

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-6d4b75cb6d   2         2         2       6m24s

controlplane ~ ➜  
```

+ etcd, kube-scheduler and kube-apiserver are all deployed as static pods, you can see the node name controlplane appended to them

+ kube-proxy and kube-flannel-ds (elasticserach service) are both deployed as DaemonSet

+ coredns is deployed as a deployment

+ kube-dns is deployed as a service

#### Viewing nodes
```
controlplane ~ ➜  kubectl get nodes -o wide --all-namespaces
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
controlplane   Ready    control-plane   11m   v1.24.0   10.12.14.3    <none>        Ubuntu 18.04.6 LTS   5.4.0-1092-gcp   containerd://1.6.6
node01         Ready    <none>          10m   v1.24.0   10.12.14.6    <none>        Ubuntu 18.04.6 LTS   5.4.0-1092-gcp   containerd://1.6.6
```


#### Inspecting the path of the directory holding the static pod definition files?

```bash
controlplane ~ ➜  ps -aux | grep kubelet
root        1822  0.0  0.1 1249400 383320 ?      Ssl  06:59   0:40 kube-apiserver --advertise-address=10.12.14.3 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root        2324  0.0  0.0 4301412 100864 ?      Ssl  06:59   0:17 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=remote --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=k8s.gcr.io/pause:3.7
root        7835  0.0  0.0  13444  1124 pts/0    S+   07:14   0:00 grep kubelet

controlplane ~ ➜  
```

Notice the option *--config=/var/lib/kubelet/config.yaml*

inspect the file */var/lib/kubelet/config.yaml* for *staticPodPath*

```bash
controlplane ~ ➜  cat /var/lib/kubelet/config.yaml | grep -i staticpodpath
staticPodPath: /etc/kubernetes/manifests

controlplane ~ ➜  
```

#### This reveals the path to be /etc/kubernetes/manifests

```
controlplane ~ ➜  ls -l /etc/kubernetes/manifests
total 16
-rw------- 1 root root 2257 Nov 27 06:59 etcd.yaml
-rw------- 1 root root 3844 Nov 27 06:59 kube-apiserver.yaml
-rw------- 1 root root 3365 Nov 27 06:59 kube-controller-manager.yaml
-rw------- 1 root root 1435 Nov 27 06:59 kube-scheduler.yaml

controlplane ~ ➜  
```

#### Create a static pod named static-busybox that uses the busybox image and the command sleep 1000

+ Name: static-busybox
+ Image: busybox

```bash
controlplane /etc/kubernetes/manifests ➜  kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml

controlplane /etc/kubernetes/manifests ➜  
```

#### view the yaml file

```bash
controlplane /etc/kubernetes/manifests ➜  vi static-busybox.yaml 

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: static-busybox
  name: static-busybox
spec:
  containers:
  - command:
    - sleep
    - "1000"
    image: busybox
    name: static-busybox
  restartPolicy: Never
```

#### Apply the defintion yaml file to create the pod
```bash
controlplane /etc/kubernetes/manifests ➜  kubectl apply -f static-busybox.yaml pod/static-busybox created

controlplane /etc/kubernetes/manifests ➜  
```

#### Find below the yam definition files for the various static pods in the master node

+ [etcd](etcd)
+ [kube-apiserver](kube-apiserver.yaml)
+ [kube-controller-manager](kube-controller-manager.yaml)
+ [kube-scheduler](kube-scheduler.yaml)