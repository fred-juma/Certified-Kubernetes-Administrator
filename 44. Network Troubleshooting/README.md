#### Network Plugin in Kubernetes

Kubernetes uses CNI plugins to setup network. The kubelet is responsible for executing plugins 

Parameters:

*- cni-bin-dir:*   Kubelet probes this directory for plugins on startup

*- network-plugin:* The network plugin to use from cni-bin-dir. It must match the name reported by a plugin probed from the plugin directory.

Some of the plugins available are listed below together with their installation process:

1. Weave Net:

*kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml*

2. Flannel :

*kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml*

3. Calico :

curl https://docs.projectcalico.org/manifests/calico.yaml -O

kubectl apply -f calico.yaml

Note: If there are multiple CNI configuration files in the directory, the kubelet uses the configuration file that comes first by name in lexicographic order.



#### DNS in Kubernetes
- Kubernetes uses CoreDNS. 
- CoreDNS is a flexible, extensible DNS server that can serve as the Kubernetes cluster DNS.

**Memory and Pods**

- In large scale Kubernetes clusters, CoreDNS's memory usage is predominantly affected by the number of Pods and Services in the cluster. 
- Other factors include the size of the filled DNS answer cache, and the rate of queries received (QPS) per CoreDNS instance.
- Kubernetes resources for coreDNS are:   

1. a service account named *coredns*

2. cluster-roles named *coredns* and *kube-dns*

3. clusterrolebindings named *coredns* and *kube-dns* 

4. a deployment named *coredns*

5. a configmap named *coredns* 

6. service named *kube-dns*

- While analyzing the coreDNS deployment you can see that the the Corefile plugin consists of important configuration which is defined as a configmap.

```bash
kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }

    ```

- This is the backend to k8s for cluster.local and reverse domains.

```bash
proxy . /etc/resolv.conf
```

- Forward out of cluster domains directly to right authoritative DNS server.


##### Troubleshooting issues related to coreDNS

1. If you find CoreDNS pods in pending state first check network plugin is installed.

2.  coredns pods have CrashLoopBackOff or Error state: 

If you have nodes that are running SELinux with an older version of Docker you might experience a scenario where the coredns pods are not starting. To solve that you can try one of the following options:
- Upgrade to a newer version of Docker.
- Disable SELinux.
- Modify the coredns deployment to set allowPrivilegeEscalation to true:


```bash
kubectl -n kube-system get deployment coredns -o yaml | \
  sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
  kubectl apply -f -
  ```

- Another cause for CoreDNS to have CrashLoopBackOff is when a CoreDNS Pod deployed in Kubernetes detects a loop. Work around:

- Add the following to your kubelet config yaml: resolvConf: <path-to-your-real-resolv-conf-file> This flag tells kubelet to pass an alternate resolv.conf to Pods. For systems using systemd-resolved, /run/systemd/resolve/resolv.conf is typically the location of the "real" resolv.conf, although this can be different depending on your distribution.

- Disable the local DNS cache on host nodes, and restore /etc/resolv.conf to the original.

- A quick fix is to edit your Corefile, replacing forward . /etc/resolv.conf with the IP address of your upstream DNS, for example forward . 8.8.8.8. But this only fixes the issue for CoreDNS, kubelet will continue to forward the invalid resolv.conf to all default dnsPolicy Pods, leaving them unable to resolve DNS.


3. If *CoreDNS* pods and the *kube-dns* service is working fine, check the *kube-dns* service has valid endpoints.

              *kubectl -n kube-system get ep kube-dns*

If there are no endpoints for the service, inspect the service and make sure it uses the correct selectors and ports.

#### Kube-Proxy

- *kube-proxy* is a network proxy that runs on each node in the cluster. *kube-proxy* maintains network rules on nodes. These network rules allow network communication to the Pods from network sessions inside or outside of the cluster.

- In a cluster configured with kubeadm, you can find kube-proxy as a daemonset.

- *kubeproxy* is responsible for watching services and endpoint associated with each service. When the client is going to connect to the service using the virtual IP the *kubeproxy* is responsible for sending traffic to actual pods.

- If you run a *kubectl describe ds kube-proxy -n kube-system* you can see that the kube-proxy binary runs with following command inside the kube-proxy container.

