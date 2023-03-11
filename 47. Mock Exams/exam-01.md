Deploy a pod named nginx-pod using the nginx:alpine image.

Once done, click on the Next Question button in the top right corner of this panel. You may navigate back and forth freely between all questions. Once done with all questions, click on End Exam. Your work will be validated at the end and score shown. Good Luck!

ontrolplane ~ ➜  kubectl run nginx-pod --image=nginx:alpine
pod/nginx-pod created

controlplane ~ ➜  kubectl get all
NAME            READY   STATUS    RESTARTS   AGE
pod/nginx-pod   1/1     Running   0          6s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   43m

controlplane ~ ➜  

Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.

controlplane ~ ➜ kubectl run messaging --image=redis:alpine --labels=tier=msg
pod/messaging created

controlplane ~ ➜  


controlplane ~ ➜  kubectl get pod --show-labels
NAME        READY   STATUS    RESTARTS   AGE     LABELS
messaging   1/1     Running   0          2m45s   tier=msg
nginx-pod   1/1     Running   0          5m49s   run=nginx-pod

controlplane ~ ➜  

Create a namespace named apx-x9984574.

controlplane ~ ➜  kubectl create namespace apx-x9984574
namespace/apx-x9984574 created

controlplane ~ ➜  


Get the list of nodes in JSON format and store it in a file at /opt/outputs/nodes-z3444kd9.json


controlplane ~ ➜  kubectl get nodes -o json > /opt/outputs/nodes-z3444kd9.json

controlplane ~ ➜  ls /opt/outputs/
nodes-z3444kd9.json

controlplane ~ ➜  

Create a service messaging-service to expose the messaging application within the cluster on port 6379.

Use imperative commands.



controlplane ~ ➜  kubectl expose pod messaging --name=messaging-service --port=6379
service/messaging-service exposed

controlplane ~ ➜  


controlplane ~ ➜  kubectl get service
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP    58m
messaging-service   ClusterIP   10.96.154.241   <none>        6379/TCP   17s

controlplane ~ ➜  



Create a deployment named hr-web-app using the image kodekloud/webapp-color with 2 replicas.


controlplane ~ ➜  kubectl create deployment hr-web-app --image=kodekloud/webapp-color --replicas=2
deployment.apps/hr-web-app created

controlplane ~ ➜  



controlplane ~ ➜  kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/hr-web-app-5d67dbb499-495mt   1/1     Running   0          12s
pod/hr-web-app-5d67dbb499-qgwfg   1/1     Running   0          12s
pod/messaging                     1/1     Running   0          13m
pod/nginx-pod                     1/1     Running   0          16m

NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP    59m
service/messaging-service   ClusterIP   10.96.154.241   <none>        6379/TCP   107s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hr-web-app   2/2     2            2           12s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/hr-web-app-5d67dbb499   2         2         2       12s

controlplane ~ ➜  



Create a static pod named static-busybox on the controlplane node that uses the busybox image and the command sleep 1000.



controlplane ~ ➜  ps -aux | grep kubelet
root        3278  0.0  0.1 1125288 320736 ?      Ssl  03:54   2:55 kube-apiserver --advertise-address=192.14.124.3 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root        4241  0.0  0.0 3940168 112884 ?      Ssl  03:54   1:34 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9
root       19186  0.0  0.0   6744   656 pts/0    S+   05:01   0:00 grep --color=auto kubelet

controlplane ~ ➜  

--config=/var/lib/kubelet/config.yaml


controlplane ~ ➜  cat /var/lib/kubelet/config.yaml | grep -i staticpodpath
staticPodPath: /etc/kubernetes/manifests

controlplane ~ ➜  


controlplane ~ ➜  kubectl run static-busybox --image=busybox -o yaml --dry-run=client --command -- sleep 1000 > /etc/kubernetes/manifests/static-busyibox.yaml  

controlplane /etc/kubernetes/manifests ➜  ls -l
total 20
-rw------- 1 root root 2382 Mar  9 03:54 etcd.yaml
-rw------- 1 root root 3859 Mar  9 03:54 kube-apiserver.yaml
-rw------- 1 root root 3370 Mar  9 03:54 kube-controller-manager.yaml
-rw------- 1 root root 1440 Mar  9 03:54 kube-scheduler.yaml
-rw-r--r-- 1 root root   27 Mar  9 05:09 static-busybox.yaml

