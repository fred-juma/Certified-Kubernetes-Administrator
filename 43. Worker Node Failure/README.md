#### Troubleshooting

##### Check node status

kubectl get nodes

kubectl describe node <node_name>

##### Check node cpu and disk spaces

top
df -h

##### Check kubelet status and logs

service kubelet status

sudo journalctl -u kubelet

##### Check kubelet certificates for issuer, expiry and group allocations ()o = system:nodes

```bash

openssl x509 -in /var/lib/kubelet/<node_name>.crt -text
```

Fix the broken cluster

Check the cluster resources

```bash
controlplane ~ ➜  kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4m39s

controlplane ~ ➜  
```

View cluster nodes

```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS     ROLES           AGE     VERSION
controlplane   Ready      control-plane   4m57s   v1.26.0
node01         NotReady   <none>          4m25s   v1.26.0

controlplane ~ ➜  
```

Describe node01 to view its events

```bash
EConditions:
  Type                 Status    LastHeartbeatTime                 LastTransitionTime                Reason              Message
  ----                 ------    -----------------                 ------------------                ------              -------
  NetworkUnavailable   False     Tue, 28 Feb 2023 07:02:43 -0500   Tue, 28 Feb 2023 07:02:43 -0500   FlannelIsUp         Flannel is running on this node
  MemoryPressure       Unknown   Tue, 28 Feb 2023 07:08:07 -0500   Tue, 28 Feb 2023 07:09:19 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
  DiskPressure         Unknown   Tue, 28 Feb 2023 07:08:07 -0500   Tue, 28 Feb 2023 07:09:19 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
  PIDPressure          Unknown   Tue, 28 Feb 2023 07:08:07 -0500   Tue, 28 Feb 2023 07:09:19 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
  Ready                Unknown   Tue, 28 Feb 2023 07:08:07 -0500   Tue, 28 Feb 2023 07:09:19 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
```

SSH to node01

```bash
controlplane ~ ➜  ssh node01

```


View kubelet status

```bash
root@node01 ~ ✖ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead) since Tue 2023-02-28 01:28:02 EST; 13min ago
       Docs: https://kubernetes.io/docs/home/
    Process: 2033 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=0/SUCCESS)
   Main PID: 2033 (code=exited, status=0/SUCCESS)
```


Start kubelet service

```bash
root@node01 ~ ➜  systemctl start kubelet

root@node01 ~ ➜  systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Tue 2023-02-28 01:43:13 EST; 5s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 7074 (kubelet)
      Tasks: 20 (limit: 251382)
     Memory: 50.4M
```


View node status


```bash
controlplane ~ ➜  kubectl get node
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   19m   v1.26.0
node01         Ready    <none>          19m   v1.26.0

controlplane ~ ➜  
```


The cluster is broken again. Investigate and fix the issue.


Check node status

```bash
controlplane ~ ➜  kubectl get node
NAME           STATUS     ROLES           AGE   VERSION
controlplane   Ready      control-plane   23m   v1.26.0
node01         NotReady   <none>          22m   v1.26.0

controlplane ~ ➜  
```


Describe node to view events