```bash
Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/config.conf
      --hostname-override=$(NODE_NAME)
      ```
-   So it fetches the configuration from a configuration file ie, */var/lib/kube-proxy/config.conf* and we can override the hostname with the node name of at which the pod is running.

-  In the config file we define the *clusterCIDR, kubeproxy mode, ipvs, iptables, bindaddress, kube-config* etc.

##### Troubleshooting issues related to kube-proxy

1. Check *kube-proxy* pod in the kube-system namespace is running.
2. Check *kube-proxy* logs.

3. Check *configmap* is correctly defined and the config file for running *kube-proxy* binary is correct.

4. *kube-config* is defined in the *config map*.

5. check *kube-proxy* is running inside the container

```bash
# netstat -plan | grep kube-proxy
tcp        0      0 0.0.0.0:30081           0.0.0.0:*               LISTEN      1/kube-proxy
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      1/kube-proxy
tcp        0      0 172.17.0.12:33706       172.17.0.12:6443        ESTABLISHED 1/kube-proxy
tcp6       0      0 :::10256                :::*                    LISTEN      1/kube-proxy
```

**Troubleshooting Test 1:**

A simple 2 tier application is deployed in the *triton* namespace. It must display a green web page on success. Click on the app tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.


Triton Architecture              |  
:-------------------------:|
![TritonArchitecture](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/troubleshoot_network01.JPG)


View all resources in the *triton* namespace

```bash
root@controlplane ~ ➜  kubectl get all -n triton
NAME                                READY   STATUS              RESTARTS   AGE
pod/mysql                           0/1     ContainerCreating   0          5m5s
pod/webapp-mysql-74fc8f4448-gcxqx   0/1     ContainerCreating   0          5m4s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/mysql         ClusterIP   10.111.192.14   <none>        3306/TCP         5m5s
service/web-service   NodePort    10.107.65.207   <none>        8080:30081/TCP   5m4s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   0/1     1            0           5m4s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-74fc8f4448   1         1         0       5m4s

root@controlplane ~ ➜  
```




Set *triton* namespace as the current context

```bash
root@controlplane ~ ➜  kubectl config set-context --current --namespace=triton
Context "kubernetes-admin@kubernetes" modified.

root@controlplane ~ ➜  
```

Describe pod to view the pod events


```bash
root@controlplane ~ ➜  kubectl describe pod webapp-mysql-74fc8f4448-gcxqx

Events:
  Type     Reason                  Age                   From               Message
  ----     ------                  ----                  ----               -------
  Normal   Scheduled               10m                   default-scheduler  Successfully assigned triton/webapp-mysql-74fc8f4448-gcxqx to controlplane
  Warning  FailedCreatePodSandBox  10m                   kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox "d05b551a528fed17b11dd40bdfc0b428810ebd7ea007718991b022479e46d8de": plugin type="weave-net" name="weave" failed (add): unable to allocate IP address: Post "http://127.0.0.1:6784/ip/d05b551a528fed17b11dd40bdfc0b428810ebd7ea007718991b022479e46d8de": dial tcp 127.0.0.1:6784: connect: connection refused
```

Inspect the kubelet service and identify the container runtime endpoint value is set for Kubernetes. *unix:///var/run/containerd/containerd.sock*

```bash
root@controlplane ~ ➜  ps -aux | grep kubelet | grep --color container-runtime-endpoint
root      2592  2.7  5.1 1581800 105152 ?      Ssl  06:29   1:06 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///run/containerd/containerd.sock --hostname-override=controlplane --pod-infra-container-image=registry.k8s.io/pause:3.9

root@controlplane ~ ➜  
```

Check the path configured with all binaries of CNI supported plugins