controlplane /etc/kubernetes/manifests ➜  


controlplane /etc/kubernetes/manifests ➜  kubectl apply -f static-busybox.yaml pod/static-busybox created

controlplane /etc/kubernetes/manifests ➜  

controlplane ~ ➜  crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
85c07c2a0320d       5130e5ba7d244       4 minutes ago       Running             temp-bus                  0                   5fdaf597fbbf1       temp-bus
36aa33fb87f9f       bab98d58e29e4       5 minutes ago       Running             static-busybox            0                   7f086d652a783       static-busybox
4b86a29d3bbbd       bab98d58e29e4       5 minutes ago       Running             static-busybox            0                   ae069d4bd544f       static-busybox-controlplane
3c158c0ec665c       32a1ce4c22f21       25 minutes ago      Running             webapp-color              0                   00e1678d335a7       hr-web-app-5d67dbb499-495mt
863680e7a7f2c       32a1ce4c22f21       25 minutes ago      Running             webapp-color              0                   e1778ee36dd10       hr-web-app-5d67dbb499-qgwfg
aacb6417cc9bf       5130e5ba7d244       38 minutes ago      Running             messaging                 0                   c55490dd7a5ba       messaging
9575123223a9c       2bc7edbc3cf2f       42 minutes ago      Running             nginx-pod                 0                   9e0c153ae45c9       nginx-pod
881a9dad6c99e       5185b96f0becf       About an hour ago   Running             coredns                   0                   9516eea9a9def       coredns-787d4945fb-j82tf
a793a304d053f       5185b96f0becf       About an hour ago   Running             coredns                   0                   0da70619d735e       coredns-787d4945fb-k4zqc
58e598e91062d       8b675dda11bb1       About an hour ago   Running             kube-flannel              0                   a790493f23071       kube-flannel-ds-gfx6g
c08d4b464fed1       556768f31eb1d       About an hour ago   Running             kube-proxy                0                   5fe7758786788       kube-proxy-p65jf
c93d8cec42c52       dafd8ad70b156       About an hour ago   Running             kube-scheduler            0                   27d263ea521de       kube-scheduler-controlplane
57947e7cf74e7       a31e1d84401e6       About an hour ago   Running             kube-apiserver            0                   c25b679f1e1f2       kube-apiserver-controlplane
397a3ac3ef958       fce326961ae2d       About an hour ago   Running             etcd                      0                   6b260fa129591       etcd-controlplane
fabc6a060e421       5d7c5dfd3ba18       About an hour ago   Running             kube-controller-manager   0                   4eb9ce47bffa4       kube-controller-manager-controlplane

controlplane ~ ➜  


Create a POD in the finance namespace named temp-bus with the image redis:alpine


controlplane /etc/kubernetes/manifests ➜  kubectl run temp-bus -n finance --image=redis:alpine
pod/temp-bus created

controlplane /etc/kubernetes/manifests ➜  kubectl get pod -n finance
NAME       READY   STATUS    RESTARTS   AGE
temp-bus   1/1     Running   0          14s

controlplane /etc/kubernetes/manifests ➜  



A new application orange is deployed. There is something wrong with it. Identify and fix the issue.


controlplane /etc/kubernetes/manifests ➜  kubectl get pod orange
NAME     READY   STATUS       RESTARTS      AGE
orange   0/1     Init:Error   2 (30s ago)   34s

controlplane /etc/kubernetes/manifests ➜  


controlplane /etc/kubernetes/manifests ➜  kubectl describe pod orange 

Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  64s                default-scheduler  Successfully assigned default/orange to controlplane
  Normal   Pulled     63s                kubelet            Successfully pulled image "busybox" in 355.805705ms (355.823442ms including waiting)
  Normal   Pulled     61s                kubelet            Successfully pulled image "busybox" in 322.645032ms (322.665545ms including waiting)
  Normal   Pulled     45s                kubelet            Successfully pulled image "busybox" in 273.041499ms (273.05591ms including waiting)
  Normal   Pulling    15s (x4 over 63s)  kubelet            Pulling image "busybox"
  Normal   Created    14s (x4 over 63s)  kubelet            Created container init-myservice
  Normal   Started    14s (x4 over 62s)  kubelet            Started container init-myservice
  Normal   Pulled     14s                kubelet            Successfully pulled image "busybox" in 298.97411ms (298.991355ms including waiting)
  Warning  BackOff    13s (x5 over 59s)  kubelet            Back-off restarting failed container init-myservice in pod orange_default(f251b516-148e-4d72-b38e-76c30ff65863)


