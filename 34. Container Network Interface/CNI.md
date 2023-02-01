#### Pod Networking Model

- Every pod should have an ip address
- Every pod should be able to communicate with every other pod in the same node
- Every pod should be able to communicate with every other pod on other nodes without NAT

Deploying Weave CNI

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

Inspect the kubelet service and identify the container runtime endpoint value is set for Kubernetes. **unix:///var/run/containerd/containerd.sock**

```bash
controlplane ~ ➜  ps -aux | grep kubelet | grep --color container-runtime-endpoint
root        3793  0.0  0.0 4308060 107536 ?      Ssl  04:42   0:10 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9

controlplane ~ ➜  

```

What is the path configured with all binaries of CNI supported plugins?

The CNI binaries are located under /opt/cni/bin by default.

```bash
controlplane ~ ➜  ls -l /opt/cni/bin
total 69916
-rwxr-xr-x 1 root root  4159518 May 13  2020 bandwidth
-rwxr-xr-x 1 root root  4671647 May 13  2020 bridge
-rwxr-xr-x 1 root root 12124326 May 13  2020 dhcp
-rwxr-xr-x 1 root root  5945760 May 13  2020 firewall
-rwxr-xr-x 1 root root  2474798 Feb  1 04:42 flannel
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

controlplane ~ ➜  
```


What is the CNI plugin configured to be used on this kubernetes cluster?

```bash
controlplane ~ ➜  ls /etc/cni/net.d/
10-flannel.conflist

controlplane ~ ➜  

```

What binary executable file will be run by kubelet after a container and its associated namespace are created? **"type": "flannel"**

```bash
controlplane ~ ➜  cat /etc/cni/net.d/10-flannel.conflist
{
  "name": "cbr0",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}

controlplane ~ ➜  
```


#### Deploy weave-net pod netwoeking solution

Deploy weave-net networking solution to the cluster.

NOTE: - We already have provided a weave manifest file under the /root/weave directory.

```bash

controlplane ~/weave ➜  kubectl apply -f weave-daemonset-k8s.yaml 
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created

controlplane ~/weave ➜  
```

Check if the weave pods are created

```bash

controlplane ~/weave ➜  kubectl get pods -A | grep weave
kube-system   weave-net-tbxtn                        2/2     Running   0          41s

controlplane ~/weave ➜  
```

Weave - IP Address Management plugins:
- DHCP
- host-local

CNI configuration file

*cat /etc/cni/net.d/net-script.conf*

- ipam section specifies the type of plugin to be used, the subnet and routes.

Weave IP Address subnet: 10.32.0./12 -> 10.32.0.1 - 10.47.255.254

How many Nodes are part of this cluster? Including master and worker nodes

```bash
controlplane ~ ➜  kubectl get nodes -A
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   36m   v1.26.0
node01         Ready    <none>          35m   v1.26.0

controlplane ~ ➜  
```

What is the Networking Solution used by this cluster? **weave**

```bash
controlplane ~ ➜  ls -l /etc/cni/net.d/
total 4
-rw-r--r-- 1 root root 318 Feb  1 04:35 10-weave.conflist

controlplane ~ ➜  
```

How many weave agents/peers are deployed in this cluster? **2 weave pods**

```bash
controlplane ~ ➜  kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS      AGE
coredns-787d4945fb-mxdxf               1/1     Running   0             39m
coredns-787d4945fb-z488v               1/1     Running   0             39m
etcd-controlplane                      1/1     Running   0             39m
kube-apiserver-controlplane            1/1     Running   0             39m
kube-controller-manager-controlplane   1/1     Running   0             39m
kube-proxy-fs6zj                       1/1     Running   0             39m
kube-proxy-ztnkm                       1/1     Running   0             39m
kube-scheduler-controlplane            1/1     Running   0             39m
weave-net-7nv48                        2/2     Running   1 (39m ago)   39m
weave-net-pfwcj                        2/2     Running   0             39m

controlplane ~ ➜  
```

