Install the kubeadm and kubelet packages on the controlplane and node01.

Use the exact version of 1.26.0-00 for both.

Check the OS version

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

Install the kubeadm and kubelet packages on the controlplane and node01.

Use the exact version of 1.26.0-00 for both.

List available kubeadm versions

```bash
controlplane ~ ➜  apt list -a kubeadm
Listing... Done
kubeadm/kubernetes-xenial 1.26.1-00 amd64 [residual-config]
kubeadm/kubernetes-xenial,now 1.26.0-00 amd64 [residual-config]
kubeadm/kubernetes-xenial 1.25.6-00 amd64 [residual-config]
kubeadm/kubernetes-xenial 1.25.5-00 amd64 [residual-config]
```

Install some prerequisite packages, certificas as well as updates on all the nodes


```bash
controlplane ~ ➜  sudo apt-get update

controlplane ~ ➜  sudo apt-get install -y apt-transport-https ca-certificates curl
Reading package lists... Done

controlplane ~ ➜  sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

controlplane ~ ➜ echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.listdeb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main

controlplane ~ ➜  sudo apt-get update
```

Install the specified versions of kubeadm and kubelet on both nodes


```bash
controlplane ~ ➜  sudo apt-get install -y kubeadm=1.26.0-00 kubelet=1.26.0-00
```

Mark the packages soo that they are cannot be removed, purged, or upgraded unless the hold mark is removed

```bash
controlplane ~ ➜  sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
```
set net.bridge.bridge-nf-call-iptables to 1

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

What is the version of kubelet installed?

```bash
controlplane ~ ➜  kubelet --version
Kubernetes v1.26.0

controlplane ~ ➜  
```

Initialize Control Plane Node (Master Node). Use the following options:


apiserver-advertise-address - Use the IP address allocated to eth0 on the controlplane node

apiserver-cert-extra-sans - Set it to controlplane

pod-network-cidr - Set to 10.244.0.0/16
Once done, set up the default kubeconfig file and wait for node to be part of the cluster.

First check the interfaces configuration

```bash
controlplane ~ ➜  ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default 
    link/ether 06:9d:42:b8:1d:13 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
3: cni0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether be:5a:ad:e9:56:93 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.1/24 brd 10.244.0.255 scope global cni0
       valid_lft forever preferred_lft forever
646: eth0@if647: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default 
    link/ether 02:42:c0:09:4b:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.9.75.9/24 brd 192.9.75.255 scope global eth0
       valid_lft forever preferred_lft forever
648: eth1@if649: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:19:00:51 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 172.25.0.81/24 brd 172.25.0.255 scope global eth1
       valid_lft forever preferred_lft forever

controlplane ~ ➜ 
```

initialize the control-plane node

```bash
controlplane ~ ➜  kubeadm init --apiserver-cert-extra-sans=controlplane --apiserver-advertise-address 192.9.75.9 --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.26.1
.
.
.
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubeadm join 192.9.75.9:6443 --token y3ew8g.e05pv2l8vnzmqzc6 \
        --discovery-token-ca-cert-hash sha256:027b774d7452fe1d094f70592146b919a028e1c57d4e00bd5975cb0717b4110b
```

Run the suggested commands in the output above

```bash
controlplane ~ ➜  mkdir -p $HOME/.kube

controlplane ~ ➜  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

controlplane ~ ➜  sudo chown $(id -u):$(id -g) $HOME/.kube/config

controlplane ~ ➜  
```

To join node01 to the cluster, First check the ipaddress of node01

```bash
controlplane ~ ➜  ssh node01 ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 192.9.75.12  netmask 255.255.255.0  broadcast 192.9.75.255
        ether 02:42:c0:09:4b:0c  txqueuelen 0  (Ethernet)
        RX packets 2645  bytes 1016068 (1.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2696  bytes 359464 (359.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


controlplane ~ ➜  
```

Generate a kubeadm join token

```bash
controlplane ~ ➜  kubeadm token create --print-join-command
kubeadm join 192.9.75.9:6443 --token fk3n1b.fti2qsi5s5soqd7q --discovery-token-ca-cert-hash sha256:027b774d7452fe1d094f70592146b919a028e1c57d4e00bd5975cb0717b4110b 

controlplane ~ ➜  
```


Then ssh into node 01 and join it to the cluster

```bash
root@node01 ~ ➜  kubeadm join 192.9.75.9:6443 --token fk3n1b.fti2qsi5s5soqd7q --discovery-token-ca-cert-hash sha256:027b774d7452fe1d094f70592146b919a028e1c57d4e00bd5975cb0717b4110b 
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

Install a Network Plugin. As a default, we will go with flannel

Refer to the official documentation for the procedure.

```bash
controlplane ~ ➜  kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.ymlnamespace/kube-flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created

controlplane ~ ➜  
```

Get nodes to see all joined nodes

```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   25m   v1.26.0
node01         Ready    <none>          10m   v1.26.0

controlplane ~ ➜  
```