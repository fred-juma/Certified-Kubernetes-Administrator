#### Task
Upgrade the current version of kubernetes from 1.25.0 to 1.26.0 exactly using the kubeadm utility. Make sure that the upgrade is carried out one node at a time starting with the controlplane node. To minimize downtime, the deployment gold-nginx should be rescheduled on an alternate node before upgrading each node.

Upgrade controlplane node first and drain node node01 before upgrading it. Pods for gold-nginx should run on the controlplane node subsequently.

List cluster nodes

```bash

controlplane ~ ➜  kubectl get nodes 
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   44m   v1.25.0
node01         Ready    <none>          44m   v1.25.0

controlplane ~ ➜  
```			

The version colums shows that cluster is running version 1.25.0

List pods and the associated node they are scheduled on

```bash

controlplane ~ ➜  kubectl get pod -A -o wide
NAMESPACE     NAME                                   READY   STATUS    RESTARTS      AGE    IP             NODE           NOMINATED NODE   READINESS GATES
admin2406     deploy1-78d8d47657-6dh8z               1/1     Running   0             7m2s   10.244.192.2   node01         <none>           <none>
admin2406     deploy2-d6b48767c-4t9hw                1/1     Running   0             7m2s   10.244.192.3   node01         <none>           <none>
admin2406     deploy3-5bdcc4b88c-x4t76               1/1     Running   0             7m2s   10.244.192.4   node01         <none>           <none>
admin2406     deploy4-6b96456775-hqssv               1/1     Running   0             7m1s   10.244.192.5   node01         <none>           <none>
admin2406     deploy5-5dc4d5d78b-bdgz8               1/1     Running   0             7m1s   10.244.192.6   node01         <none>           <none>
alpha         alpha-mysql-6d945ffc78-fflxh           0/1     Pending   0             7m3s   <none>         <none>         <none>           <none>
default       gold-nginx-69d8d76f55-7fjqs            1/1     Running   0             7m9s   10.244.192.1   node01         <none>           <none>
kube-system   coredns-565d847f94-b5crx               1/1     Running   0             49m    10.244.0.1     controlplane   <none>           <none>
kube-system   coredns-565d847f94-lqj84               1/1     Running   0             49m    10.244.0.3     controlplane   <none>           <none>
kube-system   etcd-controlplane                      1/1     Running   0             50m    192.8.135.3    controlplane   <none>           <none>
kube-system   kube-apiserver-controlplane            1/1     Running   0             50m    192.8.135.3    controlplane   <none>           <none>
kube-system   kube-controller-manager-controlplane   1/1     Running   0             50m    192.8.135.3    controlplane   <none>           <none>
kube-system   kube-proxy-wx7d9                       1/1     Running   0             49m    192.8.135.6    node01         <none>           <none>
kube-system   kube-proxy-zmwzb                       1/1     Running   0             49m    192.8.135.3    controlplane   <none>           <none>
kube-system   kube-scheduler-controlplane            1/1     Running   0             50m    192.8.135.3    controlplane   <none>           <none>
kube-system   weave-net-7475j                        2/2     Running   0             49m    192.8.135.6    node01         <none>           <none>
kube-system   weave-net-t5pxc                        2/2     Running   1 (49m ago)   49m    192.8.135.3    controlplane   <none>           <none>

controlplane ~ ➜ 
```

Upgrade A Cluster Steps

- Upgrade the control plane
- Upgrade the worker nodes in your cluster
- Upgrade clients such as kubectl
- Adjust manifests and other resources based on the API changes that accompany the new Kubernetes version

Ref:

- https://kubernetes.io/docs/tasks/administer-cluster/cluster-upgrade/


View kubeadm version

```bash
controlplane ~ ➜  kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"25", GitVersion:"v1.25.0", GitCommit:"a866cbe2e5bbaa01cfd5e969aa3e033f3282a8a2", GitTreeState:"clean", BuildDate:"2022-08-23T17:43:25Z", GoVersion:"go1.19", Compiler:"gc", Platform:"linux/amd64"}

controlplane ~ ➜ 

```


Run kubeadm upgrade plan to see controller node current components versions and target versions