```bash
Events:
  Type     Reason                   Age                    From             Message
  ----     ------                   ----                   ----             -------
  Normal   Starting                 23m                    kube-proxy       
  Normal   NodeHasSufficientMemory  23m                    kubelet          Node node01 status is now: NodeHasSufficientMemory
  Warning  InvalidDiskCapacity      23m                    kubelet          invalid capacity 0 on image filesystem
  Normal   NodeHasNoDiskPressure    23m                    kubelet          Node node01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     23m                    kubelet          Node node01 status is now: NodeHasSufficientPID
  Normal   Starting                 23m                    kubelet          Starting kubelet.
  Normal   NodeAllocatableEnforced  23m                    kubelet          Updated Node Allocatable limit across pods
  Normal   RegisteredNode           23m                    node-controller  Node node01 event: Registered Node node01 in Controller
  Normal   NodeReady                23m                    kubelet          Node node01 status is now: NodeReady
  Normal   Starting                 4m56s                  kubelet          Starting kubelet.
  Warning  InvalidDiskCapacity      4m56s                  kubelet          invalid capacity 0 on image filesystem
  Normal   NodeHasSufficientMemory  4m55s (x2 over 4m55s)  kubelet          Node node01 status is now: NodeHasSufficientMemory
  Normal   NodeAllocatableEnforced  4m55s                  kubelet          Updated Node Allocatable limit across pods
  Normal   NodeHasNoDiskPressure    4m55s (x2 over 4m55s)  kubelet          Node node01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     4m55s (x2 over 4m55s)  kubelet          Node node01 status is now: NodeHasSufficientPID
  Normal   NodeReady                4m55s                  kubelet          Node node01 status is now: NodeReady
  Normal   NodeNotReady             3m10s (x2 over 19m)    node-controller  Node node01 status is now: NodeNotReady

controlplane ~ ➜  
```

Check the status of kubelet service

```bash
root@node01 ~ ✖ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Tue 2023-02-28 01:50:29 EST; 1s ago
       Docs: https://kubernetes.io/docs/home/
    Process: 9238 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAILURE)
   Main PID: 9238 (code=exited, status=1/FAILURE)
   ```

View the kubelet logs

```bash
root@node01:~# journalctl -u kubelet -f 
.
.
Feb 28 07:22:27 node01 kubelet[7776]: E0228 07:22:27.550826    7776 run.go:74] "command failed" err="failed to construct kubelet dependencies: unable to load client CA file /etc/kubernetes/pki/WRONG-CA-FILE.crt: open /etc/kubernetes/pki/WRONG-CA-FILE.crt: no such file or directory"
```

Correct the CA cert file name by updating the kubelet config file

```bash
root@node01 ~ ➜  vi /var/lib/kubelet/config.yaml

apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
    ```


Then start the kubelet service

```bash
root@node01 ~ ➜  systemctl start kubelet

root@node01 ~ ➜  systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; >
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2023-03-01 01:06:06 EST; 12>
       Docs: https://kubernetes.io/docs/home/
   Main PID: 6466 (kubelet)
      Tasks: 27 (limit: 251382)
     Memory: 51.3M
     CGroup: /system.slice/kubelet.service
     ```

The cluster is broken again. Investigate and fix the issue.


View nodes in the cluster
```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS     ROLES           AGE   VERSION
controlplane   Ready      control-plane   56m   v1.26.0
node01         NotReady   <none>          55m   v1.26.0

controlplane ~ ➜  
```


Describe the node events

```bash
Conditions:
  Type                 Status    LastHeartbeatTime                 LastTransitionTime                Reason              Message
  ----                 ------    -----------------                 ------------------                ------              -------
  NetworkUnavailable   False     Tue, 28 Feb 2023 07:02:43 -0500   Tue, 28 Feb 2023 07:02:43 -0500   FlannelIsUp         Flannel is running on this node
  MemoryPressure       Unknown   Tue, 28 Feb 2023 07:56:45 -0500   Tue, 28 Feb 2023 07:57:39 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
  DiskPressure         Unknown   Tue, 28 Feb 2023 07:56:45 -0500   Tue, 28 Feb 2023 07:57:39 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
  PIDPressure          Unknown   Tue, 28 Feb 2023 07:56:45 -0500   Tue, 28 Feb 2023 07:57:39 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
  Ready                Unknown   Tue, 28 Feb 2023 07:56:45 -0500   Tue, 28 Feb 2023 07:57:39 -0500   NodeStatusUnknown   Kubelet stopped posting node status.
```

View kubelet service status

```bash

  root@node01 ~ ➜  systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Tue 2023-02-28 07:57:03 EST; 2min 23s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 15408 (kubelet)
      Tasks: 30 (limit: 251382)
     Memory: 50.4M
```


View kubelet logs

