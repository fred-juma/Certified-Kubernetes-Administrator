#### Exercise:

How many nodes are part of this cluster? Including the controlplane and worker nodes.

```bash
controlplane ~ ➜  kubectl get nodes -o wide  -A
NAME           STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
controlplane   Ready    control-plane   6m23s   v1.26.0   10.40.203.6   <none>        Ubuntu 20.04.5 LTS   5.4.0-1093-gcp   containerd://1.6.6
node01         Ready    <none>          5m43s   v1.26.0   10.40.203.9   <none>        Ubuntu 20.04.5 LTS   5.4.0-1093-gcp   containerd://1.6.6

controlplane ~ ➜  
```

What is the network interface configured for cluster connectivity on the controlplane node? node-to-node communication. **the network interface to which this IP: 10.40.203.6 is copnfigured is *eth0*


```bash
controlplane ~ ➜  ip a
11776: eth1@if11777: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:19:00:69 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 172.25.0.105/24 brd 172.25.0.255 scope global eth1
       valid_lft forever preferred_lft forever
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
    link/ether 6e:a8:5c:d1:8a:d2 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
3: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether e2:5f:8b:a1:f6:1d brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.1/24 brd 10.244.0.255 scope global cni0
       valid_lft forever preferred_lft forever
4: vethde1d2eb2@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP group default 
    link/ether ca:fd:cc:50:df:44 brd ff:ff:ff:ff:ff:ff link-netns cni-72116b99-4623-f641-934c-932b5d097ee6
5: veth22ecb4aa@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP group default 
    link/ether b2:55:1c:dd:2f:55 brd ff:ff:ff:ff:ff:ff link-netns cni-eb4da93c-5d18-db41-2047-cab19fccf779
11774: eth0@if11775: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default 
    link/ether 02:42:0a:28:cb:06 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.40.203.6/24 brd 10.40.203.255 scope global eth0
       valid_lft forever preferred_lft forever

controlplane ~ ➜  
```


What is the MAC address assigned to node01? **02:42:0a:28:cb:09**

```bash
controlplane ~ ➜  ssh node01

root@node01 ~ ➜  ip link show eth0
8850: eth0@if8851: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:0a:28:cb:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0

root@node01 ~ ➜  
```

We use Containerd as our container runtime. What is the interface/bridge created by Containerd on this host? **cni0**

Default gateway for node01

```bash
root@node01 ~ ➜  cat /etc/resolv.conf 
nameserver 172.25.0.1
options ndots:0

root@node01 ~ ➜  
```

What is the port the kube-scheduler is listening on in the controlplane node? **10259**

```bash
controlplane ~ ➜  netstat -nplt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      2756/kube-scheduler 
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      753/systemd-resolve 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1106/sshd: /usr/sbi 
tcp        0      0 127.0.0.1:44643         0.0.0.0:*               LISTEN      1097/containerd     
tcp        0      0 127.0.0.11:43267        0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      3797/kubelet        
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      4289/kube-proxy     
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      2755/etcd           
tcp        0      0 10.40.203.6:2379        0.0.0.0:*               LISTEN      2755/etcd           
tcp        0      0 10.40.203.6:2380        0.0.0.0:*               LISTEN      2755/etcd           
tcp        0      0 127.0.0.1:2381          0.0.0.0:*               LISTEN      2755/etcd           
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      1096/ttyd           
tcp        0      0 127.0.0.1:10257         0.0.0.0:*               LISTEN      2770/kube-controlle 
tcp6       0      0 :::22                   :::*                    LISTEN      1106/sshd: /usr/sbi 
tcp6       0      0 :::8888                 :::*                    LISTEN      4020/kubectl        
tcp6       0      0 :::10250                :::*                    LISTEN      3797/kubelet        
tcp6       0      0 :::6443                 :::*                    LISTEN      2759/kube-apiserver 
tcp6       0      0 :::10256                :::*                    LISTEN      4289/kube-proxy     

controlplane ~ ➜  
```

Note: ETCD port 2379 is the port of ETCD to which all control plane components connect to. 2380 is only for etcd peer-to-peer connectivity.

***The End***