```bash
root@controlplane ~ ➜  ls -l /opt/cni/bin
total 81668
-rwxr-xr-x 1 root root  4159518 May 13  2020 bandwidth
-rwxr-xr-x 1 root root  4671647 May 13  2020 bridge
-rwxr-xr-x 1 root root 12124326 May 13  2020 dhcp
-rwxr-xr-x 1 root root  5945760 May 13  2020 firewall
-rwxr-xr-x 1 root root  3069556 May 13  2020 flannel
-rwxr-xr-x 1 root root  4174394 May 13  2020 host-device
-rwxr-xr-x 1 root root  3614480 May 13  2020 host-local
-rwxr-xr-x 1 root root  4314598 May 13  2020 ipvlan
-rwxr-xr-x 1 root root  3209463 May 13  2020 loopback
-rwxr-xr-x 1 root root  4389622 May 13  2020 macvlan
-rwxr-xr-x 1 root root  3939867 May 13  2020 portmap
-rwxr-xr-x 1 root root  4590277 May 13  2020 ptp
-rwxr-xr-x 1 root root  3392826 May 13  2020 sbr
-rwxr-xr-x 1 root root  2885430 May 13  2020 static
-rwxr-xr-x 1 root root  3356587 May 13  2020 tuning
-rwxr-xr-x 1 root root  4314446 May 13  2020 vlan
lrwxrwxrwx 1 root root       18 Mar  1 06:29 weave-ipam -> weave-plugin-2.8.1
lrwxrwxrwx 1 root root       18 Mar  1 06:29 weave-net -> weave-plugin-2.8.1
-rwxr-xr-x 1 root root 11437320 Mar  1 06:29 weave-plugin-2.8.1

root@controlplane ~ ➜  
```


Check the CNI plugin configured to be used on this kubernetes cluster?

```bash
root@controlplane ~ ➜  ls /etc/cni/net.d/
10-weave.conflist

root@controlplane ~ ➜  
```

Check for weave pods. So far none is running

```bash
root@controlplane ~ ➜  kubectl get pods -A | grep weave

root@controlplane ~ ✖ 
```

Install weave-net pod networking solution

```bash
root@controlplane ~ ➜  kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created

root@controlplane ~ ➜  
```

Confirm the weave pod is created and running

```bash
root@controlplane ~ ➜  kubectl get pods -A | grep weave
kube-system   weave-net-66cxw                        2/2     Running             0          22s

root@controlplane ~ ➜  
```

Check that the cluster objects are now running

```bash
root@controlplane ~ ➜  kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/mysql                           1/1     Running   0          24m
pod/webapp-mysql-74fc8f4448-gcxqx   1/1     Running   0          24m

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/mysql         ClusterIP   10.111.192.14   <none>        3306/TCP         24m
service/web-service   NodePort    10.107.65.207   <none>        8080:30081/TCP   24m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   1/1     1            1           24m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-74fc8f4448   1         1         1       24m

root@controlplane ~ ➜  
```

**Troubleshooting Test 2:**

The same 2 tier application is having issues again. It must display a green web page on success. Click on the app tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.


Triton Architecture              |  
:-------------------------:|
![TritonArchitecture](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/troubleshoot_network02.JPG)


View objects in the current namespace

```bash
root@controlplane ~ ➜  kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/mysql                           1/1     Running   0          2m58s
pod/webapp-mysql-74fc8f4448-l7rg6   1/1     Running   0          2m57s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/mysql         ClusterIP   10.111.192.14   <none>        3306/TCP         28m
service/web-service   NodePort    10.107.65.207   <none>        8080:30081/TCP   28m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   1/1     1            1           2m57s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-74fc8f4448   1         1         1       2m57s
```


View all controller objects in the kube-system namespace

```bash
root@controlplane ~ ➜  kubectl get all -n kube-system
NAME                                       READY   STATUS             RESTARTS       AGE
pod/coredns-787d4945fb-jbvcb               1/1     Running            0              51m
pod/coredns-787d4945fb-ll5h6               1/1     Running            0              51m
pod/etcd-controlplane                      1/1     Running            0              52m
pod/kube-apiserver-controlplane            1/1     Running            0              52m
pod/kube-controller-manager-controlplane   1/1     Running            0              52m
pod/kube-proxy-spdq5                       0/1     CrashLoopBackOff   5 (103s ago)   4m45s
pod/kube-scheduler-controlplane            1/1     Running            0              52m
pod/weave-net-66cxw                        2/2     Running            0              6m41s

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   52m

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         0       1            0           kubernetes.io/os=linux   4m45s
daemonset.apps/weave-net    1         1         1       1            1           <none>                   6m41s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   2/2     2            2           52m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-787d4945fb   2         2         2       51m

root@controlplane ~ ➜  
```

