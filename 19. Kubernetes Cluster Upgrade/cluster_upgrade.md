#### Kubernetes Cluster Upgrade Process

Components
- kube-apiserver
- controller-manager
- kube-scheduler
- kubelet
- kube-proxy
- kubectl

kubeadm - upgrade

The upgrade workflow at high level is the following:

- Upgrade a primary control plane node.
- Upgrade additional control plane nodes.
- Upgrade worker nodes

Pre-upgrade checks to see all versions of the components on the cluster. 

What is the latest stable version of Kubernetes as of today? remote version is : v1.26.0

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
I0111 08:26:02.469070   20237 version.go:256] remote version is much newer: v1.26.0; falling back to: stable-1.25
[upgrade/versions] Target version: v1.25.5
[upgrade/versions] Latest version in the v1.25 series: v1.25.5

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     2 x v1.25.0   v1.25.5

Upgrade to the latest version in the v1.25 series:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.25.0   v1.25.5
kube-controller-manager   v1.25.0   v1.25.5
kube-scheduler            v1.25.0   v1.25.5
kube-proxy                v1.25.0   v1.25.5
CoreDNS                   v1.9.3    v1.9.3
etcd                      3.5.4-0   3.5.4-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.25.5

Note: Before you can perform this upgrade, you have to update kubeadm to v1.25.5.

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

What is the latest version available for an upgrade with the current version of the kubeadm tool installed?

 kubeadm upgrade apply v1.25.5

To see OS version

```bash

controlplane ~ ➜  cat /etc/*release*
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
DISTRIB_DESCRIPTION="Ubuntu 20.04.5 LTS"
NAME="Ubuntu"
VERSION="20.04.5 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.5 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal

controlplane ~ ➜  
```

See kubeadm version

```bash

controlplane ~ ➜  kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"25", GitVersion:"v1.25.0", GitCommit:"a866cbe2e5bbaa01cfd5e969aa3e033f3282a8a2", GitTreeState:"clean", BuildDate:"2022-08-23T17:43:25Z", GoVersion:"go1.19", Compiler:"gc", Platform:"linux/amd64"}

controlplane ~ ➜  
```

kubeadm upgrade plan

Note: kubeadm does not upgrade the kubelet

To verify current versions of the kubelet running on the nodes

```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   87m   v1.25.0
node01         Ready    <none>          86m   v1.25.0

controlplane ~ ➜  
```

See all resources in the cluster 

```bash
controlplane ~ ➜  kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/blue-5db6db69f7-29bbv   1/1     Running   0          7m26s
pod/blue-5db6db69f7-6sjkf   1/1     Running   0          7m26s
pod/blue-5db6db69f7-dnjmk   1/1     Running   0          7m26s
pod/blue-5db6db69f7-rx67z   1/1     Running   0          7m26s
pod/blue-5db6db69f7-zjgwc   1/1     Running   0          7m26s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP        90m
service/red-service   NodePort    10.96.234.238   <none>        80:30080/TCP   7m27s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/blue   5/5     5            5           7m26s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/blue-5db6db69f7   5         5         5       7m26s

controlplane ~ ➜  
```

See the nodes the pods are hosted on

```bash
controlplane ~ ➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-5db6db69f7-29bbv   1/1     Running   0          8m41s   10.244.0.5   controlplane   <none>           <none>
blue-5db6db69f7-6sjkf   1/1     Running   0          8m41s   10.244.1.4   node01         <none>           <none>
blue-5db6db69f7-dnjmk   1/1     Running   0          8m41s   10.244.1.3   node01         <none>           <none>
blue-5db6db69f7-rx67z   1/1     Running   0          8m41s   10.244.0.4   controlplane   <none>           <none>
blue-5db6db69f7-zjgwc   1/1     Running   0          8m41s   10.244.1.2   node01         <none>           <none>

controlplane ~ ➜  
```

You are tasked to upgrade the cluster. Users accessing the applications must not be impacted, and you cannot provision new VMs. What strategy would you use to upgrade the cluster?