```bash

controlplane ~ ➜  kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.25.0
[upgrade/versions] kubeadm version: v1.25.0
I0308 02:09:17.845492   16816 version.go:256] remote version is much newer: v1.26.2; falling back to: stable-1.25
[upgrade/versions] Target version: v1.25.7
[upgrade/versions] Latest version in the v1.25 series: v1.25.7

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     2 x v1.25.0   v1.25.7

Upgrade to the latest version in the v1.25 series:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.25.0   v1.25.7
kube-controller-manager   v1.25.0   v1.25.7
kube-scheduler            v1.25.0   v1.25.7
kube-proxy                v1.25.0   v1.25.7
CoreDNS                   v1.9.3    v1.9.3
etcd                      3.5.4-0   3.5.4-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.25.7

Note: Before you can perform this upgrade, you have to update kubeadm to v1.25.7.

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________


controlplane ~ ➜  

```


Find the latest patch release for Kubernetes 1.26

```bash

controlplane ~ ➜  apt-cache madison kubeadm
   kubeadm |  1.26.2-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.1-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.0-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
```

Drain controlplane node of all pods 

```bash
controlplane ~ ➜  kubectl drain controlplane --ignore-daemonsets 
node/controlplane cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-g2f8b, kube-system/weave-net-45vwm
evicting pod kube-system/coredns-565d847f94-fxg8z
evicting pod kube-system/coredns-565d847f94-24nhn
pod/coredns-565d847f94-24nhn evicted
pod/coredns-565d847f94-fxg8z evicted
node/controlplane drained

controlplane ~ ➜  
```

Install kubeadm of the specified version

```bash
controlplane ~ ➜   apt-mark unhold kubeadm && \
>  apt-get update && apt-get install -y kubeadm=1.26.0-00 && \
>  apt-mark hold kubeadm
kubeadm was already not hold.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [8,993 B]                                                
Hit:2 https://download.docker.com/linux/ubuntu focal InRelease                                                                   
Get:3 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]        
Hit:4 http://archive.ubuntu.com/ubuntu focal InRelease                
Hit:5 http://archive.ubuntu.com/ubuntu focal-updates InRelease
Get:6 http://archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Fetched 231 kB in 1s (253 kB/s)    
Reading package lists... Done
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 16 not upgraded.
Need to get 9,730 kB of archives.
After this operation, 2,974 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.26.0-00 [9,730 kB]
Fetched 9,730 kB in 0s (47.2 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 20439 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.26.0-00_amd64.deb ...
Unpacking kubeadm (1.26.0-00) over (1.25.0-00) ...
Setting up kubeadm (1.26.0-00) ...
kubeadm set on hold.

controlplane ~ ➜  
```

Confirm kubeadm is upgraded to a new version

```bash

controlplane ~ ➜  kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.0", GitCommit:"b46a3f887ca979b1a5d14fd39cb1af43e7e5d12d", GitTreeState:"clean", BuildDate:"2022-12-08T19:57:06Z", GoVersion:"go1.19.4", Compiler:"gc", Platform:"linux/amd64"}

controlplane ~ ➜  
```

Run kubeadm upgrade plan to see controller node current components versions and target versions

```bash
controlplane ~ ➜  kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.25.0
[upgrade/versions] kubeadm version: v1.26.0
[upgrade/versions] Target version: v1.26.2
[upgrade/versions] Latest version in the v1.25 series: v1.25.7

W0308 02:35:38.214812   14156 configset.go:177] error unmarshaling configuration schema.GroupVersionKind{Group:"kubeproxy.config.k8s.io", Version:"v1alpha1", Kind:"KubeProxyConfiguration"}: strict decoding error: unknown field "udpIdleTimeout"
Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     2 x v1.25.0   v1.25.7

Upgrade to the latest version in the v1.25 series:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.25.0   v1.25.7
kube-controller-manager   v1.25.0   v1.25.7
kube-scheduler            v1.25.0   v1.25.7
kube-proxy                v1.25.0   v1.25.7
CoreDNS                   v1.9.3    v1.9.3
etcd                      3.5.4-0   3.5.6-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.25.7

_____________________________________________________________________

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     2 x v1.25.0   v1.26.2

Upgrade to the latest stable version:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.25.0   v1.26.2
kube-controller-manager   v1.25.0   v1.26.2
kube-scheduler            v1.25.0   v1.26.2
kube-proxy                v1.25.0   v1.26.2
CoreDNS                   v1.9.3    v1.9.3
etcd                      3.5.4-0   3.5.6-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.26.2

Note: Before you can perform this upgrade, you have to update kubeadm to v1.26.2.

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________


controlplane ~ ➜  
```