controlplane ~ ➜  kubectl get pod orange
NAME     READY   STATUS    RESTARTS   AGE
orange   1/1     Running   0          26s

controlplane ~ ➜  



Expose the hr-web-app as service hr-web-app-service application on port 30082 on the nodes on the cluster.

The web application listens on port 8080

controlplane ~ ➜  kubectl expose deployment hr-web-app --name=hr-web-app-service --type=NodePort --port=8080 -o yaml --dry-run=client > hr-web-app-service.yaml

controlplane ~ ➜  

apiVersion: v1
kind: Service
metadata:
  labels:
    app: hr-web-app
  name: hr-web-app-service
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
    nodePort: 30082
  selector:
    app: hr-web-app
  type: NodePort


controlplane ~ ➜  kubectl apply -f hr-web-app-service.yaml 
service/hr-web-app-service created

controlplane ~ ➜  


controlplane ~ ➜  kubectl get service
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hr-web-app-service   NodePort    10.104.110.76   <none>        8080:30082/TCP   12s
kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP          71m
messaging-service    ClusterIP   10.97.143.14    <none>        6379/TCP         14m

controlplane ~ ➜  



Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os_x43kj56.txt.

The osImages are under the nodeInfo section under status of each node.



controlplane ~ ➜  kubectl get node -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
controlplane   Ready    control-plane   80m   v1.26.0   192.16.190.6   <none>        Ubuntu 20.04.5 LTS   5.4.0-1100-gcp   containerd://1.6.6

controlplane ~ ➜  


