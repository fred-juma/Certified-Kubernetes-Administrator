#### Check Node status

kubectl get nodes

#### Check pod status
kubectl get pods

#### Incase of cluster deployed with kubeadm toool, hence controlplane deployed as pods, check that pods in kube-system namespace are running

 kubectl get pods -n kube-system

#### If control planes components are deployed as services, check the status of the services kube-apiserver, kube-controller-manager and kube-scheduler on the master nodes as well as kubelet and kube-proxy on the worker nodes

service kube-apiserver status

service kube-controller-manager status

service kube-scheduler status

service kubelet status

service kube-proxy status

#### Check the logs of the control components

In case of kubeadm:

kubectl logs kube-apiserver-master -n kube-system

In case of services configured natively on master nodes:

sudo journalctl -u kube-apiserver


The cluster is broken. We tried deploying an application but it's not working. Troubleshoot and fix the issue.

Start looking at the deployments.

Check nodes status

```bash

controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   10m   v1.26.0

controlplane ~ ➜ 
```

Check status of pods, the pod is in *Pending* status

```bash
controlplane ~ ➜  kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
app-865fdf7bbf-s9rss   0/1     Pending   0          3m33s

controlplane ~ ➜  
```

Check status of pods in kube-system namespace

```bash
controlplane ~ ➜  kubectl get pods -n kube-system
NAME                                   READY   STATUS             RESTARTS        AGE
coredns-787d4945fb-cbz75               1/1     Running            0               12m
coredns-787d4945fb-qh2zf               1/1     Running            0               12m
etcd-controlplane                      1/1     Running            0               12m
kube-apiserver-controlplane            1/1     Running            0               12m
kube-controller-manager-controlplane   1/1     Running            0               12m
kube-proxy-kp9hx                       1/1     Running            0               12m
kube-scheduler-controlplane            0/1     CrashLoopBackOff   5 (2m10s ago)   5m22s

controlplane ~ ➜ 
```

The kube-scheduler is not running, status is *CrashLoopBackOff*. Describe the pod to view events

```bash

controlplane ~ ➜  kubectl describe pod kube-scheduler-controlplane -n kube-system

Events:
  Type     Reason   Age                     From     Message
  ----     ------   ----                    ----     -------
  Normal   Pulled   7m4s (x4 over 7m59s)    kubelet  Container image "registry.k8s.io/kube-scheduler:v1.26.0" already present on machine
  Normal   Created  7m4s (x4 over 7m59s)    kubelet  Created container kube-scheduler
  Warning  Failed   7m3s (x4 over 7m59s)    kubelet  Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "kube-schedulerrrr": executable file not found in $PATH: unknown
  Warning  BackOff  2m51s (x32 over 7m58s)  kubelet  Back-off restarting failed container kube-scheduler in pod kube-scheduler-controlplane_kube-system(ba87dfc82718067f5aa1792d603e49fb)
```

From the events output, we see the error message *unable to start container process: exec: "kube-schedulerrrr"*

There is typo in the service name. Edit the kube-scheduler pod definition and correct the service name to "kube-scheduler"

```bash

controlplane ~ ➜  vi /etc/kubernetes/manifests/kube-scheduler.yaml

or 

sed -i 's/kube-schedulerrrr/kube-scheduler/g' /etc/kubernetes/manifests/kube-scheduler.yaml
```

Then check status of the pods, and confirm the kube-scheduler pod is now running

```bash
controlplane ~ ➜  kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-787d4945fb-jq78f               1/1     Running   0          6m4s
coredns-787d4945fb-rdbdq               1/1     Running   0          6m5s
etcd-controlplane                      1/1     Running   0          6m20s
kube-apiserver-controlplane            1/1     Running   0          6m15s
kube-controller-manager-controlplane   1/1     Running   0          6m15s
kube-proxy-v8ddw                       1/1     Running   0          6m5s
kube-scheduler-controlplane            1/1     Running   0          37s

controlplane ~ ➜  
```