upgrade one node at a time, so that pods are migrated to the available node. Check taints and tolerations to ensure pods are able to migrate to the other nodes




#### Upgrade the control plane node

move workloads from the first node (by draining the node) to schedule them on other nodes as well as cordon the node

```bash
controlplane ~ ➜  kubectl drain controlplane --ignore-daemonsets
node/controlplane already cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-6vws9, kube-system/kube-proxy-btftr
evicting pod kube-system/coredns-565d847f94-vv5rp
evicting pod default/blue-5db6db69f7-rdfq6
evicting pod kube-system/coredns-565d847f94-pxpgc
evicting pod default/blue-5db6db69f7-888fs
pod/blue-5db6db69f7-888fs evicted
pod/blue-5db6db69f7-rdfq6 evicted
pod/coredns-565d847f94-pxpgc evicted
pod/coredns-565d847f94-vv5rp evicted
node/controlplane drained

controlplane ~ ➜  
```

Upgrade the controlplane components to exact version v1.26.0

Upgrade the kubeadm tool (if not already), then the controlplane components, and finally the kubelet. Practice referring to the Kubernetes documentation page.
Note: While upgrading kubelet, if you hit dependency issues while running the apt-get upgrade kubelet command, use the apt install kubelet=1.26.0-00 command instead.

kubeadm must be upgraded first b4 the kubernetes cluster

update the OS

```bash
controlplane ~ ➜  sudo apt update
Hit:2 https://download.docker.com/linux/ubuntu focal InRelease                                        
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease                               
Hit:3 http://security.ubuntu.com/ubuntu focal-security InRelease                                      
Hit:4 http://archive.ubuntu.com/ubuntu focal InRelease     
Hit:5 http://archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:6 http://archive.ubuntu.com/ubuntu focal-backports InRelease
Reading package lists... Done
Building dependency tree       
Reading state information... Done
4 packages can be upgraded. Run 'apt list --upgradable' to see them.

controlplane ~ ➜  
```

Determine which versions to upgrade = 1.26.0-00

```bash
controlplane ~ ➜  apt-cache madison kubeadm
   kubeadm |  1.26.0-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
```


upgrade the kubeadm tool

```bash
controlplane ~ ➜  apt-mark unhold kubeadm && \
>  apt-get update && apt-get install -y kubeadm=1.26.0-00 && \
>  apt-mark hold kubeadm
kubeadm was already not hold.
Hit:2 https://download.docker.com/linux/ubuntu focal InRelease                                        
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease                               
Hit:3 http://archive.ubuntu.com/ubuntu focal InRelease                                                
Hit:4 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:5 http://archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:6 http://archive.ubuntu.com/ubuntu focal-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 3 not upgraded.
Need to get 9,730 kB of archives.
After this operation, 2,974 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.26.0-00 [9,730 kB]
Fetched 9,730 kB in 0s (39.4 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 18569 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.26.0-00_amd64.deb ...
Unpacking kubeadm (1.26.0-00) over (1.25.0-00) ...
Setting up kubeadm (1.26.0-00) ...
kubeadm set on hold.

controlplane ~ ➜  
```

Verify that the download works and has the expected version:

```bash
controlplane ~ ➜  kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.0", GitCommit:"b46a3f887ca979b1a5d14fd39cb1af43e7e5d12d", GitTreeState:"clean", BuildDate:"2022-12-08T19:57:06Z", GoVersion:"go1.19.4", Compiler:"gc", Platform:"linux/amd64"}

controlplane ~ ➜  
```

Verify the upgrade plan. This command checks that your cluster can be upgraded, and fetches the versions you can upgrade to. It also shows a table with the component config version states.

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
[upgrade/versions] Target version: v1.26.0
[upgrade/versions] Latest version in the v1.25 series: v1.25.5