Run *kubeadm upgrade apply* to upgrade the cluster

```bash

controlplane ~ ➜  sudo kubeadm upgrade apply v1.26.0
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
.....
.....
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.26.0". Enjoy!
```

Upgrade kubelet and kubectl

```bash

controlplane ~ ➜ apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet=1.26.0-00 kubectl=1.26.0-00 &&  apt-mark hold kubelet kubectl

kubelet was already not hold.
kubectl was already not hold.
.....
.....
Setting up kubectl (1.26.0-00) ...
Setting up kubelet (1.26.0-00) ...
/usr/sbin/policy-rc.d returned 101, not running 'start kubelet.service'
kubelet set on hold.
kubectl set on hold.
```

Restart the kubelet service

```bash
controlplane ~ ➜  sudo systemctl daemon-reload

controlplane ~ ➜  

root@node01 ~ ➜  sudo systemctl restart kubelet

root@node01 ~ ➜  


controlplane ~ ➜  sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2023-03-08 02:24:05 EST; 19s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 23774 (kubelet)
      Tasks: 31 (limit: 251379)
     Memory: 55.9M
```

Uncordon the node

```bash
controlplane ~ ➜  kubectl uncordon controlplane
node/controlplane uncordoned

controlplane ~ ➜

```

Confirm the nodes are running, with the conreolplane node upgraded

```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION   
controlplane   Ready    control-plane   79m   v1.26.0   
node01         Ready    <none>          78m   v1.25.0   

```

Inorder to have the gold-nginx nodes scheduled on controlplane node when node01 is cordoned, remove taints on the controlplane node

List the taints on controplane node

```bash
controlplane ~ ➜  kubectl describe node controlplane 

Taints:             node-role.kubernetes.io/control-plane:NoSchedule
                    node.kubernetes.io/unschedulable:NoSchedule
```

Remove the taints on the controlplane node

```bash
controlplane ~ ➜  kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
node/controlplane untainted

controlplane ~ ➜  kubectl taint node controlplane node.kubernetes.io/unschedulable:NoSchedule-
node/controlplane untainted

controlplane ~ ➜  
```
Drain pods on node01

```bash

controlplane ~ ➜  kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-vrm4p, kube-system/weave-net-v4lvd
evicting pod admin2406/deploy1-78d8d47657-p5wx5
evicting pod admin2406/deploy4-6b96456775-8hfnb
evicting pod admin2406/deploy3-5bdcc4b88c-5ktz9
evicting pod kube-system/coredns-787d4945fb-klqs2
evicting pod kube-system/coredns-787d4945fb-9h5ck
evicting pod admin2406/deploy5-5dc4d5d78b-pv5gt
evicting pod default/gold-nginx-69d8d76f55-44zl6
evicting pod admin2406/deploy2-d6b48767c-4bhk9
pod/deploy2-d6b48767c-4bhk9 evicted
pod/deploy1-78d8d47657-p5wx5 evicted
pod/gold-nginx-69d8d76f55-44zl6 evicted
I0308 02:26:34.700145   25432 request.go:682] Waited for 1.095358956s due to client-side throttling, not priority and fairness, request: GET:https://controlplane:6443/api/v1/namespaces/admin2406/pods/deploy5-5dc4d5d78b-pv5gt
pod/deploy3-5bdcc4b88c-5ktz9 evicted
pod/deploy5-5dc4d5d78b-pv5gt evicted
pod/deploy4-6b96456775-8hfnb evicted
pod/coredns-787d4945fb-klqs2 evicted
pod/coredns-787d4945fb-9h5ck evicted
node/node01 drained

controlplane ~ ➜  
```


SSH into node01

```bash

controlplane ~ ➜  ssh node01
Warning: Permanently added the ECDSA host key for IP address '192.9.128.12' to the list of known hosts.

root@node01 ~ ➜  
```

Upgrade kubeadm tool

```bash

root@node01 ~ ➜  apt-mark unhold kubeadm && \
> apt-get update && apt-get install -y kubeadm=1.26.0-00 && \
> apt-mark hold kubeadm
kubeadm was already not hold.
....
....
Setting up kubeadm (1.26.0-00) ...
kubeadm set on hold.

root@node01 ~ ➜  sudo kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.

root@node01 ~ ➜  

```