Scale the deployment app to 2 pods.

View the deployment name

```bash
controlplane ~ ➜  kubectl get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
app    1/1     1            1           6m11s

controlplane ~ ➜  
```

Scale the *app* deployment to 2 pods

```bash
controlplane ~ ➜  kubectl scale deployment/app --replicas=2
deployment.apps/app scaled

controlplane ~ ➜  
```

Even though the deployment was scaled to 2, the number of PODs does not seem to increase. Investigate and fix the issue.

Inspect the component responsible for managing deployments and replicasets.


View the pods and deployments 

```bash
controlplane ~ ➜  kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
app-865fdf7bbf-glp42   1/1     Running   0          8m6s

controlplane ~ ➜  kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/app-865fdf7bbf-glp42   1/1     Running   0          8m16s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   12m

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app   1/2     1            1           8m16s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/app-865fdf7bbf   1         1         1       8m16s

controlplane ~ ➜  
```

Scaling replicas is the job of controller manager. So let's describe the status of the controller manager


```bash
controlplane ~ ➜  kubectl get all -n kube-system
NAME                                       READY   STATUS             RESTARTS        AGE
pod/coredns-787d4945fb-jq78f               1/1     Running            0               18m
pod/coredns-787d4945fb-rdbdq               1/1     Running            0               18m
pod/etcd-controlplane                      1/1     Running            0               18m
pod/kube-apiserver-controlplane            1/1     Running            0               18m
pod/kube-controller-manager-controlplane   0/1     CrashLoopBackOff   6 (4m55s ago)   10m
pod/kube-proxy-v8ddw                       1/1     Running            0               18m
pod/kube-scheduler-controlplane            1/1     Running            0               12m
```

The controller manager pod is in status *CrashLoopBackOff*. Lets check its logs

```bash
controlplane ~ ➜  kubectl logs kube-controller-manager-controlplane -n kube-system
I0227 12:24:40.515812       1 serving.go:348] Generated self-signed cert in-memory
E0227 12:24:40.516000       1 run.go:74] "command failed" err="stat /etc/kubernetes/controller-manager-XXXX.conf: no such file or directory"

controlplane ~ ➜  
```

There is the error message *err="stat /etc/kubernetes/controller-manager-XXXX.conf: no such file or directory"*

Check the kube-controller-manager manifest definition


```bash

controlplane ~ ➜  vi /etc/kubernetes/manifests/kube-controller-manager.yaml 



spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=10.244.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager-XXXX.conf
    - --leader-elect=true
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --use-service-account-credentials=true
```

We see that the command for kubeconfig is defined as *- --kubeconfig=/etc/kubernetes/controller-manager-XXXX.conf*

There is error in naming the kubeconfig, The correct config file is in /etc/kubernetes

```bash
controlplane ~ ➜  ls -l /etc/kubernetes/
total 36
-rw------- 1 root root 5640 Feb 27 07:05 admin.conf
-rw------- 1 root root 5672 Feb 27 07:05 controller-manager.conf
-rw------- 1 root root 1984 Feb 27 07:05 kubelet.conf
drwxr-xr-x 1 root root 4096 Feb 27 07:31 manifests
drwxr-xr-x 3 root root 4096 Feb 27 07:05 pki
-rw------- 1 root root 5616 Feb 27 07:05 scheduler.conf

controlplane ~ ➜  
```

Therefore we rename the file as *- --kubeconfig=/etc/kubernetes/controller-manager.conf*


After the correction, check the status of the kube-controller-manager pod. It is now in running state

```bash
controlplane ~ ➜  kubectl get all -n kube-system
NAME                              READY   STATUS    RESTARTS   AGE
pod/coredns-787d4945fb-jq78f      1/1     Running   0          29m
pod/coredns-787d4945fb-rdbdq      1/1     Running   0          29m
pod/etcd-controlplane             1/1     Running   0          29m
pod/kube-apiserver-controlplane   1/1     Running   0          29m
pod/kube-proxy-v8ddw              1/1     Running   0          29m
pod/kube-scheduler-controlplane   1/1     Running   0          23m
```

