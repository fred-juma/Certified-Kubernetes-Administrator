Create a new service account with the name pvviewer. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding.
Next, create a pod called pvviewer with the image: redis and serviceAccount: pvviewer in the default namespace.

controlplane ~ ➜  kubectl create serviceaccount pvviewer
serviceaccount/pvviewer created

controlplane ~ ➜  kubectl get serviceaccount
NAME       SECRETS   AGE
default    0         26m
pvviewer   0         5m29s

controlplane ~ ➜  

controlplane ~ ➜  kubectl create serviceaccount pvviewer
serviceaccount/pvviewer created


controlplane ~ ✖ kubectl create clusterrolebinding pvviewer-role-binding  --clusterrole=pvviewer-role --user=pvviewer
clusterrolebinding.rbac.authorization.k8s.io/pvviewer-role-binding created

controlplane ~ ➜  kubectl describe clusterrolebinding pvviewer-role-binding
Name:         pvviewer-role-binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  pvviewer-role
Subjects:
  Kind  Name      Namespace
  ----  ----      ---------
  User  pvviewer


controlplane ~ ➜  kubectl run pvviewer --image=redis -o yaml --dry-run=client > pvviewer.yaml 

controlplane ~ ➜  vi pvviewer.yaml 

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pvviewer
  name: pvviewer
spec:
  serviceAccountName: pvviewer
  containers:
  - image: redis
    name: pvviewer
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


controlplane ~ ➜  kubectl get pod
NAME       READY   STATUS    RESTARTS   AGE
pvviewer   1/1     Running   0          8s

controlplane ~ ➜  


controlplane ~ ➜  kubectl auth can-i list PersistentVolumes --as=pvviewer
Warning: resource 'persistentvolumes' is not namespace scoped

yes

controlplane ~ ➜ 



Create a new service account with the name pvviewer. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding.
Next, create a pod called pvviewer with the image: redis and serviceAccount: pvviewer in the default namespace.





