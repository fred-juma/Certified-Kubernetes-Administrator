#### Service Networking 

#### ClusterIP Service
- When a service is created, they are accessible from all pods in the cluster through its IP Address irrespective of the nodes.

#### NodePort
- When a service is created, they are accessible from all pods in the cluster through its IP Address irrespective of the nodes.
- In addition, the NodePort  exposes a port on the cluster.

- Services when created in the cluster are assigned an IP address from a predefined range

kube-api-server --service-cluster-ip-range ipNet (Default: 10.0.0.0/24)

To see your specific server service ip range allocation:

ps aux | grep kube-api-server

#### Kube-proxy modes

can be set using the kube-proxy service:

kube-proxy --proxy-mode [userspace | iptables | ipvs] ...

- The default option is the iptables

To observe service rules:

iptables -L -t nat | grep db-service

You can check kube-proxy logs for the rules

cat /var/log/kube-proxy.log

What network range are the nodes in the cluster part of? **10.19.9.9/24**

Check the nodes IP addresses

```bash

controlplane ~ ➜  kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
controlplane   Ready    control-plane   125m   v1.26.0   10.19.9.9     <none>        Ubuntu 20.04.5 LTS   5.4.0-1093-gcp   containerd://1.6.6
node01         Ready    <none>          125m   v1.26.0   10.19.9.12    <none>        Ubuntu 20.04.5 LTS   5.4.0-1093-gcp   containerd://1.6.6
```

Check the ip address configuration for *eth0* interface

```bash
controlplane ~ ➜  ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether fe:35:80:3a:f1:6c brd ff:ff:ff:ff:ff:ff
4: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP group default qlen 1000
    link/ether 0e:90:6d:db:41:4c brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.1/16 brd 10.244.255.255 scope global weave
       valid_lft forever preferred_lft forever
6: vethwe-datapath@vethwe-bridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master datapath state UP group default 
    link/ether ee:aa:70:8a:63:23 brd ff:ff:ff:ff:ff:ff
7: vethwe-bridge@vethwe-datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP group default 
    link/ether 9e:62:0e:07:58:11 brd ff:ff:ff:ff:ff:ff
8: vxlan-6784: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65535 qdisc noqueue master datapath state UNKNOWN group default qlen 1000
    link/ether 22:3f:0b:64:d4:ee brd ff:ff:ff:ff:ff:ff
10: vethwepldd8f761@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP group default 
    link/ether d6:67:f3:5b:8f:86 brd ff:ff:ff:ff:ff:ff link-netns cni-dffca24a-7196-f324-363a-c3766708972a
12: vethweplab5947b@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP group default 
    link/ether 56:dc:10:d5:90:af brd ff:ff:ff:ff:ff:ff link-netns cni-c944af72-ce46-80d7-dd2a-75a681d9dd75
6080: eth0@if6081: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default 
    link/ether 02:42:0a:13:09:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.19.9.9/24 brd 10.19.9.255 scope global eth0
       valid_lft forever preferred_lft forever
6082: eth1@if6083: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:19:01:45 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 172.25.1.69/24 brd 172.25.1.255 scope global eth1
       valid_lft forever preferred_lft forever

controlplane ~ ➜  
```

View the valid IP addresses in the cidr

```bash
controlplane ~ ➜  ipcalc -b 10.19.9.9/24
Address:   10.19.9.9            
Netmask:   255.255.255.0 = 24   
Wildcard:  0.0.0.255            
=>
Network:   10.19.9.0/24         
HostMin:   10.19.9.1            
HostMax:   10.19.9.254          
Broadcast: 10.19.9.255          
Hosts/Net: 254                   Class A, Private Internet


controlplane ~ ➜  
```

What is the range of IP addresses configured for PODs on this cluster? **ipalloc-range:10.244.0.0/16**

The network is configured with weave, check the weave pods name

```bash
controlplane ~ ➜  kubectl get pod -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS       AGE
kube-system   coredns-787d4945fb-kx22f               1/1     Running   0              132m
kube-system   coredns-787d4945fb-zq547               1/1     Running   0              132m
kube-system   etcd-controlplane                      1/1     Running   0              132m
kube-system   kube-apiserver-controlplane            1/1     Running   0              132m
kube-system   kube-controller-manager-controlplane   1/1     Running   0              132m
kube-system   kube-proxy-d7l2d                       1/1     Running   0              132m
kube-system   kube-proxy-mkzlj                       1/1     Running   0              132m
kube-system   kube-scheduler-controlplane            1/1     Running   0              132m
kube-system   weave-net-255sj                        2/2     Running   0              132m
kube-system   weave-net-x5rvw                        2/2     Running   1 (132m ago)   132m

controlplane ~ ➜  
```

Check the allocation on the weave pod logs 

```bash
controlplane ~ ➜  kubectl logs weave-net-255sj weave -n kube-system
INFO: 2023/02/02 05:52:23.763171 Command line options: map[conn-limit:200 datapath:datapath db-prefix:/weavedb/weave-net docker-api: expect-npc:true http-addr:127.0.0.1:6784 ipalloc-init:consensus=1 ipalloc-range:10.244.0.0/16 metrics-addr:0.0.0.0:6782 name:4a:69:5e:87:35:f1 nickname:node01 no-dns:true no-masq-local:true port:6783]
INFO: 2023/02/02 05:52:23.763262 weave  2.8.1
INFO: 2023/02/02 05:52:25.026923 Bridge type is bridged_fastdp

```


