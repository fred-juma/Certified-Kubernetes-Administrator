Inspect the environment and identify the authorization modes configured on the cluster. **--authorization-mode=Node,RBAC**


Check the kube-apiserver settings.

Get the pods in all namespaces to see the name of the kube-apiserver and its namespace

```bash

controlplane ~ ➜  kubectl get pods -A 
NAMESPACE      NAME                                   READY   STATUS    RESTARTS   AGE
blue           blue-app                               1/1     Running   0          2m40s
blue           dark-blue-app                          1/1     Running   0          2m40s
default        red-84c985b67c-d6vkw                   1/1     Running   0          2m41s
default        red-84c985b67c-t8bpp                   1/1     Running   0          2m41s
kube-flannel   kube-flannel-ds-x8br5                  1/1     Running   0          4m22s
kube-system    coredns-787d4945fb-fl4h2               1/1     Running   0          4m22s
kube-system    coredns-787d4945fb-mgd9s               1/1     Running   0          4m22s
kube-system    etcd-controlplane                      1/1     Running   0          4m32s
kube-system    kube-apiserver-controlplane            1/1     Running   0          4m37s
kube-system    kube-controller-manager-controlplane   1/1     Running   0          4m32s
kube-system    kube-proxy-s7lkd                       1/1     Running   0          4m22s
kube-system    kube-scheduler-controlplane            1/1     Running   0          4m34s

controlplane ~ ➜  
```

Describe the kube-apiserver  and look for --authorization-mode

```bash
controlplane ~ ➜  kubectl describe pod kube-apiserver-controlplane -n kube-system
Name:                 kube-apiserver-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 controlplane/10.29.50.9
Start Time:           Sat, 21 Jan 2023 05:53:03 -0500
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.29.50.9:6443
                      kubernetes.io/config.hash: be20f5e0c771e548d659027baa08eb84
                      kubernetes.io/config.mirror: be20f5e0c771e548d659027baa08eb84
                      kubernetes.io/config.seen: 2023-01-21T05:52:45.276239451-05:00
                      kubernetes.io/config.source: file
Status:               Running
IP:                   10.29.50.9
IPs:
  IP:           10.29.50.9
Controlled By:  Node/controlplane
Containers:
  kube-apiserver:
    Container ID:  containerd://6c709f8ced898d7aefdcf3043697a6090236d5bf048efd121119f0e2d7c70f3d
    Image:         registry.k8s.io/kube-apiserver:v1.26.0
    Image ID:      registry.k8s.io/kube-apiserver@sha256:d230a0b88a3daf14e4cce03b906b992c8153f37da878677f434b1af8c4e8cc75
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=10.29.50.9
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --enable-admission-plugins=NodeRestriction
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
      --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
      --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
      --etcd-servers=https://127.0.0.1:2379
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
      --requestheader-allowed-names=front-proxy-client
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
      --requestheader-extra-headers-prefix=X-Remote-Extra-
      --requestheader-group-headers=X-Remote-Group
      --requestheader-username-headers=X-Remote-User
      --secure-port=6443
      --service-account-issuer=https://kubernetes.default.svc.cluster.local
      --service-account-key-file=/etc/kubernetes/pki/sa.pub
      --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
      --service-cluster-ip-range=10.96.0.0/12
      --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
      --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    State:          Running
      Started:      Sat, 21 Jan 2023 05:52:49 -0500
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        250m
    Liveness:     http-get https://10.29.50.9:6443/livez delay=10s timeout=15s period=10s #success=1 #failure=8
    Readiness:    http-get https://10.29.50.9:6443/readyz delay=0s timeout=15s period=1s #success=1 #failure=3
    Startup:      http-get https://10.29.50.9:6443/livez delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/ca-certificates from etc-ca-certificates (ro)
      /etc/kubernetes/pki from k8s-certs (ro)
      /etc/ssl/certs from ca-certs (ro)
      /usr/local/share/ca-certificates from usr-local-share-ca-certificates (ro)
      /usr/share/ca-certificates from usr-share-ca-certificates (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  etc-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ca-certificates
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
  usr-local-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/local/share/ca-certificates
    HostPathType:  DirectoryOrCreate
  usr-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/share/ca-certificates
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:            <none>

controlplane ~ ➜  
```






Create a role definition file

Create the role by kubectl apply command

Create a role binding object defintion file. It links a user to a role
- you can specify other namespace below the metadata if different from default namespace
create the role binding by kubectl apply command


View roles in the default namespace

```bash
controlplane ~ ➜  kubectl get roles
No resources found in default namespace.

controlplane ~ ➜  
```

View roles in all namespaces ** 12**

```bash
controlplane ~ ➜  kubectl get roles -A -o wide
NAMESPACE     NAME                                             CREATED AT
blue          developer                                        2023-01-21T10:54:55Z
kube-public   kubeadm:bootstrap-signer-clusterinfo             2023-01-21T10:52:59Z
kube-public   system:controller:bootstrap-signer               2023-01-21T10:52:57Z
kube-system   extension-apiserver-authentication-reader        2023-01-21T10:52:57Z
kube-system   kube-proxy                                       2023-01-21T10:53:01Z
kube-system   kubeadm:kubelet-config                           2023-01-21T10:52:58Z
kube-system   kubeadm:nodes-kubeadm-config                     2023-01-21T10:52:58Z
kube-system   system::leader-locking-kube-controller-manager   2023-01-21T10:52:57Z
kube-system   system::leader-locking-kube-scheduler            2023-01-21T10:52:57Z
kube-system   system:controller:bootstrap-signer               2023-01-21T10:52:57Z
kube-system   system:controller:cloud-provider                 2023-01-21T10:52:57Z
kube-system   system:controller:token-cleaner                  2023-01-21T10:52:57Z

controlplane ~ ➜  kubectl get roles -A -o wide | wc -l
13

controlplane ~ ➜  
```

What are the resources the kube-proxy role in the kube-system namespace is given access to? **configmaps**

```bash
controlplane ~ ➜  kubectl describe role kube-proxy --namespace kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  configmaps  []                 [kube-proxy]    [get]

controlplane ~ ➜  
```

What actions can the kube-proxy role perform on configmaps? **verbs = get**

```bash

controlplane ~ ➜  kubectl describe role -n kube-system kube-proxy
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  configmaps  []                 [kube-proxy]    [get]

controlplane ~ ➜ 
```

Which account is the kube-proxy role assigned to? **Group  system:bootstrappers:kubeadm:default-node-token**

```bash
controlplane ~ ➜  kubectl describe rolebinding kube-proxy -n kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  kube-proxy
Subjects:
  Kind   Name                                             Namespace
  ----   ----                                             ---------
  Group  system:bootstrappers:kubeadm:default-node-token  

controlplane ~ ➜ 
```

A user dev-user is created. User's details have been added to the kubeconfig file. Inspect the permissions granted to the user. Check if the user can list pods in the default namespace.


Use the --as dev-user option with kubectl to run commands as the dev-user.

```bash
controlplane ~ ➜  kubectl auth can-i create pods --as dev-user
no
```


See if you have access to a particular resource

kubectl auth can-i create <resource>

Check permissions of another user

kubectl auth can-i create pods --as dev-user

Check permissions of another user on a different namespace

kubectl auth can-i create <resource> --as <user_name> --namespace test