controlplane ~ ➜  kubectl get node -o json
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "v1",
            "kind": "Node",
            "metadata": {
                "annotations": {
                    "kubeadm.alpha.kubernetes.io/cri-socket": "unix:///var/run/containerd/containerd.sock",
                    "node.alpha.kubernetes.io/ttl": "0",
                    "volumes.kubernetes.io/controller-managed-attach-detach": "true"
                },
                "creationTimestamp": "2023-03-13T11:29:59Z",
                "labels": {
                    "beta.kubernetes.io/arch": "amd64",
                    "beta.kubernetes.io/os": "linux",
                    "kubernetes.io/arch": "amd64",
                    "kubernetes.io/hostname": "controlplane",
                    "kubernetes.io/os": "linux",
                    "node-role.kubernetes.io/control-plane": "",
                    "node.kubernetes.io/exclude-from-external-load-balancers": ""
                },
                "name": "controlplane",
                "resourceVersion": "2663",
                "uid": "7c84f0de-251f-4e66-b2b6-1bc843688356"
            },
            "spec": {
                "podCIDR": "10.244.0.0/24",
                "podCIDRs": [
                    "10.244.0.0/24"
                ]
            },
            "status": {
                "addresses": [
                    {
                        "address": "192.19.82.9",
                        "type": "InternalIP"
                    },
                    {
                        "address": "controlplane",
                        "type": "Hostname"
                    }
                ],
                "allocatable": {
                    "cpu": "36",
                    "ephemeral-storage": "936398358207",
                    "hugepages-1Gi": "0",
                    "hugepages-2Mi": "0",
                    "memory": "214484652Ki",
                    "pods": "110"
                },
                "capacity": {
                    "cpu": "36",
                    "ephemeral-storage": "1016057248Ki",
                    "hugepages-1Gi": "0",
                    "hugepages-2Mi": "0",
                    "memory": "214587052Ki",
                    "pods": "110"
                },
                "conditions": [
                    {
                        "lastHeartbeatTime": "2023-03-13T11:30:32Z",
                        "lastTransitionTime": "2023-03-13T11:30:32Z",
                        "message": "Weave pod has set this",
                        "reason": "WeaveIsUp",
                        "status": "False",
                        "type": "NetworkUnavailable"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-13T11:55:51Z",
                        "lastTransitionTime": "2023-03-13T11:29:54Z",
                        "message": "kubelet has sufficient memory available",
                        "reason": "KubeletHasSufficientMemory",
                        "status": "False",
                        "type": "MemoryPressure"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-13T11:55:51Z",
                        "lastTransitionTime": "2023-03-13T11:29:54Z",
                        "message": "kubelet has no disk pressure",
                        "reason": "KubeletHasNoDiskPressure",
                        "status": "False",
                        "type": "DiskPressure"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-13T11:55:51Z",
                        "lastTransitionTime": "2023-03-13T11:29:54Z",
                        "message": "kubelet has sufficient PID available",
                        "reason": "KubeletHasSufficientPID",
                        "status": "False",
                        "type": "PIDPressure"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-13T11:55:51Z",
                        "lastTransitionTime": "2023-03-13T11:30:22Z",
                        "message": "kubelet is posting ready status",
                        "reason": "KubeletReady",
                        "status": "True",
                        "type": "Ready"
                    }
                ],
                "daemonEndpoints": {
                    "kubeletEndpoint": {
                        "Port": 10250
                    }
                },
                "images": [
                    {
                        "names": [
                            "docker.io/kodekloud/fluent-ui-running@sha256:78fd68ba8a79adcd3e58897a933492886200be513076ba37f843008cc0168f81",
                            "docker.io/kodekloud/fluent-ui-running:latest"
                        ],
                        "sizeBytes": 389734636
                    },
                    {
                        "names": [
                            "registry.k8s.io/etcd@sha256:dd75ec974b0a2a6f6bb47001ba09207976e625db898d1b16735528c009cb171c",
                            "registry.k8s.io/etcd:3.5.6-0"
                        ],
                        "sizeBytes": 102542580
                    },
                    {
                        "names": [
                            "docker.io/library/nginx@sha256:6650513efd1d27c1f8a5351cbd33edf85cc7e0d9d0fcb4ffb23d8fa89b601ba8",
                            "docker.io/library/nginx:latest"
                        ],
                        "sizeBytes": 56897816
                    },
                    {
                        "names": [
                            "registry.k8s.io/kube-apiserver@sha256:d230a0b88a3daf14e4cce03b906b992c8153f37da878677f434b1af8c4e8cc75",
                            "registry.k8s.io/kube-apiserver:v1.26.0"
                        ],
                        "sizeBytes": 35317868
                    },
                    {
                        "names": [
                            "registry.k8s.io/kube-controller-manager@sha256:26e260b50ec46bd1da7352565cb8b34b6dd2cb006cebbd2f35170d50935fb9ec",
                            "registry.k8s.io/kube-controller-manager:v1.26.0"
                        ],
                        "sizeBytes": 32244989
                    },
                    {
                        "names": [
                            "docker.io/weaveworks/weave-kube@sha256:d797338e7beb17222e10757b71400d8471bdbd9be13b5da38ce2ebf597fb4e63",
                            "docker.io/weaveworks/weave-kube:2.8.1"
                        ],
                        "sizeBytes": 30924173
                    },
                    {
                        "names": [
                            "registry.k8s.io/kube-proxy@sha256:1e9bbe429e4e2b2ad32681c91deb98a334f1bf4135137df5f84f9d03689060fe",
                            "registry.k8s.io/kube-proxy:v1.26.0"
                        ],
                        "sizeBytes": 21536465
                    },
                    {
                        "names": [
                            "quay.io/coreos/flannel@sha256:51223d328b2f85d8fe9ad35db82d564b45b03fd1002728efcf14011aff02de78",
                            "quay.io/coreos/flannel:v0.13.1-rc1"
                        ],
                        "sizeBytes": 20688989
                    },
                    {
                        "names": [
                            "registry.k8s.io/kube-scheduler@sha256:34a142549f94312b41d4a6cd98e7fddabff484767a199333acb7503bf46d7410",
                            "registry.k8s.io/kube-scheduler:v1.26.0"
                        ],
                        "sizeBytes": 17484038
                    },
                    {
                        "names": [
                            "quay.io/coreos/flannel@sha256:6d451d92c921f14bfb38196aacb6e506d4593c5b3c9d40a8b8a2506010dc3e10",
                            "quay.io/coreos/flannel:v0.12.0-amd64"
                        ],
                        "sizeBytes": 17124093
                    },
                    {
                        "names": [
                            "docker.io/library/nginx@sha256:6f94b7f4208b5d5391246c83a96246ca204f15eaf7e636cefda4e6348c8f6101",
                            "docker.io/library/nginx:alpine"
                        ],
                        "sizeBytes": 16684700
                    },
                    {
                        "names": [
                            "registry.k8s.io/coredns/coredns@sha256:8e352a029d304ca7431c6507b56800636c321cb52289686a581ab70aaa8a2e2a",
                            "registry.k8s.io/coredns/coredns:v1.9.3"
                        ],
                        "sizeBytes": 14837849
                    },
                    {
                        "names": [
                            "docker.io/weaveworks/weave-npc@sha256:38d3e30a97a2260558f8deb0fc4c079442f7347f27c86660dbfc8ca91674f14c",
                            "docker.io/weaveworks/weave-npc:2.8.1"
                        ],
                        "sizeBytes": 12814131
                    },
                    {
                        "names": [
                            "docker.io/library/busybox@sha256:7b3ccabffc97de872a30dfd234fd972a66d247c8cfc69b0550f276481852627c",
                            "docker.io/library/busybox:latest"
                        ],
                        "sizeBytes": 2597143
                    },
                    {
                        "names": [
                            "registry.k8s.io/pause@sha256:7031c1b283388d2c2e09b57badb803c05ebed362dc88d84b480cc47f72a21097",
                            "registry.k8s.io/pause:3.9"
                        ],
                        "sizeBytes": 321520
                    },
                    {
                        "names": [
                            "k8s.gcr.io/pause@sha256:3d380ca8864549e74af4b29c10f9cb0956236dfb01c40ca076fb6c37253234db",
                            "k8s.gcr.io/pause:3.6"
                        ],
                        "sizeBytes": 301773
                    }
                ],
                "nodeInfo": {
                    "architecture": "amd64",
                    "bootID": "eba1c726-ffcf-4e20-85b7-053a6d961080",
                    "containerRuntimeVersion": "containerd://1.6.6",
                    "kernelVersion": "5.4.0-1101-gcp",
                    "kubeProxyVersion": "v1.26.0",
                    "kubeletVersion": "v1.26.0",
                    "machineID": "1e8ea8fad2a94becb6a9111994cc64cb",
                    "operatingSystem": "linux",
                    "osImage": "Ubuntu 20.04.5 LTS",
                    "systemUUID": "1c118f50-8f55-130d-4ecb-5748520247f9"
                }
            }
        },
        {
            "apiVersion": "v1",
            "kind": "Node",
            "metadata": {
                "annotations": {
                    "kubeadm.alpha.kubernetes.io/cri-socket": "unix:///var/run/containerd/containerd.sock",
                    "node.alpha.kubernetes.io/ttl": "0",
                    "volumes.kubernetes.io/controller-managed-attach-detach": "true"
                },
                "creationTimestamp": "2023-03-13T11:30:41Z",
                "labels": {
                    "beta.kubernetes.io/arch": "amd64",
                    "beta.kubernetes.io/os": "linux",
                    "kubernetes.io/arch": "amd64",
                    "kubernetes.io/hostname": "node01",
                    "kubernetes.io/os": "linux"
                },
                "name": "node01",
                "resourceVersion": "2681",
                "uid": "c503a216-7063-439b-a1b0-fe8c1a7bad4c"
            },
            "spec": {
                "podCIDR": "10.244.1.0/24",
                "podCIDRs": [
                    "10.244.1.0/24"
                ]
            },
            "status": {
                "addresses": [
                    {
                        "address": "192.19.82.12",
                        "type": "InternalIP"
                    },
                    {
                        "address": "node01",
                        "type": "Hostname"
                    }
                ],
                "allocatable": {
                    "cpu": "36",
                    "ephemeral-storage": "936398358207",
                    "hugepages-1Gi": "0",
                    "hugepages-2Mi": "0",
                    "memory": "214484648Ki",
                    "pods": "110"
                },
                "capacity": {
                    "cpu": "36",
                    "ephemeral-storage": "1016057248Ki",
                    "hugepages-1Gi": "0",
                    "hugepages-2Mi": "0",
                    "memory": "214587048Ki",
                    "pods": "110"
                },
                "conditions": [
                    {
                        "lastHeartbeatTime": "2023-03-13T11:31:04Z",
                        "lastTransitionTime": "2023-03-13T11:31:04Z",
                        "message": "Weave pod has set this",
                        "reason": "WeaveIsUp",
                        "status": "False",
                        "type": "NetworkUnavailable"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-13T11:56:02Z",
                        "lastTransitionTime": "2023-03-13T11:30:40Z",
                        "message": "kubelet has sufficient memory available",
                        "reason": "KubeletHasSufficientMemory",
                        "status": "False",
                        "type": "MemoryPressure"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-13T11:56:02Z",
                        "lastTransitionTime": "2023-03-13T11:30:40Z",
                        "message": "kubelet has no disk pressure",
                        "reason": "KubeletHasNoDiskPressure",
                        "status": "False",
                        "type": "DiskPressure"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-13T11:56:02Z",
                        "lastTransitionTime": "2023-03-13T11:30:40Z",
                        "message": "kubelet has sufficient PID available",
                        "reason": "KubeletHasSufficientPID",
                        "status": "False",
                        "type": "PIDPressure"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-13T11:56:02Z",
                        "lastTransitionTime": "2023-03-13T11:31:00Z",
                        "message": "kubelet is posting ready status",
                        "reason": "KubeletReady",
                        "status": "True",
                        "type": "Ready"
                    }
                ],
                "daemonEndpoints": {
                    "kubeletEndpoint": {
                        "Port": 10250
                    }
                },
                "images": [
                    {
                        "names": [
                            "docker.io/kodekloud/fluent-ui-running@sha256:78fd68ba8a79adcd3e58897a933492886200be513076ba37f843008cc0168f81",
                            "docker.io/kodekloud/fluent-ui-running:latest"
                        ],
                        "sizeBytes": 389734636
                    },
                    {
                        "names": [
                            "registry.k8s.io/etcd@sha256:dd75ec974b0a2a6f6bb47001ba09207976e625db898d1b16735528c009cb171c",
                            "registry.k8s.io/etcd:3.5.6-0"
                        ],
                        "sizeBytes": 102542580
                    },
                    {
                        "names": [
                            "docker.io/library/nginx@sha256:0047b729188a15da49380d9506d65959cce6d40291ccfb4e039f5dc7efd33286",
                            "docker.io/library/nginx:latest"
                        ],
                        "sizeBytes": 56882284
                    },
                    {
                        "names": [
                            "docker.io/library/redis@sha256:e50c7e23f79ae81351beacb20e004720d4bed657415e68c2b1a2b5557c075ce0",
                            "docker.io/library/redis:latest"
                        ],
                        "sizeBytes": 42467382
                    },
                    {
                        "names": [
                            "registry.k8s.io/kube-apiserver@sha256:d230a0b88a3daf14e4cce03b906b992c8153f37da878677f434b1af8c4e8cc75",
                            "registry.k8s.io/kube-apiserver:v1.26.0"
                        ],
                        "sizeBytes": 35317868
                    },
                    {
                        "names": [
                            "registry.k8s.io/kube-controller-manager@sha256:26e260b50ec46bd1da7352565cb8b34b6dd2cb006cebbd2f35170d50935fb9ec",
                            "registry.k8s.io/kube-controller-manager:v1.26.0"
                        ],
                        "sizeBytes": 32244989
                    },
                    {
                        "names": [
                            "docker.io/weaveworks/weave-kube@sha256:d797338e7beb17222e10757b71400d8471bdbd9be13b5da38ce2ebf597fb4e63",
                            "docker.io/weaveworks/weave-kube:2.8.1"
                        ],
                        "sizeBytes": 30924173
                    },
                    {
                        "names": [
                            "registry.k8s.io/kube-proxy@sha256:1e9bbe429e4e2b2ad32681c91deb98a334f1bf4135137df5f84f9d03689060fe",
                            "registry.k8s.io/kube-proxy:v1.26.0"
                        ],
                        "sizeBytes": 21536465
                    },
                    {
                        "names": [
                            "quay.io/coreos/flannel@sha256:51223d328b2f85d8fe9ad35db82d564b45b03fd1002728efcf14011aff02de78",
                            "quay.io/coreos/flannel:v0.13.1-rc1"
                        ],
                        "sizeBytes": 20688989
                    },
                    {
                        "names": [
                            "registry.k8s.io/kube-scheduler@sha256:34a142549f94312b41d4a6cd98e7fddabff484767a199333acb7503bf46d7410",
                            "registry.k8s.io/kube-scheduler:v1.26.0"
                        ],
                        "sizeBytes": 17484038
                    },
                    {
                        "names": [
                            "quay.io/coreos/flannel@sha256:6d451d92c921f14bfb38196aacb6e506d4593c5b3c9d40a8b8a2506010dc3e10",
                            "quay.io/coreos/flannel:v0.12.0-amd64"
                        ],
                        "sizeBytes": 17124093
                    },
                    {
                        "names": [
                            "docker.io/library/nginx@sha256:dd8a054d7ef030e94a6449783605d6c306c1f69c10c2fa06b66a030e0d1db793",
                            "docker.io/library/nginx:alpine"
                        ],
                        "sizeBytes": 16678454
                    },
                    {
                        "names": [
                            "registry.k8s.io/coredns/coredns@sha256:8e352a029d304ca7431c6507b56800636c321cb52289686a581ab70aaa8a2e2a",
                            "registry.k8s.io/coredns/coredns:v1.9.3"
                        ],
                        "sizeBytes": 14837849
                    },
                    {
                        "names": [
                            "docker.io/weaveworks/weave-npc@sha256:38d3e30a97a2260558f8deb0fc4c079442f7347f27c86660dbfc8ca91674f14c",
                            "docker.io/weaveworks/weave-npc:2.8.1"
                        ],
                        "sizeBytes": 12814131
                    },
                    {
                        "names": [
                            "docker.io/library/busybox@sha256:05a79c7279f71f86a2a0d05eb72fcb56ea36139150f0a75cd87e80a4272e4e39",
                            "docker.io/library/busybox:latest"
                        ],
                        "sizeBytes": 2592108
                    },
                    {
                        "names": [
                            "registry.k8s.io/pause@sha256:7031c1b283388d2c2e09b57badb803c05ebed362dc88d84b480cc47f72a21097",
                            "registry.k8s.io/pause:3.9"
                        ],
                        "sizeBytes": 321520
                    },
                    {
                        "names": [
                            "k8s.gcr.io/pause@sha256:3d380ca8864549e74af4b29c10f9cb0956236dfb01c40ca076fb6c37253234db",
                            "k8s.gcr.io/pause:3.6"
                        ],
                        "sizeBytes": 301773
                    }
                ],
                "nodeInfo": {
                    "architecture": "amd64",
                    "bootID": "26a93259-4229-4090-9edf-5d03b363c76b",
                    "containerRuntimeVersion": "containerd://1.6.6",
                    "kernelVersion": "5.4.0-1101-gcp",
                    "kubeProxyVersion": "v1.26.0",
                    "kubeletVersion": "v1.26.0",
                    "machineID": "3107027b90944bf39a83cff0cb1006fe",
                    "operatingSystem": "linux",
                    "osImage": "Ubuntu 20.04.5 LTS",
                    "systemUUID": "0c798a42-137e-b810-54a0-d87766123685"
                }
            }
        }
    ],
    "kind": "List",
    "metadata": {
        "resourceVersion": ""
    }
}