What is the IP Range configured for the services within the cluster? **--service-cluster-ip-range=10.96.0.0/12**

```bash
controlplane ~ ➜  cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep service-cluster
    - --service-cluster-ip-range=10.96.0.0/12

controlplane ~ ➜  
```

What type of proxy is the kube-proxy configured to use? *iptables*

```bash
controlplane ~ ➜  kubectl logs kube-proxy-d7l2d -n kube-system
I0202 05:52:21.403256       1 node.go:163] Successfully retrieved node IP: 10.19.9.12
I0202 05:52:21.403381       1 server_others.go:109] "Detected node IP" address="10.19.9.12"
I0202 05:52:21.403426       1 server_others.go:535] "Using iptables proxy"
I0202 05:52:21.484235       1 server_others.go:176] "Using iptables Proxier"
I0202 05:52:21.484287       1 server_others.go:183] "kube-proxy running in dual-stack mode" ipFamily=IPv4
I0202 05:52:21.484297       1 server_others.go:184] "Creating dualStackProxier for iptables"
I0202 05:52:21.484337       1 server_others.go:465] "Detect-local-mode set to ClusterCIDR, but no IPv6 cluster CIDR defined, , defaulting to no-op detect-local for IPv6"
I0202 05:52:21.484374       1 proxier.go:242] "Setting route_localnet=1 to allow node-ports on localhost; to change this either disable iptables.localhostNodePorts (--iptables-localhost-nodeports) or set nodePortAddresses (--nodeport-addresses) to filter loopback addresses"
I0202 05:52:21.721455       1 server.go:655] "Version info" version="v1.26.0"
I0202 05:52:21.721490       1 server.go:657] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
I0202 05:52:21.783386       1 conntrack.go:52] "Setting nf_conntrack_max" nf_conntrack_max=1835008
I0202 05:52:21.786479       1 config.go:444] "Starting node config controller"
I0202 05:52:21.786512       1 shared_informer.go:273] Waiting for caches to sync for node config
I0202 05:52:21.786516       1 config.go:226] "Starting endpoint slice config controller"
I0202 05:52:21.786533       1 shared_informer.go:273] Waiting for caches to sync for endpoint slice config
I0202 05:52:21.786590       1 config.go:317] "Starting service config controller"
I0202 05:52:21.786620       1 shared_informer.go:273] Waiting for caches to sync for service config
I0202 05:52:21.887205       1 shared_informer.go:280] Caches are synced for service config
I0202 05:52:21.887224       1 shared_informer.go:280] Caches are synced for endpoint slice config
I0202 05:52:21.887275       1 shared_informer.go:280] Caches are synced for node config

controlplane ~ ➜  
```

How does this Kubernetes cluster ensure that a kube-proxy pod runs on all nodes in the cluster? Inspect the kube-proxy pods and try to identify how they are deployed. **using DaemonSet**

```bash
controlplane ~ ➜  kubectl describe pod kube-proxy-d7l2d -n kube-system
Name:                 kube-proxy-d7l2d
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Service Account:      kube-proxy
Node:                 node01/10.19.9.12
Start Time:           Thu, 02 Feb 2023 00:52:19 -0500
Labels:               controller-revision-hash=78545cdb7d
                      k8s-app=kube-proxy
                      pod-template-generation=1
Annotations:          <none>
Status:               Running
IP:                   10.19.9.12
IPs:
  IP:           10.19.9.12
Controlled By:  DaemonSet/kube-proxy
Containers:
  kube-proxy:
    Container ID:  containerd://f7ac2fd8d31d39c3e10fb9d2f37ab4bf426f3b14e1e4f1fac3c774abbfe6daee
    Image:         registry.k8s.io/kube-proxy:v1.26.0
    Image ID:      registry.k8s.io/kube-proxy@sha256:1e9bbe429e4e2b2ad32681c91deb98a334f1bf4135137df5f84f9d03689060fe
    Port:          <none>
    Host Port:     <none>
    Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/config.conf
      --hostname-override=$(NODE_NAME)
    State:          Running
      Started:      Thu, 02 Feb 2023 00:52:21 -0500
    Ready:          True
    Restart Count:  0
    Environment:
      NODE_NAME:   (v1:spec.nodeName)
    Mounts:
      /lib/modules from lib-modules (ro)
      /run/xtables.lock from xtables-lock (rw)
      /var/lib/kube-proxy from kube-proxy (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mvc8f (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-proxy:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-proxy
    Optional:  false
  xtables-lock:
    Type:          HostPath (bare host directory volume)
    Path:          /run/xtables.lock
    HostPathType:  FileOrCreate
  lib-modules:
    Type:          HostPath (bare host directory volume)
    Path:          /lib/modules
    HostPathType:  
  kube-api-access-mvc8f:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 op=Exists
                             node.kubernetes.io/disk-pressure:NoSchedule op=Exists
                             node.kubernetes.io/memory-pressure:NoSchedule op=Exists
                             node.kubernetes.io/network-unavailable:NoSchedule op=Exists
                             node.kubernetes.io/not-ready:NoExecute op=Exists
                             node.kubernetes.io/pid-pressure:NoSchedule op=Exists
                             node.kubernetes.io/unreachable:NoExecute op=Exists
                             node.kubernetes.io/unschedulable:NoSchedule op=Exists
Events:                      <none>

controlplane ~ ➜  kubectl get ds -n kube-system
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   145m
weave-net    2         2         2       2            2           <none>                   145m

controlplane ~ ➜  
```

***The End***