W0111 09:07:58.556770   22293 configset.go:177] error unmarshaling configuration schema.GroupVersionKind{Group:"kubeproxy.config.k8s.io", Version:"v1alpha1", Kind:"KubeProxyConfiguration"}: strict decoding error: unknown field "udpIdleTimeout"
Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     2 x v1.25.0   v1.25.5

Upgrade to the latest version in the v1.25 series:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.25.0   v1.25.5
kube-controller-manager   v1.25.0   v1.25.5
kube-scheduler            v1.25.0   v1.25.5
kube-proxy                v1.25.0   v1.25.5
CoreDNS                   v1.9.3    v1.9.3
etcd                      3.5.4-0   3.5.6-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.25.5

_____________________________________________________________________

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     2 x v1.25.0   v1.26.0

Upgrade to the latest stable version:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.25.0   v1.26.0
kube-controller-manager   v1.25.0   v1.26.0
kube-scheduler            v1.25.0   v1.26.0
kube-proxy                v1.25.0   v1.26.0
CoreDNS                   v1.9.3    v1.9.3
etcd                      3.5.4-0   3.5.6-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.26.0

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

Now apply the kubeadm upgrade 

```bash
controlplane ~ ➜  sudo kubeadm upgrade apply v1.26.0
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W0111 09:09:52.211843   22588 configset.go:177] error unmarshaling configuration schema.GroupVersionKind{Group:"kubeproxy.config.k8s.io", Version:"v1alpha1", Kind:"KubeProxyConfiguration"}: strict decoding error: unknown field "udpIdleTimeout"
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.26.0"
[upgrade/versions] Cluster version: v1.25.0
[upgrade/versions] kubeadm version: v1.26.0
[upgrade] Are you sure you want to proceed? [y/N]: 

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.26.0". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```



upgrade the kubelet and kubectl on the control node

```bash
rolplane ~ ➜  apt-mark unhold kubelet kubectl && \
> apt-get update && apt-get install -y kubelet=1.26.0-00 kubectl=1.26.0-00 && \
> apt-mark hold kubelet kubectl
kubelet was already not hold.
kubectl was already not hold.
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease  
Hit:2 https://download.docker.com/linux/ubuntu focal InRelease                                        
Hit:3 http://security.ubuntu.com/ubuntu focal-security InRelease                                      
Hit:4 http://archive.ubuntu.com/ubuntu focal InRelease                   
Hit:5 http://archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:6 http://archive.ubuntu.com/ubuntu focal-backports InRelease
Reading package lists... 5%
```

Restart the kubelet

```bash

controlplane ~ ➜  sudo systemctl daemon-reload

controlplane ~ ➜  sudo systemctl restart kubelet

controlplane ~ ➜  
```

Check kubelet status to ensure it is running

```bash
controlplane ~ ➜  sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2023-01-11 09:18:44 EST; 52s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 28305 (kubelet)
      Tasks: 32 (limit: 251382)
     Memory: 60.5M
     CGroup: /system.slice/kubelet.service
```

See status of controlnode

```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS                     ROLES           AGE   VERSION
controlplane   Ready,SchedulingDisabled   control-plane   94m   v1.26.0
node01         Ready                      <none>          94m   v1.25.0

controlplane ~ ➜  
```

Mark the controlplane node as "Schedulable" again

```bash
controlplane ~ ➜  kubectl uncordon controlplane
node/controlplane uncordoned

controlplane ~ ➜  
```

See status of controlnode in ready and scheduled state

```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   96m   v1.26.0
node01         Ready    <none>          95m   v1.25.0

controlplane ~ ➜  
```

#### Upgrade worker nodes


Next is the worker node. Drain the worker node of the workloads and mark it UnSchedulable