controlplane ~ ➜ 



controlplane ~ ➜  kubectl get node -o=jsonpath="{.items[0].status.addresses[0].address}{'\t'}{.items[1].status.addresses[0].address}"
192.19.82.9     192.19.82.12
controlplane ~ ➜  



controlplane ~ ➜  kubectl get node -o=jsonpath="{.items[0].status.addresses[0].address}{'\t'}{.items[1].status.addresses[0].address}" > /root/CKA/node_ips

controlplane ~ ➜  cat /root/CKA/node_ips
192.19.82.9     192.19.82.12
controlplane ~ ➜  

controlplane ~ ➜  kubectl get node -o=jsonpath='{.items[*].status.addresses[0].address}'
192.12.80.3 192.12.80.6
controlplane ~ ➜  kubectl get node -o=jsonpath='{.items[*].status.addresses[0].address}' > /root/CKA/node_ips

controlplane ~ ➜  cat /root/CKA/node_ips
192.12.80.3 192.12.80.6
controlplane ~ ➜  




Create a pod called multi-pod with two containers.
Container 1, name: alpha, image: nginx
Container 2: name: beta, image: busybox, command: sleep 4800

Environment Variables:
container 1:
name: alpha

Container 2:
name: beta


controlplane ~ ➜  kubectl run multi-pod --image=nginx --env=name=alpha -o yaml --dry-run=client > multi-pod.yaml

