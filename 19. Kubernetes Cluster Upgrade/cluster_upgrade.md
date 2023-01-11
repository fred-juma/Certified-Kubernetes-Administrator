Kubernetes Cluster Upgrade Process

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

Pre-upgrade checks to see all versions of the components on the cluster

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

kubeadm version

kubeadm upgrade plan

Note: kubeadm does not upgrade the kubelet

To verify current versions of the kubelet running on the nodes
kubectl get nodes

kubeadm must be upgraded first b4 the kubernetes cluster

apt-get upgrade -y kubeadm=1.12.0-0

upgrade the cluster

kubeadm upgrade apply v1.12.0

upgrade the kubelet on master node

apt-get upgrade -y kubelet=1.12.0-00
systemctl restart kubelet

upgrade the worker nodes

move workloads from the first node to schedule them on other nodes as well as cordon the node

kubectl drain node-1 --ignore-daemonsets //run on controlplane

then upgrade the kubeadm tool on the worker node

apt-get upgrade -y kubeadm=1.12.0-0
apt-get upgrade -y kubelet=1.12.0-00
kubeadm upgrade node config --kubelet-version v1.12.0
systemctl daemon-reload
systemctl restart kubelet

the node should be up with the new version

uncordon the node so that it is schedulable

kubectl uncordon node-1 //run on controlplane

To show token used to set up the cluster 

kubeadm token list

To see all pods in all the namespaces

kubectl get pods -A



We have a production cluster with applications running on it. Let us explore the setup first.


What is the current version of the cluster?