```bash
root@node01 ~ ➜  journalctl -u kubelet -f

Feb 28 07:59:43 node01 kubelet[15408]: E0228 07:59:43.644680   15408 kubelet_node_status.go:92] "Unable to register node with API server" err="Post \"https://controlplane:6553/api/v1/nodes\": dial tcp 192.21.128.12:6553: connect: connection refused" node="node01"

```

From the kubelet logs, we see that kubelet is trying to connect to the API server on the controlplane node on port *6553*. This is incorrect. Correct the port on the kubelet config file to *6443* 


```bash

root@node01 ~ ➜  vi /etc/kubernetes/kubelet.conf


apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1ESXlPREV5TURFek9Wb1hEVE16TURJeU5URXlNREV6T1Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTlVwCjl5WFZNUlJoZG53a0RjYWx5OWVuNi9LQithWUN6TTlPa3V0KzRMclF2QitYSEc0UjZzNC9xMWM3VkE0L3o5M0sKYll2c3ZoZEZUWmZld2JyOG9CanR2ejR6ZFpLNUVzeVZ3dnVpcEdERWJMdkRuelNBM3h3OXhzSkpQNGRjMEpWNwpDeWlBNHRUNzFVSWNvOHN1Qm4wcmRueVRYbzNSVEtQTDNqKzl5cDVKZzU2enBaV1F0MXV3SEdxTllZNFhQYXNqCmhGWStXNTlGN3ZWLyt4ZG5wbFhJaFVrcGhVV2pLdVZabSs5c3FuZU9NQnRPdXVMNm5CMWdndjBacVlHaE4rcUsKS3FUUnZ1anc5WVBxOFdrY21IRGJIWlVzUVdZempCQ3RIOS9Oa3owYVlzY3dOaTR5dk9mSHYyOU1LaWxUZWJCWgpDbmUrZ29uSzA0NmFCVTNvbDVzQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZDSGFVN09XTmZYc3BqU3VLbHlUaUFrWTYyd1VNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBSm5MUUY0VmxyYlNVdlVjUm9OVwpXRXUxRTMxV0FZMmNRR3hzTlFzVnpXY0pxck5zdVhWQWJaZDVGTllETENET2t2dkxqZU1PNE1jTWR2NmhxL0k5ClNQUlFNVVpHQjFvcFhXaGJmV28wZWhOa2ZjMWtYOTNrSGlOZmRSTStPcE9kUDBhZkZHNmVpNlVzNDJzNGlwaU4KWXdjemlnRWJBQ2ZxK0JlVmM1emlWam9KQVRvdHF5MllCRU1WVFY3b0RNYitkcW5vTU5qU0Ewam1KelNFcmtYNQozQWI2S2Jydll4Q0xZaHl5dndEYUR0MzBZNytrSTFiSDRhdXFLR29oWDdBbHVtT08yREwwYzJpdFFLZU9DZkovCjA5NGM0MWpIaEwwSkJYZVBCZmxBem5Edkc4UjM1M1B4U2pXTWQ5YXZUSXhQcmpVZ0pkbnJuU2thbW1GMXdOYUYKdU5vPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://controlplane:6443
  name: default-cluster
contexts:
- context:
    cluster: default-cluster
    namespace: default
    user: default-auth
  name: default-context
current-context: default-context
kind: Config
preferences: {}
users:
- name: default-auth
  user:
    client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
    client-key: /var/lib/kubelet/pki/kubelet-client-current.pem
```


Restart the kubelet service in node01

```bash
root@node01 ~ ➜  systemctl restart kubelet

root@node01 ~ ➜

root@node01 ~ ➜  systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; >
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2023-03-01 01:08:32 EST; 2m>
       Docs: https://kubernetes.io/docs/home/
   Main PID: 6955 (kubelet)
      Tasks: 29 (limit: 251382)
     Memory: 54.1M  

```

View the status of the node to confirm it is ready

```bash

controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   61m   v1.26.0
node01         Ready    <none>          60m   v1.26.0

controlplane ~ ➜  
```


***The End***