Upgrade kubelet and kubectl

```bash

root@node01 ~ ➜  apt-mark unhold kubelet kubectl && \
> apt-get update && apt-get install -y kubelet=1.26.0-00 kubectl=1.26.0-00 && \
> apt-mark hold kubelet kubectl
Canceled hold on kubelet.
Canceled hold on kubectl.

Setting up kubectl (1.26.0-00) ...
Setting up kubelet (1.26.0-00) ...
/usr/sbin/policy-rc.d returned 101, not running 'start kubelet.service'
kubelet set on hold.
kubectl set on hold.
```

Restart kubelet service

```bash
controlplane ~ ➜  sudo systemctl daemon-reload

controlplane ~ ➜  

root@node01 ~ ➜  sudo systemctl restart kubelet

root@node01 ~ ➜  

root@node01 ~ ➜  sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabl
ed)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2023-03-08 02:45:14 EST; 1min 16s ag

```


Exit from within node01

```bash
root@node01 ~ ➜ exit
logout
Connection to node01 closed.
```

Uncordon node01

```bash

controlplane ~ ➜  kubectl uncordon node01
node/node01 uncordoned

controlplane ~ ➜ 
```

View all nodes to confirm they are ready


```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   72m   v1.26.0
node01         Ready    <none>          71m   v1.26.0

controlplane ~ ➜  

```
 
Confirm *gold-nginx* pods are scheduled on controlplane node


```bash

controlplane ~ ➜  watch kubectl get pod -A -o wide

Every 2.0s: kubectl get pod -A -o wide                                                                                                    controlplane: Wed Mar  8 07:17:40 2023

NAMESPACE     NAME                                   READY   STATUS    RESTARTS      AGE     IP             NODE           NOMINATED NODE   READINESS GATES
admin2406     deploy1-78d8d47657-v9xc4               1/1     Running   0             4m17s   10.244.0.4     controlplane   <none>           <none>
admin2406     deploy2-d6b48767c-hwj2h                1/1     Running   0             4m17s   10.244.0.3     controlplane   <none>           <none>
admin2406     deploy3-5bdcc4b88c-ssgrv               1/1     Running   0             4m17s   10.244.0.2     controlplane   <none>           <none>
admin2406     deploy4-6b96456775-phch2               1/1     Running   0             4m17s   10.244.0.5     controlplane   <none>           <none>
admin2406     deploy5-5dc4d5d78b-wpcb5               1/1     Running   0             4m17s   10.244.0.7     controlplane   <none>           <none>
alpha         alpha-mysql-6d945ffc78-f5xzl           0/1     Pending   0             21m     <none>         <none>         <none>           <none>
default       gold-nginx-69d8d76f55-tl9pj            1/1     Running   0             4m17s   10.244.0.6     controlplane   <none>           <none>
kube-system   coredns-787d4945fb-7zjb4               1/1     Running   0             4m17s   10.244.0.8     controlplane   <none>           <none>
kube-system   coredns-787d4945fb-8jvwc               1/1     Running   0             4m16s   10.244.0.9     controlplane   <none>           <none>
kube-system   etcd-controlplane                      1/1     Running   0             10m     192.18.135.3   controlplane   <none>           <none>
kube-system   kube-apiserver-controlplane            1/1     Running   0             10m     192.18.135.3   controlplane   <none>           <none>
kube-system   kube-controller-manager-controlplane   1/1     Running   0             10m     192.18.135.3   controlplane   <none>           <none>
kube-system   kube-proxy-hc4ml                       1/1     Running   0             12m     192.18.135.6   node01         <none>           <none>
kube-system   kube-proxy-wswhj                       1/1     Running   0             12m     192.18.135.3   controlplane   <none>           <none>
kube-system   kube-scheduler-controlplane            1/1     Running   0             10m     192.18.135.3   controlplane   <none>           <none>
kube-system   weave-net-fq8g9                        2/2     Running   0             75m     192.18.135.6   node01         <none>           <none>
kube-system   weave-net-pcmcv                        2/2     Running   1 (75m ago)   75m     192.18.135.3   controlplane   <none>           <none>
```

***The End***