controlplane ~ ➜  vi multi-pod.yaml 

controlplane ~ ➜  


apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-pod
  name: multi-pod
spec:
  containers:
  - env:
    - name: name
      value: beta
    image: busybox
    command: ["sleep","4800"]
    name: beta
  - env:
    - name: name
      value: alpha
    image: nginx
    name: alpha
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


controlplane ~ ➜  kubectl create -f multi-pod.yaml 
pod/multi-pod created

controlplane ~ ➜  




controlplane ~ ➜  kubectl get pod
NAME        READY   STATUS    RESTARTS   AGE
multi-pod   2/2     Running   0          10s
pvviewer    1/1     Running   0          16m

controlplane ~ ➜  kubectl run non-root-pod --image=redis:alpine -o yaml --dry-run=client > non-root-pod.yaml

controlplane ~ ➜  vi non-root-pod.yaml 


apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: non-root-pod
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - image: redis:alpine
    name: non-root-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}



controlplane ~ ➜  kubectl create -f non-root-pod.yaml 
pod/non-root-pod created

controlplane ~ ➜  


controlplane ~ ➜  kubectl get pod
NAME           READY   STATUS    RESTARTS   AGE
multi-pod      2/2     Running   0          3m42s
non-root-pod   1/1     Running   0          19s
pvviewer       1/1     Running   0          20m