Then lets check of the replicaset is scaling the pods. Pod is caled up to 2.

```bash

```controlplane ~ ➜  kubectl get all 
NAME                       READY   STATUS    RESTARTS   AGE
pod/app-865fdf7bbf-6vnsj   1/1     Running   0          35s
pod/app-865fdf7bbf-glp42   1/1     Running   0          26m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   30m

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app   2/2     2            2           26m

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/app-865fdf7bbf   2         2         2       26m

controlplane ~ ➜  
```



Something is wrong with scaling again. We just tried scaling the deployment to 3 replicas. But it's not happening.

Investigate and fix the issue.


Let us view the deployment and pod status. The deployment ready is 2/3

```bash
controlplane ~ ➜  kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/app-865fdf7bbf-6vnsj   1/1     Running   0          2m36s
pod/app-865fdf7bbf-glp42   1/1     Running   0          28m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   32m

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app   2/3     2            2           28m

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/app-865fdf7bbf   2         2         2       28m

controlplane ~ ➜  
```

Scaling replicas is the job of controller manager. So let's describe the status of the controller manager


```bash
controlplane ~ ➜  kubectl get all -n kube-system
NAME                                       READY   STATUS             RESTARTS      AGE
pod/coredns-787d4945fb-jq78f               1/1     Running            0             34m
pod/coredns-787d4945fb-rdbdq               1/1     Running            0             34m
pod/etcd-controlplane                      1/1     Running            0             34m
pod/kube-apiserver-controlplane            1/1     Running            0             34m
pod/kube-controller-manager-controlplane   0/1     CrashLoopBackOff   4 (54s ago)   2m42s
pod/kube-proxy-v8ddw                       1/1     Running            0             34m
pod/kube-scheduler-controlplane            1/1     Running            0            
```

The status of the kube-controller-manager is *CrashLoopBackOff*. View the logs of its pod

```bash
controlplane ~ ✖ kubectl logs kube-controller-manager-controlplane -n kube-system
I0227 12:40:42.851886       1 serving.go:348] Generated self-signed cert in-memory
E0227 12:40:43.086362       1 run.go:74] "command failed" err="unable to load client CA provider: open /etc/kubernetes/pki/ca.crt: no such file or directory"

controlplane ~ ➜  
```

There is error message *err="unable to load client CA provider: open /etc/kubernetes/pki/ca.crt: no such file or directory"*

The way the control plane certificates are setup is in such a way that the certificate files are saved on the host nodes, and we use volumes to mount the directories within the controller manager. This is specified in the manifest file. So lets check that and look for *pki* mount

```bash
- mountPath: /etc/kubernetes/pki
      name: k8s-certs
```

The volumeMount is named *k8s-certs*. Lets check the volume host path where *k8s-certs* is pointing. From the output below, the path is pointing to */etc/kubernetes/WRONG-PKI-DIRECTORY* instead of */etc/kubernetes/pki*.

```bash
- hostPath:
      path: /etc/kubernetes/WRONG-PKI-DIRECTORY
      type: DirectoryOrCreate
    name: k8s-certs
    ```

Rectify the mount path on the kube-controller-manager manifest

```bash
controlplane ~ ➜  vi /etc/kubernetes/manifests/kube-controller-manager.yaml 


- hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name
```

Check and confirm that the deployment is now scaling to 3 replicas.

```bash
controlplane ~ ➜  kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/app-865fdf7bbf-6vnsj   1/1     Running   0          21m
pod/app-865fdf7bbf-glp42   1/1     Running   0          47m
pod/app-865fdf7bbf-l62k4   1/1     Running   0          6s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   51m

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app   3/3     3            3           47m

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/app-865fdf7bbf   3         3         3       47m

controlplane ~ ➜  
```

***The End***