```bash
controlplane ~ ➜  kubectl drain node01 --ignore-daemonsets
node/node01 already cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-l52pb, kube-system/kube-proxy-ckxgz
evicting pod kube-system/coredns-787d4945fb-xf65h
evicting pod default/blue-5db6db69f7-c7lzd
evicting pod default/blue-5db6db69f7-6nfvf
evicting pod default/blue-5db6db69f7-wdmpd
evicting pod default/blue-5db6db69f7-nw5mb
evicting pod kube-system/coredns-787d4945fb-mvrcz
evicting pod default/blue-5db6db69f7-74cd9
I0111 09:24:00.199114   30621 request.go:682] Waited for 1.094128043s due to client-side throttling, not priority and fairness, request: GET:https://controlplane:6443/api/v1/namespaces/default/pods/blue-5db6db69f7-wdmpd
pod/blue-5db6db69f7-nw5mb evicted
pod/blue-5db6db69f7-6nfvf evicted
pod/blue-5db6db69f7-wdmpd evicted
pod/blue-5db6db69f7-74cd9 evicted
pod/blue-5db6db69f7-c7lzd evicted
pod/coredns-787d4945fb-mvrcz evicted
pod/coredns-787d4945fb-xf65h evicted
node/node01 drained

controlplane ~ ➜ 
```


SSH to node01

```bash
controlplane ~ ➜  ssh node01

root@node01 ~ ➜  
```

Upgrade the kubeadm tool on the worker node


```bash
root@node01 ➜  apt-mark unhold kubeadm && \
> apt-get update && apt-get install -y kubeadm=1.26.0-00 && \
> apt-mark hold kubeadm
Canceled hold on kubeadm.
Hit:2 https://download.docker.com/linux/ubuntu focal InRelease                                        
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease                               
Hit:3 http://archive.ubuntu.com/ubuntu focal InRelease                                                
Get:4 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:6 http://archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Fetched 336 kB in 3s (112 kB/s)   
Reading package lists... Done
Reading package lists... Done
Building dependency tree       
Reading state information... Done
kubeadm is already the newest version (1.26.0-00).
0 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
kubeadm set on hold.

root@node01 ➜  
```

Call "kubeadm upgrade". For worker nodes this upgrades the local kubelet configuration:

```bash
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
root@node01 ➜  apt-mark unhold kubelet kubectl && \
> apt-get update && apt-get install -y kubelet=1.26.0-00 kubectl=1.26.0-00 && \
> apt-mark hold kubelet kubectl
Canceled hold on kubelet.
Canceled hold on kubectl.
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease                               
Hit:2 http://security.ubuntu.com/ubuntu focal-security InRelease                                      
Hit:3 https://download.docker.com/linux/ubuntu focal InRelease                                        
Hit:4 http://archive.ubuntu.com/ubuntu focal InRelease                                                
Hit:5 http://archive.ubuntu.com/ubuntu focal-updates InRelease
Get:6 http://archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Fetched 108 kB in 3s (40.7 kB/s)   
Reading package lists... Done
Reading package lists... Done
Building dependency tree       
Reading state information... Done
kubectl is already the newest version (1.26.0-00).
kubelet is already the newest version (1.26.0-00).
0 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
kubelet set on hold.
kubectl set on hold.

root@node01 ➜  
```

Restart the kubelet

```bash
root@node01 ~ ➜  sudo systemctl daemon-reload

root@node01 ~ ➜  sudo systemctl restart kubelet

root@node01 ~ ➜  sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset
: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2023-01-11 09:46:57 EST; 7s a
go
       Docs: https://kubernetes.io/docs/home/
   Main PID: 29465 (kubelet)
      Tasks: 29 (limit: 251382)
     Memory: 43.7M
     CGroup: /system.slice/kubelet.service
```

Uncordon the node - Bring the node back online by marking it schedulable. All kubectl commands are done on the control plane node

```bash
controlplane ~ ➜  kubectl uncordon node01
node/node01 uncordoned

controlplane ~ ➜  
```

Check that node01 is ready

```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE    VERSION
controlplane   Ready    control-plane   121m   v1.26.0
node01         Ready    <none>          120m   v1.26.0

controlplane ~ ➜  
```


To show token used to set up the cluster 

kubeadm token list

***The End***