controlplane ~ ➜  


controlplane ~ ➜  kubectl get pod,svc
NAME               READY   STATUS    RESTARTS   AGE
pod/multi-pod      2/2     Running   0          5m25s
pod/non-root-pod   1/1     Running   0          2m2s
pod/np-test-1      1/1     Running   0          91s
pod/pvviewer       1/1     Running   0          22m

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP   47m
service/np-test-service   ClusterIP   10.104.22.162   <none>        80/TCP    91s

controlplane ~ ➜  


controlplane ~ ➜  kubectl describe pod np-test-1
Name:             np-test-1
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/192.19.82.12
Start Time:       Mon, 13 Mar 2023 08:16:18 -0400
Labels:           run=np-test-1
Annotations:      <none>
Status:           Running
IP:               10.244.192.4
IPs:
  IP:  10.244.192.4
Containers:
  np-test-1:
    Container ID:   containerd://5d3cc7e4cde0103911b540c3497cf32c0bbca661d0cc48fb73ba1edb950bca21
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:aa0afebbb3cfa473099a62c4b32e9b3fb73ed23f2a75a65ce1d4b4f55a5c2ef2



controlplane ~ ➜  kubectl describe service np-test-service
Name:              np-test-service
Namespace:         default
Labels:            run=np-test-1
Annotations:       <none>
Selector:          run=np-test-1
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.104.22.162
IPs:               10.104.22.162
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.192.4:80
Session Affinity:  None
Events:            <none>