```json
controlplane ~ ➜  kubectl get node -o json
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "v1",
            "kind": "Node",
            "metadata": {
                "annotations": {
                    "flannel.alpha.coreos.com/backend-data": "{\"VNI\":1,\"VtepMAC\":\"c6:bb:db:2b:3c:57\"}",
                    "flannel.alpha.coreos.com/backend-type": "vxlan",
                    "flannel.alpha.coreos.com/kube-subnet-manager": "true",
                    "flannel.alpha.coreos.com/public-ip": "172.25.0.70",
                    "kubeadm.alpha.kubernetes.io/cri-socket": "unix:///var/run/containerd/containerd.sock",
                    "node.alpha.kubernetes.io/ttl": "0",
                    "volumes.kubernetes.io/controller-managed-attach-detach": "true"
                },
                "creationTimestamp": "2023-03-09T10:00:00Z",
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
                "resourceVersion": "5914",
                "uid": "c5dc305e-54e5-4b24-b1f2-abee57e490bf"
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
                        "address": "192.16.190.6",
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
                    "memory": "214484660Ki",
                    "pods": "110"
                },
                "capacity": {
                    "cpu": "36",
                    "ephemeral-storage": "1016057248Ki",
                    "hugepages-1Gi": "0",
                    "hugepages-2Mi": "0",
                    "memory": "214587060Ki",
                    "pods": "110"
                },
                "conditions": [
                    {
                        "lastHeartbeatTime": "2023-03-09T10:00:31Z",
                        "lastTransitionTime": "2023-03-09T10:00:31Z",
                        "message": "Flannel is running on this node",
                        "reason": "FlannelIsUp",
                        "status": "False",
                        "type": "NetworkUnavailable"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-09T11:08:44Z",
                        "lastTransitionTime": "2023-03-09T09:59:57Z",
                        "message": "kubelet has sufficient memory available",
                        "reason": "KubeletHasSufficientMemory",
                        "status": "False",
                        "type": "MemoryPressure"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-09T11:08:44Z",
                        "lastTransitionTime": "2023-03-09T09:59:57Z",
                        "message": "kubelet has no disk pressure",
                        "reason": "KubeletHasNoDiskPressure",
                        "status": "False",
                        "type": "DiskPressure"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-09T11:08:44Z",
                        "lastTransitionTime": "2023-03-09T09:59:57Z",
                        "message": "kubelet has sufficient PID available",
                        "reason": "KubeletHasSufficientPID",
                        "status": "False",
                        "type": "PIDPressure"
                    },
                    {
                        "lastHeartbeatTime": "2023-03-09T11:08:44Z",
                        "lastTransitionTime": "2023-03-09T10:00:27Z",
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
                            "docker.io/kodekloud/webapp-color@sha256:99c3821ea49b89c7a22d3eebab5c2e1ec651452e7675af243485034a72eb1423",
                            "docker.io/kodekloud/webapp-color:latest"
                        ],
                        "sizeBytes": 31777918
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
                            "docker.io/rancher/mirrored-flannelcni-flannel@sha256:c9786f434d4663c924aeca1a2e479786d63df0d56c5d6bd62a64915f81d62ff0",
                            "docker.io/rancher/mirrored-flannelcni-flannel:v0.19.2"
                        ],
                        "sizeBytes": 20503771
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
                            "docker.io/library/redis@sha256:8201775852e31262823ac8da9d76d0c8f36583f1a028b4800c35fc319c75289f",
                            "docker.io/library/redis:alpine"
                        ],
                        "sizeBytes": 12405358
                    },
                    {
                        "names": [
                            "docker.io/rancher/mirrored-flannelcni-flannel-cni-plugin@sha256:28d3a6be9f450282bf42e4dad143d41da23e3d91f66f19c01ee7fd21fd17cb2b",
                            "docker.io/rancher/mirrored-flannelcni-flannel-cni-plugin:v1.1.0"
                        ],
                        "sizeBytes": 3821285
                    },
                    {
                        "names": [
                            "docker.io/library/busybox@sha256:7b3ccabffc97de872a30dfd234fd972a66d247c8cfc69b0550f276481852627c"
                        ],
                        "sizeBytes": 2597143
                    },
                    {
                        "names": [
                            "docker.io/library/busybox@sha256:c118f538365369207c12e5794c3cbfb7b042d950af590ae6c287ede74f29b7d4",
                            "docker.io/library/busybox:latest"
                        ],
                        "sizeBytes": 2596507
                    },
                    {
                        "names": [
                            "docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47",
                            "docker.io/library/busybox:1.28"
                        ],
                        "sizeBytes": 727869
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
                    "bootID": "cbe9e459-8af7-4114-aed7-1cd3878bbc39",
                    "containerRuntimeVersion": "containerd://1.6.6",
                    "kernelVersion": "5.4.0-1100-gcp",
                    "kubeProxyVersion": "v1.26.0",
                    "kubeletVersion": "v1.26.0",
                    "machineID": "1e8ea8fad2a94becb6a9111994cc64cb",
                    "operatingSystem": "linux",
                    "osImage": "Ubuntu 20.04.5 LTS",
                    "systemUUID": "68cc8583-63f1-9e17-e4e4-6b7a535059d0"
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
```

controlplane ~ ➜  kubectl get node -o=jsonpath='{$.items[::].status.nodeInfo.osImage}'
Ubuntu 20.04.5 LTS
controlplane ~ ➜ 

controlplane ~ ➜  kubectl get node -o=jsonpath='{$.items[::].status.nodeInfo.osImage}' > /opt/outputs/nodes_os_x43kj56.txt

controlplane ~ ➜  cat /opt/outputs/nodes_os_x43kj56.txt
Ubuntu 20.04.5 LTS
controlplane ~ ➜  


Create a Persistent Volume with the given specification.

- Volume Name: pv-analytics
- Storage: 100Mi
-Access modes: ReadWriteMany
- Host Path: /pv/data-analytics



controlplane ~ ➜  vi pv-analytics.yaml  

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  storageClassName: manual
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/data-analytics



controlplane ~ ➜  kubectl apply -f pv-analytics.yaml 
persistentvolume/pv-analytics created

controlplane ~ ➜  


controlplane ~ ➜  kubectl get persistentvolume
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-analytics   100Mi      RWX            Retain           Available           manual                  18s

controlplane ~ ➜  