On which nodes are the weave peers present?

```bash
controlplane ~ ➜  kubectl get pods -n kube-system -o wide
NAME                                   READY   STATUS    RESTARTS      AGE   IP            NODE           NOMINATED NODE   READINESS GATES
coredns-787d4945fb-mxdxf               1/1     Running   0             40m   10.244.0.3    controlplane   <none>           <none>
coredns-787d4945fb-z488v               1/1     Running   0             40m   10.244.0.2    controlplane   <none>           <none>
etcd-controlplane                      1/1     Running   0             40m   10.32.201.3   controlplane   <none>           <none>
kube-apiserver-controlplane            1/1     Running   0             40m   10.32.201.3   controlplane   <none>           <none>
kube-controller-manager-controlplane   1/1     Running   0             40m   10.32.201.3   controlplane   <none>           <none>
kube-proxy-fs6zj                       1/1     Running   0             40m   10.32.201.3   controlplane   <none>           <none>
kube-proxy-ztnkm                       1/1     Running   0             40m   10.32.201.6   node01         <none>           <none>
kube-scheduler-controlplane            1/1     Running   0             40m   10.32.201.3   controlplane   <none>           <none>
weave-net-7nv48                        2/2     Running   1 (40m ago)   40m   10.32.201.3   controlplane   <none>           <none>
weave-net-pfwcj                        2/2     Running   0             40m   10.32.201.6   node01         <none>           <none>

controlplane ~ ➜ 
```

Identify the name of the bridge network/interface created by weave on each node. **weave**

```bash

controlplane ~ ➜  ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 3a:0f:3c:51:50:ac brd ff:ff:ff:ff:ff:ff
4: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ce:d7:3f:7d:ff:2b brd ff:ff:ff:ff:ff:ff
6: vethwe-datapath@vethwe-bridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master datapath state UP mode DEFAULT group default 
    link/ether 06:1d:81:b9:d8:ee brd ff:ff:ff:ff:ff:ff
7: vethwe-bridge@vethwe-datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP mode DEFAULT group default 
    link/ether be:bf:52:e1:13:80 brd ff:ff:ff:ff:ff:ff
8: vxlan-6784: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65535 qdisc noqueue master datapath state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether d2:f2:70:a6:44:73 brd ff:ff:ff:ff:ff:ff
10: vethwepld859de8@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP mode DEFAULT group default 
    link/ether 52:32:c8:6e:38:1a brd ff:ff:ff:ff:ff:ff link-netns cni-4ceab117-b8ba-a3c6-002d-bbc6583427e3
12: vethwepl444633d@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP mode DEFAULT group default 
    link/ether 8e:bc:35:fd:b4:32 brd ff:ff:ff:ff:ff:ff link-netns cni-c949bc0f-a46a-adda-8b01-77eb4ae7557c
6480: eth0@if6481: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:0a:20:c9:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
6482: eth1@if6483: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:ac:19:00:73 brd ff:ff:ff:ff:ff:ff link-netnsid 1

controlplane ~ ➜  

````

What is the POD IP address range configured by weave?

```bash
controlplane ~ ➜  ip a show weave
4: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP group default qlen 1000
    link/ether ce:d7:3f:7d:ff:2b brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.1/16 brd 10.244.255.255 scope global weave
       valid_lft forever preferred_lft forever

controlplane ~ ➜  
```
What is the default gateway configured on the PODs scheduled on node01?


Try scheduling a pod on node01 and check ip route output **10.244.192.0**

```bash
root@node01 ~ ➜  ip route
default via 172.25.1.1 dev eth1 
10.32.201.0/24 dev eth0 proto kernel scope link src 10.32.201.6 
10.244.0.0/16 dev weave proto kernel scope link src 10.244.192.0 
172.25.1.0/24 dev eth1 proto kernel scope link src 172.25.1.127 

root@node01 ~ ➜  
```