controlplane ~ ➜  kubectl describe service np-test-service
Name:              np-test-service
Namespace:         default
Labels:            run=np-test-1
Annotations:       <none>
Selector:          run=np-test-1
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.109.2.120
IPs:               10.109.2.120
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.192.4:80
Session Affinity:  None
Events:            <none>

controlplane ~ ➜  


apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
    - Ingress
  ingress:
    - 
      ports:
        - protocol: TCP
          port: 80


controlplane ~ ➜  kubectl apply -f ingress-to-nptest.yaml 
networkpolicy.networking.k8s.io/ingress-to-nptest created

controlplane ~ ➜  

controlplane ~ ➜  kubectl exec np-test-1 -it -- /bin/bash


root@np-test-1:/# curl http://10.104.22.162:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@np-test-1:/#



controlplane ~ ➜  kubectl taint nodes node01 env_type=production:NoSchedule
node/node01 tainted

controlplane ~ ➜  


controlplane ~ ➜  kubectl run dev-redis --image=redis:alpine
pod/dev-redis created

controlplane ~ ➜  kubectl get pod -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
dev-redis      1/1     Running   0          20s   10.244.0.4     controlplane   <none>           <none>
multi-pod      2/2     Running   0          25m   10.244.192.2   node01         <none>           <none>
non-root-pod   1/1     Running   0          22m   10.244.192.3   node01         <none>           <none>
np-test-1      1/1     Running   0          21m   10.244.192.4   node01         <none>           <none>
pvviewer       1/1     Running   0          42m   10.244.192.1   node01         <none>           <none>