The *kube-proxy* pod status is *CrashLoopBackOff*
The daemonSet *kube-proxy* has 0 ready and 0 available

View the logs of the kube-proxy pod

```bash
root@controlplane ~ ✖ kubectl logs kube-proxy-spdq5 -n kube-system
E0301 07:22:40.167988       1 run.go:74] "command failed" err="failed complete: open /var/lib/kube-proxy/configuration.conf: no such file or directory"

root@controlplane ~ ➜  
```

From the error above, The configuration file */var/lib/kube-proxy/configuration.conf* is not valid. 


Lets check the configMap. 

First list the config maps in the kube-system namespace

```bash
root@controlplane ~ ➜  kubectl -n kube-system get configMap
NAME                                 DATA   AGE
coredns                              1      57m
extension-apiserver-authentication   6      57m
kube-proxy                           2      57m
kube-root-ca.crt                     1      57m
kubeadm-config                       1      57m
kubelet-config                       1      57m
weave-net                            0      56m

root@controlplane ~ ➜  
```

Describe the kube-proxy configMap

```bash

root@controlplane ~ ➜  kubectl describe configmap kube-proxy -n kube-system


clientConnection:
  acceptContentTypes: ""
  burst: 0
  contentType: ""
  kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
```

Describe the kube-proxy daemonset

```bash
root@controlplane ~ ➜  kubectl describe daemonset kube-proxy -n kube-system

Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/configuration.conf
      --hostname-override=$(NODE_NAME)
      ```

In the DaemonSet, for kube-proxy, the command used to start the kube-proxy pod makes use of the path */var/lib/kube-proxy/configuration.conf*.

Whereas, the correct this path to */var/lib/kube-proxy/config.conf* as per the ConfigMap and recreate the *kube-proxy* pod.

```bash


root@controlplane ~ ➜  kubectl edit pod kube-proxy-k58g8 -n kube-system
error: pods "kube-proxy-k58g8" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-1439598053.yaml"
error: Edit cancelled, no valid changes were saved.

root@controlplane ~ ✖ 
containers:
  - command:
    - /usr/local/bin/kube-proxy
    - --config=/var/lib/kube-proxy/config.conf
    - --hostname-override=$(NODE_NAME)
```

Delete the kube=proxy pod

```bash
root@controlplane ~ ✖ kubectl delete pod kube-proxy-k58g8 -n kube-system
pod "kube-proxy-k58g8" deleted
```

Recreate the pod

```bash
root@controlplane ~ ➜  kubectl apply -f /tmp/kubectl-edit-1439598053.yaml
pod/kube-proxy-k58g8 created

root@controlplane ~ ➜  
```


Ediot the daemonset 

```bash

root@controlplane ~ ➜  kubectl edit daemonset kube-proxy -n kube-system

daemonset.apps/kube-proxy edited

root@controlplane ~ ➜  

spec:
      containers:
      - command:
        - /usr/local/bin/kube-proxy
        - --config=/var/lib/kube-proxy/config.conf
        - --hostname-override=$(NODE_NAME)
```

Confirm that all services are running

```bash
Every 2.0s: kubectl get all -n kube-system                                                  controlplane: Wed Mar  1 11:36:21 2023

NAME                                       READY   STATUS    RESTARTS   AGE
pod/coredns-787d4945fb-bpq6c               1/1     Running   0          52m
pod/coredns-787d4945fb-nznrh               1/1     Running   0          52m
pod/etcd-controlplane                      1/1     Running   0          52m
pod/kube-apiserver-controlplane            1/1     Running   0          52m
pod/kube-controller-manager-controlplane   1/1     Running   0          52m
pod/kube-proxy-6bzfm                       1/1     Running   0          45s
pod/kube-scheduler-controlplane            1/1     Running   0          52m
pod/weave-net-lhnfn                        2/2     Running   0          26m

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   52m

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   26m
daemonset.apps/weave-net    1         1         1       1            1           <none>                   26m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   2/2     2            2           52m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-787d4945fb   2         2         2       52m

```

Note: Kube proxy runs on each node and talks to api-server to get the details of the services and endpoints present. Based on this information, kube-proxy creates entries in iptables, which then routes the packets to the correct destination.


***The End***