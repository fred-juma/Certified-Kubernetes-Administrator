#### K8s cluster design considerations:

1. Purpose
- Education
- Development and testing
- Hosting production applications etc.

2. Cloud or on Prem
- kubeadm for on-prem
- GKE for GCP
- Kops for AWS
- AKS for Azure

3. Workloads:
- How many
- What kind: e.g. web, big data analytics etc.
- Application resource requirements: CPU, Memory, Storage
- Traffic: Heavy traffice, burst trafficc, etc.

#### Purpose
Education:
- Minikube
- Single node cluster with kubeadm/GCP/AWS/AKS

Deveolpment & Testing
- Multi-node cluster with single master and multiple worker nodes
- Kubeadm tool
- Qucik provision with GCP/AWS/AKS

Hosting production applications
- High Availability Multi-node cluster with multiple master nodes
- Kubeadm/GCP/kops on AWS or other supported platform
- up to 5,000 nodes
- upto 150,000 pods
- upto 300,000 containers
- upto 100 pods per node

#### Storage
- High perfomance - SSD backed storage
- Multiple concurrent connections - Network based storage
- Persistent shared volumes - For shared access across multiple pods
- Label nodes with specific disk types
- Use node selectors to assign applications to nodes with specific disk types


#### Nodes
- Virtual or physical machines
- Minimum 4 node cluster (size based on workload)
- Master vs Worker nodes
- Linux x86_64 Architecture
- All control plane components (etcd, API Server, Controller Manager, Kube Scheduler) deployed on master node

#### Choosing kubernetes infrastructure

Local Machine
- Minikube
- Kubeadm 

Turnkey Solutions
- OpenShift
- Cloud Foundry Container Rutime
- VMware Cloud PKS
- Vagrant

Hosted Solutions
- Google Kubernetes Engine (GKE)
- OpenShift Online
- Azure Kubernetes Service (AKS)
- Amazon Elastic Kubernetes Service (EKS)

#### High Availability

Master HA              |  
:-------------------------:|
![Master HA ](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/master-HA.JPG)

- In a HA cluster you have more than 1 Master nodes. In this discussion we assume 2 node configuration.
- API server is configured in active-active mode. kubectl is configured to access the API servers endpoint (port 6443) via a loadbalancer

API Server HA              |  
:-------------------------:|
![API Server HA  ](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/apiserver-ha.JPG)

- Scheduler and controller manager are configured in active-passive mode
- Controller manager "--leader-elect true" lease duration (15s)is used to elect the active node, the other one becomes passive

```bash
kube-controller-manager --leader-elect true
                        --leader-elect-lease-duration 15s
                        --leader-elect-renew-duration 10s
                        --leader-elect-renew-deadline 10s
                        --leader-elect-retry-period 2s
```                    


- Scheduler follows the same approach and commands

#### ETCD HA options
1. Stacked etcd topology
- Like the above examples, etcd is run within the master nodes

Stacked etcd HA              |  
:-------------------------:|
![Stacked etcd HA  ](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/stacked-etcd.JPG)

2. External etcd Topology
- Etcd is separated from the control plane and run on its own servers

External etcd HA              |  
:-------------------------:|
![External etcd HA  ](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/external-etcd.JPG)

- Whichever topology is used, the API server must be made aware of the etcd server by specifying a list of etcd server endpoints (port 2379) in the api-server configurations *cat /etc/systemd/system/kube-apiserver.service*

- Leader election is implemented by the *RAFT* protocol

Quorum = N/2 + 1. The number of etc nodes that have top be available for a write to be considered complete.

Fault tolerance = No. of all etcd instances - Quorum

- It is preferrable to select odd number of etcd clusters e.g. 3,5,7 etc. With minimum nodes recommended = 3

The peers information is passed in the etcd service configuration:


*--initial-cluster peer-1=https://$(PEER1_IP):2380,peer-2=https://$(PEER2_IP):2380,*

- Etcd utility is used to write and read data on the etcd nodes