controlplane ~ ➜  


controlplane ~ ➜  kubectl run prod-redis --image=redis:alpine -o yaml --dry-run=client > prod-redis.yaml

controlplane ~ ➜  vi prod-redis.yaml 

controlplane ~ ➜  kubectl create -f prod-redis.yaml 
pod/prod-redis created

controlplane ~ ➜  


apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: prod-redis
  name: prod-redis
spec:
  containers:
  - image: redis:alpine
    name: prod-redis
  tolerations:
  - key: "env_type"
    operator: "Equal"
    value: "production"
    effect: "NoSchedule"
~                                                                                                                                 
~    


controlplane ~ ➜  kubectl get pod -o wide                         NAME           READY   STATUS    RESTARTS   AGE     IP             NODE           NOMINATED NODE   READINESS GATES
dev-redis      1/1     Running   0          4m30s   10.244.0.4     controlplane   <none>           <none>
multi-pod      2/2     Running   0          15m     10.244.192.2   node01         <none>           <none>
non-root-pod   1/1     Running   0          12m     10.244.192.3   node01         <none>           <none>
np-test-1      1/1     Running   0          12m     10.244.192.4   node01         <none>           <none>
prod-redis     1/1     Running   0          8s      10.244.192.5   node01         <none>           <none>
pvviewer       1/1     Running   0          24m     10.244.192.1   node01         <none>           <none>

controlplane ~ ➜  

           







controlplane ~ ➜  kubectl run hr-pod --image=redis:alpine --namespace=hr --env=environment=production --env=tier=frontend
pod/hr-pod created

controlplane ~ ➜  kubectl get pod -n hr
NAME     READY   STATUS    RESTARTS   AGE
hr-pod   1/1     Running   0          7s

controlplane ~ ➜  


